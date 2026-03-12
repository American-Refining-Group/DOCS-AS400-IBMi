Based on the provided **AR932.ocl36.txt** (OCL program) and **AR932.rpgle.txt** (RPGLE program), I’ve analyzed the call stack and program logic to identify the use cases implemented and to create a function requirement document for a non-interactive version of the primary use case. The OCL program sets up the environment and calls the RPGLE program `AR932`, which handles the core functionality of **Accounts Receivable Fuels Customer Maintenance**. Below, I’ll list the use cases and provide a concise function requirement document, assuming the process is driven by input parameters rather than screen interactions.

---

### Use Cases Implemented

The program implements a single primary use case, with sub-functions handled through different modes and actions. The use case is:

1. **Maintain Accounts Receivable Fuels Customer Records**:
   - **Description**: Allows maintenance of customer records in the A/R fuels system, including adding new fuel records, updating existing records, deleting records (marking as deleted), and reactivating previously deleted records.
   - **Sub-functions**:
     - **Add Record**: Create a new fuel record for a valid company and customer.
     - **Update Record**: Modify details of an existing fuel record.
     - **Delete Record**: Mark a fuel record as deleted (`AFDEL = 'D'`).
     - **Reactivate Record**: Remove the deletion flag from a previously deleted record.
     - **Browse Records**: Navigate through fuel records (forward/backward) to review existing data.
     - **Generate Report**: Produce a printed report of maintenance actions (add, delete, reactivate, or no action).
   - **Inputs**: Company number (`CO`), customer number (`CUST`), and action type (add, update, delete, reactivate).
   - **Outputs**: Updated `ARFUEL` file, error messages, and a printed report in the `PRINT` file.
   - **Context**: The program validates inputs against `ARCONT` (control file), `ARCUST` (customer master), and `GSCONT` (system control), ensuring data integrity and compliance with business rules.

No additional use cases are implemented, as the program focuses solely on customer maintenance for the A/R fuels system.

---

### Function Requirement Document



# A/R Fuels Customer Maintenance Function Requirements

## Overview
This function maintains customer records in the Accounts Receivable (A/R) Fuels system by processing actions (add, update, delete, reactivate) on fuel-related customer data. It validates inputs, updates the `ARFUEL` file, and generates a report of actions taken. The function accepts input parameters instead of interactive screen inputs, ensuring batch or API-driven execution.

## Inputs
- **Company Number (`CO`)**: Numeric (2 digits), identifies the company.
- **Customer Number (`CUST`)**: Numeric (6 digits), identifies the customer.
- **Action Type**: String, one of:
  - `ADD`: Create a new fuel record.
  - `UPDATE`: Modify an existing fuel record.
  - `DELETE`: Mark a fuel record as deleted.
  - `REACTIVATE`: Remove deletion flag from a deleted record.
- **Customer Name (`NAME`)**: String (30 characters), required for `ADD` and `UPDATE`.

## Outputs
- **Updated `ARFUEL` File**: Modified fuel record (added, updated, deleted, or reactivated).
- **Report Output (`PRINT`)**: Listing of actions taken, including company number, customer number, customer name, and action description.
- **Error Message**: String indicating success or failure (e.g., "RECORD NOT FOUND", "INVALID CUSTOMER NUMBER").

## Process Steps
1. **Validate Inputs**:
   - Check if `CO` exists in `ARCONT`. If not, return error: "RECORD NOT FOUND".
   - Check if `CUST` exists in `ARCUST`. If not, return error: "INVALID CUSTOMER NUMBER ENTERED".
   - For `ADD`, ensure `CO` is not zero. If zero, return error: "COMPANY CAN'T BE ZERO".
2. **Check Record Existence**:
   - For `ADD`, verify no existing record in `ARFUEL` for `CO` and `CUST`. If found, return error: "CANNOT ADD - THIS RECORD EXISTS".
   - For `UPDATE`, `DELETE`, or `REACTIVATE`, verify record exists in `ARFUEL`. If not, return error: "RECORD NOT FOUND".
3. **Perform Action**:
   - **ADD**: Create new `ARFUEL` record with `CO`, `CUST`, `NAME`, and `AFDEL = ' '` (not deleted).
   - **UPDATE**: Update existing `ARFUEL` record with new `NAME`.
   - **DELETE**: Set `AFDEL = 'D'` in `ARFUEL` record. Return message: "PREVIOUS RECORD DELETED".
   - **REACTIVATE**: If `AFDEL = 'D'`, set `AFDEL = ' '`. Return message: "PREVIOUS RECORD WAS REACTIVATED".
4. **Generate Report**:
   - Write to `PRINT` file with:
     - Header: Company name (`ACNAME` from `ARCONT`), date, time, "FUELS CUSTOMER MAINTENANCE LISTING".
     - Detail: `CO`, `CUST`, `NAME`, and action ("RECORD ADDED", "RECORD DELETED", "RECORD REACTIVATED", or "NO ACTION TAKEN").
5. **Return Result**:
   - Return success message or error message based on validation and action outcome.

## Business Rules
- **Validation**:
  - Company number must exist in `ARCONT`.
  - Customer number must exist in `ARCUST`.
  - Company number cannot be zero for `ADD`.
- **Record Management**:
  - `ADD` fails if a record already exists in `ARFUEL` for the same `CO` and `CUST`.
  - `UPDATE` and `DELETE` require an existing `ARFUEL` record.
  - `REACTIVATE` only applies to records with `AFDEL = 'D'`.
- **Deletion**:
  - Deletion sets `AFDEL = 'D'` (soft delete) rather than removing the record.
  - Reactivation clears `AFDEL` to restore the record.
- **Reporting**:
  - All actions are logged in the `PRINT` file with timestamps and action details.
- **System Control**:
  - Validates against `GSCONT` for system-wide settings (e.g., company number consistency).

## Calculations
- **Date and Time**: Retrieved from system clock and formatted as `MMDDYY` (6 digits) for `DATE` and `HHMMSS` (6 digits) for `TIME` in the report.
- **Key Construction**: Combines `CO` (2 digits) and `CUST` (6 digits) into an 8-digit key (`MASKEY`) for `ARFUEL` and `ARCUST` lookups.
- **Page Number**: Increments for each report page, included in `PRINT` output.

## Dependencies
- **Files**:
  - `ARCONT`: Control data (company details).
  - `ARFUEL`: Fuel A/R data (updateable).
  - `ARCUST`: Customer master data.
  - `GSCONT`: System control data.
  - `PRINT`: Output file for reports.
- **External Programs**: None directly called by the function (OCL calls `GSGENIEC` for setup, but not required in non-interactive mode).

## Error Handling
- Returns specific error messages for validation failures (e.g., "RECORD NOT FOUND", "CANNOT ADD - THIS RECORD EXISTS").
- Ensures no action is taken if validations fail, preserving data integrity.



---

### Notes
- The function requirement document assumes a non-interactive process, replacing screen inputs (`AR932S1`, `AR932S2`) with direct parameter inputs (`CO`, `CUST`, `ACTION`, `NAME`).
- The business rules and process steps are derived from the RPGLE program’s logic, focusing on core functionality and validation.
- No calculations beyond simple key construction and date/time formatting are required, as the program primarily manages record states and reporting.
- The document is concise, focusing on business requirements while including necessary technical details (e.g., file dependencies, error messages).

If you need further refinements, additional use cases, or specific details (e.g., sample input/output formats), please let me know!