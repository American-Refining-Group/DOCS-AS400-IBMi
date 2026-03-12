The provided document is an **OCL (Operation Control Language)** script, typically used on IBM midrange systems like the IBM System/36 or AS/400 (now IBM i). This script, `AR900.ocl36.txt`, appears to control a **Customer Master Add/Update** process, likely written in RPG (Report Program Generator) or a related language. Below, I will explain the process steps of the OCL program, list the external programs called, and identify the tables (files) used.

---

### **Process Steps of the OCL Program**

OCL scripts are used to define the environment, specify files, and control the execution of programs on IBM midrange systems. The steps in this script can be broken down based on the commands and directives present:

1. **Environment Setup and Parameters**:
   - `SCPROCP ,,,,,,,,?9?`: This likely specifies the procedure name or parameters for the job, with `?9?` being a placeholder for a parameter passed at runtime (possibly a library or dataset identifier).
   - `GSY2K`: This might reference a system or configuration setting, possibly related to Y2K compliance or a specific system module.
   - `LOCAL OFFSET-480,DATA-'?9?'`: Sets a local variable or parameter at offset 480 with the value of the `?9?` placeholder.
   - `LOCAL OFFSET-410,DATA-'?WS?'`: Sets another local variable at offset 410 with the value `?WS?` (possibly a workstation ID or user-specific data).
   - `LOCAL OFFSET-400,DATA-'?USER?'`: Sets a local variable at offset 400 with the value `?USER?` (likely the user ID running the job).

2. **Conditional Logic and Index Building**:
   - `IFF ACTIVE-AR300 IFF DATAF1-?9?CRDETX BLDINDEX ?9?CRDETX,2,8,+ ?9?ARDETL,,DUPKEY,,71,8,10,7`:
     - This is a conditional statement (`IFF`) that checks if the program `AR300` is active and if the `DATAF1` field contains the value `?9?CRDETX`.
     - If true, it executes `BLDINDEX` to build an index for the file `?9?CRDETX`:
       - Key starts at position 2, length 8, in ascending order (`+`).
       - Another index is built for `?9?ARDETL` with `DUPKEY` (allowing duplicate keys), starting at position 71, with key lengths of 8, 10, and 7.
     - This step ensures that the necessary file indexes are created for efficient data access.

3. **Program Loading**:
   - `LOAD AR900`: Loads the main RPG program `AR900`, which handles the core customer master add/update logic.

4. **File Definitions**:
   - The script defines multiple files with the `FILE NAME` directive, specifying their labels (e.g., `?9?ARCUST`) and disposition (`DISP-SHR` for shared access). These files are used by the `AR900` program and other called programs for reading or updating customer-related data. The files are listed in the "Tables Used" section below.

5. **Execution of the Main Program**:
   - `RUN`: Executes the loaded `AR900` program, which likely performs the customer master add/update operations using the defined files.

6. **Support for Called Programs**:
   - The script specifies additional files for programs called by `AR900`, including `AR9009`, `AR9006`, `AR915P`, and `BI907`. These programs likely handle specific sub-tasks, such as additional file processing, validation, or reporting.

7. **Additional Procedure**:
   - `ARFX39 ,,,,,,,,?9?`: This line might initiate another procedure or job step, possibly a cleanup or follow-up process, with `?9?` as a parameter.

---

### **External Programs Called**

The OCL script references several external programs that are called by the main `AR900` program or as part of the process:

1. **AR900**: The main RPG program responsible for the customer master add/update logic.
2. **AR9009**: A supporting program, likely handling specific customer data processing or validation.
3. **AR9006**: Another supporting program, possibly related to updating or processing specific customer records.
4. **AR915P**: A program that might handle additional customer file maintenance or reporting.
5. **BI907**: Likely a program related to billing or shipping (based on the `SHIPTO` file reference).

---

### **Tables (Files) Used**

The OCL script lists several files (referred to as "tables" in some contexts) used by the programs. These files are defined with the `FILE NAME` directive and are accessed in shared mode (`DISP-SHR`). The `?9?` prefix suggests a library or dataset name provided at runtime. Below is the list of files:

1. **ARCUST** (`?9?ARCUST`): Primary customer master file.
2. **ARCUSP** (`?9?ARCUSP`): Possibly a secondary customer file or a suspense file for pending updates.
3. **ARCUPR** (`?9?ARCUPR`): Customer pricing or profile data, also referenced as `PFARCUPR` and `PTARCUPR` for specific programs.
4. **CRDETX** (`?9?CRDETX`): Credit details or transaction file, used for indexing.
5. **ARCONT** (`?9?ARCONT`): Customer contact information.
6. **BICONT** (`?9?BICONT`): Billing contact information.
7. **GSPROD** (`?9?GSPROD`): General system product file, possibly for product-related customer data.
8. **GSTABL** (`?9?GSTABL`): General system table, likely containing configuration or reference data.
9. **BISLTX** (`?9?BISLTX`): Billing or sales transaction file.
10. **ARCLGR** (`?9?ARCLGR`): Customer ledger or accounting file.
11. **ARCUST2** (`?9?ARCUST`): Likely an alias or secondary reference to the customer master file.
12. **ARCUSHS** (`?9?ARCUSHS`): Customer history or summary file.
13. **ARCUSPH** (`?9?ARCUSPH`): Customer profile history.
14. **ARCUPHS** (`?9?ARCUPHS`): Customer pricing history.
15. **SA5FIXD** (`?9?SA5FIXD`): Fixed data file for program `AR9009`, possibly for sales or customer data.
16. **SA5FIXM** (`?9?SA5FIXM`): Master fixed data file for `AR9009`.
17. **SA5BCXD** (`?9?SA5BCXD`): Billing or customer detail file for `AR9009`.
18. **SA5BCXM** (`?9?SA5BCXM`): Master billing/customer file for `AR9009`.
19. **SA5DBXD** (`?9?SA5DBXD`): Debit or detail file for `AR9009`.
20. **SA5DBXM** (`?9?SA5DBXM`): Master debit file for `AR9009`.
21. **SA5COXD** (`?9?SA5COXD`): Customer order detail file for `AR9009`.
22. **SA5COXM** (`?9?SA5COXM`): Master customer order file for `AR9009`.
23. **ARCUFM** (`?9?ARCUFM`): Customer file maintenance for `AR915P`.
24. **ARCUFMX** (`?9?ARCUFMX`): Extended customer file maintenance for `AR915P`.
25. **SHIPTO** (`?9?SHIPTO`): Shipping address file for `BI907`.
26. **ARCUP3** (`?9?ARCUP3`): Additional customer pricing or profile file for `BI907`.

---

### **Summary**

- **Process Steps**: The OCL script sets up the environment, defines parameters, conditionally builds file indexes, loads the `AR900` program, specifies files, and executes the program. It also supports additional programs for specific tasks.
- **External Programs Called**: `AR900`, `AR9009`, `AR9006`, `AR915P`, `BI907`.
- **Tables Used**: 26 files, including customer master (`ARCUST`), credit details (`CRDETX`), pricing (`ARCUPR`), contact (`ARCONT`), and various sales/customer files for supporting programs.

This OCL script orchestrates a complex customer master maintenance process, ensuring proper file access and program execution for updating customer records on an IBM midrange system. If you need further details about specific files or programs, let me know!