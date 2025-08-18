### Process Steps of the RPG .OCL Program

This .OCL file (GS973.ocl36.txt) appears to be an Operation Control Language (OCL) procedure for IBM System/36 (indicated by the .ocl36 extension and syntax), designed to set up and execute an RPG (Report Program Generator) program named GS973. The OCL acts as a job control script to prepare the environment for running the RPG program, which handles "Product / Container Master Maintenance" (likely CRUD operations on product and container data masters). The OCL procedure is named SCPROCP and accepts parameters (with the 9th parameter, ?9?, used as a variable placeholder, probably for a library prefix).

The process steps executed by this OCL procedure are as follows:

1. **Procedure Initialization and Parameter Handling**: The procedure starts with SCPROCP and a parameter list (,,,,,,,,?9?), where empty commas represent default or unused parameters, and ?9? is a substitutable variable (likely user-supplied at runtime, e.g., a library name like 'PROD' or 'TEST').

2. **Environment Setup via GSY2K**: The statement `// GSY2K` invokes or includes a separate procedure or command named GSY2K. This is likely a setup routine (possibly related to Year 2000 compliance, given the "Y2K" naming convention), which configures the runtime environment, such as date formats, global variables, or system settings before the main program loads.

3. **Load the Main RPG Program**: The command `// LOAD GS973` loads the compiled RPG program GS973 into memory, preparing it for execution.

4. **File Definitions and Assignments**: Four `// FILE` statements define and assign database files to the program, specifying their labels (physical file names prefixed with the ?9? parameter) and disposition (DISP-SHR for shared access, allowing concurrent read/write by other jobs):
   - `// FILE NAME-PRCNTR,LABEL-?9?PRCNTR,DISP-SHR`: Assigns logical file PRCNTR to physical file ?9?PRCNTR.
   - `// FILE NAME-GSPROD,LABEL-?9?GSPROD,DISP-SHR`: Assigns logical file GSPROD to physical file ?9?GSPROD.
   - `// FILE NAME-GSCNTR,LABEL-?9?GSCNTR,DISP-SHR`: Assigns logical file GSCNTR to physical file ?9?GSCNTR.
   - `// FILE NAME-INCONT,LABEL-?9?INCONT,DISP-SHR`: Assigns logical file INCONT to physical file ?9?INCONT.
   
   These files are opened in shared mode, enabling the RPG program to access them for reading/writing data related to product and container maintenance.

5. **Execute the Program**: The `// RUN` command initiates the execution of the loaded GS973 RPG program, which performs the core business logic (e.g., updating master records) using the defined files.

The OCL procedure ends after the program run, with any output or errors handled by the system. Note that the actual internal logic of the RPG program GS973 (e.g., calculations, screen interactions, or reports) is not detailed in the OCL; this file only orchestrates its execution.

### External Programs Called

- **GS973**: The primary RPG program loaded and executed for the maintenance tasks. This is the main external program being invoked.
- **GSY2K**: Likely a separate OCL procedure or utility program called at the start for environment setup (e.g., Y2K date handling). No other details are provided in the file.

No additional subprograms or calls are explicitly mentioned in the OCL.

### Tables Used

The following database files (tables) are defined and used by the RPG program GS973. These are likely keyed physical or logical files in the System/36 database, containing data for products and containers:

| File Name (Logical) | Physical Label | Description/Purpose (Inferred) | Access Mode |
|---------------------|----------------|--------------------------------|-------------|
| PRCNTR             | ?9?PRCNTR     | Likely a product-container relation or cross-reference table. | Shared (DISP-SHR) |
| GSPROD             | ?9?GSPROD     | Product master table, storing core product details. | Shared (DISP-SHR) |
| GSCNTR             | ?9?GSCNTR     | Container master table, storing container specifications. | Shared (DISP-SHR) |
| INCONT             | ?9?INCONT     | Possibly an input control or inventory container table for incoming data. | Shared (DISP-SHR) |

These files are prefixed with the runtime parameter ?9? (e.g., if ?9? is 'LIB/', the physical file for PRCNTR would be 'LIB/PRCNTR'). The RPG program would perform operations like chain, read, update, or write on these files.