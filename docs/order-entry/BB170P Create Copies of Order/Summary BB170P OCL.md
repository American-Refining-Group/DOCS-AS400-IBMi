### List of Use Cases Implemented in the Call Stack

The call stack consists of OCL and RPG programs (BB170P.ocl36.txt, BB170.rpg36.txt, BB001.rpg36.txt, BB215.rpg36.txt, BB003.rpg36.txt, BB101.rpgle.txt, BB005.rpg36.txt) that together form an order management system on the IBM System/36. The programs handle the creation, validation, and deletion of order batches, focusing on copying existing orders to create new ones. Below are the identified use cases based on the functionality of the programs:

1. **Create New Orders by Copying an Existing Order**:
   - **Description**: Allows users to select an existing order, validate its details, and create one or more new orders by copying its data into a new batch. Involves selecting a batch, validating inputs, generating new order numbers, copying order data, and releasing the batch.
   - **Programs Involved**: BB170P (OCL, initiates process), BB170 (RPG, copies order records), BB001 (RPG, batch selection), BB005 (RPG, batch release).
   - **Key Actions**: Validate company, order number, and number of copies (BB170P); select or create a batch (BB001); copy order data to new records with incremented order numbers (BB170); release or post the batch (BB005).

2. **Delete an Existing Order Batch**:
   - **Description**: Deletes an existing order batch, including associated transaction records, and resets batch state. Involves clearing locks and removing records from multiple files.
   - **Programs Involved**: BB170 (OCL, orchestrates deletion), BB215 (RPG, clears locks), BB003 (RPG, deletes transaction records), BB101 (RPG, resets batch state).
   - **Key Actions**: Set deletion switch (BB170 OCL); clear locks on order headers (BB215); delete records from transaction files (BB003); reset batch counters or state (BB101).

### Function Requirement Document: Create New Orders by Copying an Existing Order

<xaiArtifact artifact_id="3a23d19a-520b-404b-9a93-ed3ee8853830" artifact_version_id="5e4b9cd2-4792-4003-9249-85974ba5951e" title="CreateNewOrdersFunction.md" contentType="text/markdown">

# Function Requirement Document: Create New Orders by Copying an Existing Order

## Overview
This function creates one or more new orders by copying an existing order within a specified batch. It validates input parameters, assigns new order numbers, copies data to transaction and supplemental files, and updates batch status. The function operates non-interactively, taking all required inputs as parameters.

