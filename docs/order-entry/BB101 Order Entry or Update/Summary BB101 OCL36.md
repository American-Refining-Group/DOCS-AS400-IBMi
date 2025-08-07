### List of Use Cases Implemented by the BB101 Program

The `BB101` program (comprising the OCL program `BB101.ocl36.txt` and the RPGLE program `BB101.rpgle.txt`) is designed to handle the order entry process within an IBM AS/400 (iSeries) system. Based on the provided code and its functionality, the primary use case implemented is:

1. **Order Entry and Management**:
   - This use case encompasses the creation, updating, validation, and finalization of sales orders, including header, detail, and miscellaneous line items. It involves batch selection, order deletion, freight and tax calculations, and integration with various files and external programs to ensure data accuracy and compliance with business rules.

### Function Requirement Document: Order Entry and Management

<xaiArtifact artifact_id="0211796a-5e0a-478f-8234-97b13010ad55" artifact_version_id="6dcc1fe9-e126-493d-a853-3e401e91ce34" title="Order_Entry_Requirements.md" contentType="text/markdown">

# Order Entry and Management Function Requirements

## Overview
The Order Entry and Management function facilitates the creation, validation, updating, and finalization of sales orders in an IBM AS/400 system. It handles order header details, line items, miscellaneous charges, freight calculations, and tax overrides, ensuring compliance with business rules and integration with external data sources.

## Inputs
- **Company Number**: Identifies the company (e.g., `CO`).
- **Order Number**: Unique identifier for the order (6 digits, resets to 100001 at 899999).
- **Batch Number**: Identifier for the batch of orders.
- **Customer Number**: Identifies the customer (`CUST`).
- **Ship-to Number**: Specifies the delivery location (`SHIP`).
- **Order Date, PO Date, Requested Delivery Date**: Dates for order processing (`ORDT`, `PORD`, `RQDT`).
- **Freight Code**: Indicates freight terms (`FRCD`, e.g., `C`, `P`, `A`, `CYY`).
- **Carrier ID**: Specifies the carrier (`SCAID`, defaults to `BPRR` for railcar orders).
- **Product Code, Container Code, Quantity, Unit Price**: Line item details (`PROD`, `CNTR`, `QTY`, price).
- **Miscellaneous Line Details**: Includes surcharges or fees (`MXLINE`, sequence numbers).
- **Tax Codes and Amounts**: Up to 10 tax fields (`TXC`, `TXE`, `TXA`).
- **Remarks**: Order, invoice, and bill of lading remarks (`OMK`, `IMK`, `BMK`).
- **Customer-Owned Product Flag**: Indicates if the order is for customer-owned product (`COON`, `Y` or blank).
- **Hand Ticket Number**: Unique identifier for manual tickets (`HT#`).
- **Incoterms and Country Code**: For export orders (`INCT`, `CSCTY`).

## Process Steps
1. **Initialize and Validate Inputs**:
   - Validate company, customer, and ship-to numbers against master files (`ARCUST`, `SHIPTO`).
   - Ensure order, PO, and delivery dates are valid and not more than 30 days past or on holidays (unless overridden).
   - Verify carrier ID is active (not `CC`, `CT`, or `SCO` for new orders) and bill-to PO# is provided.

2. **Order Header Processing**:
   - Create or update order header in `BBORTR` with customer, ship-to, dates, freight code, and carrier ID.
   - Check for credit hold (bypassed if `COON='Y'`): Validate against credit limit, overdue invoices, or zero credit limit.
   - For export orders, validate Incoterms and country code.

3. **Order Line Item Processing**:
   - Validate product code (`PROD`) against `GSPROD`, container code (`CNTR`) against `GSCNTR1`, and unit of measure against `GSCTWT`.
   - Ensure quantity and price are non-zero for non-multi-load orders.
   - For customer-owned products (`COON='Y'`):
     - Bypass inventory checks (`INTANK`).
     - Use `GSTABL-PRODCD` for sales G/L number.
     - Force no-charge code (`Y`) and zero out miscellaneous quantities/amounts.
   - Validate freight codes are consistent across line items (`C`, `P`, `A`, or `L` for railcars).

