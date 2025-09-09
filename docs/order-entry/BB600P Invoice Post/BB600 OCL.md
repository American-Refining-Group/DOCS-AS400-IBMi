The `BB600.ocl36.txt` OCL program is a comprehensive Operation Control Language (OCL) script for an IBM System/36 or AS/400 environment, serving as the main control program for the invoice posting workflow. It orchestrates a series of steps to process invoice batches, manage EDI (Electronic Data Interchange) files, print registers, post sales analysis transactions, and clean up temporary files. Below is a detailed explanation of the process steps, business rules, external programs called, and tables (files) used.

---

### Process Steps of the BB600 OCL Program

The `BB600.ocl36.txt` OCL program coordinates multiple tasks in the invoice posting process, invoking several RPG and OCL programs, managing EDI files, and performing file maintenance. The steps are as follows:

1. **Wait for EDISNDCK Process**:
   - The program checks if the `EDISNDCK` job is active (`IF ACTIVE-'EDISNDCK'`).
   - If active, it waits for 15 seconds (`WAIT INTERVAL-000015`) and loops back to the `TRYAGN` tag to check again.
   - This ensures that no conflicting EDI send processes are running before proceeding.

2. **Create JEREMY Table for Order Sync Protection**:
   - The `// IFF DATAF1-?9?JEREMY BLDFILE ?9?JEREMY,S,RECORDS,999000,10,,T,,,NDFILE` statement checks if the `JEREMY` file exists. If it does not, it creates a temporary file (`?9?JEREMY`) with a capacity of 999,000 records, 10 bytes per record, and type `T` (temporary).
   - This file is used to prevent order synchronization during invoice posting, as noted in the comments.

3. **Invoke BB630 for Sales Journal Update**:
   - The `// BB630 ,,,,,,,,?9?` statement calls the `BB630` OCL program (likely `BB630.ocl36.txt`), which executes the `BB630` RPG program to update the Sales Journal number (`ACSLJ#`) in the `ARCONT` file for company number `10`.

4. **Set System Attributes and Date Handling**:
   - The `// GSY2K` statement enables Year 2000-compliant date handling for proper processing of two-digit year formats.
   - The `// ATTR CANCEL-NO,INQUIRY-YES` statement sets the job attributes to prevent cancellation (`CANCEL-NO`) and allow inquiry (`INQUIRY-YES`).

5. **Process EDI 810 Documents**:
   - **Check for EDI 810 Data**:
     - The `// IFF DATAF1-?9?EDOT?20? GOTO JUMP` statement checks if the batch-specific EDI output file (`?9?EDOT?20?`) exists. If not, it skips to the `JUMP` tag, bypassing EDI 810 processing.
   - **Run BB513**:
     - The `// LOAD BB513` block loads and executes the `BB513` RPG program, opening:
       - `EDIOUT` (`?9?EDOT?20?`, shared access): Temporary EDI work file.
       - `EDI810` (`?9?EDI810`, shared access): Permanent EDI invoice file.
     - `BB513` copies records from `EDIOUT` to `EDI810` for EDI 810 invoice transmission.
   - **Run BB514**:
     - The `// BB514 ,,,,,,,,?9?` statement calls the `BB514` OCL program, which executes the `BB514` RPG program to combine `FIL855` (EDI 855 purchase order acknowledgments) and `EDI810` (EDI 810 invoices) into `EDISEND` for transmission to Kleinschmidt.
   - **Run BB517**:
     - The `// LOAD BB517` block loads and executes the `BB517` RPG program, opening:
       - `EDIOTH` (`?9?EDOTH?20?`, shared access): Temporary EDI invoice header file.
       - `EDIINVH` (`?9?EDIINVH`, shared access): Permanent EDI invoice header file.
       - `EDIDPIH` (`?9?EDIDPIH`, shared access): Duplicate EDI invoice header file.
     - `BB517` processes `EDIOTH` records, adding new records to `EDIINVH`, copying duplicates to `EDIDPIH`, and logging duplicates to a printer file.

