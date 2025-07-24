### List of Use Cases Implemented by the Program Call Stack

The RPGLE program call stack, consisting of `BB928P`, `BB928`, `BB9284`, and `BB9285`, implements a set of functionalities for managing salesman ID records within a billing and invoicing system on IBM i. These programs collectively support the following use cases:

1. **View Salesman ID Records**:
   - Allows users to view a list of salesman ID records in a subfile, with options to filter by company number and salesman ID, and display all records (active and inactive) in inquiry mode.

2. **Create a Salesman ID Record**:
   - Enables users to create a new salesman ID record, specifying company number, salesman ID, name, and email address, ensuring the company exists and the record does not already exist.

3. **Update a Salesman ID Record**:
   - Permits users to modify existing salesman ID records (name and email address), protecting key fields (company number and salesman ID) to maintain data integrity.

4. **Inactivate/Reactivate a Salesman ID Record**:
   - Allows users to toggle the status of a salesman ID record between active (`A`) and inactive (`I`), ensuring the record exists and is in the appropriate state for the action.

5. **Print a Salesman ID Listing**:
   - Generates a printed report of all salesman ID records, including company number, salesman ID, name, email address, and status, sorted by company and salesman ID.

### Function Requirement Document: Manage Salesman ID Records



# Salesman ID Management Function Requirements

## Purpose
The `ManageSalesmanID` function manages salesman ID records in a billing and invoicing system, supporting creation, update, inactivation/reactivation, and printing of records without interactive screen handling. It processes inputs directly and applies business rules to maintain data integrity.

## Inputs
- **Company Number (`co`)**: Numeric, identifies the company (validated against `bicont`).
- **Salesman ID (`smid`)**: Alphanumeric, unique identifier for the salesman.
- **Salesman Name (`smnm`)**: Alphanumeric, required for create/update.
- **Email Address (`emal`)**: Alphanumeric, required for create/update.
- **File Group (`fgrp`)**: `G` or `Z`, selects database file (`gbbslsm` or `zbbslsm`).
- **Operation (`op`)**: Enum (`CREATE`, `UPDATE`, `INACTIVATE`, `REACTIVATE`, `PRINT`).
- **Output Flag (`flag`)**: Output parameter, indicates success (`1`, `A`, `I`) or failure (`E`).

## Process Steps
1. **Validate Inputs**:
   - Ensure `co` exists in `bicont`.
   - For `CREATE`, verify `co` and `smid` do not exist in `bbslsm`.
   - For `UPDATE`, `INACTIVATE`, or `REACTIVATE`, verify `co` and `smid` exist in `bbslsm`.
   - For `CREATE` and `UPDATE`, ensure `smnm` and `emal` are non-blank.
   - For `INACTIVATE`, ensure the record is not already inactive (`smdel ≠ 'I'`).
   - For `REACTIVATE`, ensure the record is inactive (`smdel = 'I'`).

2. **Select Database File**:
   - Apply file override to use `gbbslsm` (if `fgrp = 'G'`) or `zbbslsm` (if `fgrp = 'Z'`).

3. **Execute Operation**:
   - **CREATE**:
     - Write a new record to `bbslsm` with `co`, `smid`, `smnm`, `emal`, and `smdel = 'A'` (active).
     - Set `flag = '1'` on success.
   - **UPDATE**:
     - Update the existing record in `bbslsm` with new `smnm` and `emal`, preserving `co`, `smid`, and `smdel`.
     - Set `flag = '1'` on success.
   - **INACTIVATE**:
     - Update the record in `bbslsm`, setting `smdel = 'I'`.
     - Set `flag = 'I'` on success.
   - **REACTIVATE**:
     - Update the record in `bbslsm`, setting `smdel = 'A'`.
     - Set `flag = 'A'` on success.
   - **PRINT**:
     - Read all records from `bbslsm` sequentially.
     - Generate a report to `qsysprt` with:
       - Header: Company name, report title ("Salesman Id Listing By Co#/Salesman Id"), page number, job name, program name, user ID, file group, date (MM/DD/YYYY), time (HH:MM:SS), and column headers ("Co", "Slsm Id", "Salesman Name", "Email Address", "Del").
       - Detail: For each record, print `smco`, `smsmid`, `smsmnm`, `smemal`, `smdel`.
       - Format: 68 lines, 164 characters wide, 8 LPI, 15 CPI, overflow at line 62, held and saved in the job’s output queue.
     - Set `flag = '1'` on success.

4. **Handle Errors**:
   - If `co` is invalid, set `flag = 'E'` and return error message "Invalid company number."
   - If `smnm` or `emal` is blank for `CREATE` or `UPDATE`, set `flag = 'E'` and return error message "Salesman name and email required."
   - If record exists for `CREATE` or does not exist for `UPDATE`, `INACTIVATE`, or `REACTIVATE`, set `flag = 'E'` and return appropriate error message.
   - If status is invalid for `INACTIVATE` or `REACTIVATE`, set `flag = 'E'` and return error message "Invalid record status."

5. **Clean Up**:
   - Close all files and remove overrides.

## Business Rules
1. **Data Integrity**:
   - Company number must exist in `bicont`.
   - Salesman ID must be unique for `CREATE` and exist for `UPDATE`, `INACTIVATE`, or `REACTIVATE`.
   - Salesman name and email address are mandatory for `CREATE` and `UPDATE`.
2. **Status Management**:
   - Records can be active (`A`), inactive (`I`), or deleted (`D`).
   - Only non-inactive records can be inactivated; only inactive records can be reactivated.
3. **Report Generation**:
   - The `PRINT` operation includes all records, regardless of status, sorted by company number and salesman ID.
   - The report is formatted with fixed positions and includes metadata (job, user, date, time).
4. **Database Selection**:
   - File group (`G` or `Z`) determines the database file (`gbbslsm` or `zbbslsm`).
5. **Error Handling**:
   - Invalid inputs or operations result in `flag = 'E'` with a descriptive error message.

## Calculations
- **Date and Time Formatting** (for `PRINT`):
  - Date: Converted to `MM/DD/YYYY` format using system timestamp (`ps#mdy`) and time conversion structure (`t#mdcy`).
  - Time: Converted to `HH:MM:SS` format using system timestamp (`ps#hms`) and time conversion structure (`t#hms`).
- **Page Overflow** (for `PRINT`):
  - Triggers header printing when the printer file reaches line 62, as defined in the printer override.

## Outputs
- **Database Updates**: Modified `bbslsm` records for `CREATE`, `UPDATE`, `INACTIVATE`, or `REACTIVATE`.
- **Report**: Printed listing to `qsysprt` for `PRINT`.
- **Flag**: Returns `1` (success for `CREATE`, `UPDATE`, `PRINT`), `A` (success for `REACTIVATE`), `I` (success for `INACTIVATE`), or `E` (error).

## Dependencies
- **Database Files**:
  - `bbslsm`: Salesman ID records (input for `PRINT`, update/add for others).
  - `bicont`: Company records (input for validation).
- **External Programs**:
  - `QCMDEXC`: Executes file overrides for `bbslsm` and `qsysprt`.
  - `QMHSNDPM`: Sends error messages to the program message queue.
  - `QMHRMVPM`: Removes messages from the program message queue.