4. **Miscellaneous Line Processing**:
   - Process miscellaneous charges (e.g., freight surcharges, service fees).
   - Hide sequence numbers 940+ (collect freight service charges) and 950 (auto-added freight surcharges).
   - Validate miscellaneous G/L numbers and ensure zero quantities/amounts for customer-owned products.

5. **Freight and Pricing Calculations**:
   - Calculate freight based on freight code:
     - Freight Collect (`C`): Customer price = 1.00, Product price = 1.00, Freight price = 0.00.
     - Freight PPD&ADD (`A`): Customer price = 1.00, Product price = 1.00, Freight price = 0.25.
     - Freight Prepaid with Rack Price: Customer price = 1.00, Product price = 1.00, Freight price = 0.25.
     - Freight Prepaid with Sales Agreement: Customer price = 1.00, Product price = 0.75, Freight price = 0.25.
     - Freight Collect with Service Fee (`CYY`): Includes $100 service fee, freight billed by carrier.
   - If carrier ID is blank, auto-populate with the first of the top 5 carrier preferences.
   - Issue warnings if freight calculation is requested but none is calculated.

6. **Tax Processing**:
   - Support up to 10 tax codes, exemptions, and amounts.
   - Allow manual tax overrides and write to `BBTRTX`.

7. **Quantity and Weight Calculations**:
   - Calculate gross gallons (`GGAL`), net gallons (`NGAL`), shipping weight (`GWT`), quantity in unit measure (`QTUM`), and product weight (`PRWT`) using `GETSLB` subroutine.
   - Update `BBORTR` and `BBOTDS1` with calculated values.
   - Ensure total load volume does not exceed 7/0.

8. **Order Finalization**:
   - Validate no detail lines are marked with override code `X`.
   - Check for duplicate orders using `BB115` and hand ticket numbers (including customer number, ignoring deleted orders).
   - Update supplemental files (`BBOTHS1`, `BBOTDS1`) and remarks.
   - Release the batch using `BB005` after final validation.

## Business Rules
- **Order Validation**:
  - Company, customer, and ship-to must exist in master files.
  - Dates must be valid, not holidays (unless overridden with `F10`), and within 30 days.
  - Carrier ID must be active; bill-to PO# is mandatory.
  - Group by must be `S` (size) or `C` (category), defaulting to `S`.
- **Customer-Owned Product**:
  - Bypasses inventory and credit checks.
  - Forces no-charge code and zeroes miscellaneous quantities/amounts.
- **Freight and Pricing**:
  - Consistent freight codes across line items.
  - Railcar orders prohibit multi-load and require `C` or `L` freight codes.
  - Inactive rack prices (`I`) block orders; `B` allows orders until stock depletion (override with `F11`).
- **Duplicate Checks**:
  - Prevent duplicate orders and hand ticket numbers.
- **Taxes**:
  - Support up to 10 tax fields with override capability.
- **Order Completion**:
  - No detail lines with override code `X` allowed.
  - Multi-load orders require non-zero quantities and consistent freight codes.

## Outputs
- **Order Records**: Updated `BBORTR`, `BBTRTX`, `BBOTHS1`, `BBOTDS1` with header, detail, and supplemental data.
- **Batch Release**: Finalized batch released via `BB005`.
- **Error/Warning Messages**: For invalid inputs, credit holds, holiday dates, or missing calculations.
- **Order Totals**: Product, miscellaneous, freight, and overall totals (`PTOT`, `MTOT`, `FTOT`, `OTOT`).

## Dependencies
- **Files**: `BBORTR`, `BBTRTX`, `BBOTHS1`, `BBOTDS1`, `BBORDR`, `BICONT`, `SHIPTO`, `GSPROD`, `GSCNTR1`, `BBCAID`, `GSCTWT`, `GSCTUM`, `GSUMCV`, `GSTABL`, `ARCUST`, `ARCUSX`, `BBSHSA`, `BBPRXR`, `GLMAST`, `GSMLCD`, `ARCUPR`, `BICUAX`.
- **Programs**: `BB1011`, `BB1014`, `BB1015`, `BB1018`, `BB101F`, `BB106`, `BB1033`, `BB115`, `BB104A`, `BB811`, `AR822R`, `MBBQTY`, `IN805`, `LCSTSHP`, `MSHPADR`, `MGSTABL`, `BB005`.

