Based on the provided OCL (`AR933P.ocl36.txt`) and RPGLE (`AR933P.rpgle.txt`) programs, I will first identify the use cases implemented by the program. Then, I will create a function requirements document for a single use case, reimagined as a large function that accepts inputs programmatically rather than through a screen, focusing on business requirements and necessary calculations.

---

### Use Cases Implemented by the Program

The `AR933P` program (OCL and RPGLE combined) implements a single primary use case:

1. **Update Accounts Receivable (A/R) Closed Date for a Company**:
   - **Description**: The program allows a user to update the A/R closed month and year for a specified company in the `ARCONT` file, ensuring the input is valid and the new date is not earlier than the existing closed date. It retrieves and displays up to three non-deleted company records (company number, name, month, and year) for reference and provides feedback on success or errors.
   - **Inputs**: Company number, month, and year (via the `AR933PD` display file).
   - **Outputs**: Updated `ARCONT` record, success message, or error message (e.g., invalid company, month, year, or date progression).
   - **Process**: Validates inputs, checks the existing closed date, updates the `ARCONT` file, and refreshes the display with updated company data or error messages.

No additional distinct use cases are implemented, as the program focuses solely on updating the A/R closed date with associated validation and display logic.

---

### Function Requirements Document

The following document describes a function that implements the "Update Accounts Receivable Closed Date for a Company" use case, reimagined as a programmatic function that accepts inputs directly (no screen interaction). It outlines the business requirements, process steps, and calculations concisely.



# Function Requirements: Update A/R Closed Date

## Function Name
`UpdateARClosedDate`

## Purpose
Update the Accounts Receivable (A/R) closed month and year for a specified company in the `ARCONT` file, ensuring the input is valid and the new date is not earlier than the existing closed date.

## Inputs
- **Company Number** (`CompanyNo`, 2-digit numeric): The unique identifier of the company.
- **Month** (`NewMonth`, 2-digit numeric): The new A/R closed month (01-12).
- **Year** (`NewYear`, 4-digit numeric): The new A/R closed year (2000-2079).

## Outputs
- **Status** (String): `"SUCCESS"` or an error message (e.g., "INVALID COMPANY NUMBER", "INVALID MONTH ENTERED", "INVALID YEAR ENTERED", "NEW DATE IS EARLIER THAN PREVIOUS DATE").
- **Updated Company Data** (Optional, Array of Records): Up to three non-deleted company records, each containing:
  - Company Number (2-digit numeric)
  - Company Name (30-character string)
  - Closed Month (2-digit numeric)
  - Closed Year (4-digit numeric)

## Business Requirements
1. **Company Number Validation**: The `CompanyNo` must exist in the `ARCONT` file.
2. **Month Validation**: The `NewMonth` must be between 01 and 12.
3. **Year Validation**: The `NewYear` must be between 2000 and 2079.
4. **Date Progression**: The new date (`NewYear` + `NewMonth`, formatted as YYYYMM) must not be earlier than the existing closed date in `ARCONT`.
5. **Update**: If all validations pass, update the `ARCONT` record with `NewMonth` and `NewYear`.
6. **Error Handling**: Return specific error messages for invalid inputs or date issues.
7. **Company Data Retrieval**: Retrieve up to three non-deleted (`ACDEL ≠ 'D'`) company records from `ARCONT` for reference.

## Process Steps
1. **Initialize**:
   - Clear internal variables and set default status.
   - Access `GSCONT` file with key `'00'` to retrieve a control company number, if applicable.

2. **Validate Inputs**:
   - **Company Number**:
     - Query `ARCONT` with `CompanyNo`. If not found, return "INVALID COMPANY NUMBER".
   - **Month**:
     - Check if `NewMonth` is between 01 and 12. If not, return "INVALID MONTH ENTERED".
   - **Year**:
     - Check if `NewYear` is between 2000 and 2079. If not, return "INVALID YEAR ENTERED".
   - **Date Comparison**:
     - Combine `NewYear` and `NewMonth` into a 6-digit date (`YYYYMM`).
     - Compare with `ACDTCL` (existing closed date) from `ARCONT`. If new date is earlier, return "NEW DATE IS EARLIER THAN PREVIOUS DATE".

3. **Update `ARCONT`**:
   - If all validations pass, update the `ARCONT` record for `CompanyNo` with:
     - `ACMNTH` = `NewMonth` (positions 130-131)
     - `ACYEAR` = `NewYear` (positions 126-129)

4. **Retrieve Company Data**:
   - Read up to three non-deleted (`ACDEL ≠ 'D'`) records from `ARCONT`, storing:
     - Company Number (`ACCO`)
     - Company Name (`ACNAME`)
     - Closed Month (`ACMNTH`)
     - Closed Year (`ACYEAR`)
   - Return these records in the output.

5. **Return Status**:
   - If update succeeds, return "SUCCESS" and company data.
   - If any validation fails, return the corresponding error message.

## Calculations
- **Date Formatting**:
  - Combine `NewYear` and `NewMonth` into a 6-digit numeric date (`YYYYMM`) for comparison.
  - Example: For `NewYear = 2025`, `NewMonth = 06`, create `DATE = 202506`.
- **Date Comparison**:
  - Compare `DATE` (new date) with `ACDTCL` (existing closed date, formatted as `YYYYMM`) to ensure the new date is not earlier.
  - Example: If `ACDTCL = 202505` and `DATE = 202504`, the comparison fails (202504 < 202505).

## Data Sources
- **ARCONT** (Updatable Disk File):
  - Key: Company Number (`ACCO`, position 2)
  - Fields: `ACDEL` (delete flag), `ACCO` (company number), `ACNAME` (company name), `ACYEAR` (closed year), `ACMNTH` (closed month), `ACDTCL` (closed date, YYYYMM)
- **GSCONT** (Input-Only Disk File):
  - Key: Company Number (`GXCONO`, position 2)
  - Used to retrieve a control record (key `'00'`) for default company number.

## Error Messages
- "INVALID COMPANY NUMBER": `CompanyNo` not found in `ARCONT`.
- "INVALID MONTH ENTERED": `NewMonth` not 01-12.
- "INVALID YEAR ENTERED": `NewYear` not 2000-2079.
- "NEW DATE IS EARLIER THAN PREVIOUS DATE": New date (`YYYYMM`) earlier than `ACDTCL`.
- "SUCCESS": Update completed successfully.

## Notes
- The function assumes direct input without user interaction, replacing the original screen-based input (`AR933PD`).
- No external programs are called; all logic is self-contained.
- The function handles cancellation internally (e.g., via an invalid input check) rather than a cancel key.



---

This document reimagines the `AR933P` program's functionality as a single, programmatic function, maintaining the core business logic and validations while eliminating screen-based interaction. Let me know if you need further refinements or additional details!