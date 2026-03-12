The RPG program `SA870` and its associated OCL procedure (`SA870.ocl36.txt`) implement a customer inquiry system for sales analysis on an IBM System/36. Below, I will first identify the use cases implemented by the program based on the provided code. Then, I will create a function requirement document for one of the primary use cases, reimagined as a non-interactive function that processes inputs to produce outputs, focusing on business requirements and necessary calculations.

### Use Cases Implemented by the Program

The `SA870` RPG program, driven by the OCL procedure, supports several use cases related to querying and navigating customer sales data. These use cases are derived from the program’s functionality, which centers on retrieving and displaying customer information and sales records, with support for interactive navigation. The following use cases are implemented:

1. **Customer Validation and Information Retrieval**:
   - **Description**: The user provides a company number and customer number to validate the customer’s existence and retrieve their details (e.g., name, address) from the customer master file (`ARCUST`).
   - **Functionality**: The program chains to `ARCUST` using the combined company/customer key (`COCUST`). If no record is found, it displays an error message ("NO MASTER RECORD WAS FOUND"). If found, it displays customer details on the screen.
   - **Trigger**: User enters company and customer numbers on the first screen (indicator 01, subroutine `SUBSC1`).
   - **Outcome**: Validates the customer and displays their name and address, or shows an error if the customer is invalid.

2. **Sales Record Inquiry**:
   - **Description**: For a valid customer, retrieve and display sales summary records from the `SASMC1` file, filtered by company and customer number.
   - **Functionality**: The program positions the `SASMC1` file using `SETLL` with `COCUST`, reads matching records, and displays up to 12 records per screen. Each record includes product, location, month, year, invoice type, billed quantity, unit of measure, and billed dollars.
   - **Trigger**: Successful customer validation, followed by reading `SASMC1` records (subroutine `READFW` or `READBW`).
   - **Outcome**: Displays a list of sales records for the specified customer, with fields formatted for readability (e.g., quantities and dollars with decimals).

3. **Forward Navigation of Sales Records**:
   - **Description**: Allow the user to scroll forward through additional sales records for the customer when more records exist.
   - **Functionality**: Detects the Page Down key (status code 01123, indicator 19) via the `ROLLKY` subroutine, repositions the file using `ROLLFW`, and reads the next set of records (`READFW`). Updates the `MORE` field to indicate if additional records are available.
   - **Trigger**: User presses Page Down while viewing sales records.
   - **Outcome**: Displays the next batch of up to 12 sales records, maintaining the scroll position with `SVTOP` and `SVBOT`.

4. **Backward Navigation of Sales Records**:
   - **Description**: Allow the user to scroll backward through previous sales records for the customer.
   - **Functionality**: Detects the Page Up key (status code 01122, indicator 18) via `ROLLKY`, repositions the file using `ROLLBW`, and reads previous records (`READBW`). Updates the `MORE` field to indicate if earlier records exist.
   - **Trigger**: User presses Page Up while viewing sales records.
   - **Outcome**: Displays the previous batch of up to 12 sales records, updating scroll indicators.

5. **Program Exit**:
   - **Description**: Allow the user to exit the inquiry program.
   - **Functionality**: Detects the F3 key (`KG`), sets the Last Record (`LR`) indicator, and terminates the program.
   - **Trigger**: User presses F3.
   - **Outcome**: Exits the program, closing the workstation session.

6. **Screen Reset**:
   - **Description**: Reset the inquiry screen to allow a new customer query.
   - **Functionality**: Detects the F4 key (`KA`), clears the customer number (`CUST`), and redisplays the input screen.
   - **Trigger**: User presses F4.
   - **Outcome**: Clears current data and allows entry of a new company/customer number.

7. **Time Display/Refresh**:
   - **Description**: Refresh the screen with the current time.
   - **Functionality**: Detects the F9 key (indicator 09), captures the system time into `RDATE`, and redisplays the screen.
   - **Trigger**: User presses F9.
   - **Outcome**: Updates the screen with the current time, maintaining the current inquiry context.

### Function Requirement Document

For the function requirement document, I’ll focus on the primary use case: **Sales Record Inquiry**, reimagined as a non-interactive function that takes inputs (company and customer number) and returns sales records without screen interaction. This function encapsulates the core business logic of retrieving and filtering sales data, which is the program’s main purpose. The document will concisely outline the process steps, business rules, and any calculations, tailored for a business audience with technical details as needed.

<xaiArtifact artifact_id="bd936189-f162-40f1-b674-8cfced1c1583" artifact_version_id="39b11501-a06d-4489-ba4f-07b846c5fe75" title="CustomerSalesInquiryFunction.md" contentType="text/markdown">

# Customer Sales Inquiry Function Requirements

## Overview
The `CustomerSalesInquiry` function retrieves sales summary records for a specified customer, identified by company and customer number, from the sales analysis database. It validates the customer’s existence and returns a structured list of matching sales records, including product, location, date, invoice details, and financial data.

## Inputs
- **Company Number** (`CO`, 2 bytes, alphanumeric): The company identifier.
- **Customer Number** (`CUST`, 6 bytes, numeric): The customer identifier.
- **Maximum Records** (`MaxRecords`, integer, optional): The maximum number of sales records to return (default: 100).