</xaiArtifact>










### List of Use Cases Implemented by the BB001 Program

The `BB001.rpg36.txt` program, called by the `BB101.ocl36.txt` OCL program, is a component of the order entry system on an IBM AS/400 (iSeries) platform. It focuses on batch management for order entry. Based on the provided code, the primary use case implemented is:

1. **Batch Management for Order Entry**:
   - This use case involves creating, selecting, updating, or deleting batches of orders stored in the `BBBTCH` file. It ensures proper batch number assignment, user locking, and validation for order entry processes, supporting modes like standard order entry or viscosity entry ('PP').

### Function Requirement Document: Batch Management for Order Entry

<xaiArtifact artifact_id="78a6a9e7-f5c5-4704-90cc-caeedb96f909" artifact_version_id="aa67d1d1-9c45-4696-a325-f7082baba938" title="Batch_Management_Requirements.md" contentType="text/markdown">

# Batch Management for Order Entry Function Requirements

## Overview
The Batch Management for Order Entry function manages the creation, selection, updating, and deletion of order batches in an IBM AS/400 system. It assigns and validates batch numbers, enforces user-specific locking, and ensures compatibility with the order entry mode (e.g., standard or viscosity entry).

## Inputs
- **Company Number**: Identifies the company (`CO`).
- **Batch Number**: Numeric identifier for the batch (1-99, optional input for selection, `BTCH#X`).
- **Delete Flag**: Indicates if the batch should be deleted (`DEL`, 'D' or blank).
- **Program Mode**: Specifies the mode (`PGM`, e.g., 'O' for order entry, 'P' for posting).
- **Batch Source**: Indicates batch type (`PAR13C`, e.g., 'PP' for viscosity entry).
- **User ID**: Identifies the user (`USER`).
- **Workstation ID**: Identifies the workstation (`WSID`).
- **Current Date**: System date for creation/update (`UDATE`).

## Process Steps
1. **Initialize Batch Environment**:
   - Clear existing batch data and set file cursor to the start of `BBBTCH`.
   - Determine program mode (`PGM`) to restrict actions (e.g., creation allowed only if `PGM='O'`).

2. **Create New Batch**:
   - If no batch number is provided (`BTCH#X=0`):
     - Retrieve the next batch number (`ABNXTB`) from `BBBTCHX` (record '99').
     - Increment `ABNXTB` (reset to 1 if 99).
     - Assign batch fields:
       - Batch number (`BATCH#`, 1-99).
       - Lock status (`ABLOCK='O'`), workstation ID (`ABLKWS=WSID`), user ID (`ABUSER=USER`).
       - Creation date (`ABDATE=UDATE`), last update date (`ABLUDT=0`), record count (`#REC=0`).
       - Batch source (`ABSRCE=PAR13C`, e.g., 'PP').
       - BOL printed flag (`ABPRTD='N'`).
     - Write new batch record to `BBBTCHX`.

3. **Select Existing Batch**:
   - For a provided batch number (`BTCH#X≠0`):
     - Retrieve batch record from `BBBTCHX`.
     - Validate batch is not deleted (`ABDEL≠'D'`) and batch source matches (`ABSRCE=PAR13C`).
     - Check lock status:
       - If locked (`ABLOCK≠' '`) and not by the current user (`ABUSER≠USER`), reject access.
       - If locked by the current user, allow access.
     - Populate batch details (lock status, user, dates, record count).

4. **Update Batch**:
   - Update batch record with current user (`ABUSER`), workstation ID (`ABLKWS`), last update date (`ABLUDT=UDATE`), and record count (`#REC`).
   - Preserve batch source (`ABSRCE`) and BOL printed flag (`ABPRTD`).

5. **Delete Batch**:
   - If delete flag is set (`DEL='D'`), mark the batch as deleted (`ABDEL='D'`) in `BBBTCHX`.
   - Confirm deletion before finalizing.

