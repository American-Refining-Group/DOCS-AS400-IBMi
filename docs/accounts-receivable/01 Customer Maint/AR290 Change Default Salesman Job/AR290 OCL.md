The provided document is an Operations Control Language (OCL) program, specifically `AR290.ocl36.txt`, designed to update salesman codes for selected customers across various files in an accounts receivable (A/R) system. Below is a detailed explanation of the process steps, followed by a list of external programs called and tables/files used.

### Process Steps of the AR290 OCL Program:  This is called from a nightly Job not the 5250 menu.

The program updates salesman codes from an old code to a new code in multiple files, processes batches of orders, handles invoice transaction files, and generates a report. Here's a step-by-step breakdown of the process:

1. **Initial Validation Checks**:
   - The program starts by checking the existence or status of the `ARSLST` file (likely a salesman change transaction file) and the `SLSCHG` file.
   - If the `ARSLST` file does not exist or matches certain conditions (`DATAF1-?9?ARSLST'?/0000000`), the program skips to the `END` tag, effectively terminating.
   - If the `SLSCHG` file exists, the program also jumps to `END`, indicating that no processing is needed if a salesman change is already in progress or completed.

2. **GSY2K Section - Update Customer Files**:
   - **Objective**: Update selected customers' salesman codes in multiple files: `ARCUST`, `BBORDR`, `BBBOL`, `SA5FIL`, `SA5DBB`, `SA5BCM`, `SA5COP`, `SA5SHP`, and `FRBINH`.
   - **Process**:
     - Loads the `AR290` program.
     - Defines and opens multiple files with shared memory access (`DISP-SHRMM`):
       - `ARSLST`: Salesman change transaction file.
       - `ARCUST`: Customer master file.
       - `ARCUSHS`: Customer history file.
       - `BBORDH`: Order header file.
       - `BBBOLX`: Bill of lading file.
       - `SA5FIXD`, `SA5FIXM`, `SA5DBDX`, `SA5DBMX`, `SA5BCDX`, `SA5BCMX`, `SA5CODX`, `SA5COMX`: Various sales analysis files.
       - `FRBINH2`: Freight bill history file.
       - `SA5SHU`: Shipping unit file.
     - Executes the `AR290` program to update the salesman codes in these files based on the changes specified in `ARSLST`.

3. **Batch Order File Updates (BBOR01 to BBOR98)**:
   - **Objective**: Update salesman codes in batch order files (`BBOR01` through `BBOR98`).
   - **Process**:
     - Initializes a counter (`P20`) to `01` to track batch numbers.
     - Enters a loop tagged `AGNBCH`:
       - Checks if the file `BBOR?20?` (e.g., `BBOR01`, `BBOR02`, etc.) exists. If not, it moves to the next batch (`NXTBCH`).
       - Loads the `AR291` program for each valid batch file.
       - Opens `ARSLST`, `ARCUST`, and the specific batch file (`BBOR?20?`) with shared memory access.
       - Executes `AR291` to update the salesman codes in the batch file.
       - Increments the batch counter (`P20 = P20 + 01`).
       - If the counter exceeds `99`, the loop restarts at `AGNBCH` (though typically it would exit after `BBOR98`).

4. **Invoice Transaction File Updates (BBTR?WS?)**:
   - **Objective**: Update salesman codes in invoice transaction files for all workstations (`BBTR?WS?`).
   - **Process**:
     - Deletes any existing catalog file (`AR29?WS?`) using `GSDELETE`.
     - Creates a catalog file (`AR29?WS?`) listing all invoice transaction files using the `CATALOG ALL` command.
     - Enters a loop tagged `LOOP` to process each invoice transaction file:
       - Loads the `AR294` program to read the next invoice transaction file name from the catalog (`AR29?WS?`).
       - If no more files are found (empty data at offset 1, length 8), jumps to `NOMORE`.
       - If the file does not exist on disk (`IFF DATAF1-?L'1,8'?`), skips to the next iteration (`LOOP`).
       - Validates the file group to ensure it matches the expected format (`?9?` prefix and valid group code).
       - Loads the `AR295` program to update the salesman codes in the valid invoice transaction file (`BBTRAN`, labeled as `?L'1,8'?`).
       - Opens `ARSLST`, `ARCUST`, and the invoice transaction file with shared memory access.
       - Executes `AR295` to perform the update.
       - Loops back to `LOOP` to process the next file.
     - After processing all files, jumps to `NOMORE` and deletes the catalog file (`AR29?WS?`).

