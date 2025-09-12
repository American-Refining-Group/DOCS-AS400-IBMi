The provided document is an OCL (Operation Control Language) script named `AR050.ocl36.txt`, used in an IBM System/36 environment to manage the cash receipts payment statement entry process. This script is called from the main OCL program (`AR050P.ocl36`) as part of a broader accounts receivable (AR) cash receipts workflow. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called.

### Process Steps of the AR050 OCL Program

The `AR050.ocl36` script is designed to set up the environment for entering payment statement transactions related to cash receipts. It creates or verifies necessary files and indexes, loads the `AR050` program, and defines the files required for its execution. Here’s a step-by-step breakdown of the process:

1. **Switch Initialization**:
   - `// SWITCH 1XXXXXXX`: Sets the first bit of an 8-bit switch register to 1, with the remaining bits undefined (represented by `X`). This likely enables a specific mode or option for the program, such as activating a particular processing path.

2. **Year 2000 Compliance**:
   - `// GSY2K`: Executes a Year 2000 compliance routine or utility, likely to ensure date fields are handled correctly (e.g., using a 4-digit year format).

3. **File and Index Creation/Verification**:
   - The script checks for the existence of specific files using `IFF DATAF1-` conditions and creates or rebuilds them if necessary:
     - `// IFF DATAF1-?9?ARCUSX BLDINDEX ?9?ARCUSX,2,2,?9?ARCUST,,,,271,10,4,6`:
       - If the `?9?ARCUSX` file (customer alternate index) exists, rebuilds the index for `?9?ARCUST` (customer master file).
       - Parameters: Key starts at position 2, length 2; output file is `?9?ARCUSX`; additional attributes at positions 271, 10, 4, 6.
     - `// IFF DATAF1-?9?CRDETX BLDINDEX ?9?CRDETX,2,8,?9?ARDETL,,DUPKEY,,71,8,10,7`:
       - If the `?9?CRDETX` file (AR detail alternate index) exists, rebuilds the index for `?9?ARDETL` (AR detail file).
       - Parameters: Key starts at position 2, length 8; allows duplicate keys (`DUPKEY`); additional attributes at positions 71, 8, 10, 7.
     - `// IFF DATAF1-?9?CRWKGG BLDFILE ?9?CRWKGG,A,RECORDS,5000,96,,P,2,21`:
       - If the `?9?CRWKGG` file (work file for open invoices) exists, builds a file with 5000 records, each 96 bytes, in add mode (`A`), with primary key starting at position 2, length 21.
     - `// IFF DATAF1-?9?CRWXGG BLDINDEX ?9?CRWXGG,2,8,?9?CRWKGG,,DUPKEY,,64,8,16,7`:
       - If the `?9?CRWXGG` file (index for work file) exists, rebuilds the index for `?9?CRWKGG`.
       - Parameters: Key starts at position 2, length 8; allows duplicate keys; additional attributes at positions 64, 8, 16, 7.
     - `// IFF DATAF1-?9?CRCKGG BLDFILE ?9?CRCKGG,A,RECORDS,1000,96,,P,2,14`:
       - If the `?9?CRCKGG` file (work file for checks) exists, builds a file with 1000 records, each 96 bytes, in add mode, with primary key starting at position 2, length 14.

4. **Program Load and File Definitions**:
   - `// LOAD AR050`: Loads the `AR050` program, which handles the payment statement entry logic.
   - Defines the following files for the program, all in shared mode (`DISP-SHR`):
     - `// FILE NAME-ARCUSTX,LABEL-?9?ARCUSX,DISP-SHR`: Customer alternate index file.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file.
     - `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR`: AR control file.
     - `// FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHR`: General system table file.
     - `// FILE NAME-CRDETX,LABEL-?9?CRDETX,DISP-SHR`: AR detail alternate index file.
     - `// FILE NAME-CRWORK,LABEL-?9?CRWXGG,DISP-SHR,EXTEND-500`: Work file index for open invoices, with an extension of 500 records.
     - `// FILE NAME-CRCHKS,LABEL-?9?CRCKGG,DISP-SHR,EXTEND-50`: Work file for checks, with an extension of 50 records.
   - `// RUN`: Executes the `AR050` program with the defined files.