6. **Batch Navigation**:
   - Retrieve up to 10 batch records for selection, skipping deleted batches (`ABDEL='D'`) or those with mismatched source (`ABSRCE≠PAR13C`).
   - Assign lock descriptions (e.g., 'AVAILABLE', 'ORD ENTRY', 'PICK LIST', 'POSTING', 'BOL PRINT') based on `ABLOCK`.

## Business Rules
- **Batch Number**:
  - Must be 1-99, incrementing from the last used number, resetting to 1 after 99.
  - New batch creation is allowed only in order entry mode (`PGM='O'`).
- **Batch Source**:
  - Must match the program mode (`PAR13C`, e.g., 'PP' for viscosity entry).
- **Locking**:
  - Only the user who locked the batch (`ABUSER=USER`) can access it if `ABLOCK≠' '`.
  - Locked batches are inaccessible to other users until released.
- **Deletion**:
  - Batches can be marked for deletion with confirmation.
  - Deleted batches are skipped during selection.
- **Validation**:
  - Invalid or deleted batch numbers are rejected.
  - Non-order entry modes (`PGM≠'O'`) prevent batch creation.
- **Tracking**:
  - Record user ID, workstation ID, creation date, last update date, and order count for each batch.

## Outputs
- **Batch Record**: Created or updated record in `BBBTCHX` with batch number, lock status, user, dates, source, and record count.
- **Error Messages**: For invalid batch numbers, locked batches, or mode restrictions (e.g., "INVALID BATCH NO.", "BATCH # IN USE").
- **Batch List**: Up to 10 batch records for selection with details (batch number, lock status, user, dates, record count).

## Dependencies
- **Files**: `BBBTCH` (batch master file, read-only), `BBBTCHX` (batch file, update-capable).
- **Programs**: None called directly.

</xaiArtifact>










### List of Use Cases Implemented by the BB003 Program

The `BB003.rpg36.txt` program, called by the `BB101.ocl36.txt` OCL program, is part of the order entry system on an IBM AS/400 (iSeries) platform. It focuses on cleaning up transaction records associated with a deleted batch. Based on the provided code, the primary use case implemented is:

1. **Batch Transaction Record Deletion**:
   - This use case involves deleting records from order transaction files (`BBORCL`, `BBOTHS1`, `BBOTDS1`, `BBOTA1`) when a batch is marked for deletion, ensuring data consistency by removing all associated records for a given company and order number.

### Function Requirement Document: Batch Transaction Record Deletion

<xaiArtifact artifact_id="015529da-9ecb-451f-ac65-467ab782f79a" artifact_version_id="ae832476-da92-42fc-8862-fdfa107e41cc" title="Batch_Transaction_Deletion_Requirements.md" contentType="text/markdown">

# Batch Transaction Record Deletion Function Requirements

## Overview
The Batch Transaction Record Deletion function removes records from order transaction files (`BBORCL`, `BBOTHS1`, `BBOTDS1`, `BBOTA1`) associated with a deleted batch in an IBM AS/400 system. It ensures data consistency by deleting all records linked to a specific company and order number.

## Inputs
- **Company Number**: Identifies the company (`TDCO`, 2 characters).
- **Order Number**: Unique identifier for the order (`TDORD#`, 6 characters).
- **Customer Number**: Identifies the customer (`TDCUST`, 6 characters).
- **Transaction Key**: Composite key (`Tcoord`, company and order number combined, 8 characters).

## Process Steps
1. **Initialize Deletion Process**:
   - Construct a composite key (`BBKEY`) from `TDCO` (company), `TDCUST` (customer), and `TDORD#` (order) for `BBORCL` lookups.
   - Extract `Tcoord` (company and order number) from `BBORTR` input for matching records in other files.

2. **Delete BBORCL Records**:
   - Retrieve and delete records from `BBORCL` where the key (`BBKEY`) matches the input company and order number.

3. **Delete BBOTHS1 Records**:
   - Set file cursor to records in `BBOTHS1` matching `Tcoord` (company/order).
   - Read and delete each record where `BOCOOR` (company/order) matches `Tcoord`.

