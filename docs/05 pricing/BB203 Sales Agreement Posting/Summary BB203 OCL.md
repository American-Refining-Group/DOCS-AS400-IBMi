The call stack consists of three components: the OCL script (`BB203.ocl36.txt`), the CLP program (`BB203AC.clp.txt`), and the RPGLE program (`BB203.rpgle.txt`). These work together to automate the posting of customer sales agreement pricing in an IBM System i (AS/400) environment. Below, I’ll identify the use cases implemented by this call stack and then provide a function requirement document for a non-interactive version of the primary use case.

### Use Cases Implemented in the Call Stack

The call stack implements the following use cases:

1. **Cleanup of Temporary Work File**:
   - **Description**: Ensures that any existing temporary work file (`BB203W`) in the `QTEMP` library is deleted before processing to prevent data conflicts from previous runs.
   - **Component**: Handled by `BB203AC.clp.txt`.
   - **Input**: A 1-character file group parameter (`&P$FGRP`).
   - **Process**: Deletes the `QTEMP/BB203W` file, ignoring errors if the file does not exist.
   - **Output**: None (clean slate for subsequent processing).
   - **Business Rule**: Ensures a clean environment by removing residual temporary files.

2. **Validation and Preparation of Input Data**:
   - **Description**: Validates the existence and content of the input workstation file (`?9?BB203?WS?`) to ensure it contains records for posting.
   - **Component**: Handled by the OCL script (`BB203.ocl36.txt`).
   - **Input**: Workstation file (`?9?BB203?WS?`) and a file group parameter (`?9?`).
   - **Process**:
     - Checks if the file exists; if not, pauses with a message prompting the user to run "Option 15" and cancels.
     - Verifies the file has records; if empty, pauses, deletes the file, and cancels.
   - **Output**: None (proceeds to call `BB203AC` and `BB203` if valid).
   - **Business Rule**: Ensures only valid, non-empty input data proceeds to processing.

3. **Automated Posting of Customer Sales Agreement Pricing**:
   - **Description**: Processes records from the temporary work file to update or expire existing pricing records, create new pricing records, log history, and generate a report.
   - **Component**: Handled by `BB203.rpgle.txt`, with preprocessing by `BB203AC` and setup by the OCL script.
   - **Input**:
     - Temporary work file (`BB203W`) containing pricing records.
     - User-entered start date (via display file `BB203D`).
     - File group parameter (`p$fgrp`, 'G' or 'Z') for file overrides.
   - **Process**:
     - Validates the user-entered start date against the work file’s start date.
     - Expires existing pricing records if their expiry date is greater than the new start date or zero.
     - Creates new pricing records with incremented sequence numbers.
     - Logs new and expired records to a history file.
     - Generates a printed report of old and new pricing details.
   - **Output**:
     - Updated pricing records in `BICUAG`.
     - History records in `BICUAGH`.
     - Updated sequence numbers in `BICONT`.
     - Printed report via `QSYSPRT`.
   - **Business Rules**:
     - Start date must match the work file’s start date.
     - Existing records are expired only if their expiry date is greater than the new start date or zero.
     - New records use unique sequence numbers.
     - All changes are logged to the history file.

### Function Requirement Document for Non-Interactive Automated Posting

Assuming the primary use case (Automated Posting of Customer Sales Agreement Pricing) is implemented as a non-interactive function that receives all inputs programmatically, below is a concise function requirement document. The document focuses on business requirements, process steps, and necessary calculations, omitting the interactive screen components.

<xaiArtifact artifact_id="d202cf67-3e07-45be-9a27-9c90ec65e8ce" artifact_version_id="d75a6cb4-f2c4-483d-84d9-240ba7e30840" title="Automated_Posting_Requirements.md" contentType="text/markdown">

# Function Requirement Document: Automated Customer Sales Agreement Pricing Posting

## Purpose
Automate the posting of customer sales agreement pricing by processing records from a temporary work file, updating or expiring existing pricing records, creating new pricing records, logging history, and generating a report, without user interaction.

## Inputs
- **File Group Parameter** (`p$fgrp`): 1-character code ('G' or 'Z') to determine database file overrides (e.g., `GBICUAG` or `ZBICUAG`).
- **Work File** (`BB203W` in `QTEMP`): Contains pricing records with fields:
  - `new_bacono`: Company number.
  - `new_baseqn`: Sequence number.
  - `new_bacust`: Customer number.
  - `new_baloc`: Location.
  - `new_banprc`: New price.
  - `new_baoffp`: New off-price.
  - `new_banstd`: Start date (CCYYMMDD).
  - Other pricing fields (e.g., `new_bapr01` to `new_bapr10`, `new_bamnqy`, `new_bamxqy`, etc.).
