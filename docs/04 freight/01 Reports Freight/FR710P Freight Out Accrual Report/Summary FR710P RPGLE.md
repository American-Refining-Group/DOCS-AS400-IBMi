Below, I outline the use cases implemented by the call stack of programs (`FR710P`, `FR710PC`, `FR710C`, `FR710A`, `FR710B`) for the Freight Out Accrual Report system. Following that, I provide a concise **Function Requirement Document** for a non-interactive function that encapsulates the primary use case, focusing on business requirements and calculations.

### Use Cases Implemented by the Call Stack

The call stack collectively implements the Freight Out Accrual Report generation process. Based on the program analysis, the following use cases are identified:

1. **Generate Freight Out Accrual Report (Primary Use Case)**:
   - **Description**: Generates a detailed report of freight accrual data for a specified company and report date, including billed, expected, and actual carrier freight amounts, and their differences, organized by company and freight general ledger (GL) account. The report is produced in two formats: a standard printed report and an Excel-compatible format.
   - **Components Involved**:
     - `FR710P`: Collects and validates user input (company number, report date, file group).
     - `FR710PC`: Coordinates the report generation by calling `FR710C` and displaying a completion message.
     - `FR710C`: Manages the creation and clearing of the `FR710W` work file and orchestrates calls to `FR710A` and `FR710B`.
     - `FR710A`: Builds the `FR710W` work file by selecting and processing freight billing and invoice data based on specific criteria.
     - `FR710B`: Prints the report using data from `FR710W`, including detail lines, GL totals, company totals, and positive/negative difference summaries.
   - **Inputs**: Company number, report date (month and year, with day fixed to 31), file group ('G' for production or 'Z' for development).
   - **Outputs**: A printed report (`LIST164`) and an Excel-compatible report (`LIST378`) with freight accrual details and totals.
   - **Key Features**:
     - Validates company number against `GLCONT`.
     - Filters freight billing records based on shipment date, close status, customer-owned product, and freight calculation requirements.
     - Calculates freight differences (invoiced vs. estimated or booked) at header and detail levels.
     - Supports multi-user environments by using `QTEMP` for the work file.
     - Provides two output formats for flexibility in reporting.

2. **Validate Input Parameters for Report Generation**:
   - **Description**: Ensures that the company number, report month, and report year provided by the user are valid before proceeding with report generation.
   - **Components Involved**:
     - `FR710P`: Performs validation checks on input fields (company number, month, year).
   - **Inputs**: Company number (2 characters), report month (1-12), report year (non-zero), file group ('G' or 'Z').
   - **Outputs**: Validated parameters or error messages if validation fails.
   - **Key Features**:
     - Checks if the company number exists in `GLCONT` and retrieves the company name.
     - Ensures the month is between 1 and 12 and the year is non-zero.
     - Displays error messages for invalid inputs and allows re-entry.

3. **Support Multi-Environment Data Access**:
   - **Description**: Allows the report to be generated using different data sets (production or development) based on the file group parameter, ensuring flexibility across environments.
   - **Components Involved**:
     - `FR710P`, `FR710A`, `FR710B`: Apply file overrides for `GLCONT`, `GLMAST`, `FRBINH`, `FRBINF`, `FRCINHJ1`, and `FRBIND` based on `P$FGRP`.
     - `FR710C`: Applies overrides for `FR710W` based on `P$FGRP`.
   - **Inputs**: File group parameter ('G' or 'Z').
   - **Outputs**: Access to appropriate files (`GGL*` for production, `ZGL*` for development, `DATA` or `DATADEV` for `FR710W`).
   - **Key Features**:
     - Dynamically overrides files to point to production (`G`) or development (`Z`) libraries.
     - Ensures correct data sources are used without modifying program logic.

