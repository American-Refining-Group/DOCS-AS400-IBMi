### Process Steps of the OCL Program (BB170.ocl36.txt)

This Operation Control Language (OCL) procedure, BB170, is part of an IBM System/36 order management system and is called by the main OCL procedure (BB170P.ocl36.txt) after the RPG program BB170P validates user inputs for creating new orders by copying an existing one. The BB170 procedure handles order batch selection, file creation, and copying of order records. It uses parameter substitution (e.g., ?9? for a prefix, ?13? for order type, ?USER?, ?WS?) to make it reusable across datasets and contexts. The procedure also supports batch deletion and cleanup, with conditional logic for cancellation and iteration. Below is a step-by-step breakdown of the process:

1. **Set Local Data Area Values**:
   - Set offset 470 to ?13? (order type, e.g., 'PP' or 'PM').
   - Set offset 480 to ?9? (prefix for file labels, e.g., 'TEST').
   - Set offset 494 to ?USER? (user ID of the job).
   - Set offset 502 to ?WS? (workstation ID).
   - Set offset 504 to 'O' (possibly an order processing flag).
   - Conditionally set offset 60 (screen title) based on ?13?:
     - If blank, set to '*         ORDER ENTRY         *'.
     - If 'PP', set to '*     VISCOSITY ASN ENTRY     *'.
     - If 'PM', set to '*      PRODUCT MOVES ENTRY    *'.
   - These settings prepare metadata for the job, tailoring the process to the order type and user context.

2. **Set Switch for Batch Deletion**:
   - Set switch 1 to 0 (SWITCH 0XXXXXXX), disabling batch deletion by default. Switch 1 is later checked for deletion logic.

3. **Load and Run BB001 Program**:
   - Load the RPG program BB001.
   - Define two logical files, BBBTCH and BBBTCHX, both pointing to the same physical file ?9?BBBTCH (e.g., TESTBBBTCH), opened in shared mode (DISP-SHR).
   - Execute BB001, likely to select or validate an order batch, as the comment indicates "ORDER BATCH SELECTION".

4. **Check for Cancellation**:
   - If the local data area at offset 121 (6 characters) equals '/CANCEL' (set by BB170P if user cancels), return immediately, terminating the procedure.

5. **Evaluate Batch Suffix (P20)**:
   - Set variable P20 to the 2-character value from local data area offset 490 (likely a batch identifier, e.g., '01' or 'AA').
   - This suffix is used to construct file labels for order-related files (e.g., ?9?BBOR?20? becomes TESTBBOR01).

6. **Batch Deletion Logic (if SWITCH1=1)**:
   - If switch 1 is on (indicating a delete request):
     - Call BB215 with *ALL parameters (likely clears batch-related data).
     - Call BB003 with *ALL parameters (likely additional cleanup or logging).
     - If file ?9?BBOR?20? exists (e.g., TESTBBOR01), delete it (DELETE command, F1 for file type).
     - If file ?9?BBOX?20? exists (e.g., TESTBBOX01), delete it.
     - Reset program BB101 with *ALL parameters (likely resets counters or batch state).

7. **Create Order Files (if they don’t exist)**:
   - If file ?9?BBOR?20? does not exist, create it using BLDFILE:
     - Name: ?9?BBOR?20? (e.g., TESTBBOR01).
     - Type: Indexed (I), 999000 records, 512 bytes/record, 2-byte key, 11-byte index, type DFILE.
   - If file ?9?BBOX?20? does not exist, create it similarly.
   - These files likely store order header and detail records for the new batch.

8. **Load and Run BB170 Program (Loop)**:
   - Load the RPG program BB170 (not to be confused with this OCL procedure’s name).
   - Define multiple files, all shared (DISP-SHR):
     - BBORDR (?9?BBORDR): Order file.
     - BICONT (?9?BICONT): Control/company file.
     - BBOTHS (?9?BBOTHS): Possibly another order header file.
     - BBORHS (?9?BBORHS): Likely a historical order header file.
     - BBOTDS (?9?BBOTDS): Possibly order details file.
     - BBORDS (?9?BBORDS): Another order details file.
     - SHIPTO (?9?SHIPTO): Shipping address file.
     - SHIPTOO (?9?SHIPTO): Alias for SHIPTO.
     - BBOTA (?9?BBOTA): Possibly a temporary order file.
     - BBORA (?9?BBORA): Possibly an archive order file.
     - BBTRAN (?9?BBOR?20?): Transaction/batch order file for the new orders.
   - Execute BB170 (RPG), which likely copies the validated order (from BB170P) to the new batch file (?9?BBOR?20?).

