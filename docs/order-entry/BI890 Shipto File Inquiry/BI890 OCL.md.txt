The provided document is an Operation Control Language (OCL) program, specifically `BI890.ocl36.txt`, used to manage a "Ship To Master Inquiry" process in an IBM System/36 or AS/400 environment. OCL is a control language used to define job steps, file operations, and program execution in these systems. Below, I’ll explain the process steps of the OCL program, list the external programs called, and identify the tables (files) used.

---

### **Process Steps of the OCL Program**

The OCL program orchestrates a sequence of operations to perform a ship-to master inquiry, likely for retrieving and processing customer or shipping-related data. Here’s a step-by-step breakdown of the process:

1. **Initial Setup and Program Invocation**:
   - `CALL STRPCOCLP`:
     - This invokes the `STRPCOCLP` program, which is likely a system or utility program to initialize the environment or set up processing parameters for the job. It may handle tasks like setting up the library list or initializing system variables.
   - `CALL BI890P PARM('?9?' 'INQ')`:
     - The main program `BI890P` is called with two parameters: `'?9?'` (likely a placeholder for a library or file prefix) and `'INQ'` (indicating an inquiry operation).
     - This suggests `BI890P` is the primary program responsible for the ship-to master inquiry logic, such as retrieving or displaying data based on the inquiry mode.

2. **Conditional Logic (Commented Out)**:
   - `// IF ?L'112,3'?/F03 GOTO END`:
     - This line is commented out (indicated by `//`), so it’s not executed. If active, it would check a condition based on a system variable `?L'112,3'?` (likely a status code or flag at position 112, length 3). If the condition matches `/F03`, the program would jump to the `END` tag, terminating execution.
     - Since it’s commented, this logic is currently bypassed.

3. **Local Offset Specification**:
   - `// LOCAL OFFSET-480,DATA-'?9?'`:
     - Defines a local variable or parameter with an offset of 480 bytes, initialized with the value `'?9?'`. This is likely used to pass the library or file prefix to subsequent programs or file operations, ensuring consistency in file access.

4. **File Declarations**:
   - The program declares multiple files to be used by the called programs. Each file is specified with a name, label (prefixed with `?9?` for dynamic library resolution), and disposition (`DISP-SHR` for shared access). These files are:
     - `SHIPTO`: Likely contains ship-to address or customer location data.
     - `ARCUST`: Customer master file, storing customer details.
     - `ARCUPR`: Customer pricing or profile data.
     - `BICONT`: Possibly a control or configuration file for the inquiry process.
     - `GSPROD`: Product master file, containing product details.
     - `GSTABL`: General system table, likely containing reference or lookup data.
     - `TRRTCD`: Territory or region code table, for geographic data.
     - `CUADR` and `CUADRRD`: Customer address files (possibly redundant or aliased for compatibility).
     - `BBORA1`, `BBORDH`, `BBSHSA1`: Files related to order processing (order details, order headers, and shipping data).
     - `SHIPTHS`, `ARCUPHS`: Historical or archived versions of ship-to and customer profile data.
   - All files are opened in shared mode (`DISP-SHR`), allowing multiple programs or jobs to access them concurrently without locking.

5. **Program Execution**:
   - `// RUN`:
     - This command triggers the execution of the programs and file operations defined earlier. It indicates that the environment is set up, files are allocated, and the called program (`BI890P`) should run with the specified files and parameters.
   - The `RUN` command is a standard OCL directive to execute the job step.

6. **Loop Control (Commented Out)**:
   - `// GOTO AGAIN`:
     - This line is commented out, but if active, it would create a loop by jumping back to the `AGAIN` tag (defined earlier with `// TAG AGAIN`). This could be used for iterative processing, such as handling multiple inquiries in a batch.
     - Since it’s commented, the program executes linearly without looping.

7. **End of Program**:
   - `// TAG END`:
     - Marks the end of the program. If the commented `GOTO END` were active, the program would jump here to terminate. Since it’s not active, the program naturally ends after the `RUN` step.

---

### **External Programs Called**

The OCL program explicitly calls the following external programs:
1. **STRPCOCLP**:
   - Likely a system or utility program to initialize the job environment or set up processing parameters.
2. **BI890P**:
   - The main program for the ship-to master inquiry, handling the core logic of retrieving or displaying data based on the `'INQ'` parameter.
3. **BI9002** (Implied):
   - Not explicitly called in the OCL but referenced in the file declarations (`FOR CALLED PGM: BI9002`). This suggests `BI890P` may call `BI9002` internally for additional processing, such as address validation or formatting.
4. **BB800E** (Implied):
   - Also not explicitly called but referenced in the file declarations (`FOR CALL TO BB800E`). This indicates `BI890P` or `BI9002` may call `BB800E` for order-related processing, such as retrieving order history or shipping details.

---

### **Tables (Files) Used**

The program uses the following files (tables), all accessed in shared mode (`DISP-SHR`):
1. **SHIPTO** (`?9?SHIPTO`): Ship-to address or customer location data.
2. **ARCUST** (`?9?ARCUST`): Customer master file.
3. **ARCUPR** (`?9?ARCUPR`): Customer pricing or profile data.
4. **BICONT** (`?9?BICONT`): Control or configuration data for the inquiry.
5. **GSPROD** (`?9?GSPROD`): Product master file.
6. **GSTABL** (`?9?GSTABL`): General system table for reference data.
7. **TRRTCD** (`?9?TRRTCD`): Territory or region code table.
8. **CUADR** (`?9?CUADR`): Customer address file (for `BI9002`).
9. **CUADRRD** (`?9?CUADR`): Alias or redundant customer address file (for `BI9002`).
10. **BBORA1** (`?9?BBORA1`): Order detail file (for `BB800E`).
11. **BBORDH** (`?9?BBORDH`): Order header file (for `BB800E`).
12. **BBSHSA1** (`?9?BBSHSA1`): Shipping data file (for `BB800E`).
13. **SHIPTHS** (`?9?SHIPTHS`): Historical ship-to data.
14. **ARCUPHS** (`?9?ARCUPHS`): Historical customer profile data.

---

### **Summary**

The `BI890.ocl36.txt` program is designed to perform a ship-to master inquiry by:
1. Initializing the environment with `STRPCOCLP`.
2. Calling the main program `BI890P` with parameters for inquiry mode.
3. Allocating multiple files for customer, product, order, and historical data.
4. Executing the inquiry process via the `RUN` command.
5. Potentially looping (though currently disabled) or terminating at the `END` tag.

**External Programs Called**:
- `STRPCOCLP`
- `BI890P`
- `BI9002` (implied)
- `BB800E` (implied)

**Tables Used**:
- `SHIPTO`, `ARCUST`, `ARCUPR`, `BICONT`, `GSPROD`, `GSTABL`, `TRRTCD`, `CUADR`, `CUADRRD`, `BBORA1`, `BBORDH`, `BBSHSA1`, `SHIPTHS`, `ARCUPHS`

This program is part of a larger system, likely for managing customer and order data in a business application, with modular calls to handle specific tasks like address lookup or order processing. If you need further details or analysis of specific components, let me know!