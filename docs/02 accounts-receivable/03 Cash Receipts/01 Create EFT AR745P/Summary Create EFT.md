Based on the provided documents (`AR745P.ocl36.txt`, `AR745P.rpgle.txt`, `AR745.ocl36.txt`, `AR745.rpg36.txt`, and `AR745A.ocl36.txt`), the program call stack implements a process to generate an Electronic Funds Transfer (EFT) Customer Accounts Receivable Due Report, with additional handling for credit invoices. Below, I’ll identify the use cases implemented by this program stack and then provide a function requirement document for the primary use case, reimagined as a large function that accepts inputs directly rather than using interactive screen prompts.

### List of Use Cases

The program stack primarily implements a single use case, with a secondary conditional use case:

1. **Generate EFT Customer Accounts Receivable Due Report**:
   - The core functionality involves validating input parameters, sorting accounts receivable data, generating detailed and summary EFT reports, and producing an output file for bank upload. This includes filtering invoices for EFT-eligible customers within a specified date range and handling Y2K-compliant date conversions.
   - Components: `AR745P.ocl36.txt` (input validation and job control), `AR745P.rpgle.txt` (input processing), `AR745.ocl36.txt` (data sorting and report generation), `AR745.rpg36.txt` (report and EFT file output).

2. **Process and Distribute Credit Invoice Report (Conditional)**:
   - If credit invoices are detected in production mode (`?9? = G`), the program generates and emails a credit invoice report.
   - Component: `AR745A.ocl36.txt` (credit invoice check and report distribution).
   - This is a secondary use case triggered only under specific conditions and is dependent on the primary use case.

Since the primary use case is the generation of the EFT report, I’ll focus the function requirement document on this, incorporating the credit invoice handling as a conditional step, and assume inputs are provided directly to a function rather than through a screen.

---

<xaiArtifact artifact_id="60410ecd-87d8-4729-9116-50e8e236a460" artifact_version_id="e741f4c0-cb49-4512-912a-6b2d8c4d3d40" title="EFTReportFunctionRequirements.md" contentType="text/markdown">

# EFT Customer Accounts Receivable Due Report Function Requirements

## Purpose
The `GenerateEFTReport` function generates an Electronic Funds Transfer (EFT) report for customer accounts receivable due within a specified date range, producing detailed and summary reports and an EFT output file for bank upload. It also checks for credit invoices and emails a report if found in production mode.

## Inputs
- **Company Number** (`KYCO1`, 2-digit numeric): Identifies the company.
- **Customer Number** (`KYCUST`, 6-digit numeric, optional): Specific customer or `0` for all EFT customers.
- **From Date** (`KYFRDT`, 6-digit MMDDYY): Start of transaction date range.
- **To Date** (`KYTODT`, 6-digit MMDDYY): End of transaction date range.
- **Settlement Date** (`KYSEDT`, 6-digit MMDDYY): EFT settlement date.
- **Upload Date** (`KYUPDT`, 6-digit MMDDYY): Date for bank upload.
- **Batch Mode** (`KYBATC`, 1-character, 'Y'/'N'/blank): Indicates batch processing.
- **Job Queue Flag** (`KYJOBQ`, 1-character, 'Y'/'N'/blank): Indicates job queue submission.
- **Copy Count** (`KYCOPY`, 2-digit numeric): Number of report copies (defaults to 1 if 0).
- **Environment** (`ENV`, 1-character, e.g., 'G' for production): Determines output queue and credit invoice processing.

## Outputs
- **Detailed Report** (`LIST3`): Lists customer, invoice, ship date, discount date, settlement date, invoice amount, discount amount, and EFT amount.
- **Summary Report** (`LIST2`): Aggregates by customer without invoice/sequence details.
- **EFT Output File** (`AREFTD`): Disk file with EFT transaction data for bank upload.
- **Credit Invoice Report** (if applicable): Emailed report listing credit invoices (production mode only).

## Process Steps
1. **Validate Inputs**:
   - Verify company number exists in `ARCONT`.
   - If customer number is non-zero, verify it exists in `ARCUST` and has `AREFT = 'Y'`.
   - Validate dates (`KYFRDT`, `KYTODT`, `KYSEDT`, `KYUPDT`) as valid MMDDYY, checking month (1–12), day (1–31, 28/29 for February with leap year logic), and year.
   - Ensure `KYBATC` and `KYJOBQ` are 'Y', 'N', or blank.
   - Set `KYCOPY` to 1 if 0.
   - Convert dates to 8-digit CYYMMDD format (`KFCYMD`, `KTCYMD`, `KSCYMD`) using Y2K logic (compare year with `Y2KCMP`, set century from `Y2KCEN`).

