The provided OCL program, `AP256.ocl36.txt`, is a component of an IBM System/36 or AS/400 Accounts Payable (A/P) system, called from the main OCL program (`AP250.ocl36.txt`). Its purpose is to process temporary data files related to A/P transactions and generate multiple reports. Below, I will explain the **process steps**, **business rules**, **tables (files) used**, and **external programs called** based on the provided OCL code.

---

### **Process Steps of the AP256 OCL Program**

The `AP256.ocl36.txt` OCL program is conditionally executed if a specific temporary file (`?9?APDT?WS?`) exists and has a non-zero size, as checked in the main OCL program (`AP250.ocl36.txt`). It involves two programs, `AP256A` and `AP256`, to process data and produce reports. Here are the detailed process steps:

1. **Conditional File Creation**:
   - **Check for File Existence**: The program checks if the temporary file `?9?APDT?WS?` exists and has a non-zero size (this check is performed in `AP250.ocl36.txt` before reaching this block).
   - **Build Indexed File**:
     - If the condition `DATAF1-?9?APDT?WS?C` is true (indicating the file exists or needs creation), the `BLDFILE` command creates an indexed file:
       - File: `?9?APDT?WS?C`.
       - Type: Indexed (`I`).
       - Capacity: 500 records.
       - Record length: 10 bytes.
       - Key fields: Positions 2-7 (likely company and vendor numbers or similar identifiers).
       - File type: `DFILE` (data file).
   - This file (`APDTWSC`) is used for temporary storage of processed data.

2. **Execute AP256A Program**:
   - **Load Program**: The `LOAD AP256A` command loads the `AP256A` program.
   - **File Assignments**:
     - **APDTWS** (`?9?APDT?WS?`): Temporary data file, likely containing raw A/P transaction data.
     - **APDTWSC** (`?9?APDT?WS?C`, `RETAIN-T`, `RECORDS-50`): Indexed temporary file created in the previous step, retained temporarily with a capacity of 50 records.
     - **APVNFMX** (`?9?APVNFMX`, `DISP-SHR`): Vendor master extension file, accessed in shared mode for reference data.
   - **Run Program**: The `RUN` command executes `AP256A`, which likely processes `APDTWS`, populates `APDTWSC` with indexed data, and uses `APVNFMX` for vendor-related information (e.g., extended vendor attributes or payment terms).

3. **Execute AP256 Program**:
   - **Load Program**: The `LOAD AP256` command loads the `AP256` program.
   - **File Assignments**:
     - **APDTWS** (`?9?APDT?WS?`): Same temporary data file as in `AP256A`.
     - **APDTWSC** (`?9?APDT?WS?C`, `DISP-SHR`): Indexed temporary file, now accessed in shared mode.
     - **APVEND** (`?9?APVEND`, `DISP-SHR`): Primary vendor master file, containing vendor details (e.g., name, address).
     - **APCONT** (`?9?APCONT`, `DISP-SHR`): A/P control file, containing company-level control data (e.g., company name, next check number).
     - **APVNFMX** (`?9?APVNFMX`, `DISP-SHR`): Vendor master extension file, reused for additional vendor data.
   - **Printer Overrides**:
     - Overrides output queues for four report files (`REPORT1`, `REPORT2`, `REPORT3`, `REPORT4`) based on the condition `?9?/G`:
       - If `?9?/G` is true, directs reports to `QUSRSYS/APACHOUTQ` (standard A/P output queue).
       - If `?9?/G` is false, directs reports to `QUSRSYS/TESTOUTQ` (test output queue).
   - **Run Program**: The `RUN` command executes `AP256`, which processes the temporary files (`APDTWS`, `APDTWSC`), references vendor and control data (`APVEND`, `APCONT`, `APVNFMX`), and generates up to four reports (`REPORT1` to `REPORT4`).

4. **Completion**:
   - After `AP256A` and `AP256` complete, the OCL script ends, and control returns to the main OCL program (`AP250.ocl36.txt`).
   - The temporary files (`APDTWS`, `APDTWSC`) are later deleted by `GSDELETE` commands in the main OCL program.

---

### **Business Rules**

The OCL script itself is minimal, so business rules are inferred from its structure and context within the A/P system:

1. **Conditional Execution**:
   - The program only runs if the temporary file `?9?APDT?WS?` exists and has data (checked in `AP250.ocl36.txt`).
   - The creation of `APDTWSC` is conditional on the existence of `DATAF1-?9?APDT?WS?C`, ensuring resources are allocated only when necessary.

2. **Temporary File Management**:
   - Creates an indexed file (`APDTWSC`) with a small record length (10 bytes) and a key spanning positions 2-7, suggesting it stores summarized or indexed A/P transaction data (e.g., company and vendor keys).
   - `APDTWSC` is initially created with a capacity of 500 records but retained with only 50 records in `AP256A`, indicating a compact dataset for processing.

