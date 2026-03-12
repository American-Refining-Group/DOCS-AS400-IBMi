### **List of Use Cases Implemented in the Call Stack**

The call stack for the rack pricing update process, as defined by the OCL and RPG programs (`BB956P.ocl36.txt`, `BB9541.rpg36.txt`, `BB956P.rpg36.txt`, `BB9563.ocl36.txt`, `BB9563.rpg36.txt`, `BB9564.ocl36.txt`, `BB9564.rpg36.txt`, `BB9565.ocl36.txt`, `BB9565.rpg36.txt`, `BB9566.ocl36.txt`, `BB9566.rpg36.txt`), implements several use cases related to updating and managing rack pricing data. Below is a comprehensive list of the use cases derived from the programs:

1. **Use Case 1: Create a Deduplicated Customer List for Rack Pricing**
   - **Description**: Generate a unique list of company and customer numbers from the customer master file (`PRCTUM`) for use in the rack pricing update process.
   - **Program**: `BB9541`
   - **Details**: Reads `PRCTUM`, checks for duplicates in `BB956S`, and writes new records to `BB956S` with an active flag (`'A'`).

2. **Use Case 2: Validate User Input for Rack Pricing Update**
   - **Description**: Validate user-provided inputs (company number, customer number, and date) to ensure they are valid before proceeding with pricing updates.
   - **Program**: `BB956P`
   - **Details**: Interactively validates company number against `BICONT`, customer number against `BB956S`, and month (01–12). Handles Y2K date conversion and displays errors on a screen.

3. **Use Case 3: Filter and Validate Rack Pricing Records**
   - **Description**: Process rack pricing records to identify new pricing entries that match specific criteria (date, non-existing in `PRCTUM`, and specific container descriptions) and generate a report.
   - **Program**: `BB9563`
   - **Details**: Validates `RKPRCE` records against a provided date, checks for non-existence in `PRCTUM`, and verifies container descriptions ("BRAD PENN" or "DA LUBRICANTS") in `GSCNTR1`. Writes valid records to `NEWPROD` and a printed report.

4. **Use Case 4: Display New Pricing Records for User Review**
   - **Description**: Display new pricing records from `NEWPROD` for user review, allowing confirmation or cancellation.
   - **Program**: `BB9564`
   - **Details**: Reads up to 10 records from `NEWPROD` and displays company, location, product, container, and unit of measure on a screen. Supports cancellation.

5. **Use Case 5: Update Customer Master with New Pricing Records**
   - **Description**: Add new pricing records from `NEWPROD` to the `PRCTUM` customer master file, ensuring no duplicates.
   - **Program**: `BB9565`
   - **Details**: Checks for duplicates in `PRCTUM` and adds new records with an active flag (`'A'`).

6. **Use Case 6: Generate Pricing Download File**
   - **Description**: Create a download file containing validated rack pricing records that match customer data in `PRCTUM`.
   - **Program**: `BB9566`
   - **Details**: Copies `RKPRCE` records to `RKPRCEO` (`?9?PRCDOWN`) if they match records in `PRCTUM`, after clearing the output file.

---

### **Function Requirement Document for Rack Pricing Update**

<xaiArtifact artifact_id="d153c58e-9aa7-4554-b0af-1f521e67a982" artifact_version_id="2e4bc75c-36b6-4a89-89c6-bb5701657082" title="Rack_Pricing_Update_Requirements.md" contentType="text/markdown">

# Rack Pricing Update Function Requirements

## **Overview**
The Rack Pricing Update function processes rack pricing data to generate a deduplicated customer list, validate inputs, filter pricing records, update the customer master file, and produce a download file for external use. The function takes inputs programmatically (no screen interaction) and ensures data integrity through validation and deduplication.

## **Inputs**
- **Company Number** (`KYCO`, 2 chars): Identifies the company.
- **Customer Number** (`KYCUST`, 6 digits): Identifies the customer.
- **Date** (`KYDATE`, 6 digits, YYMMDD): Date for pricing records.
- **Customer Master File** (`PRCTUM`): Contains company number, customer number, product code, container code, unit of measure, and delete flag.
- **Rack Pricing File** (`RKPRCE`): Contains company number, location, price class, product code, description, container code, unit of measure, date, and time.
- **Company Control File** (`BICONT`): Contains company number and name.
- **Container File** (`GSCNTR1`): Contains container code, descriptions, and type.