2. **Prepare Environment**:
   - If `KYBATC = 'Y'`, create temporary work files (`AREFTD`) named by settlement date (e.g., `EMMDDYY`, `XMMDDYY`) in `QS36F`, deleting existing files.
   - Set processing mode: `I*C` for all customers (`KYCUST = 0`) or `IAC` for specific customer.

3. **Sort Accounts Receivable Data**:
   - Filter `ARDETL` records:
     - Non-deleted (`ADDEL ≠ 'C'`).
     - Match company number (`ADCO = KYCO1`).
     - Invoice type 'I' (`ADTYPE`).
     - Transaction date (`ADTYM8`) within `KFCYMD` to `KTCYMD`.
     - Match customer number if specified (`ADCUST = KYCUST`).
   - Sort by company (`ADCO`), customer (`ADCUST`), invoice (`ADINV#`), type (`ADTYPE`), and sequence (`ADSEQ#`).
   - Output to `AR745S` (up to 999,000 records).

4. **Generate Reports**:
   - Read `AR745S`, validate against `ARCUST` (`AREFT = 'Y'`) and `ARCONT`.
   - Calculate:
     - **Invoice Amount** (`INVAMT`): Sum of `ADAMT` per invoice.
     - **Discount Amount** (`ADDSAL`): Discount allowed (`ADDSAL` or `ADDSA7`).
     - **EFT Amount** (`EFTAMT`): `INVAMT` minus `ADDSAL` (if applicable).
     - Accumulate totals at customer (`L1IAMT`, `L1DAMT`, `L1EAMT`) and company (`L2IAMT`, `L2DAMT`, `L2EAMT`) levels.
   - Write to:
     - **Detailed Report** (`LIST3`): Sequence number, customer, name, ship date (`ADTXMM/DD/YY`), discount date (`DISMD/CY`), settlement date (`KSMM/DD/YY`), invoice number, amounts.
     - **Summary Report** (`LIST2`): Customer, name, dates, amounts (no sequence/invoice).
     - **EFT File** (`AREFTD`): Sequence number, company, customer, invoice, amounts, G/L codes (`ACARGL`, `ACCSGL`, `ACDSGL`), dates.
   - Include headers with company name, date range, upload date, run date/time.

5. **Process Credit Invoices (Production Mode)**:
   - If `ENV = 'G'`, check for credit invoices (`ADAMT < 0`) using `AREFTCREDT` query.
   - If found, generate credit invoice report (`AREFTCRPRT`) and email via `SFARDST` to queue `AREFTCRINV`.

## Business Rules
1. **Data Validation**:
   - Company must exist in `ARCONT`.
   - Customer (if specified) must exist in `ARCUST` with `AREFT = 'Y'`.
   - Dates must be valid MMDDYY, with leap year checks for February.
   - `KYBATC` and `KYJOBQ` must be 'Y', 'N', or blank.
   - `KYCOPY` defaults to 1 if 0.

2. **Data Filtering**:
   - Include non-deleted (`ADDEL ≠ 'C'`), invoice-type 'I', and date-range-compliant records.
   - Process all EFT customers (`KYCUST = 0`) or a specific customer.

3. **Calculations**:
   - Invoice amount: Sum `ADAMT`.
   - Discount amount: Use `ADDSAL` or `ADDSA7` if available.
   - EFT amount: Invoice amount minus discount (if applied).
   - Accumulate totals at customer and company levels.

4. **Output Formatting**:
   - Detailed report includes sequence and invoice details; summary report omits them.
   - Printer output uses 10 CPI, 6 LPI, EFT forms.
   - EFT file includes G/L codes and 8-digit dates.

5. **Credit Invoice Handling**:
   - In production mode, generate and email credit invoice report if negative amounts are found.

## Dependencies
- **Files**:
  - `ARCONT`: Company data.
  - `ARCUST`: Customer data (EFT flag).
  - `ARDETL`: Accounts receivable details (input to `AR745S`).
  - `GSCONT`: Default company data.
  - `AR745S`: Sorted work file.
  - `AREFTD`: EFT output and temporary batch files.
  - `?9?ARINVCR`: Credit invoice condition.
  - `LIST`, `LIST1`, `LIST2`, `LIST3`: Printer outputs.
  - `QPQUPRFIL`: Credit invoice report output.
