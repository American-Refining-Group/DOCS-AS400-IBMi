The provided document is an OCL (Operation Control Language) program, `BB956P.ocl36.txt`, used on IBM midrange systems like the AS/400 (now IBM i) to control job execution and file operations. Below is an explanation of the process steps, the external programs called, and the tables (files) used in this program.

---

### **Process Steps of the RPG .OCL Program**

The OCL program `BB956P` updates a rack pricing file with new prices and prints the changes. It involves file manipulation, conditional logic, and calls to external programs based on specific conditions. Here’s a step-by-step breakdown of the process:

1. **Initial File Deletion and Clear**:
   - `// IF DATAF1-?9?BB956S DELETE ?9?BB956S,F1`:
     - Checks if the file `?9?BB956S` (likely a pricing file with a prefix `?9?` for library or environment) exists in the `DATAF1` file.
     - If it exists, the file `?9?BB956S` is deleted to prepare for new data.
   - `CLRPFM FILE(?9?NWPROD)`:
     - Clears the physical file `?9?NWPROD` (likely a new product or pricing file) to ensure it’s empty before new data is loaded.

2. **Create Customer List from PRCTUM**:
   - `// LOAD BB9541`:
     - Loads and executes the program `BB9541`, which processes the customer master file to create a customer list.
   - `// FILE NAME-PRCTUM,LABEL-?9?PRCTUM,DISP-SHR`:
     - Specifies the input file `PRCTUM` (likely a customer master file) with the label `?9?PRCTUM` (prefixed by the library/environment `?9?`) and shared access (`DISP-SHR`).
   - `// FILE NAME-BB954S,LABEL-?9?BB956S,RECORDS-999000,RETAIN-T`:
     - Specifies the output file `BB954S` (aliased to `?9?BB956S`), which will store the processed customer list with a capacity of up to 999,000 records. The `RETAIN-T` parameter indicates the file is temporary and retained after the job ends.
   - `// RUN`:
     - Executes the `BB9541` program to process `PRCTUM` and generate the customer list in `?9?BB956S`.

3. **Load and Process Pricing Data**:
   - `// LOAD BB956P`:
     - Loads the main program `BB956P` (likely an RPG program) to perform the core pricing update logic.
   - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHR`:
     - Specifies the input file `BICONT` (likely a control or configuration file) with the label `?9?BICONT` and shared access.
   - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`:
     - Specifies the input file `ARCUST` (likely an accounts receivable or customer file) with the label `?9?ARCUST` and shared access.
   - `// FILE NAME-BB956S,LABEL-?9?BB956S,DISP-SHR`:
     - Specifies the file `BB956S` (the customer list created by `BB9541`) as input with shared access.
   - `// RUN`:
     - Executes the `BB956P` program to process the pricing updates using `BICONT`, `ARCUST`, and `BB956S`.

4. **Conditional Program Execution**:
   - `// IF ?L'129,6'?/CANCEL GOTO END`:
     - Checks a local variable or condition at position 129,6 (likely a return code or status). If the condition is `CANCEL`, the program jumps to the `END` tag, terminating the job.
   - `// IF SWITCH1-0 GOTO END`:
     - Checks if `SWITCH1` (a system or program switch) is set to `0`. If true, the program jumps to the `END` tag, terminating the job.
   - `// IF SWITCH1-1 BB9563 ,,,,,,,,?9?`:
     - If `SWITCH1` is set to `1`, the program `BB9563` is called with the parameter `?9?` (likely the library/environment prefix).
   - `// IF SWITCH2-1 BB9564 ,,,,,,,,?9?`:
     - If `SWITCH2` is set to `1`, the program `BB9564` is called with the parameter `?9?`.
   - `// IF ?L'207,1'?/Y BB9565 ,,,,,,,,?9?`:
     - Checks a local variable or condition at position 207,1. If the value is `Y`, the program `BB9565` is called with the parameter `?9?`.
   - `// BB9566 ,,,,,,,,?9?`:
     - Unconditionally calls the program `BB9566` with the parameter `?9?`.

5. **Program Termination**:
   - `// TAG END`:
     - Marks the `END` label where the program jumps if certain conditions are met (e.g., `CANCEL` or `SWITCH1-0`).
   - `// LOCAL BLANK-*ALL`:
     - Clears all local variables used in the job to ensure no residual data remains.
   - `// SWITCH 00000000`:
     - Resets all switches (e.g., `SWITCH1`, `SWITCH2`) to `0`, ensuring a clean state for the next run.

---

### **External Programs Called**

The OCL program calls the following external programs:
1. `BB9541`: Creates a customer list from the `PRCTUM` file and outputs it to `BB956S`.
2. `BB956P`: The main program that processes pricing updates using `BICONT`, `ARCUST`, and `BB956S`.
3. `BB9563`: Called conditionally if `SWITCH1` is `1`.
4. `BB9564`: Called conditionally if `SWITCH2` is `1`.
5. `BB9565`: Called conditionally if the condition at `?L'207,1'?` is `Y`.
6. `BB9566`: Called unconditionally at the end of the program.

---

### **Tables (Files) Used**

The program interacts with the following files:
1. **Input Files**:
   - `PRCTUM` (labeled `?9?PRCTUM`): Customer master file used by `BB9541` to create the customer list.
   - `BICONT` (labeled `?9?BICONT`): Control or configuration file used by `BB956P`.
   - `ARCUST` (labeled `?9?ARCUST`): Accounts receivable or customer file used by `BB956P`.
   - `BB956S` (labeled `?9?BB956S`): Customer list file created by `BB9541` and used as input by `BB956P`.

2. **Output/Cleared Files**:
   - `NWPROD` (labeled `?9?NWPROD`): New product or pricing file cleared at the start of the program.
   - `BB956S` (labeled `?9?BB956S`): Output file for the customer list created by `BB9541`, also used as input by `BB956P`.

3. **Deleted Files** (if they exist):
   - `BB956S` (labeled `?9?BB956S`): Deleted if it exists in `DATAF1` at the start of the program.

---

### **Summary**

The `BB956P.ocl36` program orchestrates a process to update rack pricing and print changes. It starts by clearing and deleting existing files, creates a customer list using `BB9541`, processes pricing updates with `BB956P`, and then conditionally calls additional programs (`BB9563`, `BB9564`, `BB9565`) based on switches and conditions, followed by an unconditional call to `BB9566`. The program uses files `PRCTUM`, `BICONT`, `ARCUST`, `BB956S`, and `NWPROD` to manage customer and pricing data. The use of switches and local variables allows flexible control over the execution flow.