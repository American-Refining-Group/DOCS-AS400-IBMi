Based on the provided RPGLE program `AR810U.rpgle`, I will first identify the use cases implemented by the program. Then, I will create a concise function requirement document for a large function that encapsulates the primary use case, assuming it processes inputs programmatically rather than through screen interaction. The function will focus on business requirements, process steps, and necessary calculations.

---

### List of Use Cases Implemented by AR810U

The `AR810U.rpgle` program implements a single primary use case:

1. **Accounts Receivable Invoice Inquiry**:
   - **Description**: Retrieves and displays detailed Accounts Receivable (AR) transaction information (invoices, payments, adjustments) for a specified company and customer. The program filters transactions based on open/closed status, excludes deleted or inactive records, and provides aging bucket information for open invoices.
   - **Purpose**: Allows users to inquire about a customer’s AR transactions, including invoice amounts, payment status, terms, and aging details, to support financial analysis or customer account management.
   - **Key Features**:
     - Filters transactions to show only open invoices by default (`cOPEN = 'O'`).
     - Retrieves customer and company details for context.
     - Calculates open invoice amounts and aging buckets.
     - Displays terms descriptions and days between transaction and due dates for adjustments/payments.

No additional use cases are implemented, as the program focuses solely on this inquiry functionality within the Genie environment.

---

### Function Requirement Document



# AR Invoice Inquiry Function Requirements

## Function Name
`GetARInvoiceDetails`

## Purpose
Retrieve detailed Accounts Receivable (AR) transaction information for a specified company and customer, including invoices, payments, and adjustments, with calculated open amounts, aging buckets, and terms descriptions. The function supports financial analysis and customer account management by providing filtered transaction data programmatically.

## Inputs
- `companyCode` (Numeric, 2 digits): Company identifier (e.g., `cco`).
- `customerCode` (Numeric, 6 digits): Customer identifier (e.g., `ccust`).
- `openOnly` (Character, 1): Flag to filter open transactions ('O') or include all ('A').
- `currentDate` (Date, CCYYMMDD): System date for aging calculations.

## Outputs
- `transactionList` (Array of Transaction Records):
  - `transactionType` (String): 'INVOICE', 'PAYMENT', or 'ADJUST'.
  - `typeCode` (Character, 1): '1' (Invoice), '0' (Payment/Adjustment).
  - `amount` (Numeric, 15,2): Transaction amount (open amount for invoices if `openOnly = 'O'`).
  - `transactionDate` (Date, MM/DD/YYYY): Transaction date.
  - `dueDate` (Date, MM/DD/YYYY): Due date.
  - `termsCode` (Numeric, 2): Invoice terms code (if applicable).
  - `termsDescription` (String, 30): Terms description (e.g., '02-NET 30').
  - `agingBucket` (String, 10): Aging description (e.g., 'CURRENT', '1-30').
  - `daysDifference` (Numeric, 10,2): Days between transaction and due date (for payments/adjustments).
- `customerDetails`:
  - `name` (String, 40): Customer name.
  - `address` (Array of String): Address lines (up to 4).
  - `zipCode` (String, 5): ZIP code.
- `agingBuckets` (Array of String): Descriptions for aging buckets (e.g., ['CURRENT', '1-30', '31-60', ...]).
- `moreRecords` (String): Indicator if more records exist (e.g., 'More history...').
- `errorMessage` (String): Error description if processing fails (e.g., 'Customer not found').

## Process Steps
1. **Validate Inputs**:
   - Ensure `companyCode` and `customerCode` are valid (non-zero, correct format).
   - Default `openOnly` to 'O' if invalid.
   - Validate `currentDate` as a valid CCYYMMDD date.
2. **Retrieve Company Control Data**:
   - Read AR control record (`ARCONT`) for `companyCode`.
   - Extract aging bucket limits (`aclmt1`, `aclmt2`, `aclmt3`, `aclmt4`).
   - Define aging bucket descriptions:
     - Bucket 1: 'CURRENT'
     - Bucket 2: '1 - `aclmt1`'
     - Bucket 3: '`aclmt1 + 1` - `aclmt2`'
     - Bucket 4: '`aclmt2 + 1` - `aclmt3`'
     - Bucket 5: '`aclmt3 + 1` - `aclmt4`'
   - If not found, set limits to 0.