## Outputs
- **Status**: Success (valid customer with records), InvalidCustomer (no customer found), or NoRecords (valid customer but no sales records).
- **Customer Details**:
  - Name (`ARNAME`, 30 bytes, alphanumeric)
  - Address Line 1 (`ARADR1`, 30 bytes, alphanumeric)
  - Address Line 2 (`ARADR2`, 30 bytes, alphanumeric)
  - Address Line 3 (`ARADR3`, 30 bytes, alphanumeric)
- **Sales Records** (array, up to `MaxRecords` entries):
  - Product (`SYPROD`, 4 bytes, alphanumeric)
  - Location (`SYLOC`, 3 bytes, alphanumeric)
  - Month (`SYMO`, 2 bytes, numeric)
  - Year (`SYYR`, 2 bytes, numeric)
  - Invoice Type (`SYINTY`, 1 byte, alphanumeric)
  - Invoice Style (`SYINST`, 1 byte, alphanumeric)
  - Unit of Measure (`SYUNMS`, 3 bytes, alphanumeric)
  - Billed Quantity (`SYBQTY`, 11 digits, 2 decimals, numeric, signed)
  - Billed Dollars (`SYBDOL`, 11 digits, 2 decimals, numeric, signed)

## Process Steps
1. **Validate Customer**:
   - Combine `CO` and `CUST` into a composite key (`COCUST`, 8 bytes).
   - Query the customer master file (`ARCUST`) using `COCUST`.
   - If no record is found, return `Status: InvalidCustomer`.
   - If found, extract `ARNAME`, `ARADR1`, `ARADR2`, `ARADR3`.

2. **Retrieve Sales Records**:
   - Query the sales summary file (`SASMC1`) using `COCUST` as the key.
   - Read records where `SYCOCU` matches `COCUST`, up to `MaxRecords`.
   - For each record, extract `SYPROD`, `SYLOC`, `SYMO`, `SYYR`, `SYINTY`, `SYINST`, `SYUNMS`, `SYBQTY`, `SYBDOL`.
   - If no records are found, return `Status: NoRecords`.

3. **Format Output**:
   - Package customer details and sales records into a structured response.
   - Ensure numeric fields (`SYBQTY`, `SYBDOL`) are formatted with two decimal places and negative signs if applicable.
   - Return `Status: Success` with the data.

## Business Rules
1. **Customer Validation**:
   - The customer must exist in `ARCUST` for the inquiry to proceed.
   - Invalid company/customer combinations result in an `InvalidCustomer` status.
2. **Data Filtering**:
   - Only sales records where `SYCOCU` matches the input `COCUST` are returned.
   - Records are returned in the order they appear in `SASMC1` (typically sorted by key fields).
3. **Data Limits**:
   - The function returns up to `MaxRecords` sales records to prevent excessive data retrieval.
   - Default `MaxRecords` is 100 to balance performance and usability.
4. **Data Integrity**:
   - Numeric fields (`SYBQTY`, `SYBDOL`) must be valid numbers, with negative values preserved (e.g., for returns or credits).
   - Date fields (`SYMO`, `SYYR`) must represent valid months (1–12) and years (e.g., two-digit format, assumed 19XX or 20XX based on system context).

## Calculations
- **Key Construction**: Concatenate `CO` (2 bytes) and `CUST` (6 bytes) to form `COCUST` (8 bytes) for file queries.
- **Numeric Formatting**: Ensure `SYBQTY` and `SYBDOL` are formatted with two decimal places (e.g., 1234.56 or -1234.56).
- **Record Counting**: Track the number of records retrieved to enforce the `MaxRecords` limit.

## Assumptions
- The `ARCUST` file contains one record per customer, uniquely identified by `COCUST`.
- The `SASMC1` file contains multiple sales records per customer, indexed by `SYCOCU`.
- No external programs are called; all data is retrieved from `ARCUST` and `SASMC1`.

## Error Handling
- **InvalidCustomer**: Return status with no further processing if the customer is not found.
- **NoRecords**: Return status and customer details if no sales records match.
- **Data Errors**: Invalid numeric or date fields in `SASMC1` are skipped, with a warning included in the response.

</xaiArtifact>

### Explanation of the Function Requirement Document

The document reimagines the **Sales Record Inquiry** use case as a non-interactive function, aligning with the request to treat it as a large function that processes inputs directly. It focuses on:
- **Inputs**: Company and customer number, with an optional limit on records to replace the interactive screen’s 12-record limit.
- **Outputs**: Structured customer and sales data, replacing screen displays with a data structure suitable for API or batch processing.
- **Process Steps**: Simplified to validation, retrieval, and formatting, removing interactive elements like scrolling and screen output.
- **Business Rules**: Retained from the RPG program (e.g., customer validation, data filtering) but adapted for a non-interactive context.
- **Calculations**: Limited to key construction and numeric formatting, as the original program performs no complex calculations.
- **Error Handling**: Covers invalid customers and no records, aligning with the RPG’s error messaging.

The other use cases (e.g., navigation, exit, reset) are not included in the function, as they are inherently interactive and tied to screen navigation. If you need a function requirement document for another use case or further refinements, please let me know!