4. **Delete BBOTDS1 Records**:
   - Set file cursor to records in `BBOTDS1` matching `Tcoord`.
   - Read and delete each record where `BDCOOR` (company/order) matches `Tcoord`.

5. **Delete BBOTA1 Records**:
   - Set file cursor to records in `BBOTA1` matching `Tcoord`.
   - Read and delete each record where `BACOOR` (company/order) matches `Tcoord`.

6. **Completion**:
   - Continue deletion until all matching records in each file are removed.

## Business Rules
- **Record Matching**:
  - Delete records only when the company and order number (`Tcoord` or `BBKEY`) match the input from `BBORTR`.
- **File Access**:
  - `BBORTR` is read-only; `BBORCL`, `BBOTHS1`, `BBOTDS1`, and `BBOTA1` are update-capable for deletion.
- **Deletion Scope**:
  - All records associated with the input company and order number are deleted to ensure no orphaned data remains.
- **Error Handling**:
  - Skip deletion for files where no matching records are found, proceeding to the next file.

## Outputs
- **Deleted Records**: Removed records from `BBORCL`, `BBOTHS1`, `BBOTDS1`, and `BBOTA1`.
- **No Output Data**: The function does not produce output data beyond file modifications.

## Dependencies
- **Files**:
  - `BBORTR`: Input order transaction file.
  - `BBORCL`: Order transaction file for deletion.
  - `BBOTHS1`: Order header supplemental file for deletion.
  - `BBOTDS1`: Order detail supplemental file for deletion.
  - `BBOTA1`: Order accessorial file for deletion.
- **Programs**: None called directly.

</xaiArtifact>










### List of Use Cases Implemented by the BB005 Program

The `BB005.rpg36.txt` program, called by the `BB101.ocl36.txt` OCL program, is part of the order entry system on an IBM AS/400 (iSeries) platform. It focuses on updating the status of order batches to reflect their processing stage. Based on the provided code, the primary use case implemented is:

1. **Batch Status Update for Order Processing**:
   - This use case involves updating the status of an order batch in the `BBBTCH` file to release, mark for pick list, mark as bill of lading printed, or post the batch, based on the program mode, ensuring proper progression through the order entry workflow.

### Function Requirement Document: Batch Status Update for Order Processing

<xaiArtifact artifact_id="a71b365d-9f06-49c4-8d55-a3c62b493593" artifact_version_id="4deac554-5d2b-49ff-9706-ed01cf4fbd90" title="Batch_Status_Update_Requirements.md" contentType="text/markdown">

# Batch Status Update for Order Processing Function Requirements

## Overview
The Batch Status Update for Order Processing function updates the status of an order batch in an IBM AS/400 system to reflect its processing stage (release, pick list, bill of lading printed, or posted). It ensures batches progress correctly through the order entry workflow by updating lock status, record count, and other flags.

## Inputs
- **Batch Number**: Numeric identifier for the batch (1-99, `BATCH#`).
- **Program Mode**: Specifies the processing mode (`PGM`, e.g., 'O' for order entry, 'L' for pick list, 'B' for bill of lading, 'P' for posting).
- **Record Count**: Number of orders in the batch (`RECCNT`, 8 digits).

## Process Steps
1. **Validate Batch Existence**:
   - Retrieve the batch record from `BBBTCH` using the batch number (`BATCH#`).
   - If no record exists, terminate processing.

2. **Update Batch Status**:
   - Based on the program mode (`PGM`):
     - **Order Entry (`O`)**:
       - Clear lock status (`ABLOCK=' '`) and workstation ID (`ABLKWS=' '`) to release the batch.
       - Update record count (`RECCNT`).
     - **Pick List (`L`)**:
       - Clear lock status and workstation ID to mark the batch for pick list processing.
       - Update record count.
     - **Bill of Lading (`B`)**:
       - Clear lock status and workstation ID.
       - Set BOL printed flag (`ABPRTD='Y'`).
       - Update record count.
     - **Posting (`P`)**:
       - Mark batch as deleted (`ABDEL='D'`).
       - Set lock status to posted (`ABLOCK='P'`).
       - Update record count.
     - **Default (Other Modes)**:
       - Release the batch (clear lock status and workstation ID).
       - Update record count.