### Business Rules

The `AR050.ocl36` script enforces the following business rules for the cash receipts payment statement entry process:

1. **File and Index Management**:
   - Ensures that necessary files and indexes (`?9?ARCUSX`, `?9?CRDETX`, `?9?CRWKGG`, `?9?CRWXGG`, `?9?CRCKGG`) are created or rebuilt if they exist, maintaining data integrity for customer and AR detail records.
   - Supports duplicate keys in alternate indexes (`DUPKEY` for `?9?CRDETX` and `?9?CRWXGG`), allowing multiple records with the same key values where applicable.

2. **Workstation-Specific Files**:
   - Uses the `?9?` parameter (likely a session or workstation identifier) to create unique file names, ensuring that multiple users or sessions can process cash receipts concurrently without conflicts.
   - Replaces `?WS?` with `GG` (per JB01 comment, 11/28/23) to adapt the system for the Atrium environment, ensuring compatibility with the target system.

3. **File Extensions**:
   - Extends the `CRWORK` file by 500 records and `CRCHKS` by 50 records, allowing additional space for temporary data storage during processing.

4. **Shared File Access**:
   - All files are opened in shared mode (`DISP-SHR`), enabling concurrent access by multiple users or processes, which is critical for a multi-user AR system.

5. **Year 2000 Compliance**:
   - The `GSY2K` utility ensures that date-related processing is Y2K-compliant, preventing issues with date fields in the payment statement entry process.

### Tables (Files) Used

The script interacts with the following files:
1. **?9?ARCUSX** (aliased as `ARCUSTX`):
   - Type: Alternate index file for `?9?ARCUST`.
   - Usage: Provides indexed access to customer master data.

2. **?9?ARCUST** (aliased as `ARCUST`):
   - Type: Customer master file.
   - Usage: Contains core customer data for payment statement processing.

3. **?9?ARCONT** (aliased as `ARCONT`):
   - Type: AR control file.
   - Usage: Stores control data, such as default GL accounts, for the AR system.

4. **?9?GSTABL** (aliased as `GSTABL`):
   - Type: General system table file.
   - Usage: Likely contains configuration or reference data for the system.

5. **?9?CRDETX** (aliased as `CRDETX`):
   - Type: Alternate index file for `?9?ARDETL`.
   - Usage: Provides indexed access to AR detail records (e.g., invoices).

6. **?9?ARDETL**:
   - Type: AR detail file (referenced in index creation).
   - Usage: Contains detailed AR transactions, such as open invoices.

7. **?9?CRWKGG** (aliased as `CRWORK`):
   - Type: Work file for open invoices.
   - Record Length: 96 bytes, 5000 records.
   - Usage: Temporarily stores open invoice data for a customer during processing.

8. **?9?CRWXGG** (aliased as `CRWORK`):
   - Type: Alternate index for `?9?CRWKGG`.
   - Usage: Provides indexed access to the work file for efficient retrieval.

9. **?9?CRCKGG** (aliased as `CRCHKS`):
   - Type: Work file for checks.
   - Record Length: 96 bytes, 1000 records.
   - Usage: Temporarily stores check data entered for a customer.

### External Programs Called

The script explicitly calls the following program:
1. **AR050**:
   - Loaded and executed via `// LOAD AR050` and `// RUN`.
   - Purpose: Handles the core logic for cash receipts payment statement entry, interacting with the defined files to process customer payments and apply them to open invoices.

Additionally, the `GSY2K` utility is invoked, which may be a system-level routine rather than a user program, ensuring Y2K compliance.

### Summary

The `AR050.ocl36` script is a critical component of the cash receipts payment statement entry process in the AR system. It:
- Initializes the environment with a switch setting and Y2K compliance.
- Creates or verifies necessary files and indexes for customer, AR detail, and temporary work data.
- Loads and runs the `AR050` program, passing files for customer data, AR control, system tables, invoices, and checks.
- Enforces business rules for file management, concurrent access, and workstation-specific processing in the Atrium environment (using `GG` suffix per JB01).
- Supports temporary storage of open invoices and check data with extensible work files.

The script sets the stage for the `AR050` program to process payment statements, ensuring that the necessary data structures are in place for efficient and accurate cash receipts entry.