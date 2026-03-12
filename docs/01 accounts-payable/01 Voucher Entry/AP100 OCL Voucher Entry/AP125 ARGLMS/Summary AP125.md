### **Freight Voucher Usecase AP125 and AP1012 Programs**

The `AP125.rpg` and `AP1012.rpg` programs together implement a single primary use case, as they form a cohesive process for creating accounts payable (A/P) voucher transactions from freight invoices. The use case is described below:

1. **Create A/P Voucher Transactions from Freight Invoices**:
   - **Description**: This use case processes freight invoices from the `FRCINH` or `FRCFBH` files to generate A/P voucher transactions in the `APTRAN` file, including header and detail records. It calculates due dates, applies discounts, prorates freight amounts across sales detail or miscellaneous records, and validates general ledger (G/L) accounts and vendor information. The process ensures accurate allocation of freight charges and compliance with payment terms and hold statuses.
   - **Inputs**: Company code (`CO`), carrier ID (`CAID`), invoice number (`CAIN`), record type (`RCD`), reference number (`RDNO`), and sales-related data (order number, shipping reference number, freight total, etc.).
   - **Outputs**: A/P transaction header and detail records in `APTRAN`, updated freight invoice records in `FRCINH` or `FRCFBH`, marked as processed.
   - **Components**:
     - `AP125.rpg`: Handles header creation, vendor validation, due date calculation, and G/L validation, and calls `AP1012.rpg` for detail line creation.
     - `AP1012.rpg`: Prorates freight amounts across sales detail (`SA5FIUD`, `SA5MOUD`) or miscellaneous (`SA5FIUM`, `SA5MOUM`) records and creates detail lines in `APTRAN`.

---

### **Function Requirement Document**



# Function Requirement: Create A/P Voucher Transactions from Freight Invoices

## **Purpose**
Automate the creation of accounts payable (A/P) voucher transactions from freight invoices, including header and detail records, by processing invoice data, validating vendor and G/L information, calculating due dates, applying discounts, and prorating freight amounts across sales detail or miscellaneous records.

## **Inputs**
- **Company Code (`CO`)**: 2-digit code identifying the company.
- **Carrier ID (`CAID`)**: 6-character identifier for the carrier.
- **Invoice Number (`CAIN`)**: 25-character freight invoice number.
- **Record Type (`RCD`)**: Indicates invoice source (`FRCFBHP4` for freight billed balance header, else `FRCINHP4`).
- **Reference Number (`RDNO`)**: Numeric reference number for `FRCFBH` invoices.
- **Sales Data** (via `SALES` data structure):
  - Order Number (`SAORD`): 6-digit sales order number.
  - Shipping Reference Number (`SASRN#`): 3-digit shipping reference number.
  - Freight Total (`FRTTOT`): Total freight amount to allocate (7.2 digits).
  - Vendor Number (`VEND`): 5-digit vendor number.
  - Entry Number (`ENTNUM`): 5-digit A/P entry number.
  - Discount Percentage (`DSPC`): 4.3-digit discount percentage.
  - Comparison Date (`CMPDT8`): Invoice date minus one year (8 digits).
  - File Indicator (`S@FIMO`): `F` (SA5FI files) or `M` (SA5MO files).
  - Detail/Misc Indicator (`S@DM`): `D` (detail) or `M` (miscellaneous).

## **Outputs**
- **A/P Transaction Records**: Header and detail records in `APTRAN` with validated vendor, G/L, and prorated freight amounts.
- **Updated Freight Invoice Records**: `FRCINH` or `FRCFBH` records marked as processed (`FRAPST = 'Y'`).

## **Process Steps**
1. **Retrieve Invoice Data**:
   - Access `FRCINH` (if `RCD ≠ 'FRCFBHP4'`) or `FRCFBH` (if `RCD = 'FRCFBHP4'`) using `CO`, `CAID`, `CAIN`, and `RDNO`.
   - Extract invoice number (`FRCAIN`), type (`FRINTY`), date (`FRIYMD`), amount (`FRINAM - FRFBOA`), sales order (`FRRDNO`), and shipping reference (`FRSRN`).

2. **Validate Company and Generate Entry Number**:
   - Retrieve company details (`ACAPGL`, `ACCAGL`, `ACRTGL`, `ACNXTE`) from `APCONT` using `CO`.
   - Assign a new entry number (`ENT#`) from `ACNXTE` for new transactions, incrementing and updating `ACNXTE`.

3. **Retrieve Vendor Information**:
   - Get vendor number (`VYVEND`) from `APVENY` using `CO` and `CAID`.
   - Retrieve vendor details (`VNVNAM`, `VNAD1-4`, `VNHOLD`, `VNSNGL`, `VNTERM`, `VNEXGL`) from `APVEND` using `CO` and `VEND`.
   - Set hold description (`HLDD`) based on hold code (`VNHOLD`):
     - `H`: "VENDOR ON HOLD"
     - `A`: "ON HOLD FOR ACH"
     - `W`: "ON HOLD FOR WIRE TRANSFER"
     - `U`: "ON HOLD FOR UTILITY AUTO-PAYMENT"

