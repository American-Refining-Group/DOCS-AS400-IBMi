The provided CLP (Control Language Program) named `BI907C` serves as a wrapper program that facilitates the maintenance of customer/ship-to product description and freight terms fields. It is called from the main RPGLE program `BI907P` and handles the setup of a work file before invoking another program (`BI907`). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BI907C CLP Program

The `BI907C` program performs the following steps to prepare and execute the file maintenance process for customer/ship-to entries:

1. **Parameter Declaration**:
   - The program declares five input parameters:
     - `&P$CONO`: Company number (2-character string).
     - `&P$CUST`: Customer number (6-character string).
     - `&P$SHIP`: Ship-to number (3-character string).
     - `&P$MODE`: Run mode (`MNT` for maintenance or `INQ` for inquiry, 3-character string).
     - `&P$FGRP`: File group (`Z` or `G`, 1-character string).

2. **Clear/Create Work File**:
   - Attempts to clear the physical file `BI907W` in the `QTEMP` library using the `CLRPFM` command.
   - Monitors for message `CPF3142` (file not found) using `MONMSG`. If the file does not exist:
     - Creates the physical file `BI907W` in `QTEMP` using `CRTPF` with the following specifications:
       - Source file: `QSRC` in `*LIBL`.
       - Options: No source or listing (`*NOSRC *NOLIST`).
       - Size: Unlimited (`*NOMAX`).
       - Authority: Full authority (`*ALL`).
   - This ensures the work file exists and is empty before proceeding.

3. **Override Work File**:
   - Overrides the database file `BI907W` to point to `QTEMP/BI907W` using the `OVRDBF` command (added per revision JK01 on 08/01/2021).
   - This ensures that the program `BI907` uses the temporary work file in `QTEMP` rather than any other library.

4. **Call BI907 Program**:
   - Calls the program `BI907`, passing the input parameters:
     - `&P$CONO` (company number).
     - `&P$CUST` (customer number).
     - `&P$SHIP` (ship-to number).
     - `&P$MODE` (run mode: `MNT` or `INQ`).
     - `&P$FGRP` (file group: `Z` or `G`).
   - The `BI907` program is responsible for the core file maintenance logic (e.g., creating, updating, or displaying customer/ship-to records).

5. **Delete Override**:
   - Removes the override for `BI907W` using the `DLTOVR` command (added per revision JK01).
   - This cleanup ensures that subsequent operations do not inadvertently use the temporary file in `QTEMP`.

6. **Program Termination**:
   - The program ends with the `ENDPGM` command, returning control to the calling program (`BI907P`).

---

### Business Rules

The `BI907C` program enforces the following business rules:

1. **Work File Management**:
   - A temporary work file (`BI907W`) must be available in the `QTEMP` library for the `BI907` program to use.
   - If the file exists, it is cleared to ensure no residual data affects the current operation.
   - If the file does not exist, it is created with unlimited size and full authority to accommodate the program's needs.

2. **File Override**:
   - The program ensures that `BI907W` is accessed from `QTEMP` by applying a file override, isolating the work file to the current job's temporary library.
   - The override is removed after the `BI907` program executes to prevent unintended file access in subsequent operations.

3. **Parameter Passing**:
   - The program passes all input parameters (`&P$CONO`, `&P$CUST`, `&P$SHIP`, `&P$MODE`, `&P$FGRP`) directly to `BI907` without modification, ensuring that the called program receives the exact context provided by the caller (`BI907P`).

4. **Mode-Based Processing**:
   - The `&P$MODE` parameter (`MNT` or `INQ`) determines whether the called program (`BI907`) operates in maintenance mode (allowing record creation or updates) or inquiry mode (read-only display).

5. **File Group Flexibility**:
   - The `&P$FGRP` parameter (`Z` or `G`) allows the program to work with different sets of database files, supporting flexibility in accessing data from different environments or configurations.

---

### Tables Used

The `BI907C` program directly interacts with the following table:
1. **BI907W**:
   - Library: `QTEMP`.
   - Purpose: Temporary work file used by the `BI907` program for processing customer/ship-to data.
   - Operations: Cleared (`CLRPFM`) or created (`CRTPF`) if it does not exist, and overridden (`OVRDBF`) to ensure it is accessed from `QTEMP`.

No other database files are directly referenced in `BI907C`. However, the called program `BI907` likely accesses additional files (e.g., `bicont`, `arcupr`, `gsprod`, `arcust`, `shipto`) as inherited from the context provided by `BI907P`.

---

### External Programs Called

The `BI907C` program calls the following external program:
1. **BI907**:
   - Purpose: Performs the core logic for maintaining or displaying customer/ship-to product description and freight terms fields.
   - Parameters:
     - `&P$CONO`: Company number.
     - `&P$CUST`: Customer number.
     - `&P$SHIP`: Ship-to number.
     - `&P$MODE`: Run mode (`MNT` or `INQ`).
     - `&P$FGRP`: File group (`Z` or `G`).
   - Called in: Main program flow after setting up the work file.

---

### Summary

The `BI907C` CLP program acts as a preparatory wrapper for the `BI907` program, ensuring that a temporary work file (`BI907W`) in `QTEMP` is available and properly configured. It handles file creation, clearing, and overriding, then delegates the core file maintenance logic to `BI907`. The business rules focus on ensuring a clean and isolated work environment for the called program, with flexibility for different modes and file groups. The program is tightly integrated with the `BI907P` RPGLE program, which provides the user interface and invokes `BI907C` for specific operations (create, change, or display).