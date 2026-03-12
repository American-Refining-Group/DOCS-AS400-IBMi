The provided document is an **OCL (Operation Control Language)** program named `BB150P.ocl36.txt`, used for controlling the execution of an **Order Entry Bills of Lading Print** process, likely on an IBM System/36 or AS/400 system. OCL is a scripting language used to define job control procedures, manage file operations, and invoke programs. Below, I’ll explain the process steps of the program, list the external programs called, and identify the tables (files) used.

---

### **Process Steps of the BB150P OCL Program**

The OCL program is designed to manage the printing of Bills of Lading (BOL) for order processing, with logic to handle batch selection, concurrency checks, and program execution. The steps are broken down based on the program’s structure and flow:

1. **Concurrency Checks**:
   - The program checks if any of the processes `BB150`, `BB150P`, `BB152`, or `BB152P` are already active (i.e., running on another workstation).
   - If any of these processes are active:
     - A message is displayed: *"IBOX BILL OF LADINGS CREATION IS BEING RUN BY ANOTHER WORKSTATION"* and *"PLEASE TRY AGAIN IN A FEW MINUTES..."*.
     - The program pauses, prompting the user to reply with `0` to cancel the job.
     - If the user replies with `0`, the job is canceled (`CANCEL`).

   **Purpose**: Prevents multiple instances of the same or related processes from running concurrently, avoiding data conflicts or resource contention.

2. **Initialization**:
   - The program sets the switch to `00000000` (resets all switches).
   - Clears all local variables with `LOCAL BLANK-*ALL`.
   - Calls `GSY2K` (likely a utility or system initialization program, possibly for Y2K compliance or date handling).

   **Purpose**: Ensures a clean state before proceeding with the main logic.

3. **Order Batch Selection**:
   - Sets local variables for user interface and configuration:
     - `OFFSET-470`: Stores `?13?` (likely a parameter for batch type or mode).
     - `OFFSET-494`: Stores `?USER?` (the user ID).
     - `OFFSET-502`: Stores `?WS?` (workstation ID).
     - `OFFSET-504`: Sets the value `B` (possibly indicating Bills of Lading mode).
     - `OFFSET-60`: Sets a display message based on the value of `?13?`:
       - Default: `'*   PRINT BILLS OF LADING    *'`.
       - If `?13?` is not blank: `'*    PRINT BILLS OF LADING   *'`.
       - If `?13?` is `PM`: `'* PRODUCT MOVES BOL CREATION *'`.
       - If `?13?` is `PP`: `'* VISCOSITY ASN BOL CREATION *'`.
   - Sets switch 1 to allow batch deletion (`SWITCH 0XXXXXXX`).
   - Loads and runs the program `BB001`:
     - Opens files `BBBTCH` and `BBBTCHX` (both labeled `?9?BBBTCH`, accessed with `DISP-SHR` for shared access).
   - Checks if the condition `?L'121,6'?` equals `CANCEL`. If true, the program returns (exits).
   - Evaluates and stores the value of `?L'490,2'?` into variable `P20`.

   **Purpose**: Allows the user to select a batch for processing Bills of Lading, with different modes (e.g., standard BOL, Product Moves, or Viscosity ASN). The `BB001` program likely handles batch selection or validation.

4. **Order Entry Bills of Lading Selection**:
   - Loads and runs the program `BB150P`:
     - Opens files:
       - `BICONT` (labeled `?9?BICONT`, accessed with `DISP-SHRRM` for shared read/modify access).
       - `BBORTR` (labeled `?9?BBOR?20?`, where `?20?` is likely the value of `P20` from the previous step, accessed with `DISP-SHRRM`).
   - If `SWITCH8-1` is set (indicating cancellation):
     - Displays the message: *"PROCEDURE IS CANCELLED"*.
     - Jumps to the `REL` (Release Batch) tag.
   - If `?L'166,1'?` equals `Y`:
     - Submits the `BB150` job to the job queue (`JOBQ ?CLIB?,BB150,*ALL`).
   - Otherwise:
     - Runs `BB150 *ALL` directly.
     - Jumps to the `END` tag.

   **Purpose**: Executes the core Bills of Lading processing logic via `BB150P` or `BB150`, depending on conditions. The `BB150P` program likely handles the selection or preparation of BOL data, while `BB150` may perform the actual printing or processing.

5. **Release Batch**:
   - At the `REL` tag:
     - Sets a local variable at `OFFSET-475` with the value `?F'A,?9?BBOR?20?'?` (likely constructing a file name or parameter for the batch release).
     - Loads and runs the program `BB005`:
       - Opens the file `BBBTCH` (labeled `?9?BBBTCH`, accessed with `DISP-SHR`).
   - **Purpose**: Releases the selected batch, possibly updating batch status or cleaning up after processing.

6. **End of Program**:
   - At the `END` tag:
     - Resets the switch to `00000000`.
     - Clears all local variables with `LOCAL BLANK-*ALL`.

   **Purpose**: Ensures a clean exit, resetting the environment for the next run.

---

### **External Programs Called**

The OCL program invokes the following external programs:
1. **GSY2K**: Likely a system utility program, possibly for date handling or Y2K compliance.
2. **BB001**: Handles order batch selection, likely presenting a user interface or validating batch data.
3. **BB150P**: Manages the selection or preparation of Bills of Lading data.
4. **BB150**: Executes the main Bills of Lading processing or printing logic.
5. **BB005**: Handles batch release, possibly updating batch status or performing cleanup.

---

### **Tables (Files) Used**

The program references the following files (referred to as "tables" in some contexts, though they are physical files on the system):
1. **BBBTCH**: Batch file, labeled `?9?BBBTCH`, used in batch selection and release (`DISP-SHR`).
2. **BBBTCHX**: An alternate or indexed version of the batch file, also labeled `?9?BBBTCH`, used in batch selection (`DISP-SHR`).
3. **BICONT**: Control file, labeled `?9?BICONT`, used in BOL selection (`DISP-SHRRM`).
4. **BBORTR**: Order transaction file, labeled `?9?BBOR?20?` (where `?20?` is a dynamic parameter from `P20`), used in BOL selection (`DISP-SHRRM`).

---

### **Summary**

The `BB150P` OCL program orchestrates the printing of Bills of Lading by:
1. Checking for concurrent processes to avoid conflicts.
2. Initializing the environment and selecting a batch via `BB001`.
3. Processing BOL data using `BB150P` and `BB150`, with conditional job queue submission.
4. Releasing the batch using `BB005`.
5. Cleaning up and exiting.

The program uses a combination of user inputs (`?13?`, `?USER?`, `?WS?`), dynamic file naming (`?9?`, `?20?`), and conditional logic to manage the workflow. The files `BBBTCH`, `BBBTCHX`, `BICONT`, and `BBORTR` store batch and order data, while the programs `GSY2K`, `BB001`, `BB150P`, `BB150`, and `BB005` handle specific processing tasks.

If you need further clarification or analysis (e.g., specific logic flows or file structures), let me know!