- **Programs/Queries**:
  - `#GSORT`: Sorts `ARDETL` into `AR745S`.
  - `AREFTCREDT`: Checks for credit invoices.
  - `AREFTCRPRT`: Generates credit invoice report.
  - `SFARDST`: Emails credit invoice report.

## Error Handling
- Return error codes for invalid company, customer, dates, or flags.
- Log errors for file access or query failures.
- Skip credit invoice processing if none are found.

## Assumptions
- Inputs are provided as parameters, not via interactive screens.
- Y2K logic uses `Y2KCMP` and `Y2KCEN` for century determination.
- Production mode (`ENV = 'G'`) triggers credit invoice processing.

</xaiArtifact>

### Notes
- The function requirement document consolidates the functionality of the entire call stack (`AR745P`, `AR745`, `AR745A`) into a single function, assuming direct input parameters.
- Calculations (e.g., `EFTAMT`) are inferred from context, as the `CALEFT` subroutine in `AR745.rpg36.txt` is truncated.
- If further details on specific calculations or file formats are needed, please provide additional context or clarify requirements.


## Call Stack 


| **Program** | **Order Called** | **Main Purpose** | **Tables Used** | **Outputs (Files or Side Effects)** |
|-------------|------------------|------------------|-----------------|-------------------------------|
| **GSGENIEC** | 1 | Initializes the environment for EFT processing. | None explicitly defined | Sets system flags or parameters (not specified, likely updates control areas). |
| **AR745P.ocl36.txt** | 2 | Controls job execution, validates inputs, and submits the EFT report generation job. | `ARCONT` (input, shared), `ARCUST` (input, shared) | Submits job to queue (`AR745`) or runs directly; updates control area (e.g., `KYCANC`, `KYCO1`). |
| **AR745P.rpgle.txt** | 3 | Validates user inputs (company, customer, dates) and prepares data for EFT report generation. | `AR745PD` (workstation display), `ARCONT` (input, keyed), `ARCUST` (input, keyed), `GSCONT` (input, keyed) | Updates `UDS` fields (e.g., `KYCO1`, `KYFRDT`, `KFCYMD`); displays error messages on `AR745PD`. |
| **#GSORT** | 4 | Sorts accounts receivable detail records for EFT report processing. | `ARDETL` (input, shared), `AR745S` (output, 999,000 records, job-retained) | Creates sorted `AR745S` file with filtered records. |
| **AR745.ocl36.txt** | 5 | Orchestrates sorting and report generation, creating temporary files in batch mode. | `ARDETL` (input, shared), `AR745S` (output/input, shared), `ARCUST` (input, shared), `ARCONT` (input, shared), `AREFTD` (output/input, shared, batch mode), `LIST` (printer) | Creates `AREFTD` files (batch mode); routes report to `AREFTOUTQ` or `TESTOUTQ`; calls `AR745A` in production mode. |
| **AR745.rpg36.txt** | 6 | Generates detailed and summary EFT reports and an EFT output file for bank upload. | `ARDETL` (input, from `AR745S`), `ARCUST` (input, keyed), `ARCONT` (input, keyed), `LIST`, `LIST1`, `LIST2`, `LIST3` (printer), `AREFTD` (output, keyed) | Produces detailed (`LIST3`) and summary (`LIST2`) reports; writes EFT data to `AREFTD`; accumulates totals. |
| **AR745A.ocl36.txt** | 7 (Conditional) | Checks for credit invoices and emails a report if found in production mode. | `?9?ARINVCR` (data area), `QPQUPRFIL` (printer output), implied `ARDETL` (via queries) | Emails credit invoice report to `AREFTCRINV` queue; no direct file output. |

### Notes on Call Order
- **GSGENIEC** is called first by `AR745P.ocl36.txt` for initialization.
- **AR745P.ocl36.txt** is the main entry point, invoking `AR745P.rpgle.txt` and conditionally submitting `AR745.ocl36.txt` (via job queue or direct call).
- **#GSORT** is called by `AR745.ocl36.txt` to sort data before report generation.
- **AR745.ocl36.txt** loads `AR745.rpg36.txt` and conditionally calls `AR745A.ocl36.txt` if `?9? = G`.
- **AR745.rpg36.txt** processes sorted data and generates reports.
- **AR745A.ocl36.txt** is only called in production mode for credit invoice processing.