3. **Retrieve Customer Data**:
   - Read customer master record (`ARCUST`) using `companyCode` and `customerCode`.
   - Extract name, address lines, and ZIP code.
   - If not found, set `name` to 'Customer Master not found' and clear other fields.
4. **Retrieve AR Transactions**:
   - Read AR detail records (`ARDETL`) for `companyCode` and `customerCode`.
   - Filter records:
     - Exclude deleted (`adDEL = 'D'`) or inactive (`adDEL = 'I'`) records.
     - If `openOnly = 'O'`:
       - Exclude payments (`ADTYPE = 'P'`) or adjustments (`ADTYPE = 'J'`).
       - Exclude fully paid invoices (`ADTYPE = 'I'` where `ADPART + ADPAY = ADAMT`).
   - Limit output to 9990 records; set `moreRecords` if exceeded.
5. **Process Each Transaction**:
   - Convert transaction date (`adtym8`) and due date (`addud8`) from CCYYMMDD to MM/DD/YYYY.
   - For each valid record:
     - **Invoice (`ADTYPE = 'I'`)**:
       - Set `transactionType = 'INVOICE'`, `typeCode = '1'`.
       - If `openOnly = 'O'`, calculate `amount = ADAMT - ADPART - ADPAY`; else `amount = ADAMT`.
       - Set `termsCode = ADTERM`.
       - Retrieve terms description from `GSTABL` using key 'ARTERM' + `ADTERM`. If not found, use `ADTERM` as description.
       - Set `transactionDate` to converted `adtym8`.
     - **Payment (`ADTYPE = 'P'`)**:
       - Set `transactionType = 'PAYMENT'`, `typeCode = '0'`.
       - Set `amount = ADAMT`.
       - Calculate `daysDifference` using `GSDTCLC2` (difference between `adtym8` and `addud8` in days).
     - **Adjustment (`ADTYPE = 'J'`)**:
       - Set `transactionType = 'ADJUST'`, `typeCode = '0'`.
       - Set `amount = ADAMT`.
       - Calculate `daysDifference` using `GSDTCLC2`.
     - Assign `agingBucket` based on `ADAGE` (1-5, mapping to bucket descriptions).
6. **Return Results**:
   - Package `transactionList`, `customerDetails`, `agingBuckets`, `moreRecords`, and `errorMessage`.
   - If no errors, set `errorMessage` to empty; otherwise, describe the issue (e.g., 'Invalid company code').

## Business Rules
- **Transaction Filtering**:
  - Deleted or inactive records are always excluded.
  - Open-only mode (`openOnly = 'O'`) excludes payments, adjustments, and fully paid invoices.
- **Amount Calculation**:
  - For open invoices, amount is reduced by partial payments (`ADPART`) and payments applied (`ADPAY`).
- **Aging Buckets**:
  - Defined by AR control limits, with 'CURRENT' as the first bucket.
  - Transactions are categorized based on `ADAGE` (1-5).
- **Terms Description**:
  - Must include terms code and description (e.g., '02-NET 30') from `GSTABL` or default to terms code.
- **Date Conversion**:
  - All dates are converted to MM/DD/YYYY for consistency.
- **Record Limit**:
  - Maximum 9990 transactions; excess triggers `moreRecords` warning.
- **Error Handling**:
  - Invalid inputs or missing master data (customer, company) return descriptive errors.
  - Date conversion errors set dates to default low-value.

## External Dependencies
- **Programs**:
  - `GSDTCLC2`: Calculates days between two dates (CCYYMMDD format).
- **Files**:
  - `ARCONT`: AR control data for aging buckets.
  - `ARCUST`: Customer master data.
  - `ARDETL`: AR transaction details.
  - `GSTABL`: Terms descriptions.

## Assumptions
- Input dates are in CCYYMMDD format.
- `GSDTCLC2` is available and reliable for date calculations.
- All files (`ARCONT`, `ARCUST`, `ARDETL`, `GSTABL`) are accessible and correctly keyed.
- The function runs in a non-interactive environment, replacing screen-based input/output.

