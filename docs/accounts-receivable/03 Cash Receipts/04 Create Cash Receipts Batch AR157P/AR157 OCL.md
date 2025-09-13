The provided document, `AR157.ocl36.txt`, is an Operations Control Language (OCL) program used in IBM midrange systems (e.g., AS/400, iSeries) to handle the copying of AR (Accounts Receivable) EFT (Electronic Funds Transfer) individual entry data into a cash receipts batch file. This program is called from the main OCL program (`AR157P.ocl36.txt`) as part of the EFT cash receipts batch file creation process. Below is a detailed explanation of the process steps, business rules, tables/files used, and external programs called, based on the OCL code.

---

### Process Steps of the OCL Program (`AR157`)

The `AR157` OCL program performs a straightforward but critical task of copying data from a source file to a target cash receipts batch file and cleaning up afterward. Here’s a step-by-step breakdown of the process:

1. **Call the AR157A Program**:
   - The statement `// AR157A ,,,,,,,,?9?` calls an external program named `AR157A`, passing a parameter represented by `?9?` (likely a dynamic value such as a company code or batch identifier).
   - This program likely performs preparatory or related processing for the EFT cash receipts batch file creation, such as validating or formatting data before the copy operation.

2. **Copy File Operation (CPYF)**:
   - The statement `CPYF (QS36F/?9?E?L'110,6'?) (QS36F/?9?CRIEGG) CRTFILE(*YES) MBROPT(*REPLACE) INCCHAR(*RCD 1 *NE 'D')` executes a Copy File (`CPYF`) command:
     - **Source File**: `QS36F/?9?E?L'110,6'?` – The source file is dynamically named, where `?9?` is a placeholder (likely a company code or batch identifier) and `?L'110,6'?` represents a 6-character value from location 110 in the program’s local storage (likely the bank upload date `KYUPDT` from the `AR157P` RPGLE program). For example, this could resolve to a file like `QS36F/CO123E202311` (where `CO123` is the company code and `202311` is the upload date).
     - **Target File**: `QS36F/?9?CRIEGG` – The target file is the cash receipts batch file, dynamically named with `?9?` (same placeholder) and `GG` (replacing `?WS?` as per the change noted by Jan Beccari). For example, `QS36F/CO123CRIEGG`.
     - **CRTFILE(*YES)**: If the target file does not exist, it is created.
     - **MBROPT(*REPLACE)**: If the target file already exists, its contents are replaced with the copied data.
     - **INCCHAR(*RCD 1 *NE 'D')**: Only records where the first character of the record is not `'D'` (likely indicating non-deleted records) are copied.
   - This step copies relevant EFT transaction records from the source file to the cash receipts batch file, filtering out deleted records.

3. **Delete the Source Workfile**:
   - The statement `** IF DATAF1-?9?E?L'110,6'? DELETE ?9?E?L'110,6'?,F1` checks if the source file `?9?E?L'110,6'?` exists in the `DATAF1` library.
   - If the file exists, it is deleted using the `DELETE` command, with `F1` specifying the file’s library or format.
   - This step cleans up the temporary workfile after its data has been copied to the target batch file.

4. **Clear Local Variables**:
   - The statement `// LOCAL BLANK-*ALL` clears all local variables in the program’s storage, resetting the environment to prevent data leakage between runs.

---

### Business Rules

The `AR157` OCL program enforces the following business rules:

1. **Selective Data Copy**:
   - Only records from the source file (`?9?E?L'110,6'?`) where the first character is not `'D'` are copied to the target cash receipts batch file (`?9?CRIEGG`). This ensures that only active (non-deleted) EFT transaction records are included in the batch.

2. **Dynamic File Naming**:
   - Both source and target files use dynamic naming with placeholders:
     - `?9?` represents a company code or batch identifier.
     - `?L'110,6'?` represents the bank upload date (likely `KYUPDT` from the `AR157P` RPGLE program).
     - The target file uses `GG` (replacing `?WS?`) to align with the Atrium environment, as per the change noted by Jan Beccari.

3. **File Creation or Replacement**:
   - If the target cash receipts batch file (`?9?CRIEGG`) does not exist, it is created (`CRTFILE(*YES)`).
   - If it exists, its contents are replaced with new data (`MBROPT(*REPLACE)`), ensuring the batch file contains only the latest copied records.

4. **Workfile Cleanup**:
   - The source workfile (`?9?E?L'110,6'?`) is deleted after the copy operation if it exists, ensuring no residual temporary files remain.

5. **Environment Reset**:
   - All local variables are cleared at the end of the program to maintain a clean state for subsequent executions.

---

### Tables/Files Used

The program interacts with the following files:

1. **Source File: `QS36F/?9?E?L'110,6'?`**:
   - Library: `QS36F`
   - Name: Dynamically constructed as `?9?E?L'110,6'?` (e.g., `QS36F/CO123E202311`, where `?9?` is a company code and `?L'110,6'?` is the bank upload date).
   - Purpose: Temporary workfile containing EFT individual entry data to be copied to the cash receipts batch file.
   - Usage: Read during the `CPYF` operation, deleted afterward if it exists.

2. **Target File: `QS36F/?9?CRIEGG`**:
   - Library: `QS36F`
   - Name: Dynamically constructed as `?9?CRIEGG` (e.g., `QS36F/CO123CRIEGG`, where `?9?` is a company code and `GG` replaces `?WS?`).
   - Purpose: Cash receipts batch file that stores the copied EFT transaction data.
   - Usage: Created or replaced during the `CPYF` operation.

---

### External Programs Called

The program calls the following external program:
1. **AR157A**:
   - Purpose: Likely performs preparatory processing or validation related to the EFT individual entry data before the copy operation. Its exact functionality is not specified in the provided code.
   - Parameter: `?9?` (likely a company code or batch identifier).

Additionally, the `CPYF` command is a system utility command, not a user-defined program, used to copy data between files.

---

### Notes
- **Integration with Main OCL Program**: The `AR157` OCL program is called by the main `AR157P.ocl36` program after validation in the `AR157P` RPGLE program. The `?9?` parameter and `?L'110,6'?` (likely `KYUPDT`) are passed from the RPGLE program to ensure consistent file naming.
- **Jan Beccari Change**: The change from `?WS?` to `GG` (noted on 11/28/23) modifies the target file name to `?9?CRIEGG`, aligning with the Atrium environment.
- **File Deletion Safety**: The program checks for the existence of the source workfile before deletion, preventing errors if the file is missing.
- **Dynamic Naming**: The use of `?9?` and `?L'110,6'?` placeholders indicates a flexible naming convention to support multiple companies or batch dates.

If you need further details, such as the functionality of `AR157A` or additional context about related programs, let me know!