The GS800 program, consisting of the OCL (GS800.ocl36.txt) and RPG III (GS800.rpg36.txt) components, is designed for table file maintenance on an IBM System/36 or AS/400 system. Below, I will first identify the **use cases** implemented by the program based on the provided code. Then, I will provide a **Function Requirement Document** for one of these use cases, reimagined as a non-interactive function that processes inputs to complete the task, focusing on business requirements and necessary calculations.

---

### List of Use Cases Implemented by GS800

The GS800 program supports multiple use cases, each corresponding to the maintenance (add/update/delete) of specific record types through interactive screens (S1–S8, SA, SB). Each use case is tied to a specific record type and its associated validation and processing logic. Based on the RPG code and comments, the following use cases are implemented:

1. **Maintain Record Types (TYPE)**:
   - **Description**: Allows users to define or update the structure of record types (e.g., code length and data type: Alpha or Numeric) using screen S3.
   - **Details**: Validates code length (1–6) and data type (A or N). Used for configuring the `TYPE` records in the GSTABL file.
   - **Screens**: S1 (select type), S3 (add/update).

2. **Maintain Payment Terms (APTERM, ARTERM)**:
   - **Description**: Manages payment terms records, including discounts, net days, prox days, rack price add-on, and discount days, using screen S3.
   - **Details**: Ensures description is not blank, validates terms percentages (must total 100% for ARTERM), and stores data in GSTABL.
   - **Screens**: S1 (select type), S3 (add/update).

3. **Maintain Product Codes (PRODCD)**:
   - **Description**: Manages product code records, including product group, class, VCF code, fluid code, STCC code, inactive date, and report-as-sales flag, using screen S4.
   - **Details**: Validates product code (0001–9999), company code (01–98), VCF code (exists in INVCF), fluid code (Y/N/blank), report-as-sales (Y/N/blank), and inactive date. Stores data in GSPROD. Note: As of JB13, PRODCD maintenance is restricted and handled in the GS916 series.
   - **Screens**: S1 (select type), S4 (add/update).

4. **Maintain Product Groups (PRODGR)**:
   - **Description**: Manages product group records, including description and sequence, using screen S5.
   - **Details**: Ensures description is not blank and stores data in GSTABL.
   - **Screens**: S1 (select type), S5 (add/update).

5. **Maintain Inventory Counts (INCOUN)**:
   - **Description**: Manages inventory count records, including description and unit type, using screen S5.
   - **Details**: Validates unit type (B or blank) and ensures description is not blank. Stores data in GSTABL.
   - **Screens**: S1 (select type), S5 (add/update).

6. **Maintain Product Classes (PRODCL)**:
   - **Description**: Manages product class records, including description and sort code, using screen S6.
   - **Details**: Ensures description is not blank and validates sort code. Stores data in GSTABL.
   - **Screens**: S1 (select type), S6 (add/update).

7. **Maintain Container Codes (CNTRCD)**:
   - **Description**: Manages container code records, including container type, source, and IMS unit of measure, using screen S7.
   - **Details**: Ensures container code is non-zero, container type is B (Bulk) or P (Packaged), container source is valid or blank, and IMS unit of measure exists in BIUNMS for packaged containers. Stores data in GSCNTR. Note: As of JB15, CNTRCD maintenance is restricted and handled elsewhere.
   - **Screens**: S1 (select type), S7 (add/update).

8. **Maintain Sales Managers (SLSMGR)**:
   - **Description**: Manages sales manager records, including salesman numbers, using screen S8.
   - **Details**: Validates salesman numbers against SLSMAN records in GSTABLZ. Stores data in GSTABL.
   - **Screens**: S1 (select type), S8 (add/update).

9. **Maintain BI Unit Measures (BIUNMS)**:
   - **Description**: Manages BI unit measure records, including description and fluid code, using screen SA.
   - **Details**: Ensures description is not blank and fluid code is Y or N. Stores data in GSTABL.
   - **Screens**: S1 (select type), SA (add/update).

10. **Maintain Container Profiles (CNTRPF)**:
    - **Description**: Manages container profile records, including description and sales G/L prefix, using screen SB.
    - **Details**: Ensures description and sales G/L prefix are not blank/zero. Stores data in GSTABL.
    - **Screens**: S1 (select type), SB (add/update).