4. **Create Multi-User Capable Work File**:
   - **Description**: Creates a job-specific temporary work file (`FR710W`) in `QTEMP` to support concurrent report generation by multiple users without data conflicts.
   - **Components Involved**:
     - `FR710C`: Creates and clears `FR710W` in `QTEMP`, applying overrides to ensure job-specific access.
   - **Inputs**: File group parameter ('G' or 'Z') to determine the source library (`DATA` or `DATADEV`).
   - **Outputs**: A cleared `FR710W` file in `QTEMP` ready for data population.
   - **Key Features**:
     - Checks for the existence of `FR710W` and creates it if absent, copying from the appropriate library.
     - Clears the file to ensure no residual data.
     - Overrides file references to `QTEMP/FR710W` for isolation.

### Function Requirement Document: Generate Freight Out Accrual Report

<xaiArtifact artifact_id="6d40d45d-6702-46fa-91c6-c5acb7b82c69" artifact_version_id="70ab595f-4504-423e-9006-e3ea7b313214" title="FreightOutAccrualFunctionRequirements.md" contentType="text/markdown">

# Function Requirement Document: Generate Freight Out Accrual Report

## Purpose
Generate a Freight Out Accrual Report for a specified company and report date, calculating freight differences (invoiced vs. estimated or booked) and producing outputs in standard and Excel-compatible formats.

## Inputs
- **Company Number** (`P$CO`): 2-character code identifying the company.
- **Report Date** (`P$RDAT`): 8-digit date (`CCYYMMDD`) with day fixed to 31.
- **File Group** (`P$FGRP`): 1-character code ('G' for production, 'Z' for development).

## Outputs
- **Standard Report** (`LIST164`): 164-character wide printed report with detail lines, freight GL totals, and company totals.
- **Excel-Compatible Report** (`LIST378`): 378-character wide report for spreadsheet export.
- **Temporary Work File** (`FR710W`): Populated with processed freight data in `QTEMP`.

## Process Steps
1. **Validate Inputs**:
   - Verify company number exists in `GLCONT` and retrieve company name (`GCNAME`).
   - Ensure report month is 1-12 and year is non-zero.
   - Return errors if validation fails.

2. **Apply File Overrides**:
   - For `P$FGRP` = 'G', use production files (`GGLMAST`, `GGLCONT`, `GFRBINH`, `GFRBINF`, `GFRCINHJ1`, `GFRBIND`, `DATA/FR710W`).
   - For `P$FGRP` = 'Z', use development files (`ZGLMAST`, `ZGLCONT`, `ZFRBINH`, `ZFRBINF`, `ZFRCINHJ1`, `ZFRBIND`, `DATADEV/FR710W`).

3. **Create/Clear Work File**:
   - Check if `FR710W` exists in `QTEMP`; if not, copy from `DATA` ('G') or `DATADEV` ('Z').
   - Clear `FR710W` and override to `QTEMP/FR710W` for job-specific access.

4. **Build Work File**:
   - Read `FRBINH` (freight billing header) for the specified company, filtering records:
     - Exclude if customer-owned product (`BOCOON` = 'Y').
     - Exclude if freight calculation not required (`BOCAFR` = 'N') and freight total (`BOFRTO`) is zero.
     - Include if shipped on/before report date (`BOSHD8` ≤ `P$RDAT`) and not closed (`BOCLOS` ≠ 'C'), or closed (`BOCLOS` = 'C') but closed after report date (`BOCLD8` > `P$RDAT`).
   - For each header record, aggregate:
     - Booked freight (`BFTFAM`) and estimated freight (`CFTFAM`) from `FRBINF`.
     - Invoiced freight (`FRINAM` - `FRFBOA`) from `FRCINHJ1`.
   - Calculate header-level difference (`W$DIFF`):
     - If estimated freight (`W$EFRT`) ≠ 0, `W$DIFF` = `W$CFRT` - `W$EFRT`.
     - Else, `W$DIFF` = `W$CFRT` - `W$BFRT`.
     - Set `W$DIFF` to 0 if order closed in selected month (`BOCLOS` = 'C' and `BOCLD8` ≤ `P$RDAT`).
   - For each detail record in `FRBIND`:
     - Prorate amounts using percentage (`BDPCTT`): `W$BFRTDTL` = `W$BFRT` × `BDPCTT`, `W$EFRTDTL` = `W$EFRT` × `BDPCTT`, `W$CFRTDTL` = `W$CFRT` × `BDPCTT`.
     - Calculate detail-level difference (`W$DIFFDTL`):
       - If `W$EFRTDTL` ≠ 0, `W$DIFFDTL` = `W$CFRTDTL` - `W$EFRTDTL`.
       - Else, `W$DIFFDTL` = `W$CFRTDTL` - `W$BFRTDTL`.
       - Set `W$DIFFDTL` to 0 if order closed in selected month.
     - Write to `FR710W` if order is not closed or `W$DIFFDTL` ≠ 0, including fields: company, customer, name, freight GL, route code, ship date, order number, serial number, invoice number, type, quantity, unit of measure, close status, sequence, freight amounts, and difference.

