The provided document is an **Operation Control Language (OCL)** program, specifically `BB110P.ocl36.txt`, used for managing the **Order Entry Picking Ticket Print** process in an IBM System/36 or AS/400 environment. OCL is a control language used to define job steps, file operations, and program execution in these systems. Below, I’ll explain the process steps of the program, identify the external programs called, and list the tables (files) used.

---

### Process Steps of the BB110P.ocl36 Program

The OCL program orchestrates the printing of picking tickets for order processing, including batch selection, validation, and release. It uses conditional logic, file operations, and external program calls to achieve this. Here’s a step-by-step breakdown of the process:

1. **Initial Setup and Configuration**:
   - The program begins with a switch setting (`SWITCH 00000000`), resetting all switches to off.
   - Local variables are cleared (`LOCAL BLANK-*ALL`).
   - The program checks if parameter `?3?` is set to `'Y'`. If true:
     - Sets switch bit 5 to 1 (`SWITCH XXXX1XXX`), indicating the process is for printing picking tickets only.
     - Sets a local variable at offset 231 to a blank space (`LOCAL OFFSET-231,DATA-' '`).

2. **Order Batch Selection**:
   - The program sets local variables for batch processing:
     - `OFFSET-470,DATA-'?13?'`: Stores a batch-related parameter (likely a batch type or identifier).
     - `OFFSET-494,DATA-'?USER?'`: Stores the user ID.
     - `OFFSET-502,DATA-'?WS?'`: Stores the workstation ID.
     - `OFFSET-504,DATA-'L'`: Sets a constant value `'L'`.
   - Depending on the value of `?13?`, it sets a descriptive message at offset 60:
     - If `?13?` is blank, it sets `* PRINT PICK LISTS *`.
     - If `?13?` equals `'PM'`, it sets `* PRODUCT MOVES PICK LISTS *`.
     - If `?13?` equals `'PP'`, it sets `* VISCOSITY ASN PICK LISTS *`.
   - These messages likely indicate the type of pick list being processed (standard, product moves, or viscosity ASN).

3. **Batch Deletion Check**:
   - The program checks switch 1 (`SWITCH 0XXXXXXX`). If set, it implies a batch deletion operation, but no specific logic is defined here for deletion.

4. **Load and Run Batch Selection Program**:
   - The program loads `BB001`, an external program for batch selection.
   - It opens two files:
     - `BBBTCH` (labeled `?9?BBBTCH`, shared access).
     - `BBBTCHX` (also labeled `?9?BBBTCH`, shared access).
   - The `RUN` command executes `BB001`.
   - If the value at location `?L'121,6'?` equals `'CANCEL'`, the program terminates (`RETURN`).

5. **Evaluate Parameter**:
   - The program evaluates `P20` by extracting a 2-character value from location `?L'490,2'?`. This likely represents a dynamic order or batch identifier used later.

6. **Order Entry Picking Ticket Selection**:
   - The program loads `BB110P` (itself or another instance) and opens three files:
     - `BICONT` (labeled `?9?BICONT`, shared read-modify access).
     - `BBORTR` (labeled `?9?BBOR?20?`, shared read-modify access, where `?20?` is the evaluated `P20` value).
     - `BBFRPR` (labeled `?9?BBFRPR`, shared read-modify access).
   - The `RUN` command executes `BB110P`.
   - If switch 8 is set (`SWITCH8-1`), the procedure is canceled, and the program jumps to the `REL` tag.

7. **Check for Active Process and Error Handling**:
   - The program checks if `BB110` (likely another program or module) is active:
     - If active, it loads `BB110E` (an error-handling or continuation program) and runs it.
     - If the value at `?L'300,6'?` equals `'CANCEL'`, it jumps to the `REL` tag.
     - Otherwise, it loops back to the `AGAIN` tag to recheck.
   - If `BB110` is not active:
     - If switch 5 is set and `?L'166,1'?` equals `'Y'`, it submits a job to the job queue (`JOBQ ?CLIB?,BB110,*ALL`).
     - If switch 5 is set but `?L'166,1'?` is not `'Y'`, it runs `BB110 *ALL`.
     - The program then jumps to the `END` tag.

8. **Release Batch**:
   - At the `REL` tag, the program:
     - Sets a local variable at offset 475 to `?F'A,?9?BBOR?20?'?`, which likely constructs a file name or parameter for the batch release.
     - Loads `BB005`, a program for releasing the batch.
     - Opens the `BBBTCH` file (labeled `?9?BBBTCH`, shared access).
     - Executes `BB005`.

9. **Program Termination**:
   - At the `END` tag, the program:
     - Resets all switches (`SWITCH 00000000`).
     - Clears all local variables (`LOCAL BLANK-*ALL`).
   - The program ends.

---

### External Programs Called

The OCL program explicitly calls the following external programs:
1. **BB001**: Handles order batch selection.
2. **BB110P**: The main program for picking ticket selection (potentially recursive, as it’s the same program name).
3. **BB110E**: Handles error checking or continuation when `BB110` is active.
4. **BB005**: Manages batch release.
5. **BB110**: Referenced in a job queue or direct execution, likely a related program for processing picking tickets.

---

### Tables (Files) Used

The program interacts with the following files (referred to as "tables" in RPG/OCL terminology):
1. **BBBTCH**: A batch control file, used in batch selection and release (shared access).
2. **BBBTCHX**: Likely an index or alternate version of the `BBBTCH` file, used in batch selection (shared access).
3. **BICONT**: A control file, possibly for inventory or order control (shared read-modify access).
4. **BBORTR**: An order transaction file, dynamically named using `?9?BBOR?20?` (shared read-modify access).
5. **BBFRPR**: A file likely related to freight or pricing for orders (shared read-modify access).

---

### Summary

The `BB110P.ocl36` program manages the printing of picking tickets by:
- Setting up initial conditions and parameters.
- Selecting and validating batches using `BB001`.
- Processing picking tickets with `BB110P` and related files.
- Handling errors or active processes with `BB110E`.
- Releasing batches with `BB005`.
- Using conditional logic to handle cancellations and different pick list types (standard, product moves, viscosity ASN).

**External Programs**: BB001, BB110P, BB110E, BB005, BB110.
**Files Used**: BBBTCH, BBBTCHX, BICONT, BBORTR, BBFRPR.