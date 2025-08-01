The call stack involves two RPG programs, `AR9009.rpg36.txt` and `BI907.rpgle.txt`, called from the main OCL script `AR900.ocl36.txt` within the Bradford Order Entry/Invoices system. Together, these programs support customer master file maintenance, specifically focusing on validating customer deletion (`AR9009`) and managing customer and shipto product data (`BI907`). Below, I identify the use cases implemented by these programs and provide a function requirement document for a large function that performs customer and shipto product maintenance without screen interaction, incorporating the process steps and business rules from both programs.

---

### **Use Cases Implemented**

The programs collectively implement the following use cases:

1. **Customer Deletion Validation (`AR9009`)**:
   - Validates whether a customer can be deleted from the customer master file by checking for existing activity (invoices or sales) in related files.
   - Ensures data integrity by preventing deletion of customers with active records.

2. **Customer and Shipto Product Maintenance (`BI907`)**:
   - Manages product-specific data for customers and shiptos, including alternate descriptions, freight codes, gallons billed codes, and container types.
   - Supports adding, updating, deleting (marking as inactive), reactivating, and copying alternate descriptions, with history tracking.
   - Provides inquiry mode to view data without modifications.

Given that `AR9009` is a validation step within the broader customer maintenance process, and `BI907` handles detailed product data maintenance, these use cases are interconnected. For the function requirement document, I will focus on a consolidated function that performs **Customer and Shipto Product Maintenance**, incorporating the deletion validation logic from `AR9009` as a prerequisite check, assuming the function processes inputs programmatically rather than through a screen.

---

### **Function Requirement Document**



# Customer and Shipto Product Maintenance Function Requirements

## **Overview**
The **Customer and Shipto Product Maintenance** function programmatically manages product-specific data for customers and shiptos in the Bradford Order Entry/Invoices system, including validation to prevent deletion of customers with active records. It supports adding, updating, deleting (marking as inactive), reactivating, and copying alternate product descriptions, with history tracking.

## **Inputs**
- **Company Number** (`co`, 2 bytes, numeric): Identifies the company.
- **Customer Number** (`cust`, 6 bytes, numeric): Identifies the customer.
- **Shipto Number** (`ship`, 3 bytes, numeric): Identifies the shipto location.
- **Mode** (`mode`, 3 bytes, string): `MNT` (maintenance) or `INQ` (inquiry).
- **File Group** (`fgrp`, 1 byte, string): `Z` or `G` for file overrides.
- **Billing Instruction Code** (`bcinst`, 1 byte, string): Determines if sales files are checked (`'5'` for sales check).
- **Operation** (`operation`, string): `ADD`, `UPDATE`, `DELETE`, `REACTIVATE`, `COPY`, or `INQUIRE`.
- **Product Data List** (array of records):
  - `prod` (4 bytes, string): Product code.
  - `cnty` (1 byte, string): Container type code.
  - `cpds` (20 bytes, string): Alternate product description.
  - `glcd` (1 byte, string): Gallons billed code (`'G'` or blank).
  - `frcd` (1 byte, string): Freight code (`'C'`, `'P'`, `'A'`, or blank).
  - `sfrt` (1 byte, string): Separate freight code (`'Y'`, `'N'`, or blank).
  - `cafr` (1 byte, string): Calculate freight code (`'Y'`, `'N'`, or blank).
- **Copy Source** (for `COPY` operation):
  - `copy_cust` (6 bytes, numeric): Source customer number.
  - `copy_ship` (3 bytes, numeric): Source shipto number.

## **Outputs**
- **Result** (string): `SUCCESS`, `ERROR`, or `NO_ACTIVITY` (for deletion validation).
- **Found Flag** (`found`, 1 byte, string): `'A'` (invoices found), `'S'` (sales found), or blank (no activity, for deletion validation).
- **Error Messages** (array of strings): List of validation errors, if any.
- **Processed Records** (array of records): Returns updated or inquired records with fields as in input `Product Data List`, plus:
  - `exis` (1 byte, string): `'Y'` if record exists, `'N'` otherwise.
  - `del` (1 byte, string): `'D'` if deleted, `'A'` if active.

