The call stack consists of the OCL procedure (`SA880.ocl36.txt`) and the RPG program (`SA880.rpg36.txt`) for a Sales Analysis Inquiry by Product on IBM System/36 or its environment on IBM i. Below, I’ll identify the use cases implemented by this program and then provide a function requirement document for a non-interactive version of the primary use case, focusing on business requirements, process steps, and calculations.

### Use Cases Implemented by the Program

The `SA880` program, executed via the OCL procedure, is an interactive inquiry application that retrieves and displays sales analysis data by product. Based on the RPG code and OCL setup, the following use cases are implemented:

1. **Interactive Sales Analysis Inquiry by Product and Company**:
   - **Description**: Allows a user to input a company number, product code, and optionally a location to retrieve and display sales summary data (e.g., billed quantity, billed dollars, invoice type, month, year) for the specified product and company, with scrolling capabilities.
   - **Details**:
     - Users enter company (`CO`), product (`PROD`), and optional location (`LOC`) on the input screen (`SA880S1`).
     - Validates the product against the `GSPROD` file; displays "INVALID PRODUCT" if not found or deleted.
     - Retrieves matching records from `SASMY1`, displaying up to 12 records per screen (`SA880S3`) with fields like location, month, year, invoice type, unit of measure, billed quantity, and billed dollars.
     - Supports forward/backward scrolling using roll keys (`ROLLFW`, `ROLLBW`) with navigation messages ("MORE RECORDS EXIST", "END OF RECORDS", "START OF RECORDS").
     - Handles non-stock items (`PROD='NS'`) by displaying a specific description.
     - Exits on function keys `KA` (cancel) or `KG` (end job).
   - **Inputs**: Company number, product code, optional location (via screen).
   - **Outputs**: Displayed sales data, error messages, or navigation messages.

2. **Non-Stock Item Inquiry**:
   - **Description**: Handles inquiries for non-stock items by displaying a predefined description when the product code is 'NS'.
   - **Details**:
     - If `PROD='NS'`, the program sets the product description to `NSDESC` ("NON STOCK ITEM") and skips further product validation in `GSPROD`.
     - Proceeds to retrieve sales data from `SASMY1` for the non-stock item, if any.
     - Displays results or "NO RECORDS EXIST FOR THIS PRODUCT" if no data is found.
   - **Inputs**: Company number, product code 'NS', optional location.
   - **Outputs**: Sales data for non-stock items or error message.

3. **Error Handling for Invalid Inputs**:
   - **Description**: Validates user inputs and provides feedback for invalid or non-existent products or no matching sales records.
   - **Details**:
     - Checks if the product exists in `GSPROD` and is not deleted (`TPDEL≠'D'`); if invalid, displays "INVALID PRODUCT".
     - If no records in `SASMY1` match the input criteria, displays "NO RECORDS EXIST FOR THIS PRODUCT".
     - Ensures location is optional; if blank, retrieves all locations for the company/product.
   - **Inputs**: Invalid product code or criteria yielding no results.
   - **Outputs**: Error messages on the screen.

### Function Requirement Document

Below is a function requirement document for a non-interactive version of the primary use case (Use Case 1: Sales Analysis Inquiry by Product and Company). The function assumes inputs are provided programmatically (not via a screen) and returns sales data, adapting the interactive logic to a batch-like process. The document focuses on business requirements, process steps, and calculations, keeping it concise.

<xaiArtifact artifact_id="e58ab604-f30b-49c5-9bd9-f2096c364339" artifact_version_id="cee1bf32-e35a-4d0c-8338-041dc70f6f3e" title="Sales_Analysis_Inquiry_Requirements.md" contentType="text/markdown">

# Sales Analysis Inquiry by Product Function Requirements

## Overview
The `SalesAnalysisInquiry` function retrieves sales summary data for a specified company, product, and optional location, returning a structured dataset of sales records. It validates inputs, retrieves product and sales data, and handles non-stock items and errors, designed for programmatic execution without user interaction.

## Inputs
- **Company Number** (`company`, string, 3-40 chars): Identifies the company for the inquiry.
- **Product Code** (`product`, string, 6 chars): Product to query (e.g., product code or 'NS' for non-stock).
- **Location** (`location`, string, 3 chars, optional): Specific location to filter sales data; if blank, includes all locations.
- **File Access**:
  - `GSPROD`: Product master file (fields: `TPDEL`, `TPCODE`, `TPDES1`).
  - `SASMY1`: Sales summary file (fields: `SYCONO`, `SYPROD`, `SYLOC`, `SYYR`, `SYMO`, `SYINTY`, `SYINST`, `SYUNMS`, `SYBQTY`, `SYBDOL`).

