The call stack consists of three programs: the OCL program (`AR901P.ocl36.txt`), the RPGLE program (`AR901P.rpgle.txt`), and the RPG II program (`AR901.rpg36.txt`). These programs work together to generate a "Customer Master File List" report on the IBM System/36. Below, I’ll identify the use cases implemented by this program suite and provide a concise function requirements document for a non-interactive version of the primary use case, assuming inputs are provided programmatically rather than via a screen.

---

### Use Cases Implemented

The program suite implements a single primary use case:

1. **Generate Customer Master File List Report**:
   - **Description**: Produces a printed report listing customer details, including customer number, name, address, phone number, contact name, and payment terms, grouped by company. The report can be filtered by company number(s) or customer number(s), with options to include address details, run interactively or in batch, and specify the number of copies.
   - **Components**:
     - **OCL Program (`AR901P.ocl36.txt`)**: Sets up the job environment, calls initialization utilities (`GSGENIEC`, `GSY2K`), loads the RPGLE program (`AR901P`), and conditionally queues or runs the RPG II program (`AR901`).
     - **RPGLE Program (`AR901P.rpgle`)**: Collects and validates user inputs (company/customer selections, address list flag, job queue flag, number of copies) and prepares parameters for the report generation.
     - **RPG II Program (`AR901.rpg36`)**: Processes customer data from files (`ARCUST`, `ARCONT`, `ARCUSP`, `GSTABL`) and generates the printed report.

No additional distinct use cases are evident, as the programs focus on a single workflow: generating a customer master list report with configurable parameters.

---

### Function Requirements Document



# Customer Master List Function Requirements

## Overview
The `GenerateCustomerMasterList` function generates a customer master file list report, listing customer details (customer number, name, address, phone, contact, payment terms) grouped by company. It accepts input parameters programmatically (no screen interaction) and produces a printed report, supporting filtering by company/customer and configuration options.

## Inputs
- **Company Selection** (`kyalco`, string):
  - Values: 'ALL' (all companies) or 'CO ' (specific companies).
- **Company Numbers** (`kyco1`, `kyco2`, `kyco3`, numeric, 2 digits each):
  - Required if `kyalco = 'CO '`. Must exist in `ARCONT` and not be deleted (`acdel ≠ 'D'`).
- **Customer Selection** (`kyalcs`, string):
  - Values: 'ALL' (all customers) or 'SEL' (selected customers).
- **Customer Numbers** (`kycs01` to `kycs10`, numeric, 6 digits each):
  - Required if `kyalcs = 'SEL'`. At least one non-zero value.
- **Address List Flag** (`addlst`, string):
  - Values: 'Y' (include address line 4), 'N' (exclude), or blank.
- **Dollar List Flag** (`dollst`, string):
  - Values: 'Y', 'N', or blank (not used in report generation but validated).
- **Job Queue Flag** (`kyjobq`, string):
  - Values: 'Y' (queue report job), 'N' (run interactively), or blank.
- **Number of Copies** (`kycopy`, numeric, 2 digits):
  - Number of report copies (default: 1 if zero).

## Outputs
- **Printed Report** (`PRINTER` file):
  - Format: 164-character lines, grouped by company.
  - Header: Company name, page number, date (YY/MM/DD), time (HH.MM.SS).
  - Columns: Customer number (6 digits), name (30 chars), address (4 lines, 30 chars each, line 4 conditional on `addlst = 'Y'`), phone (XXX-XXXX), contact name (25 chars), payment terms (20 chars).

## Process Steps
1. **Validate Inputs**:
   - Check `kyalco` is 'ALL' or 'CO '; else return error "ENTER ALL OR CO".
   - If `kyalco = 'CO '`, verify `kyco1`, `kyco2`, or `kyco3` are non-zero, exist in `ARCONT`, and not deleted; else return "ENTER COMPANY #" or "INVALID COMPANY #".
   - Check `kyalcs` is 'ALL' or 'SEL'; else return "ENTER ALL OR SEL".
   - If `kyalcs = 'SEL'`, ensure at least one `kycs01` to `kycs10` is non-zero; else return "ENTER CUSTOMER #".
   - Check `addlst`, `dollst`, `kyjobq` are 'Y', 'N', or blank; else return "ENTER Y, N OR BLANK".
   - Set `kycopy` to 1 if zero.
   - If `addlst ≠ 'Y'` and `dollst ≠ 'Y'`, set `addlst = 'Y'`.

2. **Initialize Data**:
   - Retrieve up to three company records from `ARCONT` (company number, name) for filtering.
   - Check `GSCONT` for default company number (`gxcono`); set `kyalco = 'CO '` and `kyco1 = gxcono` if found, else `kyalco = 'ALL'`.
   - Set defaults: `kyalcs = 'ALL'`, `addlst = 'Y'`, `dollst = ' '`, `kyjobq = 'N'`, `kycopy = 1`.