6. **Process EDI 945 Documents**:
   - **Check for EDI 945 Data**:
     - The `// IFF DATAF1-?9?EDWSA?20? GOTO JUMP1` statement checks if the batch-specific EDI warehouse shipping advice file (`?9?EDWSA?20?`) exists. If not, it skips to the `JUMP1` tag, bypassing EDI 945 processing.
   - **Run BB516**:
     - The `// BB516 ,,,,,,,,?9?,,,,,,,,,,,?20?` statement calls the `BB516` OCL program, which executes the `BB516` RPG program to copy records from `EDWSA` (`?9?EDWSA?20?`) to `EDI945` (`?9?EDI945`) for EDI 945 warehouse shipping advice transmission.

7. **Print Registers**:
   - The `// SWITCH 1XXXXXXX` statement sets switch 1 to suppress the gross profit report.
   - The `// LOAD BB600` block loads and executes the `BB600` RPG program (not to be confused with this OCL program), opening:
     - `INVTOT` (`?9?BBTT?20?`): Invoice totals file.
     - `BBTRAN` (`?9?BBTR?20?`): Invoice transaction file.
     - `BICONT` (`?9?BICONT`, shared access): Billing control file.
     - `ARCUST` (`?9?ARCUST`, shared access): A/R customer file.
     - Printer files: `LIST`, `LIST2`, `MILIST` (device `P5`, priority 0), `MILISTP1` (device `P1`, priority 0).
   - Conditional printer overrides for the `REPORT` file based on the `?9?` environment variable:
     - If `?9? = G`, `OVRPRTF FILE(REPORT) OUTQ(QUSRSYS/SJPOST)`.
     - If `?9? = G` and another condition, `OVRPRTF FILE(REPORT) OUTQ(QUSRSYS/TESTOUTQ)`.
   - The `BB600` RPG program likely prints invoice registers or reports using these files.
   - The `// SWITCH 0XXXXXXX` statement resets switch 1 after execution.

8. **Post Sales Analysis Transactions**:
   - The `// LOAD BB605` block loads and executes the `BB605` RPG program, opening multiple files:
     - `SA5TRN` (`?9?BBSA?20?`): Sales analysis transaction file.
     - `SA5TSH` (`?9?BBSS?20?`): Sales analysis transaction header file.
     - `SHPADR` (`?9?SHPADR`, shared access): Ship-to address file.
     - `SA5FILD`, `SA5FILM`, `SA5DBBD`, `SA5DBBM`, `SA5BCMD`, `SA5BCMM`, `SA5COPD`, `SA5COPM`, `SA5SHA` (`?9?` prefix, shared access): Various sales analysis files.
     - `BBIBCH`, `BBIBCHX` (`?9?` prefix, shared access): Invoice batch header files.
     - `BBTRA1`, `BBTRANU` (`?9?BBTR?20?`), `BBTRTX` (`?9?BBTX?20?`): Additional transaction files.
   - `BB605` likely processes sales analysis data, updating relevant files based on invoice transactions.

9. **Reorganize and Delete Transaction Files**:
   - The `// IFF ?F'A,?9?BBTR?20?'?/0 RGZPFM FILE(?9?BBTR?20?)` statement reorganizes the `BBTR?20?` file if it exists and is empty.
   - The `// IFF ?F'A,?9?BBTX?20?'?/0 RGZPFM FILE(?9?BBTX?20?)` statement reorganizes the `BBTX?20?` file if it exists and is empty.
   - Conditional deletion of files if both `BBTR?20?` and `BBTX?20?` are empty:
     - `GSDELETE BBTX?20?,,,,,,,,?9?`
     - `GSDELETE BBTR?20?@,,,,,,,,?9?`
     - `GSDELETE BBTR?20?,,,,,,,,?9?`
   - This cleans up empty transaction files to free up space.

10. **Copy Electronic Invoices to Permanent File**:
    - The `// IF ?F'A,?9?BBEIP?20?'?/00000000 GOTO JUMP2` statement checks if the batch-specific electronic invoice file (`?9?BBEIP?20?`) is empty. If so, it skips to the `JUMP2` tag.
    - Otherwise, the `CPYF FROMFILE(*LIBL/?9?BBEIP?20?) TOFILE(*LIBL/?9?SA5IVEI) MBROPT(*ADD) FMTOPT(*NOCHK)` statement copies records from `BBEIP?20?` to `SA5IVEI`, adding records (`MBROPT(*ADD)`) without format checking (`FMTOPT(*NOCHK)`).

