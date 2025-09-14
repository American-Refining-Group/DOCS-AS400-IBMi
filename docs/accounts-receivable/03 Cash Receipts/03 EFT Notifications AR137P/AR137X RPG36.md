The `AR137X.rpg36.txt` is an RPG III program used in IBM i (AS/400) systems, invoked by the `AR137.ocl36.txt` OCL script as part of the EFT (Electronic Funds Transfer) draft notice filtering process. This program is responsible for processing the `CRTRAN` file to mark or process records for deletion. Below, I’ll explain the process steps, business rules, tables used, and external programs called, providing a clear and concise analysis.

### Process Steps of the AR137X RPG Program

The `AR137X` program is a simple RPG III program designed to process records in the `CRTRAN` file, checking for a delete flag and performing an exception output operation. Here’s a step-by-step breakdown of its execution:

1. **Program Header**:
   - `H P064 AR137X`: Specifies the program identifier (`P064`) and name (`AR137X`). The `B` indicates it’s a batch program.
   - This sets up the program context within the IBM i environment.

2. **File Definition**:
   - `FCRTRAN UP F 256 256 DISK`:
     - Defines the `CRTRAN` file as an update-capable (`UP`) disk file with a fixed record length of 256 bytes.
     - The file is both input and output capable, allowing records to be read and updated.

3. **Input Specification**:
   - `ICRTRAN NS`:
     - Defines the input record layout for the `CRTRAN` file.
     - `ATDEL` (position 1, 1 byte) is defined as the delete flag field (`D - DELETE`).
   - This specifies that the program reads the `ATDEL` field to check for deletion status.

4. **Calculation Logic**:
   - The program processes each record in the `CRTRAN` file:
     - `C ATDEL IFEQ 'D'`: Checks if the `ATDEL` field equals ‘D’ (indicating a deleted record).
     - `C EXCPT`: If true, executes an exception output operation (writes to the output file or updates the record).
     - `C END`: Ends the conditional block.
   - The program loops through all records in `CRTRAN`, processing only those marked for deletion.

5. **Output Specification**:
   - `OCRTRAN EDEL`:
     - Defines an exception output operation (`EDEL`) for the `CRTRAN` file.
     - When the `EXCPT` operation is triggered, it writes or updates the record according to the output specification.
   - This likely marks the record for deletion or writes it to an exception file.

6. **Program Termination**:
   - The program ends after processing all records in the `CRTRAN` file, as is typical in RPG III’s implicit file processing cycle.

### Business Rules

The `AR137X` program enforces the following business rules:

1. **Delete Flag Check**:
   - Only records with `ATDEL` equal to ‘D’ in position 1 of the `CRTRAN` file are processed.
   - This ensures that only records explicitly marked for deletion are affected.

2. **Exception Output**:
   - Records meeting the delete condition (`ATDEL = 'D'`) are processed via an exception output operation (`EDEL`), which may involve marking the record for deletion, updating it, or writing it to an output file.
   - The exact action depends on the `EDEL` output specification, which is not detailed but typically involves flagging records for removal during reorganization.

3. **File Update**:
   - The `CRTRAN` file is opened in update mode (`UP`), allowing the program to modify records directly.
   - This supports the reorganization process by preparing deleted records for removal.

### Integration with AR137 OCL Script

The `AR137X` program is called by the `AR137.ocl36.txt` script via the `// LOAD AR137X` and `// RUN` statements, with the `CRTRAN` file labeled as `?9?E?L'110,6'?` (where `?L'110,6'?` is the update date from the LDA, prefixed by ‘E’ and `?9?` for the library). The script then executes the `RGZPFM` (Reorganize Physical File Member) command on the same file to remove deleted records. The `AR137X` program processes records marked with `ATDEL = 'D'`, setting them up for the reorganization step, which physically removes them from the file.

### Tables (Files) Used

The program uses the following file:

1. **CRTRAN**:
   - Type: Update-capable disk file (256 bytes, fixed length).
   - Fields: `ATDEL` (position 1, 1 byte, delete flag).
   - Purpose: Stores transaction data, with the program processing records marked for deletion.

### External Programs Called

The `AR137X` program does not call any external programs. It operates solely on the `CRTRAN` file and performs its logic internally.

### Summary

The `AR137X` RPG III program is a straightforward utility that processes the `CRTRAN` file to handle records marked for deletion (`ATDEL = 'D'`). It performs an exception output operation for these records, preparing them for the `RGZPFM` reorganization step in the `AR137` OCL script. The program ensures that only deleted records are processed, supporting the EFT draft notice filtering process by cleaning up the transaction file.

**Tables Used**: `CRTRAN`.
**External Programs Called**: None.