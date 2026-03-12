Below, I’ll first identify the **use cases** implemented by the RPG programs `AR190` and `AR191`, as called from the provided `.OCL` program. Then, I’ll provide a **Function Requirement Document** for a single, consolidated use case, assuming it processes inputs programmatically (without screen interaction) to add, update, or edit salesman change transactions in the Accounts Receivable (A/R) system.

---

### Use Cases Implemented

Based on the provided `.OCL` (AR190.ocl36.txt), `AR190.rpg36.txt`, and `AR191.rpg36.txt`, the programs implement the following use cases:

1. **Add/Update Salesman Change Transactions (AR190)**:
   - **Description**: Allows users to add new salesman change transactions or update existing ones in the `ARSLST` file. Users specify a company number, customer number, old and new salesman codes, and a delete code via interactive screens (`S1` and `S2`). The program validates inputs against `ARCONT` (company), `ARCUST` (customer), and `GSTABL` (salesman) files, and updates the `ARSLST` file accordingly.
   - **Scope**: Interactive data entry and validation, supporting up to 10 transactions per screen, with roll-up/roll-down navigation.

2. **Generate Salesman Change Edit Report (AR191)**:
   - **Description**: Produces a printed report listing non-deleted salesman change transactions from the `ARSLST` file, grouped by company number. The report includes customer names (from `ARCUST`) and salesman descriptions (from `GSTABL`), with validation to flag invalid entries. It is a batch process triggered by the `.OCL` file.
   - **Scope**: Batch reporting for verification of transactions, with no user interaction.

---

### Function Requirement Document

Assuming a single, consolidated use case that programmatically processes salesman change transactions (combining the functionality of `AR190` and `AR191` without screen interaction), the following document outlines the requirements for a function that adds, updates, or deletes transactions and generates a report.



# Function Requirement Document: Salesman Change Transaction Processor

## Purpose
The `SalesmanChangeTransactionProcessor` function processes salesman change transactions in the Accounts Receivable (A/R) system. It validates and applies additions, updates, or deletions to the `ARSLST` file and generates a report summarizing the transactions, without requiring interactive screen input.

## Inputs
- **Company Number** (`CO`, 2 digits, numeric): Identifies the company for the transaction.
- **Transaction List**: Array of up to 10 transaction records, each containing:
  - **Customer Number** (`CUS`, 6 digits, numeric): Identifies the customer.
  - **New Salesman Code** (`SLN`, 2 digits, numeric): The new salesman to assign.
  - **Delete Code** (`DL`, 1 character): `' '` (blank), `'A'` (add/update), or `'D'` (delete).
- **Reference Files**:
  - `ARCONT`: Contains valid company numbers and names.
  - `ARCUST`: Contains customer numbers and names.
  - `GSTABL`: Contains salesman codes and descriptions under the `SLSMAN` table.

## Outputs
- **Updated `ARSLST` File**: Records added, updated, or deleted based on input transactions.
- **Report File** (`ARPRINT`): A text file summarizing processed transactions, including:
  - Company number, customer number, customer name, old salesman code, new salesman code, and salesman description.
  - Header with date, time, and page number, grouped by company.
- **Error List**: Array of error messages for invalid inputs (e.g., invalid company, customer, or salesman).

## Process Steps
1. **Validate Company Number**:
   - Check if `CO` is non-zero and exists in `ARCONT`. If invalid, return error: "Company number cannot be blank" or "Company not in control file."
2. **Process Transactions**:
   - For each transaction in the input list (up to 10):
     - **Validate Customer**:
       - If `SLN ≠ 0` and `CUS = 0`, return error: "Customer cannot be zero when new salesman is keyed."
       - If `CUS ≠ 0`, validate `CUS` against `ARCUST` using key `CO + CUS`. If not found, return error: "Invalid customer number."
       - Retrieve customer name (`ARNAME`) and old salesman code (`ARSLMN`) from `ARCUST`.
     - **Validate Salesman**:
       - If `CUS ≠ 0`, validate `SLN` against `GSTABL` using key `'SLSMAN' + SLN`. If not found, return error: "Invalid new salesman code."
       - Retrieve salesman description (`TBDESC`) from `GSTABL`.
     - **Validate Delete Code**:
       - If `CUS ≠ 0`, ensure `DL` is `' '`, `'A'`, or `'D'`. If invalid, return error: "Delete code must be ' ', A, or D."
     - **Update `ARSLST`**:
       - Build key `CO + CUS` to check `ARSLST`.
       - If `CUS ≠ 0`:
         - If `DL ≠ 'D'` and record exists, update `ARSLST` with `DL`, old salesman (`ARSLMN` or existing `ASSLSO`), and `SLN`.
         - If `DL ≠ 'D'` and record does not exist, add new record to `ARSLST` with `DL`, old salesman (`ARSLMN`), and `SLN`.
         - If `DL = 'D'` and record does not exist, delete the record.
         - If record is marked deleted (`ASDEL = 'D'`), flag with warning: "This record previously deleted."
