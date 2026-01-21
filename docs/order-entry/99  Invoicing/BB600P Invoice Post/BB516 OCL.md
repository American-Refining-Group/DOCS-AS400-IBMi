The `BB516.ocl36.txt` OCL program is part of an IBM System/36 or AS/400 environment and is designed to manage the archiving of EDI (Electronic Data Interchange) files, specifically for EDI 945 (Warehouse Shipping Advice) files, and to execute the `BB516` program to process these files. It also clears a specific file after processing. Below is a detailed explanation of the process steps, external programs called, and tables (files) used.

---

### Process Steps of the BB516 OCL Program

The `BB516.ocl36.txt` OCL program performs file management operations to archive prior versions of EDI 945 files, invokes the `BB516` program to process the EDI data, and clears a source file to prepare for new data. The steps are as follows:

1. **Load and Execute BB516 Program**:
   - The `// LOAD BB516` statement loads the `BB516` program into memory for execution.
   - The program opens two files:
     - `// FILE NAME-EDWSA,LABEL-?9?EDWSA?20?,DISP-SHR`: Opens the EDI warehouse shipping advice file (`EDWSA`) with a label that includes a batch number (`?20?`) and shared access (`DISP-SHR`).
     - `// FILE NAME-EDI945,LABEL-?9?EDI945,DISP-SHR`: Opens the EDI 945 file with shared access (`DISP-SHR`).
   - The `// RUN` statement executes the `BB516` program, which likely processes the EDI 945 data in `EDI945` and `EDWSA` for transmission or further processing (though the exact functionality of `BB516` is not provided).

2. **Archiving EDI 945 Files**:
   - **Delete Oldest Archive**:
     - The `// GSDELETE BBVA40,,,,,,,,?9?` statement deletes the oldest archive file (`?9?BBVA40`) for viscosity shipping advice (denoted as `VA` in the comments).
   - **Rename Chain for EDI 945 Files**:
     - A series of `// IF DATAF1-?9?BBVAXX RENAME ?9?BBVAXX,?9?BBVA(XX+1)` statements (for `XX` from 39 down to 01) checks if each file exists (`DATAF1-?9?BBVAXX`) and renames it to the next higher number (e.g., `?9?BBVA39` to `?9?BBVA40`, `?9?BBVA38` to `?9?BBVA39`, etc.).
     - This creates a rolling archive of the 20 most recent versions of the EDI 945 files, with the oldest (`?9?BBVA40`) being deleted and each file shifted up one number.
   - **Copy Current EDI 945 File**:
     - The `CPYF FROMFILE(QS36F/?9?EDI945) TOFILE(QS36F/?9?BBVA01) MBROPT(*REPLACE) CRTFILE(*YES)` statement copies the current EDI 945 file (`?9?EDI945`) to the newest archive file (`?9?BBVA01`), replacing any existing content (`MBROPT(*REPLACE)`) and creating the file if it does not exist (`CRTFILE(*YES)`).

3. **Clear Source File**:
   - The `CLRPFM ?9?EDWSA?20?` statement clears all records from the `EDWSA` file specific to the batch number (`?20?`).
   - This prepares the file for new EDI 945 data in the next processing cycle.

4. **Program Termination**:
   - The OCL program terminates after clearing the `EDWSA` file, completing the archiving and processing cycle.

---

### External Programs Called

1. **BB516**:
   - This program is loaded and executed to process the EDI data in `EDI945` and `EDWSA`.
   - The exact functionality is not specified in the OCL code, but it likely prepares or formats the EDI 945 warehouse shipping advice data for transmission to Kleinschmidt, a third-party EDI service provider, as noted in the comments.

---

### Tables (Files) Used

1. **EDWSA**:
   - **Description**: EDI warehouse shipping advice file, specific to a batch.
   - **Label**: `?9?EDWSA?20?` (system-generated label with a prefix and batch number `?20?`).
   - **Disposition**: Shared access (`DISP-SHR`).
   - **Purpose**: Contains EDI 945 warehouse shipping advice data for a specific batch, used by the `BB516` program.
   - **Usage**: Opened for shared access by `BB516` and cleared after processing.

2. **EDI945**:
   - **Description**: Current EDI 945 warehouse shipping advice file.
   - **Label**: `?9?EDI945` (system-generated label with a prefix).
   - **Disposition**: Shared access (`DISP-SHR`).
   - **Purpose**: Contains the latest EDI 945 data, which is copied to the archive file `BBVA01`.
   - **Usage**: Copied to `BBVA01` for archiving and used by `BB516` for processing.

3. **BBVA01 to BBVA40**:
   - **Description**: Archive files for EDI 945 warehouse shipping advice.
   - **Label**: `?9?BBVA01` to `?9?BBVA40` (system-generated labels).
   - **Purpose**: Store the 20 most recent versions of EDI 945 data for historical reference.
   - **Usage**: The current `EDI945` is copied to `BBVA01`, and existing archives are renamed to higher numbers, with `BBVA40` being deleted.

---

### Summary

The `BB516.ocl36.txt` OCL program is a file management utility that:
- Loads and executes the `BB516` program to process EDI 945 data in `EDI945` and `EDWSA`.
- Archives the current EDI 945 file (`EDI945`) by copying it to `BBVA01` and maintaining a rolling set of 20 archive versions (`BBVA01` to `BBVA40`) by renaming and deleting the oldest file (`BBVA40`).
- Clears the `EDWSA` file for the specific batch (`?20?`) to prepare for new data.
- Notes that an external FTP program also handles the transmission of the `EDI945` file to Kleinschmidt, indicating that this archiving process is supplementary.

**External Programs Called**: `BB516`.
**Tables Used**: `EDWSA` (batch-specific EDI 945), `EDI945` (current EDI 945), `BBVA01` to `BBVA40` (EDI 945 archives).

The program ensures that EDI 945 warehouse shipping advice data is preserved for historical purposes and processed by `BB516` for transmission to a third-party EDI service provider like Kleinschmidt.