The provided document is an Operation Control Language (OCL) program, `BB600P.ocl36.txt`, used in IBM System/36 or AS/400 environments to manage invoice posting. Below is a detailed explanation of the process steps, external programs called, and tables (files) used in the program.

---

### Process Steps of the BB600P OCL Program

The OCL program `BB600P` is designed to handle the invoice posting process, ensuring that invoices are posted to the accounting system only under specific conditions (e.g., no concurrent conflicting processes, invoices printed, etc.). The program includes checks for system conditions, batch selection, and journal date prompting, and it performs the actual invoice posting. Below is a step-by-step breakdown of the process:

1. **Initial Checks for Concurrent Processes**:
   - The program checks if any conflicting processes are running, such as:
     - Invoicing Inventory Transaction Entry or Post (`INXXXS`).
     - Accounts Receivable (A/R) Customer Master Maintenance (`AR900`).
   - If any of these processes are active (`IF DATAF1-?9?INXXXS` or `IF ACTIVE-AR900`), the program displays a warning message indicating that these processes cannot run simultaneously with the invoice post.
   - The user is prompted to press `0` and `Enter` to cancel the job (`PAUSE 'PRESS 0,ENTER AND JOB WILL CANCEL'`), and the program branches to the `END` tag to terminate.

2. **Check for EDI Send Check Process**:
   - The program checks if the `EDISNDCK` process is active (`IF ACTIVE-'EDISNDCK'`).
   - If active, it waits for 3 seconds (`WAIT INTERVAL-000003`) and loops back to the `TRYAGN` tag to recheck until the process is no longer active.
   - If the `INXXXS` file exists, it builds a temporary file (`BLDFILE ?9?INXXXS,S,RECORDS,1,10`) for processing.

3. **Invoice Batch Selection**:
   - The program sets up local variables for batch processing:
     - `OFFSET-470,DATA-'?13?'`: Stores a batch type identifier (e.g., `PP` for Viscosity Post, `PM` for Product Moves Shipped Post, or empty for standard Invoice Post).
     - `OFFSET-400,DATA-'?USER?'`: Stores the user ID.
     - `OFFSET-410,DATA-'?WS?'`: Stores the workstation ID.
     - `OFFSET-504,DATA-'P'`: Sets a default flag for posting.
   - Based on the batch type (`?13?`), it sets a display message at `OFFSET-60`:
     - Empty `?13?`: `* INVOICE POST *`.
     - `PP`: `* VISCOSITY POST *`.
     - `PM`: `* PRODUCT MOVES SHIPPED POST *`.
   - A switch (`SWITCH 0XXXXXXX`) is set, likely to control batch deletion behavior.
   - The program loads and runs `BB051` to handle invoice batch selection:
     - Files `BBIBCH` and `BBIBCHX` (both labeled `?9?BBIBCH`, disposition `SHR` for shared access) are opened.
     - If the user cancels the selection (`?L'121,6'?/CANCEL`), the program branches to the `FIN` tag.
     - The program evaluates `P20` based on input at `OFFSET-490,2` (`EVALUATE P20='?L'490,2'?'`), which likely determines the batch number or identifier.

4. **Check for Printed Invoices**:
   - The program checks if invoices for the selected batch have been printed (`IFF DATAF1-?9?BBOK?20?`).
   - If not printed, it displays a message indicating that invoices must be printed or reprinted before posting can proceed.
   - The user is prompted to cancel the procedure (`PAUSE 'THIS PROCEDURE WILL BE CANCELLED!'`), and the program branches to the `REL` tag.

5. **Check for Concurrent Invoice Post Processes**:
   - The program ensures that only one instance of invoice posting is running at a time by checking for active processes:
     - `BB600P`, `BB600`, `BI600P`, or `BI600`.
   - If any of these are active, a message is displayed indicating that invoice posting is running on another workstation, and the user is prompted to wait and try again.
   - The user can cancel by pressing `0` and `Enter` (`PAUSE 'PRESS 0, ENTER AND JOB WILL CANCEL'`), and the program branches to the `REL` tag.

6. **Prompt for Invoicing Journal Date**:
   - If the batch type is not `PP` (`IF ?13?/PP GOTO SKIPAR`), the program loads `BB600P` to prompt for the A/R posting date.
   - Files opened include:
     - `GSCONT` (labeled `?9?GSCONT`, disposition `SHR`).
     - `ARCONT` (labeled `?9?ARCONT`, disposition `SHR`).
     - `BBTRAN` (labeled `?9?BBTR?20?`, disposition `SHR`, where `?20?` is the batch number).
   - The program runs to collect the journal date.
   - If `SWITCH1-1` is set, the program branches to the `REL` tag, skipping the posting.

7. **Post the Invoices**:
   - If the batch type is `PP`, the program skips the journal date prompt (`GOTO SKIPAR`).
   - The program executes `BB600 *ALL` to post all invoices in the selected batch.
   - Note: The comment indicates that this step is not run in a job queue due to batch processing requirements.

8. **Post/Release Batch**:
   - At the `REL` tag, the program sets a local variable at `OFFSET-475` with the batch file name (`?F'A,?9?BBTR?20?'?`).
   - It loads and runs `BB055` to release the batch, opening the `BBIBCH` file (labeled `?9?BBIBCH`, disposition `SHR`).

9. **Cleanup and Termination**:
   - At the `FIN` tag, if the `INXXXS` file exists, it is deleted (`DELETE ?9?INXXXS,F1`).
   - The program clears all local variables (`LOCAL BLANK-*ALL`) and terminates at the `END` tag.

---

### External Programs Called

The OCL program calls the following external programs:
1. **BB051**: Handles invoice batch selection.
2. **BB600**: Performs the actual invoice posting (executed with `*ALL` to process all invoices in the batch).
3. **BB055**: Manages the post/release of the batch.

Additionally, the program references `BB600P` itself as both the OCL program and a potential loadable module for prompting the journal date.

---

### Tables (Files) Used

The program interacts with the following files (referred to as tables in System/36 terminology):
1. **INXXXS**: A temporary file used to check for concurrent invoicing processes.
2. **BBIBCH**: Invoice batch header file, used for batch selection and release (opened with `DISP-SHR`).
3. **BBIBCHX**: Likely an index or alternate file for `BBIBCH`, used during batch selection (opened with `DISP-SHR`).
4. **GSCONT**: General system control file, used when prompting for the A/R posting date (opened with `DISP-SHR`).
5. **ARCONT**: Accounts Receivable control file, used when prompting for the A/R posting date (opened with `DISP-SHR`).
6. **BBTRAN**: Invoice transaction file, specific to the batch being processed (labeled `?9?BBTR?20?`, opened with `DISP-SHR`).
7. **BBOK**: A file (likely a flag or status file) used to check if invoices for the batch (`?20?`) have been printed.

---

### Summary

The `BB600P` OCL program orchestrates the invoice posting process by:
- Ensuring no conflicting processes (e.g., inventory transactions, A/R maintenance, or other invoice posts) are running.
- Selecting an invoice batch and verifying that invoices are printed.
- Prompting for the A/R journal date (unless processing a Viscosity Post batch).
- Posting invoices and releasing the batch.
- Cleaning up temporary files and terminating.

**External Programs**: `BB051` (batch selection), `BB600` (invoice posting), `BB055` (batch release).
**Files Used**: `INXXXS`, `BBIBCH`, `BBIBCHX`, `GSCONT`, `ARCONT`, `BBTRAN`, `BBOK`.