5. **Generate Salesman Change Transaction Post Report**:
   - **Objective**: Produce a report summarizing the salesman code changes.
   - **Process**:
     - Loads the `AR299` program.
     - Opens `ARSLST`, `ARCUST`, and `GSTABL` (a general table file) with shared memory access.
     - Overrides the printer file (`ARPRINT`) to output to either `QUSRSYS/SLSMCHANGE` (for production) or `QUSRSYS/TESTOUTQ` (for testing), based on the environment (`?9?/G`).
     - Executes `AR299` to generate the report.

6. **Cleanup and Termination**:
   - Deletes the `SLSCHG` file using `GSDELETE`.
   - Clears the `ARSLST` file using `CLRPFM`.
   - Jumps to the `END` tag to terminate the program.

### External Programs Called
The OCL program invokes the following external programs:
1. **AR290**: Updates salesman codes in customer-related files (`ARCUST`, `BBORDR`, `BBBOL`, etc.).
2. **AR291**: Updates salesman codes in batch order files (`BBOR01` to `BBOR98`).
3. **AR294**: Retrieves the next invoice transaction file name from the catalog.
4. **AR295**: Updates salesman codes in invoice transaction files (`BBTRAN`).
5. **AR299**: Generates the salesman change transaction post report.

### Tables/Files Used
The program interacts with the following files:
1. **ARSLST**: Salesman change transaction file (contains old and new salesman codes).
2. **ARCUST**: Customer master file.
3. **ARCUSHS**: Customer history file.
4. **BBORDH**: Order header file.
5. **BBBOLX**: Bill of lading file.
6. **SA5FIXD**: Sales analysis fixed data file.
7. **SA5FIXM**: Sales analysis fixed master file.
8. **SA5DBDX**: Sales analysis database data file.
9. **SA5DBMX**: Sales analysis database master file.
10. **SA5BCDX**: Sales analysis billing control data file.
11. **SA5BCMX**: Sales analysis billing control master file.
12. **SA5CODX**: Sales analysis customer order data file.
13. **SA5COMX**: Sales analysis customer order master file.
14. **FRBINH2**: Freight bill history file.
15. **SA5SHU**: Shipping unit file.
16. **BBOR01 to BBOR98**: Batch order files (up to 98 files, dynamically referenced as `BBOR?20?`).
17. **BBTRAN**: Invoice transaction files (dynamically referenced as `?L'1,8'?` for each workstation).
18. **AR29WS**: Temporary catalog file for listing invoice transaction file names.
19. **GSTABL**: General table file (used for report generation).
20. **SLSCHG**: Temporary file indicating salesman change status.

### Additional Notes
- **Environment Variables**: The `?9?` placeholder likely represents a system-specific prefix or library name, dynamically resolved at runtime.
- **Shared Memory Access**: Files are opened with `DISP-SHRMM`, indicating shared memory access, allowing multiple processes to access the files simultaneously.
- **Loop Control**: The program uses tags (`AGNBCH`, `LOOP`, `NOMORE`, `END`) for flow control, typical in OCL for structuring loops and conditional jumps.
- **Error Handling**: The program checks for file existence and validity to avoid processing non-existent or invalid files.

This OCL program is a batch process typical in legacy systems (e.g., IBM AS/400 or System/36), designed to systematically update salesman codes across a suite of related files while ensuring data integrity and generating a report for verification.