- **Start Date** (`start_date`): CCYYMMDD format, provided programmatically (replacing user input via screen).

## Outputs
- **Updated Pricing File** (`BICUAG`): Updated with expired records and new pricing records.
- **History File** (`BICUAGH`): Logs new and expired pricing records.
- **Sequence File** (`BICONT`): Updated with incremented sequence numbers.
- **Printed Report** (`QSYSPRT`): Details old and new pricing information (company, customer, location, prices, dates, sequence numbers).

## Process Steps
1. **Validate Input Data**:
   - Verify `BB203W` exists and contains records.
   - Ensure `start_date` matches `new_banstd` in `BB203W` for all records.
   - If invalid or mismatched, return an error message and abort.

2. **Clean Up Temporary File**:
   - Delete any existing `QTEMP/BB203W` file to ensure a clean slate.
   - Create a new `BB203W` file by copying input data (replacing OCL’s `CRTDUPOBJ`).

3. **Process Each Work File Record**:
   - Read all records from `BB203W`.
   - For each record:
     - **Expire Existing Record**:
       - Chain to `BICUAG` using `new_bacono` and `new_baseqn`.
       - If found and `baend8` (expiry date) is greater than `start_date` or zero:
         - Calculate expiry date as `start_date - 1 day` (using date calculation logic).
         - Set `baend8` to expiry date, `baentm` to 23:59, and update `BICUAG`.
         - Write expired record to `BICUAGH`.
       - If not found, note for reporting.
     - **Create New Record**:
       - Retrieve and increment sequence number from `BICONT` for `new_bacono`.
       - Chain to `BICUAG` with new sequence number to ensure no conflict.
       - Populate new `BICUAG` record with `BB203W` fields, setting:
         - `bastd8` to `start_date`, `basttm` to 00:01.
         - `baend8` and `baendt` to 0 (no expiry).
       - Write new record to `BICUAG` and `BICUAGH`.
     - **Generate Report**:
       - Print a detail line to `QSYSPRT` with old and new pricing data (company, customer, location, prices, dates, sequence numbers).
       - Handle overflow by printing headers as needed.

4. **Update Sequence File**:
   - Update `BICONT` with the incremented sequence number for the company.

5. **Clean Up**:
   - Delete `QTEMP/BB203W` and remove file overrides.

## Business Rules
- **Data Validation**:
  - `start_date` must match `new_banstd` in all `BB203W` records.
  - `BB203W` must exist and contain records.
- **Expiry Logic**:
  - Expire existing `BICUAG` records only if `baend8` > `start_date` or `baend8` = 0.
  - Expiry date is set to `start_date - 1 day` at 23:59.
- **Sequence Number**:
  - New records use a unique sequence number, incremented from `BICONT` for the company.
- **History Logging**:
  - All new and expired records are logged to `BICUAGH` for audit purposes.
- **File Overrides**:
  - Use `p$fgrp` ('G' or 'Z') to select appropriate database files (e.g., `GBICUAG` or `ZBICUAG`).
- **Error Handling**:
  - Abort with an error message if validation fails (e.g., missing file, mismatched start date).
- **Reporting**:
  - Generate a report with headers and detail lines, including old/new prices, dates, and sequence numbers.

## Calculations
- **Expiry Date Calculation**:
  - Input: `start_date` (CCYYMMDD).
  - Output: Expiry date = `start_date - 1 day` (CCYYMMDD).
  - Logic: Use a date calculation routine (e.g., equivalent to `GSDTCLC1`) to subtract 1 day, handling month/year boundaries and leap years.
  - Example: If `start_date = 20250115`, expiry date = `20250114`.
- **Sequence Number Increment**:
  - Retrieve `bcseqn` from `BICONT` for `new_bacono`.
  - Increment by 1 and update `BICONT`.
  - Example: If `bcseqn = 100`, new sequence number = `101`.

## Dependencies
- **Files**:
  - `BICUAG`: Pricing records (update).
  - `BB203W`: Temporary work file (input).
  - `BICONT`: Sequence numbers (update).
  - `BICUAGH`: History records (output).
  - `QSYSPRT`: Printer file (output).
- **External Program**:
  - Date calculation routine (equivalent to `GSDTCLC1`) for expiry date computation.

</xaiArtifact>

### Notes
- The function requirement document assumes a non-interactive process, replacing the interactive screen (`BB203D`) with a programmatically provided `start_date`. This simplifies the process by removing user interaction while preserving all business logic.
- The cleanup use case (`BB203AC`) is integrated into the process steps to ensure a clean slate.
- The validation use case from the OCL script is incorporated into the initial validation step.
- The document focuses on business requirements and calculations, ensuring clarity and conciseness for implementation.