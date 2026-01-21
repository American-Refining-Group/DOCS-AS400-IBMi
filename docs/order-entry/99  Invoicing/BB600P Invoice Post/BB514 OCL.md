The `BB514.ocl36.txt` OCL program is part of an IBM System/36 or AS/400 environment and is designed to manage the archiving of EDI (Electronic Data Interchange) files, specifically for purchase order acknowledgments (EDI 855) and invoices (EDI 810), as well as to execute the `BB514` program to process these files. The program also clears the source files after processing. Below is a detailed explanation of the process steps, external programs called, and tables (files) used.

---

### Process Steps of the BB514 OCL Program

The `BB514.ocl36.txt` OCL program performs a sequence of file management operations to archive prior versions of EDI files and then invokes the `BB514` program to process the EDI data. The program also clears the source files to prepare for new data. The steps are as follows:

1. **Archiving Purchase Order Acknowledgment Files (EDI 855)**:
   - **Delete Oldest Archive**:
     - The `// GSDELETE BBPA40,,,,,,,,?9?` statement deletes the oldest archive file (`?9?BBPA40`) for purchase order acknowledgments (denoted as `PA` in the comments).
   - **Rename Chain for EDI 855 Files**:
     - A series of `// IF DATAF1-?9?BBPAXX RENAME ?9?BBPAXX,?9?BBPA(XX+1)` statements (for `XX` from 39 down to 01) checks if each file exists (`DATAF1-?9?BBPAXX`) and renames it to the next higher number (e.g., `?9?BBPA39` to `?9?BBPA40`, `?9?BBPA38` to `?9?BBPA39`, etc.).
     - This creates a rolling archive of the 20 most recent versions of the EDI 855 files, with the oldest (`?9?BBPA40`) being deleted and each file shifted up one number.
   - **Copy Current EDI 855 File**:
     - The `CPYF FROMFILE(QS36F/?9?FIL855) TOFILE(QS36F/?9?BBPA01) MBROPT(*REPLACE) CRTFILE(*YES)` statement copies the current EDI 855 file (`?9?FIL855`) to the newest archive file (`?9?BBPA01`), replacing any existing content (`MBROPT(*REPLACE)`) and creating the file if it does not exist (`CRTFILE(*YES)`).

2. **Archiving Invoice Files (EDI 810)**:
   - **Delete Oldest Archive**:
     - The `// GSDELETE BBIV40,,,,,,,,?9?` statement deletes the oldest archive file (`?9?BBIV40`) for invoices (denoted as `IV` in the comments).
   - **Rename Chain for EDI 810 Files**:
     - A series of `// IF DATAF1-?9?BBIVXX RENAME ?9?BBIVXX,?9?BBIV(XX+1)` statements (for `XX` from 39 down to 01) checks if each file exists (`DATAF1-?9?BBIVXX`) and renames it to the next higher number (e.g., `?9?BBIV39` to `?9?BBIV40`, `?9?BBIV38` to `?9?BBIV39`, etc.).
     - This creates a rolling archive of the 20 most recent versions of the EDI 810 files, with the oldest (`?9?BBIV40`) being deleted and each file shifted up one number.
   - **Copy Current EDI 810 File**:
     - The `CPYF FROMFILE(QS36F/?9?EDI810) TOFILE(QS36F/?9?BBIV01) MBROPT(*REPLACE) CRTFILE(*YES)` statement copies the current EDI 810 file (`?9?EDI810`) to the newest archive file (`?9?BBIV01`), replacing any existing content (`MBROPT(*REPLACE)`) and creating the file if it does not exist (`CRTFILE(*YES)`).

3. **Load and Execute BB514 Program**:
   - The `// LOAD BB514` statement loads the `BB514` program into memory for execution.
   - The program opens three files:
     - `// FILE NAME-FIL855,LABEL-?9?FIL855,DISP-SHR`: Opens the EDI 855 purchase order acknowledgment file with shared access (`DISP-SHR`).
     - `// FILEzysz4>FILE NAME-EDI810,LABEL-?9?EDI810,DISP-SHR`: Opens the EDI 810 invoice file with shared access (`DISP-SHR`).
     - `// IF ?9?/G + FILE NAME-EDISEND,LABEL-EDISEND,DISP-SHR // ELSE + FILE NAME-EDISEND,LABEL-?9?EDISEND,DISP-SHR`: Opens the `EDISEND` file with shared access, using the label `EDISEND` if the environment variable `?9?` is set to `G`, otherwise `?9?EDISEND`.
   - The `// RUN` statement executes the `BB514` program, which likely processes the EDI data in `FIL855` and `EDI810` for transmission (though the exact functionality of `BB514` is not provided).

