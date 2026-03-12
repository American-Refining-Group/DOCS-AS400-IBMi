### List of Use Cases Implemented by the Program

The call stack consists of three components:
1. **AR415.ocl36.txt**: An OCL program that orchestrates the process by invoking `AR415P` and `AR415` programs and sorting data with `#GSORT`.
2. **AR415P.rpgle.txt**: An RPGLE program that handles interactive company code input and validation.
3. **AR415.rpg36.txt**: An RPG III program that generates the Customer Credit Report.

**Use Case Identified**:
- **Generate Customer Credit Report**: The primary use case is to produce a report listing customers' credit status, including those exceeding their credit limits, for a specified company. The report details credit limits, total amounts due, open orders, unposted amounts, and available credit, with a secondary report listing specific orders that exceed credit limits.

Only one use case is implemented, as the programs collectively focus on generating this report, with `AR415P` validating the company code and `AR415` processing and printing the report data.

---

### Function Requirement Document



# Customer Credit Report Function Requirements

## Overview
The `generateCustomerCreditReport` function produces a Customer Credit Report for a specified company, listing customers' credit details and identifying those over their credit limits. A secondary report lists specific orders exceeding credit limits. The function takes input parameters and processes data from predefined files without interactive screen input.

## Inputs
- **Company Code** (`co`, 2-digit numeric): Identifies the company for which the report is generated.
- **Cancel Flag** (`kycanc`, 6-character): Indicates if the process should be canceled ("CANCEL").
- **Input Files**:
  - `BICONT`: Company control file (company name, deletion status).
  - `ARCUST`: Sorted customer file (customer number, name, total due, credit limit).
  - `ARCUSA`: Alternate customer file (customer number, total due, credit limit).
  - `ARCLGR`: Ledger file (customer numbers array).
  - `BBORCL`: Order file (order details, amounts, over-limit status).

## Outputs
- **ARLIST Report**: Primary report listing all customers with:
  - Customer number, name, credit limit, total due, open orders total, unposted amount, available credit.
  - Indicates customers over their credit limit.
- **ARLIS2 Report**: Secondary report listing orders exceeding credit limits with:
  - Customer number, name, credit limit, total due, order number, unposted amount, batch number.
- **Return Status**: Success, invalid company, deleted company, or canceled.

## Process Steps
1. **Validate Company Code**:
   - Check if `co` is non-zero and exists in `BICONT`.
   - Verify `co` is not marked as deleted (`bcdel <> 'D'`).
   - If invalid or deleted, return error status.
2. **Sort Customer Data**:
   - Use sorted `ARCUST` file (pre-sorted by `#GSORT` on positions 2–9, filtering non-deleted customers and specific conditions).
3. **Process Company Data**:
   - Retrieve company name (`BCNAME`) from `BICONT` for report headers.
   - Capture system date and time for report headers.
4. **Process Customer Data**:
   - For each customer in `ARCUST` (control break on customer number):
     - Initialize `LIMIT`, `OWED`, `ORDVAL`, `TRXVAL`, `OVER` to zero.
     - Retrieve additional customer numbers from `ARCLGR` (up to 25, skipping deleted entries).
     - For each customer (from `ARCLGR` and `ARCUST`):
       - Chain to `ARCUSA` to get `AXCLMT` (credit limit) and `AXTOTD` (total due).
       - Add `AXCLMT` to `LIMIT` if non-zero; add `AXTOTD` to `OWED`.
       - Process `BBORCL` for matching orders (skip deleted, `BLDEL = 'D'`):
         - Add `BLTAMT` (unposted amount) to `TRXVAL` and `OWED`.
         - Add `BLOAMT` (order amount) to `ORDVAL` and `OWED`.
         - If `BLOVCL = 'Y'`, output order details to `ARLIS2`.
     - Set `LIMIT = ARCLMT` from `ARCUST` if `LIMIT = 0`.
     - Add `ARTOTD` to `OWED`.
   - Calculate `OVER = LIMIT - OWED`.
5. **Generate Reports**:
   - **ARLIST**: Print for each customer:
     - Customer number (`ARCUST`), name (`ARNAME`), credit limit (`ARCLMT`), total due (`ARTOTD`), open orders (`ORDVAL`), unposted amount (`TRXVAL`), available credit (`OVER`).
     - Mark "OVER CREDIT LIMIT" if `OVER < 0`.
   - **ARLIS2**: Print for orders with `BLOVCL = 'Y'`:
     - Customer number, name, credit limit, total due, order number (`BLORDR`), unposted amount (`BLTAMT`), batch number (`BLBTCH`).
   - Include headers with company name, date, time, and page numbers.

## Business Rules
1. **Company Validation**:
   - Company code must be non-zero, exist in `BICONT`, and not be deleted (`bcdel <> 'D'`).
   - Cancel process if `kycanc = 'CANCEL'`.
2. **Customer Filtering**:
   - Skip deleted customers (`CGDEL = 'D'` in `ARCLGR`) and orders (`BLDEL = 'D'` in `BBORCL`).
   - Process only non-zero amounts (`AXCLMT`, `AXTOTD`, `BLTAMT`, `BLOAMT`).
3. **Credit Calculations**:
   - `LIMIT = AXCLMT` (from `ARCUSA`) or `ARCLMT` (from `ARCUST` if `AXCLMT = 0`).
   - `OWED = ARTOTD + AXTOTD + BLTAMT + BLOAMT`.
   - `TRXVAL = sum(BLTAMT)` (unposted amounts).
   - `ORDVAL = sum(BLOAMT)` (open order amounts).
   - `OVER = LIMIT - OWED`; negative `OVER` indicates over credit limit.
4. **Report Output**:
   - `ARLIST`: Lists all customers with credit details, marking over-limit cases.
   - `ARLIS2`: Lists orders exceeding credit limits (`BLOVCL = 'Y'`).
   - Reports include company name, date, time, and page numbers.

## Calculations
- **Credit Limit (`LIMIT`)**: Sum of `AXCLMT` from `ARCUSA` for valid customers; defaults to `ARCLMT` from `ARCUST` if zero.
- **Total Owed (`OWED`)**: Sum of `ARTOTD` (from `ARCUST`), `AXTOTD` (from `ARCUSA`), `BLTAMT` (unposted from `BBORCL`), and `BLOAMT` (orders from `BBORCL`).
- **Unposted Amount (`TRXVAL`)**: Sum of `BLTAMT` from `BBORCL`.
- **Open Orders (`ORDVAL`)**: Sum of `BLOAMT` from `BBORCL`.
- **Available Credit (`OVER`)**: `LIMIT - OWED`; negative values trigger "OVER CREDIT LIMIT" in `ARLIST`.

## Error Handling
- Return "Invalid Company" if `co = 0` or not found in `BICONT`.
- Return "Company Deleted" if `bcdel = 'D'` in `BICONT`.
- Return "Canceled" if `kycanc = 'CANCEL'`.

