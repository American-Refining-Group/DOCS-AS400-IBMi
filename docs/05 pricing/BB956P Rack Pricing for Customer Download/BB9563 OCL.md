The provided document is an OCL (Operation Control Language) program, `BB9563.ocl36.txt`, used on IBM midrange systems like the AS/400 (now IBM i). This program is called conditionally from the main OCL program `BB956P.ocl36.txt` when `SWITCH1` is set to `1`. It appears to be part of the rack pricing update process. Below is an explanation of the process steps, external programs called, and tables (files) used.

---

### **Process Steps of the OCL Program `BB9563`**

The `BB9563` OCL program is straightforward and primarily serves to execute an RPG program (or similar) with specified input and output files. Here’s a step-by-step breakdown of the process:

1. **Load the Program**:
   - `// LOAD BB9563`:
     - Loads the program `BB9563`, which is likely an RPG program or another executable on the system, responsible for processing rack pricing data.

2. **File Definitions**:
   - The program specifies several files to be used by `BB9563`:
     - `// FILE NAME-PRCTUM,LABEL-?9?PRCTUM,DISP-SHR`:
       - Defines the `PRCTUM` file (customer master file) with the label `?9?PRCTUM` (prefixed by the library/environment `?9?`) and shared access (`DISP-SHR`).
     - `// FILE NAME-RKPRCE,LABEL-?9?RKPRCE,DISP-SHR`:
       - Defines the `RKPRCE` file (likely the rack pricing file) with the label `?9?RKPRCE` and shared access.
     - `// FILE NAME-GSCNTR,LABEL-?9?GSCNTR,DISP-SHR`:
       - Defines the `GSCNTR` file (possibly a container or product control file) with the label `?9?GSCNTR` and shared access.
     - `// FILE NAME-GSCNTR1,LABEL-?9?GSCNTR1,DISP-SHR`:
       - Defines the `GSCNTR1` file (likely a secondary container or product file) with the label `?9?GSCNTR1` and shared access.
     - `// FILE NAME-NEWPROD,LABEL-?9?NWPROD,RETAIN-T,RECORDS-999000`:
       - Defines the `NEWPROD` file (new product or pricing file) with the label `?9?NWPROD`, temporary retention (`RETAIN-T`), and a capacity of up to 999,000 records.

3. **Execute the Program**:
   - `// RUN`:
     - Executes the `BB9563` program, which processes the input files (`PRCTUM`, `RKPRCE`, `GSCNTR`, `GSCNTR1`) and writes output to the `NEWPROD` file.

4. **Program Termination**:
   - The OCL script ends after the `RUN` command, with no additional logic or conditional processing defined.

---

### **Business Rules (Inferred)**

Since the OCL program itself does not contain explicit business logic (it only sets up and runs `BB9563`), the business rules are inferred based on its context within the rack pricing update process and the files involved:
1. **Purpose**:
   - The program likely processes customer and pricing data to update or generate new rack pricing records, storing the results in `NEWPROD`.
2. **Input Processing**:
   - Uses `PRCTUM` for customer data, `RKPRCE` for existing rack prices, and `GSCNTR`/`GSCNTR1` for container or product-related data.
3. **Output Generation**:
   - Produces or updates the `NEWPROD` file, which is likely used for downstream processing or reporting in the rack pricing workflow.
4. **Shared Access**:
   - All input files are accessed in shared mode (`DISP-SHR`), indicating that multiple processes can read these files simultaneously.
5. **Temporary Output**:
   - The `NEWPROD` file is marked as temporary (`RETAIN-T`), suggesting it is used for intermediate results within the job.

---

### **External Programs Called**

The OCL program explicitly calls the following external program:
1. **BB9563**:
   - Loaded and executed with the `LOAD` and `RUN` commands, this program performs the core processing logic using the specified input and output files.

---

### **Tables (Files) Used**

The program interacts with the following files:
1. **Input Files**:
   - `PRCTUM` (labeled `?9?PRCTUM`): Customer master file, likely containing customer numbers, company numbers, and related data.
   - `RKPRCE` (labeled `?9?RKPRCE`): Rack pricing file, containing existing pricing data.
   - `GSCNTR` (labeled `?9?GSCNTR`): Container or product control file, possibly containing container codes or product details.
   - `GSCNTR1` (labeled `?9?GSCNTR1`): Secondary container or product file, likely containing additional or related product data.
2. **Output File**:
   - `NEWPROD` (labeled `?9?NWPROD`): New product or pricing file, used to store the results of the program’s processing, with a capacity of up to 999,000 records and temporary retention.

---

### **Summary**

The `BB9563.ocl36` program is a simple OCL script that loads and executes the `BB9563` program as part of the rack pricing update process. It specifies input files (`PRCTUM`, `RKPRCE`, `GSCNTR`, `GSCNTR1`) for customer, pricing, and product/container data, and an output file (`NEWPROD`) for the processed results. The program is called conditionally from the main OCL (`BB956P.ocl36`) when `SWITCH1` is set to `1`. The only external program called is `BB9563`, and the files used are `PRCTUM`, `RKPRCE`, `GSCNTR`, `GSCNTR1`, and `NEWPROD`.