11. **Create Copies for Spoolflex (Advanced Shipping Notices and Invoices)**:
    - For Advanced Shipping Notices (ASN) if `?13? = PP` and `?9? = G`:
      - The `CALL PGM(BB6002ASN) PARM('?20?' '?9?')` statement calls the `BB6002ASN` program, passing the batch number (`?20?`) and library prefix (`?9?`), to create copies of ASN data for Spoolflex (a system for splitting, copying, and distributing output).
    - For Invoices if `?9? = G`:
      - The `CALL PGM(BB6002NEW) PARM('?20?' '?9?')` statement calls the `BB6002NEW` program, passing the same parameters, to create copies of invoice data for Spoolflex.
    - A commented-out call to `BB6002` suggests a legacy or alternative program for invoices, replaced by `BB6002NEW`.

12. **Delete Temporary and Transaction Files**:
    - A series of `// IF DATAF1-?9?XXXX?20? DELETE ?9?XXXX?20?,F1` statements deletes batch-specific temporary and transaction files if they exist:
      - `BBIN?20?`, `BBRR?20?`, `BBTT?20?`, `BBSA?20?`, `BBSS?20?`, `BBCF?20?`, `BBOK?20?`, `BBKE?20?`, `BBTL?20?`, `ORIN?20?`, `ORBB?20?`, `OXBB?20?`, `EDOT?20?`, `EDOTH?20?`, `BBEIP?20?`, `JEREMY`.
    - This cleans up all batch-specific files after processing to free up disk space.

13. **Process Viscosity ASN Shipments**:
    - If `?13? = PP` (Viscosity Post), the `// IF ?13?/PP BB607 ,,,,,,,,?9?` statement calls the `BB607` OCL program, which likely processes or removes orders related to Viscosity ASN shipments.
    - This step is noted to be activated after the "PP Barcode Project" is active.

14. **Program Termination**:
    - The OCL program terminates after completing all steps, having processed invoices, EDI files, sales analysis data, and cleaned up temporary files.

---

### Business Rules

1. **Process Synchronization**:
   - The program waits for the `EDISNDCK` job to complete to avoid conflicts with EDI send processes, ensuring data integrity during invoice posting.

2. **Order Sync Protection**:
   - The `JEREMY` file is created to prevent order synchronization during invoice posting, ensuring that invoice processing is isolated from order updates.

3. **EDI Processing**:
   - EDI 810 (invoices) and EDI 945 (warehouse shipping advice) files are processed only if their respective batch-specific files (`EDOT?20?`, `EDWSA?20?`) exist.
   - Records are copied to permanent files (`EDI810`, `EDI945`, `EDIINVH`, `SA5IVEI`) and duplicates are managed (`EDIDPIH`).
   - Files are archived (via `BB514` and `BB516`) to maintain historical data for up to 20 versions.

4. **Sales Journal Update**:
   - The Sales Journal number is updated for company `10` via `BB630`, ensuring unique journal entries for A/R posting.

5. **Reporting**:
   - Invoice registers are printed using `BB600`, with the gross profit report suppressed (switch 1).
   - Output is directed to specific printer queues based on the environment (`?9? = G`).

6. **Sales Analysis**:
   - Sales analysis transactions are posted to multiple files via `BB605`, ensuring accurate tracking of sales data.

7. **File Cleanup**:
   - Empty transaction files (`BBTR?20?`, `BBTX?20?`) are reorganized and deleted if empty.
   - All batch-specific temporary files are deleted after processing to maintain system efficiency.
   - The `EDWSA` file is cleared after EDI 945 processing.

8. **Spoolflex Integration**:
   - For Viscosity Post (`?13? = PP`), ASN and invoice data are prepared for Spoolflex distribution, ensuring copies are available for splitting and distribution.

9. **Conditional Processing**:
   - Many steps are conditional based on file existence (`DATAF1`) or environment variables (`?9?`, `?13?`), allowing flexibility for different configurations (e.g., `PP` for Viscosity Post, `G` for specific output queues).

---

### External Programs Called

1. **BB630**:
   - OCL program that executes the `BB630` RPG program to update the Sales Journal number (`ACSLJ#`) in the `ARCONT` file for company `10`.

2. **BB513**:
   - RPG program that copies records from `EDIOUT` to `EDI810` for EDI 810 invoice processing.

