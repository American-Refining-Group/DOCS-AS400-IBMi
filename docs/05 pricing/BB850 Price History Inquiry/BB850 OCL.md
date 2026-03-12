### Process Steps of the RPG .OCL Program

This .OCL file (BB850.ocl36.txt) appears to be an Operations Control Language (OCL) procedure used on IBM midrange systems (e.g., AS/400 or iSeries) to set up and execute an RPG (Report Program Generator) program named BB850, which is described as a "PRICE HISTORY INQUIRY" tool. OCL is not the RPG program itself but a scripting language that prepares the runtime environment, loads the program, assigns files (which act as database tables or data sources), and initiates execution. The OCL procedure handles setup for both the main program (BB850) and a called subprogram (BB1016).

The process steps, based on the OCL syntax and structure, are as follows:

1. **Procedure Initialization**: 
   - The procedure begins with `// SCPROCP ,,,,,,,,?9?`, which likely invokes a system procedure or sets up processing parameters (the placeholders like `,,,,,,,?9?` may represent default or variable values for job control).
   - Followed by `// GSY2K`, which could load a library or module related to Y2K compliance (common in legacy systems for date handling).

2. **Program Loading**:
   - `// LOAD BB850` loads the main RPG program BB850 into memory for execution.

3. **File Assignments for the Main Program (BB850)**:
   - A series of `// FILE` statements assign physical files (database files or tables) to logical names or labels, with `DISP-SHR` indicating shared access (allowing concurrent read access by other processes).
   - This step maps external data sources to the program, enabling BB850 to read from these files during its price history inquiry logic (e.g., querying historical pricing data).

4. **File Assignments for the Called Program (BB1016)**:
   - Marked by the comment `** FOR CALLED PGM: BB1016`, additional `// FILE` statements assign files specifically for the subprogram BB1016, which is invoked by BB850.
   - This ensures the called program has its own file mappings, potentially for modular operations like data updates or calculations related to the inquiry.

5. **Program Execution**:
   - `// RUN` initiates the execution of the loaded program BB850.
   - During runtime, BB850 would perform its RPG logic (e.g., reading files, processing data, generating reports or inquiries), and call BB1016 as needed.
   - The procedure ends after this, with the system handling any output or termination.

Overall, this OCL acts as a job control script to orchestrate the RPG program's environment. The actual RPG code (in BB850 and BB1016) isn't provided here, so the "process steps" are inferred from the OCL setup. The focus is on data access for a price history inquiry, likely involving customer, product, or contract data based on file names.

### External Programs Called

- BB1016: This is explicitly noted as a called program (subprogram) within the OCL, invoked by the main program BB850. No other external programs are referenced.

### Tables Used

In this context, "tables" refer to the database files assigned via the `// FILE` statements. These are physical or logical files on the system, treated as tables for data storage and retrieval. The files are listed below, grouped by their association (main program or called program). Note that some files have asterisks (**) in the original, possibly indicating comments or special notations, but they are still valid assignments. File names like GSTABL and GSTABL6 may represent aliases or versions of the same underlying table.

#### For Main Program (BB850):
- BBPRCE (likely a price history file)
- BICUAX (possibly a customer or index file)
- BICUAY (possibly another customer or auxiliary file)
- BICONT (possibly a contract file)
- INLOC (possibly an inventory location file)
- GSTABL (possibly a general system table)
- GSPROD (possibly a product file)
- GSCTUM (possibly a customer or unit master file)

#### For Called Program (BB1016):
- ARCUP16 (possibly an archive or update file for program version 16)
- ARCUP36 (possibly an archive or update file for program version 36)
- GSTABL6 (possibly a versioned general system table)
- GSPRD6 (possibly a versioned product file)