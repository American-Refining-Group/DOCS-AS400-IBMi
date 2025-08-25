The provided document is an OCL (Operation Control Language) script for an IBM System i (AS/400) environment, specifically for a process named `BB203`. This script automates a customer sales agreement posting process. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used.

### Process Steps of the BB203 OCL Program

The OCL script orchestrates a sequence of operations to prepare, process, and clean up data for posting customer sales agreements. Here’s a step-by-step breakdown:

1. **Initial Checks for Data Existence**:
   - The script uses `IFF` (If) statements to verify if the workstation file `?9?BB203?WS?` exists and contains data:
     - `// IFF DATAF1-?9?BB203?WS? PAUSE 'NO SELECTIONS HAVE BEEN MADE....PLEASE RUN OPTION 15 TO SELECT'`
       - Checks if the file exists. If it doesn’t, the script pauses and displays a message prompting the user to run "Option 15" to select data, then cancels the job.
     - `// IF ?F'A,?9?BB203?WS?'?/0000000 PAUSE 'OPTION 15 DID NOT CREATE RECORDS TO BE POSTED'`
       - Verifies if the file has records (non-zero record count). If no records exist, it pauses with an error message, deletes the file (`DLTF *LIBL/?9?BB203?WS?`), and cancels the job.
   - These checks ensure that the process only proceeds if valid data is available in the workstation file.

2. **Call Pre-Processing Program**:
   - `CALL PGM(BB203AC) PARM('?9?')`
     - Calls the program `BB203AC`, passing a parameter `?9?` (a placeholder for a specific value, likely a company code or job identifier).
     - This program likely performs preliminary processing, such as validating or preparing data in the workstation file for posting.

3. **Create Temporary File**:
   - `CRTDUPOBJ OBJ(?9?BB203?WS?) FROMLIB(*LIBL) OBJTYPE(*FILE) TOLIB(QTEMP) NEWOBJ(BB203W) DATA(*YES)`
     - Creates a duplicate of the workstation file `?9?BB203?WS?` in the `QTEMP` library, naming it `BB203W`. The `DATA(*YES)` parameter copies the data from the source file.
     - `QTEMP` is a temporary library unique to the job, ensuring isolation of the data for this process.

4. **Override Database File**:
   - `OVRDBF FILE(BB203W) TOFILE(QTEMP/BB203W)`
     - Sets a database file override so that any reference to the logical file `BB203W` points to the physical file `QTEMP/BB203W`. This ensures the subsequent program uses the temporary file.

5. **Call Main Processing Program**:
   - `CALL BB203 PARM(?9?)`
     - Calls the main program `BB203`, passing the same parameter `?9?`.
     - This program likely performs the core logic of posting the customer sales agreements, using the data in `QTEMP/BB203W`.

6. **Cleanup**:
   - `DLTF QTEMP/BB203W`
     - Deletes the temporary file `BB203W` from `QTEMP` to free up resources.
   - `DLTF *LIBL/?9?BB203?WS?`
     - Deletes the original workstation file `?9?BB203?WS?` from the library list (`*LIBL`).
   - `DLTOVR FILE(BB203W)`
     - Removes the database file override for `BB203W`, ensuring no lingering overrides affect subsequent jobs.

### External Programs Called

The OCL script explicitly calls the following external programs:
1. **BB203AC**:
   - Called with the parameter `?9?`.
   - Likely a preprocessing or validation program that prepares the data in the workstation file.
2. **BB203**:
   - Called with the parameter `?9?`.
   - The main program responsible for processing and posting the customer sales agreements.

### Tables (Files) Used

The script interacts with the following files:
1. **?9?BB203?WS?**:
   - A workstation file (likely dynamically named, with `?9?` as a placeholder for a prefix like a company code).
   - Located in the library list (`*LIBL`).
   - Contains the input data for the posting process, checked for existence and record count.
2. **BB203W**:
   - A temporary file created in `QTEMP` by duplicating `?9?BB203?WS?`.
   - Used by the `BB203` program for processing, accessed via a database file override.

### Summary

The `BB203` OCL script is a control procedure that automates the posting of customer sales agreements. It:
- Verifies the existence and validity of input data in the workstation file.
- Calls a preprocessing program (`BB203AC`).
- Creates a temporary file (`BB203W`) in `QTEMP` and sets up a file override.
- Calls the main processing program (`BB203`) to post the data.
- Cleans up by deleting temporary and input files and removing the override.

**External Programs**: `BB203AC`, `BB203`.
**Tables Used**: `?9?BB203?WS?` (workstation file), `BB203W` (temporary file in `QTEMP`).