### List of Use Cases Implemented by AR830U RPGLE Program

The `AR830U.rpgle` program implements a single primary use case:

1. **Accounts Receivable Invoice History Inquiry**:
   - **Description**: Allows users to view accounts receivable (AR) invoice history for a specified company and customer, filtered by an invoice date (defaulting to the last 2 years). The program retrieves and displays invoice, adjustment, and payment records, including details like invoice type, terms, and date differences between transaction and due dates, using a subfile interface in a Genie environment.
   - **Inputs**: Company number (`cco`), customer number (`ccust`), and invoice date filter (`cinvdat`).
   - **Outputs**: A subfile displaying AR history records with fields such as invoice number, type, dates, terms description, and days between transaction and due dates for adjustments/payments.
   - **Constraints**: Only runs in Genie environment (`genievar = 'YES'`), skips deleted records (`ahDEL = 'D' or 'I'`), and limits records to those with invoice dates within the specified range.

---

### Function Requirement Document for AR Invoice History Inquiry



# AR Invoice History Inquiry Function Requirements

## Purpose
Provide a function to retrieve and return accounts receivable (AR) invoice history records for a specified company and customer, filtered by a date range, to support financial analysis and customer account management.

## Inputs
- **Company Number** (`cco`): String (company identifier).
- **Customer Number** (`ccust`): String (customer identifier).
- **Invoice Date Filter** (`cinvdat`): Date (MM/DD/YYYY format, defaults to today minus 2 years).
- **Environment Flag** (`genievar`): String (`'YES'` to confirm Genie environment, otherwise function exits).

## Outputs
- A list of AR history records containing:
  - Invoice number (`ahinvc`).
  - Invoice type description (`sftype`): `'INVOICE'`, `'ADJUST'`, or `'PAYMENT'`.
  - Terms description (`sftmdesc`): For invoices, terms code plus description (e.g., `'01-Net 30'`); for others, blank.
  - Invoice date (`invdat`): MM/DD/YYYY format.
  - Due date (`dudat`): MM/DD/YYYY format.
  - Transaction date (`trdat`): MM/DD/YYYY format.
  - Days difference (`sfdays`): Days between transaction and due dates for adjustments/payments; zero for invoices.
  - Customer details: Name (`arname`), address (`aradr1`, `aradr2`, `aradr3`, `aradr4`), zip code (`arzip5`).
- Status message (`MORE`): Indicates if record limit (9990) was reached (`'More history...not all included...'`).

## Process Steps
1. **Validate Environment**: Check if `genievar = 'YES'`. If not, exit with no output.
2. **Retrieve Default Company**: Query `GSCONT` with key `gskey = 0` to get default company number (`gxcono`). Set `cco = gxcono` if valid; otherwise, use provided `cco`.
3. **Retrieve Customer Details**: Query `ARCUST` with keys `cco` and `ccust`. If not found, set `arname = 'Customer Master not found'` and clear address fields.
4. **Convert Date Filter**: Convert `cinvdat` (MM/DD/YYYY) to `cinvcymd` (YYYYMMDD) for comparison.
5. **Retrieve AR History**:
   - Query `ARHIST` with keys `cco` and `ccust`.
   - Filter records:
     - Exclude records where `ahDEL = 'D'` or `'I'` (deleted/inactive).
     - Exclude records where invoice date (`ahind8`, YYYYMMDD) is earlier than `cinvcymd`.
6. **Process Each Record**:
   - **Invoice Type Handling**:
     - If `ahtype = 'I'` (Invoice):
       - Set `sftype = 'INVOICE'`, `sftypeI = '1'`, `sftermI = ahterm`.
       - Query `GSTABL` with keys `'ARTERM'` and `ktbcode = '    ' + sftermI` to get terms description (`TBDESC`). Set `sftmdesc = ktbcode + '-' + TBDESC` or `ktbcode` if not found.
       - Set `sfdays = 0`.
     - If `ahtype = 'J'` (Adjustment) or `'P'` (Payment):
       - Set `sftype = 'ADJUST'` or `'PAYMENT'`.
       - Calculate days difference (`sfdays`) between transaction date (`ahtda8`) and due date (`ahdud8`) using `GSDTCLC2`.
   - **Date Conversion**:
     - Convert `ahind8`, `ahdud8`, `ahtda8` (YYYYMMDD) to MM/DD/YYYY for `invdat`, `dudat`, `trdat`.
     - Handle invalid dates by setting to a default low value.
7. **Record Limit Check**: Stop processing if record count exceeds 9990 and set `MORE` message.
8. **Return Results**: Output the list of formatted records and status message.

## Business Rules
- **Environment Restriction**: Function only processes if running in Genie environment (`genievar = 'YES'`).
- **Date Filter**: Only include records with invoice dates on or after the specified date (default: today minus 2 years).
- **Record Exclusion**: Skip deleted (`ahDEL = 'D'`) or inactive (`ahDEL = 'I'`) records.
- **Record Limit**: Maximum of 9990 records returned to prevent overflow.
- **Customer Validation**: If customer not found in `ARCUST`, return default error message for name and clear address fields.
- **Terms Description**: For invoices, append terms code and description from `GSTABL`; for adjustments/payments, leave blank.
- **Date Difference Calculation**: For adjustments and payments, compute days between transaction and due dates; for invoices, set to zero.

## Calculations
- **Date Difference** (`sfdays`):
  - For `ahtype = 'J'` or `'P'`, call `GSDTCLC2` with:
    - `p#dat1 = ahtda8` (transaction date, YYYYMMDD).
    - `p#dat2 = ahdud8` (due date, YYYYMMDD).
    - `p#fmt = 'D'` (days format).
    - Output: `p#diff` (days difference, Packed(10:2)).
    - Error handling: Set `p#err` if calculation fails.
  - Result stored in `sfdays`.
- **Date Conversion**:
  - Convert YYYYMMDD dates (`ahind8`, `ahdud8`, `ahtda8`) to MM/DD/YYYY using `%date` and `%char(*iso0)`.
  - Handle invalid dates by setting to a low value.

## External Dependencies
- **Programs**:
  - `GSGENIE2C`: Returns `genievar` to validate Genie environment.
  - `GSDTCLC2`: Calculates days difference between two dates.
- **Files**:
  - `GSCONT`: Provides default company number (`gxcono`).
  - `ARCUST`: Provides customer details (name, address, zip).
  - `ARHIST`: Stores AR invoice history records.
  - `GSTABL`: Stores terms descriptions (`TBDESC`).

## Constraints
- Maximum 9990 records to prevent subfile overflow.
- Invalid dates are handled gracefully with default low values.
- Function assumes valid company/customer inputs; invalid inputs may result in no records or default customer error message.