## **Process Steps**
1. **Validate Inputs**:
   - Verify `co` exists in `BICONT`.
   - Verify `cust` exists in `ARCUST`.
   - Verify `ship` exists in `SHIPTO` for the given `cust` and `co`.
   - If `operation = DELETE`, validate `bcinst` is `'5'` for sales checks.
   - For each record in `Product Data List`:
     - Ensure `prod` exists in `GSPROD` and is sellable (`tpsell = 'Y'`).
     - Ensure `cnty` exists in `GSTABL` (`CNTRTY` table) if not blank or `'A'`.
     - Ensure `glcd` is `'G'` or blank.
     - Ensure `frcd` exists in `GSTABL` (`BBFRCD` table) if not blank.
     - Ensure `sfrt` and `cafr` are `'Y'`, `'N'`, or blank.

2. **Deletion Validation** (if `operation = DELETE`):
   - Check `CRDETX` for active invoices (`ADDEL ≠ 'D'`, `ARDKEY = co + cust`).
     - If found, set `found = 'A'` and return `ERROR` with message "Customer has active invoices".
   - If `bcinst = '5'`, check sales files (`SA5FIXD`, `SA5FIXM`, `SA5BCXD`, `SA5BCXM`, `SA5DBXD`, `SA5DBXM`, `SA5COXD`, `SA5COXM`) for records matching `SACOCU = co + cust`.
     - If found, set `found = 'S'` and return `ERROR` with message "Customer has sales activity".
   - If no activity found, set `found = ''` and proceed.

3. **Process Operation**:
   - **INQUIRE** (`mode = 'INQ'`):
     - Retrieve records from `ARCUPR` for `co`, `cust`, `ship`, optionally filtered by `prod` and `cnty`.
     - Include deleted records if specified.
     - Return records with `exis = 'Y'` and `del` status.
   - **ADD** (`mode = 'MNT'`):
     - For each record in `Product Data List`, check if it exists in `ARCUPR` (`cpcono = co`, `cpcust = cust`, `cpship = ship`, `cpprod = prod`, `cpcnty = cnty`).
     - If exists or marked deleted, return error "Record already exists or is deleted".
     - Create new `ARCUPR` record with `cpdel = 'A'`, populate fields (`cpprod`, `cpcnty`, `cpcpds`, `cpglcd`, `cpfrcd`, `cpsfrt`, `cpcafr`), and write.
     - Log to `ARCUPHS` with current date, time, and user ID.
   - **UPDATE** (`mode = 'MNT'`):
     - For each record, verify existence in `ARCUPR`.
     - Update existing record with new values, retaining `cpdel = 'A'`.
     - Log to `ARCUPHS`.
   - **DELETE** (`mode = 'MNT'`):
     - Verify no activity via Step 2.
     - For each record, verify existence in `ARCUPR` and `cpdel ≠ 'D'`.
     - Set `cpdel = 'D'`, update `ARCUPR`, and log to `ARCUPHS`.
   - **REACTIVATE** (`mode = 'MNT'`):
     - For each record, verify existence in `ARCUPR` and `cpdel = 'D'`.
     - Set `cpdel = 'A'`, update `ARCUPR`, and log to `ARCUPHS`.
   - **COPY** (`mode = 'MNT'`):
     - Verify `copy_cust` and `copy_ship` exist in `ARCUST` and `SHIPTO`.
     - Copy alternate descriptions (`cpcpds`) from source `ARCUPR` records (`cpcono = co`, `cpcust = copy_cust`, `cpship = copy_ship`) to target records.
     - Add or update target `ARCUPR` records, log to `ARCUPHS`.