3. **Generate Report**:
   - For each non-deleted `ARSLST` record (`ASDEL ≠ 'D'`), grouped by `ASCO`:
     - Retrieve customer name (`ARNAME`) from `ARCUST` (or `'INVALID'` if not found).
     - Retrieve salesman description (`TBDESC`) from `GSTABL` (or `'INVALID'` if not found).
     - Write to `ARPRINT`:
       - **Header** (on company change or page overflow): Page number, system date (MM/DD/YY), time (HH.MM.SS), title ("SALESMAN CHANGE EDIT"), and column headers ("CO", "CUSTOMER #", "NAME", "SALESMAN OLD NEW").
       - **Detail**: Company number, customer number, customer name, old salesman code, new salesman code, salesman description.
4. **Return Results**:
   - Return updated `ARSLST` file, generated `ARPRINT` report, and any error messages.

## Business Rules
1. **Company Validation**:
   - Company number must be non-zero and exist in `ARCONT`.
2. **Customer and Salesman Requirements**:
   - A customer number is required if a new salesman is specified.
   - Customer numbers must exist in `ARCUST`.
   - New salesman codes must exist in `GSTABL` under the `SLSMAN` table.
3. **Delete Code Handling**:
   - Delete code must be `' '`, `'A'`, or `'D'`.
   - `'D'` marks a record for deletion; other values indicate add/update.
4. **Transaction Processing**:
   - Update existing records or add new ones in `ARSLST` based on input.
   - Deleted records are excluded from the report.
5. **Report Formatting**:
   - Group by company number, with headers on each group or page overflow.
   - Flag invalid customers or salesmen with `'INVALID'` in the report.
   - Include system date and time in the header.

## Calculations
- **Key Construction**:
  - Customer key: Concatenate `CO` (2 digits) and `CUS` (6 digits) for `ARSLST` and `ARCUST` lookups.
  - Salesman key: Concatenate `'SLSMAN'` (literal) and `SLN` (2 digits) for `GSTABL` lookups.
- **Date and Time Formatting**:
  - System date formatted as MM/DD/YY.
  - System time formatted as HH.MM.SS.
- **Page Numbering**:
  - Increment page number (`PAGE`) for each company change or overflow in the report.

## Error Handling
- Return specific error messages for:
  - Blank or invalid company number.
  - Missing customer number when salesman is keyed.
  - Invalid customer number or salesman code.
  - Invalid delete code.
  - Previously deleted records.



---

### Explanation of Approach

- **Use Cases**: The two identified use cases reflect the distinct functionalities of `AR190` (interactive transaction entry/update) and `AR191` (batch report generation). These are derived from the `.OCL` file’s calls to `LOAD AR190` and `LOAD AR191` and their respective RPG code.
- **Consolidated Function**: The `SalesmanChangeTransactionProcessor` combines `AR190`’s transaction processing and `AR191`’s reporting into a single programmatic function, assuming inputs are provided (e.g., via API or file) rather than through interactive screens. This eliminates the need for `SCREEN` file interactions in `AR190`.
- **Conciseness**: The document focuses on business requirements (validation, data processing, report formatting) and necessary calculations (key construction, date/time formatting, page numbering), avoiding technical implementation details like RPG-specific operations.
- **File Usage**: The function uses the same files as the original programs (`ARSLST`, `ARCONT`, `ARCUST`, `GSTABL`, `ARPRINT`), maintaining consistency with the legacy system.

If you need further refinements or additional details (e.g., sample input formats or specific error codes), let me know!