3. **Save Batch Record**:
   - Write updated batch record to `BBBTCH` with new status, lock, and count values.

## Business Rules
- **Batch Status**:
  - Status updates depend on the program mode:
    - `O`: Release batch for further processing.
    - `L`: Mark for pick list generation.
    - `B`: Indicate bill of lading printed.
    - `P`: Post batch and mark as deleted.
  - Unrecognized modes default to releasing the batch.
- **Lock Management**:
  - Clear locks (`ABLOCK`, `ABLKWS`) for modes `O`, `L`, and `B` to allow further processing.
  - Set lock to `P` for posting mode.
- **Deletion**:
  - Only posting mode (`P`) marks the batch as deleted (`ABDEL='D'`).
- **Record Count**:
  - Update `RECCNT` to reflect the current number of orders in the batch for all modes.
- **Validation**:
  - Batch must exist in `BBBTCH`; non-existent batches are skipped.

## Outputs
- **Updated Batch Record**: Modified `BBBTCH` record with updated lock status, BOL printed flag, deletion flag, and record count.
- **No Error Messages**: The function assumes valid inputs and does not produce explicit error outputs.

## Dependencies
- **Files**: `BBBTCH` (batch master file, update-capable).
- **Programs**: None called directly.

</xaiArtifact>










### List of Use Cases Implemented by the BB215 Program

The `BB215.rpg36.txt` program, called by the `BB101.ocl36.txt` OCL program, is part of the order entry system on an IBM AS/400 (iSeries) platform. It focuses on removing lockouts from order header records. Based on the provided code, the primary use case implemented is:

1. **Order Lockout Removal**:
   - This use case involves clearing lock status and workstation ID fields in order header records (`BBORDRH`) for orders identified in transaction records (`BBORTR`), ensuring orders are accessible for further processing after batch deletion or other lock-inducing activities.

### Function Requirement Document: Order Lockout Removal

<xaiArtifact artifact_id="74aa4c93-6e7b-4e77-8c53-1b36bdca94d6" artifact_version_id="7d67bdc3-afc3-4276-b0a4-5b71d829fc82" title="Order_Lockout_Removal_Requirements.md" contentType="text/markdown">

# Order Lockout Removal Function Requirements

## Overview
The Order Lockout Removal function clears lock status and workstation ID fields in order header records in an IBM AS/400 system, ensuring orders are unlocked for further processing after batch deletion or other lock-inducing activities.

## Inputs
- **Company Number**: Identifies the company (`TDCO`, 2 characters, part of `BOCOS`).
- **Order Number**: Unique identifier for the order (`TDORD#`, 6 characters, part of `BOCOS`).

## Process Steps
1. **Initialize Processing**:
   - Set default values for lock status (`LOCK=' '`) and workstation ID (`WSID=' '`) once at program start.

2. **Process Transaction Records**:
   - Read each order transaction record from `BBORTR`.
   - Extract company and order number (`BOCOS`) from the record.

3. **Update Order Header**:
   - Retrieve the corresponding order header record in `BBORDRH` using `BOCOS` (company/order).
   - If found, update:
     - Lock status (`LOCK`) to blank.
     - Workstation ID (`WSID`) to blank.
   - If not found, skip to the next transaction record.

4. **Completion**:
   - Process all `BBORTR` records until the end of the file.

## Business Rules
- **Lock Removal**:
  - Clear `LOCK` and `WSID` fields in `BBORDRH` for records matching `BOCOS` from `BBORTR`.
- **Record Matching**:
  - Update only `BBORDRH` records with a valid company and order number match.
- **File Access**:
  - `BBORTR` is read-only; `BBORDRH` is update-capable.
- **Validation**:
  - Skip updates for non-existent `BBORDRH` records without error reporting.

## Outputs
- **Updated Order Headers**: Modified `BBORDRH` records with cleared `LOCK` and `WSID` fields.
- **No Error Messages**: The function assumes valid inputs and does not produce explicit error outputs.