4. **Apply Freight Code Rules**:
   - If `frcd = 'C'` (collect):
     - `sfrt` and `cafr` must be `'Y'`, `'N'`, or blank.
     - Defaults: `sfrt = 'N'`, `cafr = 'N'` if blank.
     - If `cafr = 'Y'`, calculate freight for non-Bradford locations (e.g., Anchor).
     - If `sfrt = 'Y'`, apply $100 service fee for ARG-arranged shipping.
   - If `frcd = 'P'` (prepaid):
     - Defaults: `sfrt = 'N'`, `cafr = 'Y'` if blank.
   - If `frcd = 'A'` (prepaid & add):
     - `sfrt` must be `'Y'`.
     - Defaults: `cafr = 'Y'` if blank.

5. **Return Results**:
   - Return `SUCCESS` with processed records if no errors.
   - Return `ERROR` with error messages if validations fail.
   - For `DELETE`, return `NO_ACTIVITY` if no invoices or sales found.

## **Business Rules**
1. **Data Validation**:
   - Company, customer, and shipto must exist in respective files.
   - Product codes must be sellable (`GSPROD.tpsell = 'Y'`).
   - Container types and freight codes must exist in `GSTABL`.
   - Gallons billed code must be `'G'` or blank.
   - Separate and calculate freight codes must be `'Y'`, `'N'`, or blank.

2. **Freight Code Logic**:
   - Collect (`frcd = 'C'`): Supports non-Bradford locations (`cafr = 'Y'`) and ARG-arranged shipping with $100 fee (`sfrt = 'Y'`).
   - Prepaid (`frcd = 'P'`): Freight included in price.
   - Prepaid & Add (`frcd = 'A'`): Freight added separately, requires `sfrt = 'Y'`.

3. **Deletion Restrictions**:
   - Customers with active invoices (`CRDETX`) or sales (if `bcinst = '5'`) cannot be deleted.
   - Deletion marks records as inactive (`cpdel = 'D'`) rather than physical deletion.

4. **Record Management**:
   - Prevent adding duplicate or deleted records.
   - Reactivation restores deleted records (`cpdel = 'A'`).
   - All changes (add, update, delete, reactivate, copy) are logged to `ARCUPHS`.

5. **Inquiry Mode**:
   - Read-only access, no updates allowed.

## **Calculations**
- **Freight Service Fee** (if `frcd = 'C'` and `sfrt = 'Y'`):
  - Add $100 fee for ARG-arranged shipping (per `JB02`).
- **Timestamp for History**:
  - Use system date and time (YYYYMMDD, HHMMSS) for `ARCUPHS` records.

## **Dependencies**
- **Files**: `BICONT`, `ARCUST`, `SHIPTO`, `GSPROD`, `GSTABL`, `CRDETX`, `SA5FIXD`, `SA5FIXM`, `SA5BCXD`, `SA5BCXM`, `SA5DBXD`, `SA5DBXM`, `SA5COXD`, `SA5COXM`, `ARCUPR`, `ARCUPHS`.
- **External Logic**: Assumes external validation for copying alternate descriptions (similar to `BI9078`).

## **Error Handling**
- Return specific error messages for invalid inputs, duplicate records, or deletion restrictions.
- Log errors in the output `Error Messages` array.



---

### **Explanation**

- **Use Cases**: The two programs cover customer deletion validation (`AR9009`) and comprehensive product data maintenance (`BI907`). These are treated as distinct but related use cases, with `AR9009` serving as a prerequisite for deletion in the broader maintenance process.
- **Function Design**: The function consolidates both use cases into a single, programmatic process, eliminating screen interactions. It incorporates `AR9009`’s deletion validation as a step within the `DELETE` operation and extends `BI907`’s functionality to handle all operations (add, update, delete, reactivate, copy, inquire) via structured inputs.
- **Conciseness**: The document focuses on business requirements (validations, freight rules, deletion restrictions, history tracking) and briefly describes calculations (freight fee, timestamp). Technical details like file structures are implied through dependencies.
- **Assumptions**: The function assumes external logic for copying alternate descriptions (akin to `BI9078`) and omits screen-specific logic (e.g., subfile handling) to focus on data processing.

If you need further refinements or additional details, let me know!