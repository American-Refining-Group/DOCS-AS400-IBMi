### List of Use Cases Implemented in the GS928P Call Stack

The call stack consists of four RPG III programs: **GS928P** (main interactive program), **GS928** (maintenance/inquiry), **GS9284** (inactivate/reactivate), and **GS9285** (print listing). These programs collectively implement a system for managing Product/Container entries on an IBM i (AS/400) system, supporting both interactive user operations and batch reporting. Below are the identified use cases based on the functionality provided by the call stack:

1. **Browse and Filter Product/Container Entries (GS928P)**:
   - **Description**: Users can view a list of Product/Container records in a subfile, filter by company, product, or container, and toggle inclusion/exclusion of inactive records.
   - **Actions**: Scroll through records, reposition the list based on input criteria, and refresh the display.
   - **Details**: Supports inquiry (INQ) and maintenance (MNT) modes, with inactive records optionally filtered out (F8 toggle).

2. **Create a New Product/Container Entry (GS928P, GS928)**:
   - **Description**: Users can create a new Product/Container record by entering details directly or selecting an option from the subfile.
   - **Actions**: Validate input (company, product, container existence), call GS928 to create the record, and display a confirmation message.
   - **Details**: Only available in MNT mode; ensures description is not blank and keys are unique.

3. **Update an Existing Product/Container Entry (GS928P, GS928)**:
   - **Description**: Users can modify an existing Product/Container record’s details (e.g., descriptions) via the subfile or direct input.
   - **Actions**: Validate that the record is active and input is valid, call GS928 to update, and display a confirmation message.
   - **Details**: Only in MNT mode; key fields are protected if the record exists.

4. **InActivate or ReActivate a Product/Container Entry (GS928P, GS9284)**:
   - **Description**: Users can mark a Product/Container record as inactive ('I') or reactivate it ('A') via the subfile.
   - **Actions**: Select a record, call GS9284 to toggle the `ptdel` field, and display a confirmation message.
   - **Details**: Only in MNT mode; ensures the record’s current state allows the action (e.g., cannot inactivate an already inactive record).

5. **Display a Product/Container Entry (GS928P, GS928)**:
   - **Description**: Users can view details of a specific Product/Container record in a read-only format.
   - **Actions**: Select a record or enter keys directly, call GS928 in INQ mode to display details.
   - **Details**: Available in both MNT and INQ modes; no database changes.

6. **Print a Product/Container Listing (GS928P, GS9285)**:
   - **Description**: Users can generate a printed report of all Product/Container records, including active and inactive.
   - **Actions**: Trigger via F15, call GS9285 to produce a spooled report with headers and details.
   - **Details**: Includes all records, sorted by company, product, and container; report is held and saved for later retrieval.

7. **Prompt for Valid Product or Container Codes (GS928P, LGSPROD, LGSCNTR)**:
   - **Description**: Users can prompt for valid product or container codes to populate input fields.
   - **Actions**: Press F4 on product or container fields, call LGSPROD or LGSCNTR to return valid codes.
   - **Details**: Ensures valid input by fetching from GSPROD or GSCNTR1; only in MNT mode.

These use cases cover the full scope of the system’s functionality, from browsing and editing to reporting and input assistance, supporting both interactive and batch operations.

---

### Function Requirement Document: Product/Container Management System

<xaiArtifact artifact_id="5fa2d6c9-7241-424d-9c12-7f53c6fe5480" artifact_version_id="f8c0e6de-8054-495f-a836-2da2ecc08f31" title="ProductContainerManagementRequirements.md" contentType="text/markdown">

# Product/Container Management System Requirements

## Overview
This document outlines the functional requirements for a Product/Container Management System that manages records containing company, product, and container data. The system supports creating, updating, inactivating/reactivating, displaying, and printing records, with validation and reporting capabilities. Functions are designed to process inputs programmatically (no screen interaction) and return results or status flags.

## Business Requirements and Process Steps

### 1. Browse and Filter Product/Container Records
- **Inputs**: File group ('G' or 'Z'), company (optional), product (optional), container (optional), includeInactive (boolean).
- **Outputs**: List of records (company, product, container, description1, description2, deleteFlag).
- **Process Steps**:
  1. Apply file overrides for GSPRCT based on file group ('G' → GGSPRCT, 'Z' → ZGSPRCT).
  2. If company, product, or container provided, position cursor to matching records.
  3. Read GSPRCT sequentially, filtering out inactive records (deleteFlag = 'I') if includeInactive = false.
  4. Return records with fields: company, product, container, description1, description2, deleteFlag.
- **Business Rules**:
  - Records sorted by company, product, container.
  - Include all records if no filters provided.
  - Inactive records (deleteFlag = 'I') excluded unless includeInactive = true.
- **Calculations**: None.

### 2. Create a New Product/Container Record
- **Inputs**: File group ('G' or 'Z'), company, product, container, description1, description2.
- **Outputs**: Success flag (1 = success, 0 = failure), error message (if any).
- **Process Steps**:
  1. Apply file overrides for GSPRCT, BICONT, GSPROD, GSCNTR1 based on file group.
  2. Validate inputs:
     - Company exists in BICONT.
     - Product exists in GSPROD.
     - Container exists in GSCNTR1.
     - Description1 is not blank.
     - Record (company, product, container) does not exist in GSPRCT.
  3. If valid, write new record to GSPRCT with deleteFlag = 'A' (active).
  4. Return success flag and error message (if validation fails).