3. **BB514**:
   - OCL program that executes the `BB514` RPG program to combine `FIL855` (EDI 855) and `EDI810` (EDI 810) records into `EDISEND` for transmission to Kleinschmidt.

4. **BB517**:
   - RPG program that processes `EDIOTH` records, adding new records to `EDIINVH`, copying duplicates to `EDIDPIH`, and logging duplicates to a printer file.

5. **BB516**:
   - OCL program that executes the `BB516` RPG program to copy `EDWSA` records to `EDI945` for EDI 945 warehouse shipping advice transmission.

6. **BB600**:
   - RPG program that prints invoice registers using `INVTOT`, `BBTRAN`, `BICONT`, and `ARCUST`, with output directed to multiple printer files.

7. **BB605**:
   - RPG program that posts sales analysis transactions to multiple files (`SA5TRN`, `SA5TSH`, etc.) for sales tracking.

8. **BB6002ASN**:
   - Program called for Viscosity Post (`?13? = PP`) to create copies of Advanced Shipping Notices for Spoolflex distribution.

9. **BB6002NEW**:
   - Program called to create copies of invoices for Spoolflex distribution, replacing a legacy `BB6002` program.

10. **BB607**:
    - OCL program called for Viscosity Post (`?13? = PP`) to process or remove orders related to Viscosity ASN shipments, activated post-PP Barcode Project.

---

### Tables (Files) Used

1. **JEREMY**:
   - Temporary file created to prevent order synchronization (`?9?JEREMY`, 999,000 records, 10 bytes, type `T`).
   - Deleted after processing.

2. **ARCONT**:
   - Accounts Receivable control file (`?9?ARCONT`, shared access, used by `BB630`).
   - Stores Sales Journal number (`ACSLJ#`) and other A/R control data.

3. **EDIOUT**:
   - Temporary EDI work file (`?9?EDOT?20?`, shared access, used by `BB513`).
   - Copied to `EDI810` for invoice transmission. Deleted after processing.

4. **EDI810**:
   - Permanent EDI 810 invoice file (`?9?EDI810`, shared access, used by `BB513`, `BB514`).
   - Archived to `BBIV01`–`BBIV40` by `BB514`. Cleared after processing.

5. **FIL855**:
   - EDI 855 purchase order acknowledgment file (`?9?FIL855`, shared access, used by `BB514`).
   - Archived to `BBPA01`–`BBPA40` and cleared by `BB514`.

6. **EDISEND**:
   - EDI send file (`EDISEND` or `?9?EDISEND`, shared access, used by `BB514`).
   - Combines `FIL855` and `EDI810` for transmission to Kleinschmidt.

7. **EDIOTH**:
   - Temporary EDI invoice header file (`?9?EDOTH?20?`, shared access, used by `BB517`).
   - Processed into `EDIINVH` or `EDIDPIH`. Deleted after processing.

8. **EDIINVH**:
   - Permanent EDI invoice header file (`?9?EDIINVH`, shared access, used by `BB517`).
   - Stores unique EDI invoice header records.

9. **EDIDPIH**:
   - Duplicate EDI invoice header file (`?9?EDIDPIH`, shared access, used by `BB517`).
   - Stores duplicate EDI invoice header records.

10. **EDWSA**:
    - Batch-specific EDI 945 warehouse shipping advice file (`?9?EDWSA?20?`, shared access, used by `BB516`).
    - Copied to `EDI945` and cleared after processing.

11. **EDI945**:
    - Permanent EDI 945 warehouse shipping advice file (`?9?EDI945`, shared access, used by `BB516`).
    - Archived to `BBVA01`–`BBVA40` by `BB516`.

12. **INVTOT**:
    - Invoice totals file (`?9?BBTT?20?`, used by `BB600`).
    - Deleted after processing.

13. **BBTRAN**:
    - Invoice transaction file (`?9?BBTR?20?`, used by `BB600`, `BB605`).
    - Reorganized and deleted if empty.

14. **BICONT**:
    - Billing control file (`?9?BICONT`, shared access, used by `BB600`).

15. **ARCUST**:
    - A/R customer file (`?9?ARCUST`, shared access, used by `BB600`).

