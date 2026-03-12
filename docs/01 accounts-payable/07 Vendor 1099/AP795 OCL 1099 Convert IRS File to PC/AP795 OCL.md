The provided document is an OCL (Operation Control Language) script, typically used in IBM midrange systems like the AS/400 (now IBM i). This script, named `AP795.ocl36.txt`, is designed to convert 1099 information as required by government regulations, preparing the `IRSTAX` file for download to a PC. Below, I’ll explain the process steps of the program, list the external programs called, and identify the tables (files) used.

---

### Process Steps of the OCL Program

The OCL script performs a series of conditional checks and operations to manage the creation or replacement of the `IRSTAX` file, potentially in a test or production environment. Here’s a step-by-step breakdown of the process:

1. **Initial Setup and Comments**:
   - The script begins with comments (`// *`) explaining its purpose: converting 1099 information and preparing the `IRSTAX` file for download to a PC.
   - A specific comment indicates that in a test environment, the file is named `?9?IRSTAX` (where `?9?` is a placeholder for a prefix, likely a library or environment identifier, such as `TEST` or `PROD`).

2. **Environment Check and Pause**:
   - The script checks if the environment variable `?9?` is set to `/G` (likely indicating a specific environment or mode, such as production or test).
   - A `PAUSE` statement is executed, prompting the user to interact with the system, possibly to confirm the operation or review the setup before proceeding.

3. **Check for Existing `IRSTAX` File**:
   - The script checks if the `IRSTAX` file exists in the `DATAF1` library:
     - If `?9?` is `/G` and `DATAF1-IRSTAX` exists, it displays a message: `'IRSTAX ALREADY EXISTS. DO YOU WANT TO DELETE IT'`.
     - If `?9?` is not `/G` and `DATAF1-?9?IRSTAX` exists, it displays the same message but for the prefixed file (e.g., `TESTIRSTAX`).
   - Further messages ask the user if they want to continue (`'AND CONTINUE? IF NO, PRESS "SYS RQ" AND TAKE OPTION 3'`), suggesting a manual intervention option (e.g., pressing the System Request key to abort or choose an alternative action).
   - If the file exists, another `PAUSE` statement halts execution, likely to allow the user to confirm whether to proceed with deletion.

4. **Delete Existing `IRSTAX` File**:
   - If the `IRSTAX` file exists:
     - For `?9?=/G`, the script deletes `IRSTAX` from the `DATAF1` library using `DELETE IRSTAX,F1`.
     - For `?9?!=/G`, the script deletes the prefixed file (e.g., `?9?IRSTAX`) using `DELETE ?9?IRSTAX,F1`.
   - This step ensures that any existing `IRSTAX` file is removed before creating a new one, avoiding conflicts or data duplication.

5. **Copy Data to Create New `IRSTAX` File**:
   - The script uses the `COPYDATA` command to create a new `IRSTAX` file by copying data from the `AP1099` file (or `?9?AP1099` in a test environment):
     - For `?9?=/G`, it executes: `COPYDATA AP1099,,IRSTAX,,,,,,,750,NE,'D'`.
     - For `?9?!=/G`, it executes: `COPYDATA ?9?AP1099,,?9?IRSTAX,,,,,,,750,NE,'D'`.
   - Parameters in the `COPYDATA` command:
     - Source file: `AP1099` or `?9?AP1099` (the 1099 data source).
     - Target file: `IRSTAX` or `?9?IRSTAX` (the output file).
     - `750`: Likely specifies the record length or a processing parameter (common in IBM i for file creation).
     - `NE`: Indicates "No Error" handling, meaning the operation will not stop on certain errors.
     - `'D'`: Specifies a disposition or mode, possibly indicating a direct copy or a specific data format.

6. **Environment Handling**:
   - The script uses conditional logic (`IF ?9?/G`) to handle different environments (e.g., production vs. test). The `?9?` variable allows the script to dynamically adjust file names (e.g., `TESTIRSTAX` vs. `IRSTAX`) based on the environment.

---

### External Programs Called

The OCL script explicitly calls the following external program or command:
- **COPYDATA**: This is an IBM i system command used to copy data from one file to another, with options for formatting, error handling, and record length specification. It is used to create the `IRSTAX` file from `AP1099`.

No other external programs (e.g., RPG programs) are explicitly called in the script. The `GSY2K` reference appears to be a label or comment, possibly indicating a system or module context, but it is not a program call.

---

### Tables (Files) Used

The script interacts with the following files (referred to as "tables" in some contexts, though they are physical files in IBM i terminology):
1. **AP1099** (or `?9?AP1099`):
   - The source file containing 1099 data.
   - In a test environment, it may be prefixed (e.g., `TESTAP1099`).
   - Located in an unspecified library (assumed to be accessible in the job’s library list).
2. **IRSTAX** (or `?9?IRSTAX`):
   - The target file created or replaced by the script, containing the converted 1099 data formatted for government requirements.
   - Located in the `DATAF1` library (as indicated by `DATAF1-IRSTAX`).
   - In a test environment, it may be prefixed (e.g., `TESTIRSTAX`).

---

### Summary

The OCL program `AP795.ocl36.txt` automates the process of preparing the `IRSTAX` file for 1099 reporting by:
1. Checking the environment and pausing for user confirmation.
2. Verifying if the `IRSTAX` file exists and prompting the user to delete it if necessary.
3. Deleting the existing `IRSTAX` file (if present).
4. Copying data from `AP1099` to create a new `IRSTAX` file, formatted for government requirements.

**External Programs Called**:
- `COPYDATA` (IBM i system command).

**Tables Used**:
- `AP1099` (or `?9?AP1099`): Source 1099 data file.
- `IRSTAX` (or `?9?IRSTAX`): Target file in the `DATAF1` library.

The script is designed to handle both production and test environments, using the `?9?` variable to adjust file names dynamically. It includes user interaction points (via `PAUSE`) to ensure manual oversight before critical operations like file deletion.