11. **List Records for a Selected Type**:
    - **Description**: Displays a list of records for a selected type, showing codes, descriptions, and type-specific fields (e.g., discount for APTERM/ARTERM), using screen S2.
    - **Details**: Supports scrolling forward/backward through records in GSTABLX. Used as a precursor to selecting records for maintenance.
    - **Screens**: S2 (list display).

---

### Function Requirement Document: Maintain Product Codes (PRODCD)

Below is a **Function Requirement Document** for the **Maintain Product Codes (PRODCD)** use case, reimagined as a non-interactive function that processes inputs to add or update product code records. The document focuses on business requirements, process steps, and calculations, with minimal technical implementation details.

<xaiArtifact artifact_id="ab64a850-ff0d-44e8-83a6-705881bec315" artifact_version_id="e385e608-74cd-4034-a99e-22fea8021229" title="ProductCodeMaintenanceRequirements.md" contentType="text/markdown">

# Function Requirement Document: Maintain Product Codes (PRODCD)

## Purpose
The `MaintainProductCodes` function allows the addition or update of product code records in the product file (GSPROD) to support inventory and sales reporting processes. The function ensures data integrity through strict validation of input fields and adherence to business rules.

## Inputs
- **Company Code (TPCONO)**: 2-digit numeric (01–98).
- **Product Code (TPPROD)**: 4-digit numeric (0001–9999).
- **Description (TPDESC)**: 30-character alphanumeric, mandatory.
- **Product Group (TPPRGP)**: 2-character alphanumeric.
- **Product Class (TPPRCL)**: 3-character alphanumeric, mandatory.
- **Product Ledger Group (TPPLGR)**: 2-digit numeric.
- **Product Ledger Class (TPPLCL)**: 2-digit numeric.
- **Product Ledger High Group (TPPLGH)**: 15-character alphanumeric.
- **Product Ledger High Class (TPPLCH)**: 10-character alphanumeric.
- **Container Code (TPTOCO)**: 2-digit numeric.
- **Color Code (TPCOL)**: 2-digit numeric.
- **Inventory Color (TPICOL)**: 2-digit numeric.
- **Ship Code (TPSHDS)**: 30-character alphanumeric.
- **Sales Message (TPSMSG)**: 1-character (D/K/G or blank).
- **Gravity (TPGRAV)**: 3-digit numeric (with 1 decimal place, e.g., 12.3).
- **Inventory Group (TPINGP)**: 2-digit numeric.
- **Sales G/L Number (TPSGL#)**: 8-digit numeric.
- **Alternate Description (TPALTD)**: 20-character alphanumeric.
- **Alternate Business Description (TPABDS)**: 10-character alphanumeric.
- **Sell Product (TPSELL)**: 1-character (Y/N).
- **Sequence Number (TPTISQ)**: 6-digit numeric (with 1 decimal place, e.g., 12345.6).
- **VCF Code (TPVCF)**: 2-character alphanumeric, must exist in INVCF type.
- **Fluid Code (TPFLCD)**: 1-character (Y/N or blank).
- **STCC Code (TPSTCC)**: 8-character alphanumeric.
- **Inactive Date (TPIND8)**: 8-digit numeric (MMDDYYYY), optional.
- **Report as Sales (TPRPSL)**: 1-character (Y/N or blank).
- **System Date/Time (DATTIM)**: 14-character numeric (YYYYMMDDHHMMSS), auto-populated.

## Outputs
- **Status**: Success or failure with an error message (if applicable).
- **Updated Record**: The added or updated record in the GSPROD file.

## Process Steps
1. **Validate Inputs**:
   - Check that all mandatory fields (Description, Product Class) are non-blank.
   - Validate Company Code (01–98).
   - Validate Product Code (0001–9999).
   - Ensure VCF Code exists in INVCF type records in GSTABL.
   - Validate Fluid Code (Y, N, or blank).
   - Validate Report as Sales (Y, N, or blank).
   - Validate Inactive Date (if provided) using the Date Validation process.
   - Ensure Product Class and Inventory Group are non-blank.
   - Validate STCC Code (8 characters, alphanumeric).
   - Check other fields for correct format (e.g., numeric fields like Product Ledger Group).

2. **Check Record Existence**:
   - Construct a key (Company Code + Product Code) and check if a record exists in GSPROD.
   - If the record exists, prepare for update; otherwise, prepare for addition.

3. **Populate Record**:
   - Map input fields to GSPROD file fields (e.g., TPDESC, TPPRGP, TPVCF).
   - Set Delete Flag (TPDEL) to blank for active records.
   - Auto-populate System Date/Time (DATTIM) with the current date and time in YYYYMMDDHHMMSS format.

4. **Write/Update Record**:
   - If adding, write a new record to GSPROD.
   - If updating, overwrite the existing record in GSPROD.

5. **Return Status**:
   - Return "Success" if the operation completes without errors.
   - Return "Failure" with an appropriate error message if validation fails (e.g., "Description May Not Be Blank", "Invalid Product Code").

## Business Rules
- **Mandatory Fields**: Description and Product Class must be provided.
- **Code Ranges**:
  - Company Code: Must be between 01 and 98.
  - Product Code: Must be between 0001 and 9999.
- **VCF Code Validation**: Must exist in INVCF type records in GSTABL.
- **Fluid Code**: Must be Y, N, or blank.
- **Report as Sales**: Must be Y, N, or blank.
- **Product Class and Inventory Group**: Must be non-blank to ensure proper categorization.
- **Inactive Date**: If provided, must be a valid date (MMDDYYYY) with:
  - Month: 1–12.
  - Day: 1–31 (or 1–28/29 for February, accounting for leap years).
  - Leap Year: Years divisible by 4 (non-century) or 400 (century) allow February 29.
- **STCC Code**: Must be 8 characters, alphanumeric.
- **Restricted Maintenance**: PRODCD records are no longer maintained in GS800; this function assumes it is called by an authorized program (e.g., GS916 series).

## Calculations
- **System Date/Time (DATTIM)**:
  - Retrieve system date (YYYYMMDD) and time (HHMMSS).
  - Combine into a 14-digit string: YYYYMMDDHHMMSS.
  - Example: For 2025-09-04 12:39:00, DATTIM = 20250904123900.
- **Inactive Date Validation**:
  - Input: 8-digit MMDDYYYY.
  - Extract month (MM), day (DD), and year (YYYY).
  - Validate month (1–12).
  - Validate day based on month:
    - February: 28 days (non-leap year) or 29 days (leap year).
    - Months 4, 6, 9, 11: 30 days.
    - Other months: 31 days.
  - Leap Year Check:
    - Non-century year: Divisible by 4 (e.g., 2024).
    - Century year: Divisible by 400 (e.g., 2000).
  - Output: Valid date or error (e.g., "Inactive Date is Invalid").

## Error Messages
- "Description May Not Be Blank" (if Description is empty).
- "Product Code range is 0001 - 9999" (if Product Code is invalid).
- "Company Range is 01 thru 98" (if Company Code is invalid).
- "VCF Code is not found in Type: INVCF" (if VCF Code is invalid).
- "Fluid Code must be Y or N" (if Fluid Code is invalid).
- "Report as Sales must be Y, N, or blank" (if Report as Sales is invalid).
- "Inactive Date is Invalid" (if Inactive Date fails validation).
- "Product Class cannot be blank" (if Product Class is empty).
- "Inventory Group cannot be blank" (if Inventory Group is empty).

## Assumptions
- The function is called by an authorized program (e.g., GS916 series) due to restrictions on PRODCD maintenance in GS800.
- The GSPROD and GSTABL files are accessible and properly configured.
- System date/time is available from the environment.

</xaiArtifact>

---

### Notes
- The **Maintain Product Codes (PRODCD)** use case was chosen for the function requirement document because it is one of the most complex, involving multiple validations and fields, and it interacts with the GSPROD file, which was added to the program via updates (JB01, JB03).
- The document assumes a non-interactive function, so screen interactions (e.g., S1, S4) are replaced with direct input validation and processing.
- Other use cases (e.g., APTERM, CNTRCD) could be documented similarly but involve fewer fields and validations. If you need documents for additional use cases, please specify!
- The program does not call external programs directly, but restrictions (JB11, JB13, JB15) indicate dependencies on other programs (BB905, GS916 series) for certain types, which are noted in the business rules.

Let me know if you need further clarification, additional use case documents, or specific modifications to the provided artifact!