4. **Calculate Due Date**:
   - If `VNTERM` is non-zero, retrieve terms (`TBNETD`, `TBPRXD`, `TBDISC`) from `GSTABL` (table `APTERM`).
   - Calculate due date:
     - For net days (`TBNETD`): Add `TBNETD` to invoice date (`FRIYMD`) using Julian date conversion.
     - For prox days (`TBPRXD`): Set to `TBPRXD` day of the next month.
     - Default to invoice date if no terms.
   - Adjust due date to a non-holiday/non-weekend date using `APDATE` (`ADNED8`).

5. **Validate G/L Accounts**:
   - Validate A/P (`APGL`), bank (`BKGL`), and retention (`RTGL`) accounts against `GLMAST`, retrieving descriptions.

6. **Create A/P Header**:
   - Write/update header in `APTRAN` with invoice, vendor, G/L, due date, hold status, and discount (`DSPC`) data.

7. **Prorate Freight and Create Detail Lines**:
   - Call `AP1012` with `SALES` data structure to prorate `FRTTOT` across sales records.
   - **For Detail Records (`SA5FIUD` or `SA5MOUD`)**:
     - Calculate total gallons (`TTLQTY`) for records with `S5SHD8 >= CMPDT8`.
     - Prorate freight: `AMT = (S5NGAL / TTLQTY) * FRTTOT`.
     - Adjust the last record to ensure the sum equals `FRTTOT`.
     - Retrieve freight G/L (`FEGL`):
       - If product code (`S5PROD`) has alpha characters, use `CUFEGL` from `GSCTUM`.
       - Else, use `TBFEG4` from `GSTABL` (table `CNTRPF`) with `S5PROD` appended.
       - Default to `BCFRGL` from `BICONT`.
     - Write detail line to `APTRAN` with `FEGL`, `AMT`, `DSPC`, and description "XXXXXXXXX XXXX XXX FRTCHG".
   - **For Miscellaneous Records (`SA5FIUM` or `SA5MOUM`)**:
     - If no detail records, calculate total miscellaneous freight (`TTLMFT`) for records with `SMMSTY = 'F'`, `SMGLNO ≠ 0`, and `SMSHD8 >= CMPDT8`.
     - Prorate freight: `FRTAMT = (SMMAMT * SMMQTY / TTLMFT) * FRTTOT`.
     - Adjust the last record to match `FRTTOT`.
     - Use `SMGLNO` as `FEGL`.
     - Write detail line to `APTRAN` with `FRTAMT`, `DSPC`, and description "MISC CHARGE".

8. **Update Freight Invoice**:
   - Mark `FRCINH` or `FRCFBH` as processed (`FRAPST = 'Y'`).

## **Business Rules**
- **Invoice Source**: Process `FRCINH` or `FRCFBH` based on `RCD` (`FRCFBHP4` for freight billed balance).
- **Freight Adjustment**: Subtract `FRFBOA` from `FRINAM` for invoice amount.
- **Hold Status**: Support `H` (hold), `A` (ACH), `W` (wire transfer), `U` (utility auto-payment) with corresponding descriptions.
- **Due Date**: Calculate based on `VNTERM` (net or prox days) or default to invoice date; adjust for non-holiday/weekend.
- **Discounts**: Apply `TBDISC` from `GSTABL` to detail lines.
- **Date Restriction**: Process sales records with ship date within one year of invoice date (`S5SHD8` or `SMSHD8 >= CMPDT8`).
- **Freight Proration**:
  - Detail records: Prorate based on net gallons (`S5NGAL`).
  - Miscellaneous records: Prorate based on `SMMAMT * SMMQTY` for freight-type records (`SMMSTY = 'F'`).
  - Ensure total prorated amounts equal `FRTTOT`.
- **G/L Validation**: Validate all G/L accounts against `GLMAST`; use `GSCTUM`, `GSTABL`, or `BICONT` for freight G/L.
- **Record Selection**: Use `S@FIMO` (`F` or `M`) and `S@DM` (`D` or `M`) to select appropriate sales files; determine dynamically if blank.

## **Calculations**
- **Due Date (Net Days)**: `DUDT = INDT + TBNETD` (converted via Julian date).
- **Due Date (Prox Days)**: `DUDT = TBPRXD` day of the next month from `INDT`.
- **Freight Proration (Detail)**: `AMT = (S5NGAL / TTLQTY) * FRTTOT`; adjust last record to match `FRTTOT`.
- **Freight Proration (Misc)**: `FRTAMT = ((SMMAMT * SMMQTY) / TTLMFT) * FRTTOT`; adjust last record to match `FRTTOT`.

## **Dependencies**
- **Files**:
  - Update: `APTRAN`, `APCONT`, `FRCINH`, `FRCFBH`.
  - Input: `APVEND`, `APVENY`, `GLMAST`, `GSTABL`, `APDATE`, `SA5FIUD`, `SA5FIUM`, `SA5MOUD`, `SA5MOUM`, `BICONT`, `GSCTUM`.
- **External Program**: `AP1012` (called by `AP125`).

## **Assumptions**
- Input data is valid and exists in the respective files.
- `APTRAN` can accommodate new or updated records without conflicts.
- Sales records are available within the one-year date range.

## **Error Handling**
- Invalid company, vendor, or G/L accounts: Skip record creation and log errors (via indicators).
- No matching sales records: Skip detail line creation.
- Invalid dates: Reject and log errors.

