The `AR890.ocl36` OCL script and `AR890.rpgle` RPGLE program together implement a **Customer Master Inquiry** system on an IBM midrange platform. Below, I’ll identify the use cases implemented by this program and provide a **Function Requirement Document** for the primary use case, reimagined as a non-interactive function that processes inputs to produce customer inquiry data.

---

### **List of Use Cases**

Based on the OCL and RPGLE code, the program implements the following use cases, all centered around retrieving and presenting customer-related data for inquiry purposes:

1. **Customer Master Inquiry**:
   - **Description**: Allows users to retrieve and view comprehensive customer information, including basic details (name, address), financial data (balances, credit limits), supplemental data (tax codes, comments), product codes, and contact information. Users can navigate through customer records and access specific inquiries (e.g., customer history, product codes, contacts) using command keys.
   - **Details**: The program supports multiple display formats (`AR890S01` to `AR890S05`) for different data views, validates inputs, and integrates with external programs (`AR915P`, `BI907AC`) for specific inquiries. It handles navigation (roll forward/backward) and error conditions (e.g., invalid company, deleted customer).

No additional distinct use cases are evident, as the program’s core functionality revolves around this single inquiry process, with variations in data displayed based on user input (e.g., command keys F2, F3, F4).

---

### **Function Requirement Document**



# Customer Master Inquiry Function Requirements

## **Overview**
The **Customer Master Inquiry Function** retrieves comprehensive customer information based on provided company and customer numbers, returning data from multiple related files without requiring interactive screen input. The function supports business needs for retrieving customer details, financials, supplemental data, product codes, and contact information for reporting, auditing, or integration purposes.

## **Inputs**
- **Company Number** (`co`): 2-digit numeric (e.g., 01).
- **Customer Number** (`cust`): 6-digit numeric (e.g., 000123).
- **Ship-To Number** (`ship`): 3-digit numeric (optional, default 000 for product inquiries).
- **Inquiry Type** (`inquiryType`): String indicating the type of inquiry:
  - "CUST": Customer history (ARCUST).
  - "SUPP": Supplemental data (ARCUSP).
  - "PROD": Product code history (ARCUPR).
  - "CONT": Form type contacts (ARCUFM, via AR915P).
- **File Group** (`fileGroup`): 1-character library prefix (e.g., 'X').

## **Outputs**
A structured data object containing:
- **Customer Details**: Name, address, zip, phone, salesman, terms, credit limit, etc.
- **Financial Data**: Total due, current due, aged balances, sales MTD/YTD, etc.
- **Supplemental Data**: Tax codes, comments, freight details, ACH information, etc.
- **Product Data**: Up to 17 product codes, descriptions, freight codes, etc.
- **Contact Data**: Ageing periods, invoicing style.
- **Error Messages**: Descriptive messages for invalid inputs or missing records.

## **Process Steps**
1. **Validate Inputs**:
   - Verify `co` exists in `ARCONT`. If not, return error: "INVALID COMPANY NUMBER ENTERED".
   - Chain `co` + `cust` to `ARCUST` and `ARCUSP`. If not found, return error: "CUSTOMER NOT FOUND".
   - Check deletion status (`ardel` or `csdel = 'D'`). If deleted, return error: "THIS CUSTOMER WAS PREVIOUSLY DELETED".

2. **Retrieve Customer Data** (if `inquiryType = "CUST"` or others):
   - From `ARCUST`, retrieve:
     - Name (`arname`), address (`aradr1-4`), zip (`arzip9`), phone (`ararea`, `artele`).
     - Financials: Total due (`artotd`), current due (`arcurd`), aged balances (`ar0110`, `ar1120`, `ar2130`, `arov30`), sales MTD/YTD (`armtd$`, `arytd$`), credit limit (`arclmt`), etc.
     - Other: Salesman (`arsls#`), terms (`arterm`), group (`argrup`), class (`arcucl`), EFT status (`areft`), federal EIN (`arfein`), etc.
   - Lookup descriptions for salesman, terms, group, and class in `GSTABL` using `tbdesc`.

