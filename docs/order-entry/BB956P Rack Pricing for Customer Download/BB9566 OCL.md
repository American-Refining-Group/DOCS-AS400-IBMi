The provided document is an OCL (Operation Control Language) program, `BB9566.ocl36.txt`, used on IBM midrange systems like the AS/400 (now IBM i). This program is called unconditionally from the main OCL program `BB956P.ocl36.txt` as the final step in the rack pricing update process. It appears to be responsible for generating or updating a pricing download file. Below is an explanation of the process steps, external programs called, and tables (files) used.

---

### **Process Steps of the OCL Program `BB9566`**

The `BB9566` OCL program is straightforward and serves to clear a file and execute an RPG (or similar) program with specified input and output files. Here’s a step-by-step breakdown of the process:

1. **Clear Output File**:
   - `CLRPFM FILE(?9?PRCDOWN)`:
     - Clears the physical file `?9?PRCDOWN` (likely a pricing download file) to ensure it is empty before new data is written to it. The `?9?` prefix indicates the library or environment.

2. **Load the Program**:
   - `// LOAD BB9566`:
     - Loads the program `BB9566`, which is likely an RPG program or another executable on the system, responsible for processing rack pricing data and generating output for download.

3. **File Definitions**:
   - The program specifies three files to be used by `BB9566`:
     - `// FILE NAME-RKPRCE,LABEL-?9?RKPRCE,DISP-SHR`:
       - Defines the `RKPRCE` file (rack pricing file) with the label `?9?RKPRCE` and shared access (`DISP-SHR`).
     - `// FILE NAME-PRCTUM,LABEL-?9?PRCTUM,DISP-SHR`:
       - Defines the `PRCTUM` file (customer master file) with the label `?9?PRCTUM` and shared access (`DISP-SHR`).
     - `// FILE NAME-RKPRCEO,LABEL-?9?PRCDOWN,DISP-SHR`:
       - Defines the `RKPRCEO` file (aliased as `?9?PRCDOWN`, the pricing download file) with shared access (`DISP-SHR`).

4. **Execute the Program**:
   - `// RUN`:
     - Executes the `BB9566` program, which processes the input files `RKPRCE` and `PRCTUM` and writes output to the `RKPRCEO` (`?9?PRCDOWN`) file.

5. **Program Termination**:
   - The OCL script ends after the `RUN` command, with no additional logic or conditional processing defined.

---

### **Business Rules (Inferred)**

Since the OCL program itself does not contain explicit business logic (it only sets up and runs `BB9566`), the business rules are inferred based on its context within the rack pricing update process and the files involved:
1. **Purpose**:
   - The `BB9566` program likely processes data from the `RKPRCE` (rack pricing) and `PRCTUM` (customer master) files to generate or update a pricing download file (`RKPRCEO`/`?9?PRCDOWN`), which may be used for external systems or reporting.
2. **Input Processing**:
   - Uses `RKPRCE` for existing rack pricing data (e.g., company, location, product, container, unit of measure, price class, date).
   - Uses `PRCTUM` for customer-related data (e.g., company number, customer number, product code, container code, unit of measure).
3. **Output Generation**:
   - Clears the `?9?PRCDOWN` file before processing to ensure a fresh dataset.
   - Writes output to `RKPRCEO` (`?9?PRCDOWN`), which likely contains formatted pricing data for download or integration with other systems.
4. **Shared Access**:
   - All files are accessed in shared mode (`DISP-SHR`), indicating that multiple processes can read these files simultaneously.

---

### **External Programs Called**

The OCL program explicitly calls the following external program:
1. **BB9566**:
   - Loaded and executed with the `LOAD` and `RUN` commands, this program performs the core processing logic using the `RKPRCE`, `PRCTUM`, and `RKPRCEO` files.

---

### **Tables (Files) Used**

The program interacts with the following files:
1. **Input Files**:
   - `RKPRCE` (labeled `?9?RKPRCE`): Rack pricing file, likely containing company number, location, price class, product code, container code, unit of measure, and date.
   - `PRCTUM` (labeled `?9?PRCTUM`): Customer master file, containing customer numbers, company numbers, product codes, container codes, and unit of measure.
2. **Output File**:
   - `RKPRCEO` (labeled `?9?PRCDOWN`): Pricing download file, used to store the processed output, cleared at the start of the program.

---

### **Summary**

The `BB9566.ocl36` program is a simple OCL script that clears the `?9?PRCDOWN` file and then loads and executes the `BB9566` program as part of the rack pricing update process. It specifies two input files (`RKPRCE` and `PRCTUM`) and one output file (`RKPRCEO`/`?9?PRCDOWN`), all accessed in shared mode. The program is called unconditionally from the main OCL (`BB956P.ocl36`) as the final step. The only external program called is `BB9566`, and the files used are `RKPRCE`, `PRCTUM`, and `RKPRCEO` (`?9?PRCDOWN`). The specific processing logic (e.g., generating the download file) is handled by the `BB9566` program, which is not detailed in the OCL.