3. **Shared File Access**:
   - Files `APVNFMX`, `APVEND`, and `APCONT` are opened with shared access (`DISP-SHR`), supporting a multi-user environment where other processes may access these files concurrently.

4. **Report Output Control**:
   - Directs reports to either a production output queue (`QUSRSYS/APACHOUTQ`) or a test queue (`QUSRSYS/TESTOUTQ`) based on the `?9?/G` condition, allowing for testing or production environments.
   - Supports up to four reports (`REPORT1` to `REPORT4`), suggesting multiple types of output (e.g., vendor payment summaries, transaction details, or reconciliation reports).

5. **Dependency on Main OCL**:
   - Relies on the main OCL program (`AP250.ocl36.txt`) to provide the temporary file `APDTWS` and trigger execution.
   - Assumes `APVNFMX`, `APVEND`, and `APCONT` are populated with valid vendor and control data.

6. **No User Interaction**:
   - The script runs automatically without prompts, indicating it is part of an automated A/P workflow, possibly triggered in auto mode (`?2?/AUTO` in `AP250.ocl36.txt`).

7. **Error Handling**:
   - No explicit error handling is included in the OCL script, so errors (e.g., missing files, invalid data) are assumed to be handled within the `AP256A` and `AP256` programs.

---

### **Tables (Files) Used**

The OCL program references the following files:

1. **APDTWS** (`?9?APDT?WS?`):
   - Temporary data file, likely containing raw A/P transaction data generated by prior steps in the main OCL program.
   - Used as input by both `AP256A` and `AP256`.

2. **APDTWSC** (`?9?APDT?WS?C`, `RETAIN-T`, `RECORDS-50` in `AP256A`, `DISP-SHR` in `AP256`):
   - Indexed temporary file, created with 500 records and a 10-byte record length, with a key in positions 2-7.
   - Populated by `AP256A` and used as input/output by `AP256`.

3. **APVNFMX** (`?9?APVNFMX`, `DISP-SHR`):
   - Vendor master extension file, containing additional vendor data (e.g., extended attributes, payment terms).
   - Accessed in shared mode by both `AP256A` and `AP256`.

4. **APVEND** (`?9?APVEND`, `DISP-SHR`):
   - Primary vendor master file, containing vendor details (e.g., name, address, balance).
   - Used by `AP256` for reference data.

5. **APCONT** (`?9?APCONT`, `DISP-SHR`):
   - A/P control file, containing company-level data (e.g., company name, next check number).
   - Used by `AP256` for reference data.

---

### **External Programs Called**

The OCL program explicitly calls the following external programs:

1. **AP256A**:
   - Loaded and executed to process `APDTWS`, populate `APDTWSC`, and use `APVNFMX` for vendor data.
   - Likely an RPG program that performs initial data processing or indexing.

2. **AP256**:
   - Loaded and executed to process `APDTWS` and `APDTWSC`, reference `APVEND`, `APCONT`, and `APVNFMX`, and generate reports (`REPORT1` to `REPORT4`).
   - Likely an RPG program that produces detailed A/P reports.

No other external programs are called by this OCL script.

---

### **Summary**

The `AP256.ocl36.txt` OCL program is a conditional component of the A/P system, responsible for processing temporary transaction data and generating reports. Its key functions are:

- **Process Steps**:
  - Conditionally creates an indexed temporary file (`APDTWSC`).
  - Executes `AP256A` to process `APDTWS` and populate `APDTWSC` using `APVNFMX`.
  - Executes `AP256` to process `APDTWS` and `APDTWSC`, reference `APVEND` and `APCONT`, and generate up to four reports directed to production or test output queues.

- **Business Rules**:
  - Runs only if `APDT?WS?` exists with data.
  - Creates and uses a compact indexed file (`APDTWSC`) for efficient processing.
  - Supports shared file access for multi-user environments.
  - Directs reports to production or test queues based on the `?9?/G` condition.
  - Integrates with the main OCL program for automated A/P processing.

- **Tables Used**:
  - `APDTWS`: Temporary transaction data file.
  - `APDTWSC`: Indexed temporary file.
  - `APVNFMX`: Vendor master extension file.
  - `APVEND`: Primary vendor master file.
  - `APCONT`: A/P control file.

- **External Programs Called**:
  - `AP256A`: Processes temporary data and populates `APDTWSC`.
  - `AP256`: Generates reports using temporary and master files.

The program is tightly integrated with the main OCL program (`AP250.ocl36.txt`) and relies on prior steps to provide valid data in `APDTWS`. If you have the RPG source code for `AP256A` or `AP256`, or additional details about the reports or `APDTWS` structure, I can provide a deeper analysis of the processing logic. Please let me know if you need further clarification!