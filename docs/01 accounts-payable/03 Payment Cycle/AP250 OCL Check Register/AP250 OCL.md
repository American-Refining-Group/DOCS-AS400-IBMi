The provided document is an Operations Control Language (OCL) program, specifically `AP250.ocl36.txt`, which appears to be part of an IBM System/36 or AS/400 environment for processing an Accounts Payable (A/P) Check Register, Journal, and file updates. Below is an explanation of the process steps, the external programs called, and the tables (files) used in the program.

---

### **Process Steps of the AP250 OCL Program**

The OCL program orchestrates a sequence of operations to generate an A/P Check Register, update related files, and produce a Cash Disbursements Journal. Here’s a breakdown of the steps based on the program’s structure:

1. **Initialization and Setup**:
   - **Procedure Call**: The program starts by calling `STRPCOCLP`, likely a system procedure to initialize the environment or set up processing parameters.
   - **Conditional Wire Transfer Check**:
     - Checks if parameter `?3?` equals 'WT' (Wire Transfer).
     - If true, sets a local variable at offset 198 with the text `'WT*** WIRE TRANSFER ***'`.
     - If false, clears the local variable (sets it to blanks).
   - **Auto-Run Check**:
     - If parameter `?2?` equals 'AUTO', the program jumps to the `AP250` tag, indicating it’s running as part of an automated process (likely triggered by another program, `AP200`).

2. **Workstation Lockout Check**:
   - Checks if the data field `DATAF1-?9?APPO?WS?` exists (indicating checks are pending printing at the workstation).
   - If true, displays a warning message indicating that checks cannot be posted until printed (via Option 11 of `APMENU`).
   - Prompts the user to press `0,ENTER` to cancel the job, then jumps to the `END` tag to terminate.

3. **User Interaction**:
   - If not in auto mode and no workstation lockout, displays a message: `'A/P CHECK REGISTER, JOURNAL, AND UPDATE FILES'`.
   - Prompts the user to either cancel (press `ATTN,2,ENTER`) or continue (press `0,ENTER`).
   - Sets an attribute `INQUIRY-YES,CANCEL-NO` to handle user input.

4. **File Preparation**:
   - Deletes any existing temporary file `APCD?WS?` using `GSDELETE`.
   - Builds a new sequential file `?9?APCD?WS?` with a capacity of 999,000 records and a record length of 128 bytes.

5. **Check Register and File Update Execution**:
   - Displays a message: `'CHECK REGISTER, UPDATE FILES EXECUTING'`.
   - Loads the program `AP250` and associates it with multiple files (see **Tables Used** below for details).
   - Overrides printer output files (`APPNCF` and `APPRINT`) to specific output queues (`QUSRSYS/PRTPNC`, `QUSRSYS/APPOST`, or `QUSRSYS/TESTOUTQ`) based on the condition `?9?/G`.
   - Runs the `AP250` program to process the check register and update related files.

6. **Commission Table Update**:
   - Loads the program `AP251`.
   - Uses files `APPAY` and `APTORCY` to update a commission table with payment information.
   - Runs the `AP251` program.

7. **Sorting Cash Disbursements Data**:
   - Displays a message: `'CASH DISBURSMENTS JOURNAL EXECUTING'`.
   - Loads the `#GSORT` program to sort data.
   - Input file: `?9?APCD?WS?`.
   - Output file: `?9?APCS?WS?` (capacity of 999,000 records, extendable).
   - Sorting parameters:
     - Sorts in ascending order (`HSORTA 30A 3X N`).
     - Conditions and fields for sorting:
       - `I C 1 1NECD`: Likely a conditional check on a specific field.
       - Fields sorted: Company (positions 2-3), C/D flag (position 12), AP/Cash/Discount (positions 106-115), G/L number (positions 13-20), and Sequence number (positions 97-105).
   - Runs the sort operation.

8. **Cash Disbursements Journal Generation**:
   - Loads the program `AP255`.
   - Uses sorted data (`APCDJR` and `AP255S`) and control files (`APCONT`, `TEMGEN`).
   - Overrides printer output file `APPRINT` to `QUSRSYS/APPOST` or `QUSRSYS/TESTOUTQ` based on `?9?/G`.
   - Runs `AP255` to generate the Cash Disbursements Journal.

9. **Optional File Processing (APDT?WS?)**:
   - Checks if the file `?9?APDT?WS?` exists and has a non-zero size.
   - If it exists:
     - Builds a new indexed file `?9?APDT?WS?C` with 500 records, record length of 10 bytes, and specific key fields.
     - Loads `AP256A` to process this file with `APVNFMX` (vendor master file).
     - Runs `AP256A`.
     - Loads `AP256` to further process `APDTWS`, `APDTWSC`, `APVEND`, `APCONT`, and `APVNFMX`.
     - Overrides multiple report files (`REPORT1`, `REPORT2`, `REPORT3`, `REPORT4`) to output queues `QUSRSYS/APACHOUTQ` or `QUSRSYS/TESTOUTQ` based on `?9?/G`.
     - Runs `AP256`.