3. **Process Customer Records**:
   - Read `ARCUST` records, filtered by:
     - `kyalco = 'ALL'`: All records.
     - `kyalco = 'CO '`: Records matching `kyco1`, `kyco2`, or `kyco3`.
     - `kyalcs = 'ALL'`: All customers per company.
     - `kyalcs = 'SEL'`: Customers matching `kycs01` to `kycs10`.
   - For each record:
     - Chain `ARCO` to `ARCONT` for company name (`ACNAME`); set to blanks if not found.
     - Chain `ARCOCU` to `ARCUSP` for contact name (`CSCNCT`); set to blanks if not found.
     - Chain `ARTERM` (prefixed with 'ARTERM') to `GSTABL` for terms description (`TBDESC`); set to blanks if not found.
     - Convert `ARAREA` (3 digits) and `ARTELE` (7 digits) to zoned format; format phone as XXX-XXXX.

4. **Generate Report**:
   - For each company (`ARCO`):
     - Print header: `ACNAME`, page number, date (from system time), time (HH.MM.SS).
     - Print title: "* CUSTOMER MASTER LIST *".
     - Print column headers: "CUST#", "CUSTOMER NAME", "ADDRESS", "PHONE", "CONTACT", "PAYMENT TERMS".
     - Print decorative lines (asterisks).
   - For each customer:
     - Print: `ARCUST` (customer number), `ARNAME`, `ARADR1-3`, `ARADR4` (if `addlst = 'Y'`), `AREA-TELE`, `CSCNCT`, `TRMDSC`.
   - Repeat for `kycopy` copies, either interactively or queued based on `kyjobq`.

## Business Rules
1. **Company Filtering**:
   - If `kyalco = 'CO '`, include only customers with company numbers matching `kyco1`, `kyco2`, or `kyco3`, validated against `ARCONT` (non-deleted).
   - If `kyalco = 'ALL'`, include all companies.

2. **Customer Filtering**:
   - If `kyalcs = 'SEL'`, include only customers matching `kycs01` to `kycs10`.
   - If `kyalcs = 'ALL'`, include all customers per company.

3. **Address Inclusion**:
   - Include address line 4 only if `addlst = 'Y'`.
   - Default to `addlst = 'Y'` if both `addlst` and `dollst` are not 'Y'.

4. **Validation**:
   - Invalid inputs (e.g., `kyalco ≠ 'ALL' or 'CO '`, non-existent company numbers, zero customer numbers for `kyalcs = 'SEL'`) return specific error messages.
   - `kycopy` defaults to 1 if zero.

5. **Execution Mode**:
   - If `kyjobq = 'Y'`, queue the report job; if 'N' or blank, run interactively.

## Calculations
- **Date and Time**:
  - Retrieve system time (12 digits) and extract date (YYMMDD, formatted as YY/MM/DD) and time (HHMMSS, formatted as HH.MM.SS).
- **Page Number**:
  - Initialize to 0, increment per company group.
- **Phone Number**:
  - Combine `ARAREA` (3 digits) and `ARTELE` (7 digits) into XXX-XXXX format.
- **Default Values**:
  - Set `kycopy = 1` if zero.
  - Set `addlst = 'Y'` if both `addlst` and `dollst ≠ 'Y'`.

## Data Sources
- **ARCUST**: Customer master (customer number, name, address, phone, terms).
- **ARCONT**: Company master (company name).
- **ARCUSP**: Customer supplemental (contact name).
- **GSTABL**: Table file (terms descriptions).
- **GSCONT**: System control (default company number).

## Error Handling
- Return error messages for invalid inputs (e.g., "INVALID COMPANY #", "ENTER CUSTOMER #").
- Skip deleted records (`acdel = 'D'` in `ARCONT`, `tbdel = 'D'` in `GSTABL`).
- Set blank values for missing data (e.g., `ACNAME`, `CSCNCT`, `TRMDSC`).



---

### Additional Notes
- **Non-Interactive Design**: The function assumes inputs are provided programmatically (e.g., via API or parameter list) rather than through the interactive workstation file (`AR901PD`) used in `AR901P.rpgle`. This eliminates the need for screen handling logic (e.g., `read ar901pfm`, `*in01`).
- **Dollst Flag**: The `dollst` flag is validated but not used in `AR901.rpg36` for report generation, suggesting it may be a legacy parameter or used in related programs not provided.
- **UUID Choice**: A new UUID (`e8e616e0-d894-4936-a3f5-391682ee794d`) is used as this is a new artifact, unrelated to previous artifacts in the conversation.
- **Conciseness**: The document focuses on business requirements and key calculations, omitting low-level RPG II/LE/OCL syntax details while preserving the core logic and rules.

If you need further refinements (e.g., specific input formats, output report layout, or additional use cases), please provide additional details.