## Dependencies
- **Files**:
  - `BBORTR`: Order transaction file (input).
  - `BBORDRH`: Order header file (update-capable).
- **Programs**: None called directly.

</xaiArtifact>










### List of Use Cases Implemented by the IN805BC Program

The `IN805BC.clp.txt` program, called by the `BB101.ocl36.txt` OCL program, is part of the order entry system on an IBM AS/400 (iSeries) platform. It focuses on preparing work files for inventory processing. Based on the provided code, the primary use case implemented is:

1. **Work File Preparation for Tank Total Accumulation**:
   - This use case involves creating and populating temporary work files (`INWZHW` and `INWZ10W`) in the `QTEMP` library by copying data from S/36 files based on file group and company number, enabling tank total accumulation for inventory processing.

### Function Requirement Document: Work File Preparation for Tank Total Accumulation

<xaiArtifact artifact_id="b16b1775-39a1-4127-b169-e630d1c24dda" artifact_version_id="aab6b5b7-bdda-47c1-9069-5f6f87b21bf9" title="Work_File_Preparation_Requirements.md" contentType="text/markdown">

# Work File Preparation for Tank Total Accumulation Function Requirements

## Overview
The Work File Preparation for Tank Total Accumulation function creates and populates temporary work files (`INWZHW`, `INWZ10W`) in the `QTEMP` library to accumulate tank totals for inventory processing in an IBM AS/400 system. It copies data from S/36 files based on file group and company number.

## Inputs
- **File Group**: Identifier for the file group (`P$FGRP`, 1 character, e.g., 'G' for production).
- **Company Number**: Identifier for the company (`P$CO#`, 2 characters).

## Process Steps
1. **Construct File Names**:
   - Create source file names:
     - `WRKFIL1` = `P$FGRP` + 'INWZH' (e.g., 'GINWZH').
     - `WRKFIL2` = `P$FGRP` + 'INWZ' + `P$CO#` (e.g., 'GINWZ10').

2. **Clear or Create INWZHW**:
   - Clear `QTEMP/INWZHW`.
   - If not found:
     - For `P$FGRP='G'`, duplicate `INWZHW` from `DATA` library.
     - For `P$FGRP≠'G'`, duplicate from `DATADEV` library.
     - Create without constraints or triggers.

3. **Clear or Create INWZ10W**:
   - Clear `QTEMP/INWZ10W`.
   - If not found:
     - For `P$FGRP='G'`, duplicate `INWZ10W` from `DATA` library.
     - For `P$FGRP≠'G'`, duplicate from `DATADEV` library.
     - Create without constraints or triggers.

4. **Copy Data to Work Files**:
   - Copy data from `WRKFIL1` (e.g., `GINWZH`) to `QTEMP/INWZHW`, replacing existing records without field validation.
   - Copy data from `WRKFIL2` (e.g., `GINWZ10`) to `QTEMP/INWZ10W`, replacing existing records without field validation.

## Business Rules
- **File Naming**:
  - Source file names are dynamically constructed using `P$FGRP` and `P$CO#`.
- **Environment Selection**:
  - Production (`P$FGRP='G'`) uses `DATA` library; development/testing uses `DATADEV`.
- **File Management**:
  - Work files are created in `QTEMP` for job-specific temporary storage.
  - Existing files are cleared; non-existent files are created as duplicates.
- **Data Copying**:
  - Data is copied with replacement and no field validation to ensure S/36 compatibility.
- **Error Handling**:
  - Ignore errors for file not found, object creation, or data copy to ensure process completion.

## Outputs
- **Populated Work Files**: `QTEMP/INWZHW` and `QTEMP/INWZ10W` with copied data for tank total accumulation.
- **No Error Messages**: Errors are suppressed to allow processing to continue.

## Dependencies
- **Files**:
  - `INWZHW`: Temporary work file in `QTEMP`.
  - `INWZ10W`: Temporary work file in `QTEMP`.
  - Source files (e.g., `GINWZH`, `GINWZ10`) in `DATA` or `DATADEV`.
- **Programs**: None called directly.

</xaiArtifact>