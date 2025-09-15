### List of Use Cases Implemented by the AR156 Program Call Stack

The AR156 program call stack, comprising `AR156.ocl36`, `AR156P.rpgle`, `AR135TC.clp`, `AR137A.ocl36`, `AR137A.rpg36`, and `AR156.rpg36`, implements a single primary use case:

1. **Generate NACHA-Compliant EFT File for Accounts Receivable Withdrawals**:
   - This use case involves validating input parameters, processing EFT transaction data, creating a bank upload table, generating a NACHA-compliant file for EFT withdrawals through PNC Bank, and producing a report for auditing. The process ensures that only valid EFT participants are included, and the output file adheres to NACHA standards for submission to the bank.

---

### Function Requirement Document: Generate NACHA-Compliant EFT File

<xaiArtifact artifact_id="2e6e1908-799d-49d1-8211-dad51b39d327" artifact_version_id="0d2ea25d-0a2c-40dc-9d40-b68c505b8927" title="EFT_File_Generation_Requirements.md" contentType="text/markdown">

# Function Requirement Document: Generate NACHA-Compliant EFT File

## Overview
This function generates a NACHA-compliant file for Electronic Funds Transfer (EFT) withdrawals from accounts receivable for submission to PNC Bank, along with an audit report. It processes input data, validates parameters, and creates a bank upload table, ensuring compliance with NACHA standards and company-specific EFT rules.

## Inputs
- **Company Number** (`KYCO`, 2 digits): Identifies the company processing EFTs.
- **Batch Create Date** (`KYUPDT`, 6 digits, YYMMDD): Specifies the date for selecting EFT transactions.
- **File Group** (`P$FGRP`, 1 character): Library prefix for dynamic file naming.
- **EFT Transaction Data** (`AREFTD`, input file): Contains transaction details (e.g., customer, invoice amount, discount, date).
- **Customer Master Data** (`ARCUST`): Contains customer details, including EFT participation status.
- **Company Control Data** (`ARCONT`): Contains company settings, including EFT configuration.
- **Customer-Specific Data** (`ARCUSP`): Contains EFT banking details (e.g., routing number, account number).

## Outputs
- **NACHA File** (`EFTFILE`, 94-byte records): NACHA-compliant file with File Header, Batch Header, Entry Detail, Batch Control, and File Control records, padded to a multiple of 10 records.
- **EFT Report** (`REPORT`, 132-byte printer file): Audit report listing EFT transactions by customer.
- **Bank Upload Table** (`AREFTS`, 100-byte records): Staging file with summarized EFT data per customer.
- **Status Indicator** (`STATUS`, 1 character): 'Y' for successful validation, 'N' for failure.

## Process Steps
1. **Validate Inputs**:
   - Verify `KYCO` exists in `ARCONT`.
   - Ensure `KYUPDT` is non-zero.
   - Check if the EFT data file (`P$FGRP + 'E' + KYUPDT`) exists in `QS36F`.
   - Return `STATUS = 'N'` if any validation fails.

2. **Prepare EFT Data**:
   - Copy EFT data from `QS36F/P$FGRP + 'E' + KYUPDT` to `AREFTD`, replacing existing data.
   - Clear `AREFTS` to ensure a clean staging file.

3. **Process EFT Transactions**:
   - For each customer in `AREFTD` with `AREFT = 'Y'` in `ARCUST`:
     - Retrieve customer details (`ARNAME`, `CSARTE`, `CSABK#`) from `ARCUST` and `ARCUSP`.
     - Calculate EFT amount: `AFEAMT = AFAMT - AFDISC` (invoice amount minus discount).
     - Accumulate totals at customer (`L2`) and file (`LR`) levels: invoice amount (`AFAMT`), discount (`AFDISC`), EFT amount (`AFEAMT`).
     - Convert dates (`AFSLDT`, `KYUPDT`) to 8-digit format (20YYMMDD) for NACHA compliance.
     - Write summarized data to `AREFTS` (e.g., `AFCO`, `AFCUST`, `L2IAMT`, `L2DAMT`, `L2EAMT`, `AFDAT8`, `AFUPD8`).
   - Generate an EFT report with customer details, totals, and email addresses for spooling.

4. **Generate NACHA File**:
   - Write **File Header Record** (`RECTY1`): Includes PNC Bank ABA numbers, transmission date/time, company name ('AMERICAN REFINING GROUP'), and reference code ('EFT DRAW').
   - For each company (`AFCO`):
     - Write **Batch Header Record** (`RECTY5`): Includes service class code '200', company name ('AMERREFINING ACH'), federal tax ID, and effective date.
     - For each customer (`AFCUST`):
       - Write **Entry Detail Record** (`RECTY6`): Includes transaction code '27', customer bank routing (`CSARTE`), account number (`CSABK#`), EFT amount (`AFEAMT`), and customer name.
       - Update batch and file counters (`L2CNT`, `LRCNT`) and hash totals (`L2HASH`, `LRHASH`) using routing number.
     - Write **Batch Control Record** (`RECTY8`): Summarizes batch entry count, hash, and debit total.
   - Write **File Control Record** (`RECTY9`): Summarizes batch count, block count, entry count, hash, and debit total.
   - Pad the file with filler records ('999999...') to a multiple of 10 records (blocking factor 10).