3. **Retrieve Supplemental Data** (if `inquiryType = "SUPP"` or others):
   - From `ARCUSP`, retrieve:
     - Tax codes (`tc`), tax exemptions (`te`), start date (`csstdt`), credit comments (`cscmt1-3`), contact name (`cscnct`).
     - Freight details: Freight code (`csfrcd`), separate freight (`cssfrt`), freight bill name/address (`csfrnm`, `csfra1-3`).
     - Order/invoice marks (`csomk1-4`, `csimk1-2`), dispatch info (`csdsp1-4`), blanket PO (`csbkpo`), ACH data (`csarte`, `csabk#`).
   - Convert dates (`csstdt`, `csfsdt`, `csicdt`) from YYMMDD to MMDDYY for output.

4. **Retrieve Product Data** (if `inquiryType = "PROD"`):
   - Call `BI907AC` with `co`, `cust`, `ship`, `inquiryType = "INQ"`, and `fileGroup` to retrieve product data from `ARCUPR`.
   - For each product (up to 17):
     - From `GSPROD`, retrieve product code (`tpprod`), description (`tpdes1`).
     - From `ARCUPR`, retrieve gallons to bill (`cpglcd`), customer stock number (`cpcstk`), freight code (`cpfrcd`), separate freight (`cpsfrt`), calculated freight (`cpcafr`), container type (`cpcnty`).
   - Return error if no products found for `co` + `cust` + `ship`.

5. **Retrieve Contact Data** (if `inquiryType = "CONT"`):
   - Call `AR915P` with `co`, `cust`, `mode = "INQ"`, and `fileGroup` to retrieve contact data from `ARCUFM`/`ARCUFMX`.
   - From `ARCONT`, retrieve ageing periods (`aclmt1-4`).
   - From `BICONT`, retrieve invoicing style (`bcinst`).

6. **Handle EFT Validation**:
   - If `areft = 'Y'`, validate `csarte` (ACH routing code) and `csabk#` (ACH account number) are non-zero/non-blank. Include EFT validity flag in output.

7. **Return Results**:
   - Combine data from all relevant files into a structured output.
   - Include any error messages from validation steps.

## **Business Rules**
- **Input Validation**:
  - Company number must exist in `ARCONT`.
  - Customer must exist in `ARCUST` and `ARCUSP` and not be marked as deleted.
  - Product inquiry requires valid `ship` (defaults to 000).
- **Data Access**:
  - Only retrieve data for the specified `co` + `cust` combination.
  - Limit product data to 17 records to prevent overflow.
  - Use `fileGroup` to determine library prefix for file access.
- **EFT Processing**:
  - EFT is valid only if `areft = 'Y'` and both `csarte` and `csabk#` are populated.
- **Invoicing Style**:
  - If `bcinst = '5'` in `BICONT`, use header "To Bill Gross Gallons Enter 'G'"; otherwise, "To Bill Net Gallons Enter 'N'".
- **Date Conversion**:
  - Convert YYMMDD dates (e.g., `arhidt`, `csstdt`) to MMDDYY using multiplication by 100.0001 (e.g., `arhidt * 100.0001`).
- **Financial Calculations**:
  - Aged balances (`ar0110`, `ar1120`, `ar2130`, `arov30`) are pre-calculated in `ARCUST` and returned as-is.
  - Ageing periods (`aclmt1-4`) from `ARCONT` are formatted for display (e.g., `1-10`, `11-20`, `21-30`, `Over 30`).

## **Error Handling**
- Return descriptive error messages for:
  - Invalid company number.
  - Customer not found.
  - Deleted customer.
  - End of file reached (for product data).
- Ensure all output fields are initialized to blanks/zeros if data is unavailable.

## **Dependencies**
- **Files**:
  - `ARCUST`: Customer master data.
  - `ARCUSP`: Supplemental customer data.
  - `ARCUPR`: Customer product data.
  - `GSPROD`: Product codes and descriptions.
  - `ARCONT`: Ageing periods.
  - `BICONT`: Invoicing style.
  - `GSTABL`: Descriptions for salesman, terms, group, class.
  - `GSCONT`: Default company number.
- **External Programs**:
  - `BI907AC`: Retrieves `ARCUPR` data for product inquiries.
  - `AR915P`: Retrieves `ARCUFM`/`ARCUFMX` data for contact inquiries.



---

### **Summary**
The `AR890` program implements a single primary use case: **Customer Master Inquiry**, with variations for customer history, supplemental data, product codes, and contacts. The function requirement document reimagines this as a non-interactive function that takes company, customer, ship-to, inquiry type, and file group as inputs, retrieving and validating data from multiple files, applying business rules, and returning structured output with error handling. The document focuses on business requirements (validation, data retrieval, EFT checks) and includes calculations (e.g., date conversions) as needed.