10. **Cleanup and Termination**:
    - Deletes temporary files (`APPT?WS?`, `APPY?WS?`, `APPS?WS?`, `APPC?WS?`, `APDS?WS?`, `APPO?WS?`, `APCD?WS?`, `APCS?WS?`, `APDT?WS?`, `APDT?WS?C`) using `GSDELETE`.
    - If in auto mode (`?2?/AUTO`), clears all local variables (`LOCAL BLANK-*ALL`).
    - Ends the program.

---

### **External Programs Called**

The OCL program calls the following external programs:
1. **STRPCOCLP**: Initializes the environment or sets up processing parameters.
2. **AP250**: Main program for generating the A/P Check Register and updating files.
3. **AP251**: Updates the commission table with payment data.
4. **#GSORT**: Sorts data for the Cash Disbursements Journal.
5. **AP255**: Generates the Cash Disbursements Journal.
6. **AP256A**: Processes temporary data files (if `APDT?WS?` exists).
7. **AP256**: Further processes temporary data and generates reports.

---

### **Tables (Files) Used**

The program references the following files, which serve as input, output, or shared data tables:
1. **APPYCK** (`?9?APPC?WS?`): Likely contains check data.
2. **APPAY** (`?9?APPY?WS?`): Payment data file, used in `AP250` and `AP251`.
3. **AP250S** (`?9?APPS?WS?`): Temporary file for `AP250` processing.
4. **APPYTR** (`?9?APPT?WS?`): Transaction-related file.
5. **APCONT** (`?9?APCONT`, DISP-SHR): Control file for A/P processing, shared across programs.
6. **APVEND** (`?9?APVEND`, DISP-SHR): Vendor master file, used in multiple programs.
7. **APVEND2** (`?9?APVEND`, DISP-SHR): Likely an alias for `APVEND`.
8. **APCHKR** (`?9?APCHKR`, DISP-SHR): Check register file.
9. **APPYDS** (`?9?APDS?WS?`, DISP-SHR): Payment distribution file.
10. **APOPEN** (`?9?APOPEN`, DISP-SHR): Open A/P transactions.
11. **APOPENH** (`?9?APOPNH`, DISP-SHR): Historical A/P open transactions header.
12. **APOPEND** (`?9?APOPND`, DISP-SHR): Detailed A/P open transactions.
13. **APOPENV** (`?9?APOPNV`, DISP-SHR): Vendor-specific A/P open transactions.
14. **APHISTH** (`?9?APHSTH`, DISP-SHR): A/P history header.
15. **APHISTD** (`?9?APHSTD`, DISP-SHR): A/P history details.
16. **APHISTV** (`?9?APHSTV`, DISP-SHR): Vendor-specific A/P history.
17. **FRCINH** (`?9?FRCINH`, DISP-SHR): Possibly a financial control file.
18. **FRCFBH** (`?9?FRCFBH`, DISP-SHR): Another financial control file.
19. **APCDJR** (`?9?APCD?WS?`, EXTEND-100): Cash disbursements journal file.
20. **APDSMS** (`?9?APDSMS`, DISP-SHR): Summary data for disbursements.
21. **AP255S** (`?9?APCS?WS?`): Sorted output for the Cash Disbursements Journal.
22. **TEMGEN** (`?9?TEMGEN`, DISP-SHR): Temporary general file.
23. **APDTWS** (`?9?APDT?WS?`): Temporary data file for additional processing.
24. **APDTWSC** (`?9?APDT?WS?C`, RETAIN-T, RECORDS-50): Indexed temporary file for `AP256A` and `AP256`.
25. **APVNFMX** (`?9?APVNFMX`, DISP-SHR): Vendor master extension file.

---

### **Summary**

The `AP250` OCL program is a comprehensive A/P processing routine that:
- Validates workstation status and user input.
- Generates an A/P Check Register and updates related files (`AP250`).
- Updates commission tables (`AP251`).
- Sorts data and produces a Cash Disbursements Journal (`#GSORT`, `AP255`).
- Optionally processes additional temporary files (`AP256A`, `AP256`).
- Cleans up temporary files and terminates.

It integrates with multiple external programs and a large set of files to manage A/P transactions, vendor data, and financial reporting. The program supports both interactive and automated execution, with conditional logic for handling wire transfers and output queue assignments.