5. **Clean Up**:
   - Delete temporary file `P$FGRP + AR156S` if it exists.
   - Clear local variables.

## Business Rules
1. **Validation**:
   - Company number must exist in `ARCONT`.
   - Batch create date must be non-zero and correspond to an existing EFT data file.
   - Only customers with `AREFT = 'Y'` in `ARCUST` are processed.

2. **EFT Amount Calculation**:
   - `AFEAMT = AFAMT - AFDISC` for each transaction, accumulated at customer and file levels.

3. **NACHA Compliance**:
   - File adheres to NACHA format with fixed record types, lengths (94 bytes), and blocking factor (10 records).
   - Includes PNC Bank ABA numbers ('043000096', '04300009'), federal tax ID ('1222318612'), and company details.

4. **Dynamic File Naming**:
   - Uses `P$FGRP` for library prefix and `KYUPDT` for date-specific file names (e.g., `P$FGRP + 'E' + KYUPDT`).

5. **Reporting**:
   - Generates a report with customer details, EFT amounts, and totals, including email addresses for automated spooling.

6. **Data Integrity**:
   - Clears `AREFTS` before processing to prevent residual data.
   - Maintains accurate entry counts, hash totals (based on routing numbers), and debit totals for NACHA compliance.

## Calculations
- **EFT Amount**: `AFEAMT = AFAMT - AFDISC` (packed decimal, 9 digits, 2 decimal places).
- **Date Conversion**: `AFSLDT` and `KYUPDT` (6-digit YYMMDD) multiplied by 10000.01 and prefixed with '20' to form 8-digit dates (20YYMMDD).
- **Hash Totals**: Sum of customer bank routing numbers (`CSARTE`) for batch (`L2HASH`) and file (`LRHASH`).
- **Block Count**: `LRBLOK = RECCNT / 10`, rounded up if remainder exists.
- **Padding**: Filler records added until `RECCNT` reaches `LRBLOK * 10`.

## Dependencies
- **Files**:
  - Input: `AREFTD`, `ARCUST`, `ARCONT`, `ARCUSP`.
  - Output: `AREFTS`, `EFTFILE`, `REPORT`.
- **External Program**: `GSGENIEC` (environment setup, called by `AR156.ocl36`).

</xaiArtifact>


Below is a table summarizing the programs provided (`AR156.ocl36`, `AR156P.rpgle`, `AR135TC.clp`, `AR137A.ocl36`, `AR137A.rpg36`, `AR156.rpg36`), including their general purpose and outputs. The table is presented as a Markdown artifact, as per the requirements for generating structured content.

<xaiArtifact artifact_id="faa42e6f-2add-414b-9cc4-2c3912b860ce" artifact_version_id="7a74824e-ceda-44ea-a84f-71248db936cb" title="Program_Summary_Table.md" contentType="text/markdown">

# Program Summary Table

| Program File          | General Purpose                                                                 | Outputs                                                                 |
|-----------------------|---------------------------------------------------------------------------------|-------------------------------------------------------------------------|
| AR156.ocl36           | Orchestrates the EFT process, calling programs to validate inputs, process EFT data, and generate a NACHA file for PNC Bank. | `STATUS` ('Y' or 'N'), `AREFTS` (staging file), `EFTFILE` (NACHA file), temporary files deleted. |
| AR156P.rpgle          | Validates company number and batch date, ensuring the EFT data file exists.      | `STATUS` ('Y' or 'N'), updated `KYCO`, `KYUPDT`, error messages in `AR156PD` display file. |
| AR135TC.clp           | Checks if a specified EFT data file exists in the `QS36F` library.               | `STATUS` ('Y' if file exists, 'N' if not).                              |
| AR137A.ocl36          | Clears the staging file and initiates EFT data processing and report generation. | Cleared `AREFTS` file, triggers `AR137A.rpg36` execution.               |
| AR137A.rpg36          | Processes EFT transactions, creates a bank upload table, and generates a report. | `AREFTS` (populated with EFT data), EFT report (printer file).          |
| AR156.rpg36           | Generates a NACHA-compliant file for EFT withdrawals and an audit report.        | `EFTFILE` (NACHA file with header, batch, detail, control, and filler records), `REPORT` (printer file). |

</xaiArtifact>