- **Business Rules**:
  - Duplicate records (same company, product, container) are not allowed.
  - Description1 is mandatory.
  - New records are always active (deleteFlag = 'A').
- **Calculations**: None.

### 3. Update an Existing Product/Container Record
- **Inputs**: File group ('G' or 'Z'), company, product, container, description1, description2.
- **Outputs**: Success flag (1 = success, 0 = failure), error message (if any).
- **Process Steps**:
  1. Apply file overrides for GSPRCT, BICONT, GSPROD, GSCNTR1.
  2. Validate inputs:
     - Record exists in GSPRCT and deleteFlag = 'A' (active).
     - Description1 is not blank.
  3. If valid and fields changed, update description1 and description2 in GSPRCT.
  4. Return success flag and error message (if validation fails).
- **Business Rules**:
  - Cannot update inactive records (deleteFlag = 'I').
  - Key fields (company, product, container) cannot be changed.
  - Description1 is mandatory.
- **Calculations**: Compare input fields with existing record to detect changes.

### 4. InActivate or ReActivate a Product/Container Record
- **Inputs**: File group ('G' or 'Z'), company, product, container, action ('I' = InActivate, 'A' = ReActivate).
- **Outputs**: Status flag ('I' = inactivated, 'A' = reactivated, blank = no action), error message (if any).
- **Process Steps**:
  1. Apply file override for GSPRCT.
  2. Validate:
     - Record exists in GSPRCT.
     - For InActivate: deleteFlag ≠ 'I'.
     - For ReActivate: deleteFlag = 'I'.
  3. If valid, update deleteFlag in GSPRCT ('I' or 'A').
  4. Return status flag and error message (if validation fails).
- **Business Rules**:
  - Cannot inactivate an already inactive record.
  - Cannot reactivate an already active record.
  - Only updates deleteFlag; other fields unchanged.
- **Calculations**: None.

### 5. Display a Product/Container Record
- **Inputs**: File group ('G' or 'Z'), company, product, container.
- **Outputs**: Record details (company, product, container, description1, description2, deleteFlag), error message (if any).
- **Process Steps**:
  1. Apply file overrides for GSPRCT, BICONT, GSPROD, GSCNTR1.
  2. Chain to GSPRCT using keys.
  3. If found, fetch product description from GSPROD and container description from GSCNTR1.
  4. Return record details and error message (if not found).
- **Business Rules**:
  - Returns data regardless of active/inactive status.
  - No database changes.
- **Calculations**: None.

### 6. Print Product/Container Listing
- **Inputs**: File group ('G' or 'Z').
- **Outputs**: Success flag (1 = success, 0 = failure), spooled report.
- **Process Steps**:
  1. Apply file override for GSPRCT and printer overrides for QSYSPRT (PAGESIZE=68x164, LPI=8, CPI=15, OVRFLW=62, OUTQ=*JOB, HOLD=*YES, SAVE=*YES).
  2. Open QSYSPRT and print headers (company name, title, job/user info, date/time, column headers).
  3. Read GSPRCT sequentially, printing each record (company, product, container, description1, description2, deleteFlag).
  4. Handle page overflow at line 62, reprinting headers.
  5. Close QSYSPRT, delete overrides, and return success flag.
- **Business Rules**:
  - Includes all records (active and inactive), sorted by company, product, container.
  - Report is held and saved in the job’s output queue for later retrieval.
- **Calculations**: Page number increment for headers.

### 7. Prompt for Valid Product or Container Code
- **Inputs**: File group ('G' or 'Z'), company (for product), field type ('product' or 'container').
- **Outputs**: Valid code (product or container), description, error message (if any).
- **Process Steps**:
  1. Apply file overrides for GSPROD (product) or GSCNTR1 (container).
  2. For product: Chain to GSPROD using company and product code, return code and description.
  3. For container: Chain to GSCNTR1 using container code, return code and description.
  4. Return error if code not found.
- **Business Rules**:
  - Ensures only valid codes are returned.
  - Product validation requires company context.
- **Calculations**: None.

## General Business Rules
- **File Overrides**: Use 'G' for GG* files or 'Z' for ZG* files to separate data (e.g., by company or environment).
- **Data Validation**: 
  - Company, product, and container must exist in BICONT, GSPROD, and GSCNTR1, respectively, for create/update/display.
  - Description1 is mandatory for create/update.
  - Inactive records (deleteFlag = 'I') cannot be updated but can be displayed or reactivated.
- **Error Handling**: Return error messages for invalid inputs (e.g., non-existent keys, blank description1, duplicate records).
- **Auditing**: No user or timestamp updates in the database; relies on file-level auditing (if any).
- **Concurrency**: File overrides use SHARE(*NO) to prevent conflicts.

## Assumptions
- Input parameters are provided by a calling system (no interactive screens).
- GSPRCT contains fields: ptcono (company), ptprod (product), ptcntr (container), ptdes1 (description1), ptdes2 (description2), ptdel (delete flag: 'A' or 'I').
- BICONT, GSPROD, and GSCNTR1 provide reference data for validation.
- QSYSPRT is configured for report output.

</xaiArtifact>