## Inputs
- **Company Number (KYCO)**: 2-character alphanumeric, identifies the company.
- **Source Order Number (KYORDR)**: 6-digit numeric, identifies the existing order to copy.
- **Number of Copies (KYCPYS)**: 2-digit numeric, number of new orders to create (1 or more).
- **Order Type (PAR13C)**: 2-character alphanumeric, specifies order type (e.g., blank, 'PP' for viscosity ASN, 'PM' for product moves).
- **User ID (USER)**: 8-character alphanumeric, identifies the user.
- **Workstation ID (WSID)**: 2-character alphanumeric, identifies the workstation.
- **Batch Number (BATCH#)**: 2-digit numeric, optional; if blank, a new batch is created.

## Outputs
- **New Order Numbers**: List of generated order numbers (6-digit numeric) for the new orders.
- **Batch Number**: 2-digit numeric, the batch containing the new orders.
- **Status**: Success or error code with message (e.g., invalid company, locked order).

## Process Steps
1. **Validate Inputs**:
   - Verify KYCO exists in BICONT (control file). If not, return error: "INVALID COMPANY NUMBER".
   - Verify KYORDR is non-zero and exists in BBORDR (order file) for KYCO. If not, return error: "INVALID ORDER NUMBER".
   - Check if the order is locked (BOLOCK non-blank in BBORDR). If locked, return error: "ORDER IS CURRENTLY IN A BATCH PLEASE POST".
   - Ensure KYCPYS is non-zero. If zero, return error: "MUST ENTER NUMBER OF NEW ORDERS".
   - Validate PAR13C (blank, 'PP', 'PM'). If invalid, return error: "INVALID TYPE".

2. **Select or Create Batch**:
   - If BATCH# is provided, chain to BBBTCH using BATCH#. If not found, deleted (ABDEL='D'), or locked (ABLOCK non-blank and ABUSER≠USER), return error: "INVALID BATCH NO." or "BATCH # IN USE".
   - If BATCH# is blank and PGM='O' (order entry mode), retrieve next batch number (ABNXTB) from BBBTCH (record '99'), increment (1 to 99, wrap to 1), and create a new batch record with:
     - ABDEL=' ', ABBTCH=new batch number, ABLOCK=' ', ABLKWS=' ', ABPRTD='N', ABUSER=USER, ABDATE=current date (MMDDYY), ABLUDT=0, ABREC=0, ABSRCE=PAR13C, ABNXTB=incremented number.
   - If PGM≠'O', return error: "CANNOT CREATE A BATCH NOW".

3. **Create Transaction Files**:
   - If files ?9?BBOR?20? and ?9?BBOX?20? (e.g., TESTBBOR01, TESTBBOX01, where ?9?=KYCO prefix, ?20?=BATCH#) do not exist, create them:
     - Indexed, 999000 records, 512 bytes, 2-byte key, 11-byte index.

4. **Copy Order Records**:
   - Chain to BICONT using KYCO to get BCORDN (next order number).
   - For each copy (1 to KYCPYS):
     - Increment BCORDN to NEWORD and update BICONT.
     - Read BBORDR record (KYCO+KYORDR). If BOTYPE='M', set PAR13C='PM'.
     - Write to BBTRAN (?9?BBOR?20?) with NEWORD, current date (MMDDYY and CCYYMMDD), and copied fields (e.g., BOCUST, BOSHIP, BORQDT).
     - Read and copy records from BBORHS, BBORDS, BBORA to BBOTHS, BBOTDS, BBOTA with NEWORD.
     - If SHIPTO record exists (CSDEL='O', CSSHIP=999), copy to SHIPTOO with NEWORD.
     - Increment copy counter (KYCNT).

5. **Update Batch Status**:
   - Update BBBTCH for BATCH#:
     - Clear ABLOCK and ABLKWS (release locks).
     - Set ABREC=KYCNT (number of orders created).
   - If all copies are created (KYCNT=KYCPYS), finalize the batch.

6. **Return Results**:
   - Return list of new order numbers (NEWORDs), BATCH#, and success status. If errors occur, return error code and message.

## Business Rules
- **Validation**:
  - Company (KYCO) must exist in BICONT.
  - Source order (KYORDR) must exist in BBORDR, not be locked, and match KYCO.
  - Number of copies (KYCPYS) must be greater than zero.
  - Order type (PAR13C) must be blank, 'PP', or 'PM'.
- **Batch Management**:
  - Existing batch must be valid, not deleted, and either unlocked or locked by the current user.
  - New batch creation requires PGM='O' and assigns a unique batch number (1-99).
  - Batch source (ABSRCE) must match PAR13C.
- **Order Numbering**:
  - New orders use BCORDN from BICONT, incremented per copy, ensuring uniqueness per company.
- **File Creation**:
  - Transaction files (?9?BBOR?20?, ?9?BBOX?20?) are created if missing, with standard specs.
- **Data Integrity**:
  - Copied orders retain key fields (e.g., customer, ship-to, dates) from the source.
  - Supplemental files (BBOTHS, BBOTDS, BBOTA, SHIPTOO) are populated for each new order.
- **Batch Finalization**:
  - Batch is released (ABLOCK=' ', ABLKWS=' ') with updated record count (ABREC).

## Calculations
- **Order Number Generation**:
  - NEWORD = BCORDN (from BICONT).
  - NXTORD = BCORDN + 1, updated in BICONT.
- **Date Handling**:
  - Current date (MMDDYY) from system time.
  - CCYYMMDD = '20' + (MMDDYY * 10000.01).
- **Copy Counter**:
  - KYCNT = KYCNT + 1 per successful copy, compared to KYCPYS.

## Error Handling
- Return specific error messages for invalid inputs (e.g., "INVALID COMPANY NUMBER", "BATCH # IN USE").
- Stop processing if any validation fails, returning the error without creating records.

</xaiArtifact>

### Function Requirement Document: Delete an Existing Order Batch

<xaiArtifact artifact_id="ba535aee-4962-46bd-8e0c-32e8ba7ac453" artifact_version_id="7e2102fc-ce1e-4e59-9866-d26fb4bb348d" title="DeleteOrderBatchFunction.md" contentType="text/markdown">

# Function Requirement Document: Delete an Existing Order Batch

## Overview
This function deletes an existing order batch, including all associated transaction records, clears locks, and resets batch state. It operates non-interactively, taking input parameters to identify the batch and company.

## Inputs
- **Company Number (KYCO)**: 2-character alphanumeric, identifies the company.
- **Batch Number (BATCH#)**: 2-digit numeric, identifies the batch to delete.
- **User ID (USER)**: 8-character alphanumeric, identifies the user.

## Outputs
- **Status**: Success or error code with message (e.g., invalid batch, batch locked).

## Process Steps
1. **Validate Inputs**:
   - Verify KYCO exists in BICONT. If not, return error: "INVALID COMPANY NUMBER".
   - Chain to BBBTCH using BATCH#. If not found or ABDEL='D', return error: "INVALID BATCH NO.".
   - If ABLOCK non-blank and ABUSER≠USER, return error: "BATCH # IN USE".

2. **Clear Locks**:
   - For each BBORTR record (transaction orders for KYCO+BATCH#):
     - Chain to BBORDRH using BOCOS (company+order).
     - If found, update BOLOCK=' ' and BOWSID=' ' (positions 155-157).

3. **Delete Transaction Records**:
   - For each BBORTR record:
     - Chain to BBORCL using BBKEY (company+customer+order).
     - If found, delete the record.
     - Read BBOTHS1, BBOTDS1, BBOTA1 sequentially, matching TCOORD (company+order).
     - Delete matching records.

4. **Reset Batch State**:
   - Update BBBTCH for BATCH#:
     - Set ABDEL='D' (mark as deleted).
     - Set ABLOCK='P' (posted).
     - Update ABREC=0 (reset record count).
   - Reset counters or state in related files (e.g., BBORDR).

5. **Delete Transaction Files**:
   - Delete files ?9?BBOR?20? and ?9?BBOX?20? (e.g., TESTBBOR01, TESTBBOX01).

6. **Return Results**:
   - Return success status or error code with message.

## Business Rules
- **Validation**:
  - Batch must exist in BBBTCH, not be deleted (ABDEL≠'D'), and be accessible (unlocked or locked by USER).
- **Lock Removal**:
  - Clear locks (BOLOCK, BOWSID) on all order headers in the batch.
- **Record Deletion**:
  - Delete all transaction records (BBORCL, BBOTHS1, BBOTDS1, BBOTA1) tied to the batch’s orders.
- **Batch Finalization**:
  - Mark batch as deleted (ABDEL='D') and posted (ABLOCK='P').
  - Reset record count (ABREC=0).
- **File Deletion**:
  - Remove batch-specific transaction files to prevent orphaned data.
- **Data Integrity**:
  - Use precise key matching (BOCOS, BBKEY, TCOORD) to delete only relevant records.

## Calculations
- None. The function focuses on record deletion and status updates, with no numeric calculations.

## Error Handling
- Return specific error messages for invalid inputs (e.g., "INVALID BATCH NO.", "BATCH # IN USE").
- Stop processing if validation fails, ensuring no partial deletions.

</xaiArtifact>