## **Outputs**
- **Customer List File** (`BB956S`): Deduplicated list of company and customer numbers.
- **New Pricing File** (`NEWPROD`): Filtered pricing records.
- **Updated Customer Master File** (`PRCTUM`): Updated with new pricing records.
- **Pricing Download File** (`RKPRCEO`/`?9?PRCDOWN`): Validated pricing records for download.
- **Printed Report**: Details of filtered pricing records.

## **Process Steps**
1. **Create Deduplicated Customer List**:
   - Read `PRCTUM` records.
   - Check for duplicates in `BB956S` using company and customer number.
   - Write unique records to `BB956S` with active flag (`'A'`).

2. **Validate Inputs**:
   - Verify `KYCO` exists in `BICONT`.
   - Verify `KYCUST` exists in `BB956S`.
   - Validate `KYMO` (month from `KYDATE`) is 01–12.
   - Convert `KYDATE` to 8-digit format (e.g., 20YYMMDD) using Y2K logic:
     - If year (`KYYR`) >= 80, use century 19 (1900s); else, use 20 (2000s).

3. **Filter and Validate Pricing Records**:
   - Read `RKPRCE` records where date matches converted `KYDATE`.
   - Check non-existence in `PRCTUM` using company, customer, product, container, and unit of measure.
   - Validate container code in `GSCNTR1`, ensuring second description is "BRAD PENN" or "DA LUBRICANTS".
   - Write valid records to `NEWPROD` (company, location, product, container, unit of measure).
   - Generate printed report with company, location, price class, product, description, container, unit of measure, and date.

4. **Update Customer Master**:
   - Read `NEWPROD` records.
   - Check for duplicates in `PRCTUM` using company, customer, product, container, and unit of measure.
   - Add non-duplicate records to `PRCTUM` with active flag (`'A'`).

5. **Generate Download File**:
   - Clear `RKPRCEO` (`?9?PRCDOWN`).
   - Read `RKPRCE` records.
   - Write records to `RKPRCEO` if they match `PRCTUM` records by company, customer, product, container, and unit of measure, copying the entire record.

## **Business Rules**
1. **Deduplication**:
   - Ensure no duplicate company/customer pairs in `BB956S`.
   - Prevent duplicate pricing records in `PRCTUM` based on company, customer, product, container, and unit of measure.
2. **Validation**:
   - Company number must exist in `BICONT`.
   - Customer number must exist in `BB956S`.
   - Month must be 01–12.
   - Pricing records must match the input date and specific container descriptions ("BRAD PENN" or "DA LUBRICANTS").
3. **Data Integrity**:
   - Only active records (non-deleted, `PCDEL ≠ 'D'`) are processed.
   - New records in `PRCTUM` and `BB956S` are marked with active flag (`'A'`).
4. **Output**:
   - `NEWPROD` contains filtered pricing data for review and update.
   - `RKPRCEO` contains validated pricing records for external use.
   - Printed report documents filtered records.

## **Calculations**
- **Y2K Date Conversion**:
  - Input: `KYDATE` (6 digits, YYMMDD).
  - Multiply by 10000.01 to get 6-digit `DATYMD`.
  - If year (`KYYR`) >= 80, prepend century 19 (e.g., 19YYMMDD); else, prepend 20 (e.g., 20YYMMDD).
  - Output: 8-digit date (`DATE8`).

## **Error Handling**
- Return error if:
  - Company number is invalid (not in `BICONT`).
  - Customer number is invalid (not in `BB956S`).
  - Month is not 01–12.
- Skip records that fail date, `PRCTUM`, or container description validation.

## **Assumptions**
- Inputs are provided programmatically, bypassing screen interaction.
- All files (`PRCTUM`, `RKPRCE`, `BICONT`, `GSCNTR1`, `BB956S`, `NEWPROD`, `RKPRCEO`) are accessible in the specified library (`?9?`).
- `BB9564` (display step) is optional and skipped in a non-interactive context.

</xaiArtifact>