9. **Check Copy Counter and Loop**:
   - Compare local data area offset 150 (2 characters, likely actual copies made) to offset 140 (2 characters, likely requested copies, KYCPYS from BB170P).
   - If they match (all requested copies created), go to END.
   - Otherwise, go to AGAIN (step 8), looping to copy additional orders.

10. **Release Batch**:
    - Set local data area offset 475 to '?F'A,?9?BBOR?20?'?' (e.g., 'A,TESTBBOR01'), possibly a file action flag.
    - Load and run BB005, defining BBBTCH (?9?BBBTCH) in shared mode, likely to finalize or release the batch for processing.

11. **Clear Local Data Area**:
    - Blank all local data area contents, ensuring a clean state for the next job.

12. **End Procedure**:
    - Reach END tag, terminating the procedure.

The procedure is iterative (via AGAIN loop) to handle multiple order copies and includes cleanup for batch deletion and file creation for new orders.

### Business Rules

- **Dynamic File Labeling**: Uses ?9? prefix (e.g., 'TEST') and ?20? suffix (batch ID) to create unique file names, ensuring data separation across companies or datasets.
- **Order Type Customization**: Sets screen titles based on ?13? (order type: blank, 'PP' for viscosity ASN, 'PM' for product moves), tailoring the process to specific order workflows.
- **Cancellation Handling**: Checks for '/CANCEL' in local data area (set by BB170P), allowing immediate exit if user cancels.
- **Batch Deletion**: If switch 1 is on, deletes batch files (?9?BBOR?20?, ?9?BBOX?20?) and resets related data via BB215, BB003, BB101, ensuring no orphaned data.
- **File Creation**: Creates batch-specific order files (?9?BBOR?20?, ?9?BBOX?20?) if they don’t exist, with fixed specs (999000 records, 512 bytes, indexed), ensuring capacity for new orders.
- **Copy Control**: Loops until the number of copies made (offset 150) matches requested copies (offset 140, KYCPYS), ensuring all requested new orders are created.
- **Batch Finalization**: BB005 releases the batch, likely marking it ready for further processing (e.g., posting or shipping).
- **Data Integrity**: Uses shared file access (DISP-SHR) to allow concurrent jobs while preventing conflicts. Files are predefined to match expected formats.
- **User Context**: Stores user ID (?USER?) and workstation ID (?WS?) in the local data area, likely for audit or locking purposes.
- **Error Prevention**: Conditional file creation and deletion prevent errors from missing or existing files.

### Tables Used

In System/36, "tables" refer to disk files used as databases. The procedure defines:

- **BBBTCH** (?9?BBBTCH): Batch control file, used by BB001 and BB005.
- **BBORDR** (?9?BBORDR): Order file (likely headers), used by BB170.
- **BICONT** (?9?BICONT): Control/company file, used by BB170.
- **BBOTHS** (?9?BBOTHS): Possibly another order header file.
- **BBORHS** (?9?BBORHS): Likely historical order headers.
- **BBOTDS** (?9?BBOTDS): Possibly order details.
- **BBORDS** (?9?BBORDS): Another order details file.
- **SHIPTO** (?9?SHIPTO): Shipping address file (also aliased as SHIPTOO).
- **BBOTA** (?9?BBOTA): Possibly temporary order file.
- **BBORA** (?9?BBORA): Possibly archive order file.
- **BBTRAN** (?9?BBOR?20?): Transaction/batch order file for new orders.
- **?9?BBOR?20?** (e.g., TESTBBOR01): Dynamically created order file for the batch.
- **?9?BBOX?20?** (e.g., TESTBBOX01): Dynamically created auxiliary order file (possibly details or extensions).

All files are opened in shared mode (DISP-SHR).

### External Programs Called

- **BB001**: RPG program for batch selection/validation.
- **BB215**: Called during batch deletion, likely clears batch data.
- **BB003**: Called during batch deletion, possibly for additional cleanup.
- **BB101**: Reset during batch deletion, likely resets counters or state.
- **BB170**: RPG program to copy order records to the new batch.
- **BB005**: RPG program to release/finalize the batch.

These programs handle specific tasks in the order copying and batch management process.