5. **Generate Report**:
   - Read `FR710W` with level breaks on company (`W1CO`) and freight GL (`W1FRGL`).
   - Retrieve company name (`GCNAME`) from `GLCONT` and GL description (`GLDESC`) from `GLMAST`.
   - For each record, print detail line with company, freight GL, routing, ship date, customer number, name, order number, serial number, invoice number, type, quantity, unit of measure, billed freight (`W1BFRT`), expected carrier (`W1EFRT`), actual carrier (`W1AFRT`), difference (`W1DIFF`), close status, and sequence.
   - Accumulate totals at freight GL (`L1BFRT`, `L1EFRT`, `L1AFRT`, `L1DIFF`) and company (`L2BFRT`, `L2EFRT`, `L2AFRT`, `L2DIFF`) levels.
   - Track positive (`L2POST`) and negative (`L2NEGT`) differences.
   - Print headers with company, report title, page number, job details, date, time, and selected report date.
   - Print freight GL totals (`TOTL1`) and company totals (`TOTL2`) with positive/negative difference summaries.
   - Output to `LIST164` (standard) and `LIST378` (Excel-compatible).

6. **Notify Completion**:
   - Return a status message: "Freight Out Accrual Report By Company Has Been Printed".

## Business Rules
- **Input Validation**:
  - Company number must exist in `GLCONT`.
  - Report month must be 1-12; year must be non-zero.
  - Report date day is fixed to 31.
- **Record Selection**:
  - Exclude customer-owned products (`BOCOON` = 'Y').
  - Exclude records with no freight calculation (`BACAFR` = 'N') and zero freight total (`BOFRTO` = 0).
  - Include orders shipped on/before report date and not closed, or closed after report date.
  - Zero differences for orders closed in the selected month.
- **Calculations**:
  - Header difference: Invoiced freight (`W$CFRT` = `FRINAM` - `FRFBOA`) minus estimated (`W$EFRT`) or booked (`W$BFRT`) freight.
  - Detail difference: Prorated invoiced freight (`W$CFRTDTL`) minus prorated estimated (`W$EFRTDTL`) or booked (`W$BFRTDTL`) freight.
  - Proration uses `BDPCTT` percentage from `FRBIND`.
- **Multi-Environment Support**:
  - Use production (`G`) or development (`Z`) files based on `P$FGRP`.
- **Multi-User Support**:
  - Create `FR710W` in `QTEMP` for job-specific isolation.
- **Report Formatting**:
  - Standard report (`LIST164`) is 164 characters wide; Excel report (`LIST378`) is 378 characters wide.
  - Include headers, detail lines, freight GL totals, company totals, and positive/negative difference summaries.

## Dependencies
- **Files**:
  - Input: `GLCONT`, `GLMAST`, `FRBINH`, `FRBINF`, `FRCINHJ1`, `FRBIND`.
  - Output: `FR710W` (temporary), `LIST164`, `LIST378`.
- **System APIs**: `QCMDEXC` for file overrides.

</xaiArtifact>