4. **Clear Source Files**:
   - The `CLRPF ?9?EDI810` statement clears all records from the `EDI810` file.
   - The `CLRPF ?9?FIL855` statement clears all records from the `FIL855` file.
   - This prepares the source files for new EDI data in the next processing cycle.

5. **Program Termination**:
   - The OCL program terminates after clearing the files, completing the archiving and processing cycle.

---

### External Programs Called

1. **BB514**:
   - This program is loaded and executed to process the EDI data in `FIL855` and `EDI810`.
   - The exact functionality is not specified in the OCL code, but based on the context and the preceding `BB513.rpg36.txt` program (which copies `EDIOUT` to `EDI810`), `BB514` likely prepares or formats the EDI data for transmission to Kleinschmidt or performs additional EDI-related processing.

---

### Tables (Files) Used

1. **FIL855**:
   - **Description**: Current EDI purchase order acknowledgment file (EDI 855).
   - **Label**: `?9?FIL855` (system-generated label with a prefix).
   - **Disposition**: Shared access (`DISP-SHR`).
   - **Purpose**: Contains the latest EDI 855 data, which is copied to the archive file `BBPA01` and then cleared.
   - **Usage**: Copied to `BBPA01` and cleared after processing.

2. **EDI810**:
   - **Description**: Current EDI invoice file (EDI 810).
   - **Label**: `?9?EDI810` (system-generated label with a prefix).
   - **Disposition**: Shared access (`DISP-SHR`).
   - **Purpose**: Contains the latest EDI 810 data (likely copied from `EDIOUT` by `BB513`), which is copied to the archive file `BBIV01` and then cleared.
   - **Usage**: Copied to `BBIV01` and cleared after processing.

3. **EDISEND**:
   - **Description**: EDI send file for transmission to Kleinschmidt.
   - **Label**: `EDISEND` or `?9?EDISEND` (depending on the `?9?` environment variable).
   - **Disposition**: Shared access (`DISP-SHR`).
   - **Purpose**: Likely used by the `BB514` program to store or process EDI data for transmission.
   - **Usage**: Opened for shared access by `BB514` for processing.

4. **BBPA01 to BBPA40**:
   - **Description**: Archive files for EDI 855 purchase order acknowledgments.
   - **Label**: `?9?BBPA01` to `?9?BBPA40` (system-generated labels).
   - **Purpose**: Store the 20 most recent versions of EDI 855 data for historical reference.
   - **Usage**: The current `FIL855` is copied to `BBPA01`, and existing archives are renamed to higher numbers, with `BBPA40` being deleted.

5. **BBIV01 to BBIV40**:
   - **Description**: Archive files for EDI 810 invoices.
   - **Label**: `?9?BBIV01` to `?9?BBIV40` (system-generated labels).
   - **Purpose**: Store the 20 most recent versions of EDI 810 data for historical reference.
   - **Usage**: The current `EDI810` is copied to `BBIV01`, and existing archives are renamed to higher numbers, with `BBIV40` being deleted.

---

### Summary

The `BB514.ocl36.txt` OCL program is a file management utility that:
- Archives the current EDI 855 (`FIL855`) and EDI 810 (`EDI810`) files by copying them to `BBPA01` and `BBIV01`, respectively, and maintaining a rolling set of 20 archive versions for each by renaming and deleting the oldest files (`BBPA40`, `BBIV40`).
- Loads and executes the `BB514` program to process the EDI data in `FIL855` and `EDI810`.
- Clears the `EDI810` and `FIL855` files to prepare for new data.
- Notes that the `EDISEND` file is also saved by an external FTP program, making this archiving process redundant but retained for additional data preservation.

**External Programs Called**: `BB514`.
**Tables Used**: `FIL855`, `EDI810`, `EDISEND`, `BBPA01` to `BBPA40`, `BBIV01` to `BBIV40`.

The program ensures that EDI data is preserved for historical purposes and prepares the data for processing by `BB514`, likely for transmission to Kleinschmidt, a third-party EDI service provider.