## Outputs
- **Result** (structured data):
  - **Status**: Success, InvalidProduct, NoRecords.
  - **ProductDescription**: Description from `GSPROD` or "NON STOCK ITEM" for `product='NS'`.
  - **SalesRecords**: Array of records (up to 12 per call), each with:
    - `Location` (string, 3 chars)
    - `Year` (string, 2 chars)
    - `Month` (string, 2 chars)
    - `InvoiceType` (string, 1 char)
    - `InvoiceStyle` (string, 1 char)
    - `UnitOfMeasure` (string, 3 chars)
    - `BilledQuantity` (decimal, 11.2)
    - `BilledDollars` (decimal, 11.2)
  - **MoreRecords**: Boolean indicating if additional records exist.
- **Errors**: Returns error status for invalid product or no matching records.

## Process Steps
1. **Validate Product**:
   - If `product='NS'`, set `ProductDescription="NON STOCK ITEM"`.
   - Else, query `GSPROD` for `TPCODE=product` and `TPDEL≠'D'`.
     - If not found, return `Status=InvalidProduct`, `Message="INVALID PRODUCT"`.
     - If found, set `ProductDescription=TPDES1`.
2. **Set Search Key**:
   - Build key from `company` (and `location` if provided).
   - Set lower limit on `SASMY1` using the key.
3. **Retrieve Sales Data**:
   - Read `SASMY1` forward, filtering by:
     - `SYCONO=company`
     - `SYPROD=product`
     - `SYLOC=location` (if provided, else all locations)
   - Collect up to 12 records into `SalesRecords`.
   - Check for additional records; set `MoreRecords=true` if more exist.
4. **Handle No Data**:
   - If no records match, return `Status=NoRecords`, `Message="NO RECORDS EXIST FOR THIS PRODUCT"`.
5. **Return Result**:
   - Return `Status=Success`, `ProductDescription`, `SalesRecords`, `MoreRecords`.

## Business Rules
1. **Input Validation**:
   - `company` must be a valid company number (3-40 chars).
   - `product` must exist in `GSPROD` with `TPDEL≠'D'` or be 'NS'.
   - `location` is optional; if blank, retrieve sales for all locations.
2. **Data Filtering**:
   - Sales records must match `company`, `product`, and `location` (if specified).
   - Non-stock items (`product='NS'`) bypass `GSPROD` validation but require matching `SASMY1` records.
3. **Data Limits**:
   - Return up to 12 records per call to manage output size.
   - Indicate if more records exist for subsequent calls.
4. **Error Handling**:
   - Return "INVALID PRODUCT" for non-existent or deleted products.
   - Return "NO RECORDS EXIST FOR THIS PRODUCT" if no matching sales data.

## Calculations
- **Billed Quantity and Dollars**: Retrieved directly from `SASMY1` (`SYBQTY`, `SYBDOL`) in 11.2 decimal format; no additional calculations performed.
- **Record Counting**: Track number of records read (up to 12) to control output and set `MoreRecords` flag by checking for additional matching records in `SASMY1`.

## Dependencies
- **Files**:
  - `GSPROD`: Product master (read-only, shared access).
  - `SASMY1`: Sales summary (read-only, shared access).
- **Environment**: Assumes files are allocated by an external process (e.g., OCL) with resolved library prefixes.

</xaiArtifact>

### Notes
- The function assumes a single call retrieves up to 12 records, mimicking the original program’s screen limit. Subsequent calls could use a cursor or key to continue from the last record (not implemented in the original RPG but implied by `MoreRecords`).
- The non-interactive version eliminates roll key handling (`ROLLFW`, `ROLLBW`) and screen output, focusing on data retrieval.
- No external programs are called, as the original RPG relies on internal subroutines.
- The secondary use case (Non-Stock Item Inquiry) is subsumed into the primary function, handled by the `product='NS'` condition.
- The error handling use case is integrated into the function’s status and message outputs.

If you need further details, such as a specific implementation in a programming language or additional use case refinements, let me know!