16. **SA5TRN**:
    - Sales analysis transaction file (`?9?BBSA?20?`, used by `BB605`).
    - Deleted after processing.

17. **SA5TSH**:
    - Sales analysis transaction header file (`?9?BBSS?20?`, used by `BB605`).
    - Deleted after processing.

18. **SHPADR**, **SA5FILD**, **SA5FILM**, **SA5DBBD**, **SA5DBBM**, **SA5BCMD**, **SA5BCMM**, **SA5COPD**, **SA5COPM**, **SA5SHA**:
    - Sales analysis files (`?9?` prefix, shared access, used by `BB605`).

19. **BBIBCH**, **BBIBCHX**:
    - Invoice batch header files (`?9?` prefix, shared access, used by `BB605`).

20. **BBTRA1**, **BBTRANU**, **BBTRTX**:
    - Additional transaction files (`?9?BBTRA1`, `?9?BBTR?20?`, `?9?BBTX?20?`, used by `BB605`).
    - `BBTR?20?` and `BBTX?20?` are reorganized and deleted if empty.

21. **BBEIP**:
    - Batch-specific electronic invoice file (`?9?BBEIP?20?`, used for copying to `SA5IVEI`).
    - Deleted after processing.

22. **SA5IVEI**:
    - Permanent electronic invoice file (`?9?SA5IVEI`, receives records from `BBEIP?20?`).

23. **BBPA01 to BBPA40**:
    - Archive files for EDI 855 (`?9?` prefix, managed by `BB514`).

24. **BBIV01 to BBIV40**:
    - Archive files for EDI 810 (`?9?` prefix, managed by `BB514`).

25. **BBVA01 to BBVA40**:
    - Archive files for EDI 945 (`?9?` prefix, managed by `BB516`).

26. **BBIN?20?**, **BBRR?20?**, **BBCF?20?**, **BBOK?20?**, **BBKE?20?**, **BBTL?20?**, **ORIN?20?**, **ORBB?20?**, **OXBB?20?**:
    - Batch-specific temporary files (`?9?` prefix, deleted after processing).

---

### Summary

The `BB600.ocl36.txt` OCL program is the main control script for the invoice posting workflow, coordinating:
- Waiting for EDI send process completion (`EDISNDCK`).
- Creating a temporary file (`JEREMY`) to prevent order synchronization.
- Updating the Sales Journal number via `BB630`.
- Processing EDI 810 (`BB513`, `BB514`, `BB517`) and EDI 945 (`BB516`) files.
- Printing invoice registers via `BB600`.
- Posting sales analysis transactions via `BB605`.
- Copying electronic invoices to a permanent file.
- Creating ASN and invoice copies for Spoolflex (`BB6002ASN`, `BB6002NEW`).
- Cleaning up temporary and transaction files.
- Processing Viscosity ASN shipments via `BB607` (if `?13? = PP`).

**External Programs Called**: `BB630`, `BB513`, `BB514`, `BB517`, `BB516`, `BB600`, `BB605`, `BB6002ASN`, `BB6002NEW`, `BB607`.
**Tables Used**: `JEREMY`, `ARCONT`, `EDIOUT`, `EDI810`, `FIL855`, `EDISEND`, `EDIOTH`, `EDIINVH`, `EDIDPIH`, `EDWSA`, `EDI945`, `INVTOT`, `BBTRAN`, `BICONT`, `ARCUST`, `SA5TRN`, `SA5TSH`, `SHPADR`, `SA5FILD`, `SA5FILM`, `SA5DBBD`, `SA5DBBM`, `SA5BCMD`, `SA5BCMM`, `SA5COPD`, `SA5COPM`, `SA5SHA`, `BBIBCH`, `BBIBCHX`, `BBTRA1`, `BBTRANU`, `BBTRTX`, `BBEIP`, `SA5IVEI`, `BBPA01–BBPA40`, `BBIV01–BBIV40`, `BBVA01–BBVA40`, `BBIN?20?`, `BBRR?20?`, `BBCF?20?`, `BBOK?20?`, `BBKE?20?`, `BBTL?20?`, `ORIN?20?`, `ORBB?20?`, `OXBB?20?`.

This program ensures a complete invoice posting process, integrating EDI transmission, reporting, sales analysis, and cleanup, with conditional logic for specific environments (e.g., Viscosity Post).