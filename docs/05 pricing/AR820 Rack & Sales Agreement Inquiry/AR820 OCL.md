The provided document is an Operations Control Language (OCL) program named `AR820.ocl36.txt`, which appears to be part of an IBM System/36 or AS/400 environment. OCL is used to control job execution and manage file operations for RPG (Report Program Generator) programs. This program facilitates a customer search inquiry for current product pricing. Below is a detailed explanation of the process steps, the external programs called, and the tables/files used.

---

### Process Steps of the OCL Program

The OCL program defines a sequence of operations to set up the environment, load files, override database files, call an RPG program, and handle control flow based on switch settings. Here’s a step-by-step breakdown:

1. **Initial Setup and Switch Initialization**:
   - `// SCPROCP ,,,,,,,,?9?`: This likely specifies the procedure name or a parameter for the job, with `?9?` being a placeholder for a system-specific value (e.g., library or environment prefix).
   - `// LOCAL BLANK-*ALL`: Initializes all local variables to blank, ensuring a clean slate for the job.
   - `// SWITCH 00000000`: Sets all eight switches (used for conditional logic) to `0` (off).

2. **Conditional Check on Switch 2**:
   - `// IF SWITCH2-1 LOCAL BLANK-*ALL`: If switch 2 is set to `1`, all local variables are reset to blank.
   - `// IF SWITCH2-1 GOTO END`: If switch 2 is `1`, the program jumps to the `END` tag, effectively terminating the job early.

3. **Data Initialization**:
   - `// LOCAL OFFSET-202,DATA-'      '`: Initializes a local variable at offset 202 with six blank spaces. This could be used to clear a specific field or parameter area.

4. **Switch Modification**:
   - `// SWITCH XX01XXX0`: Modifies the switch settings. The `XX01XXX0` pattern sets switch 4 to `1`, leaves switches 1, 2, 5, 6, and 7 unchanged (`X`), and sets switch 8 to `0`. This likely enables specific program logic or conditions.

5. **File Loading**:
   - `// LOAD AR820`: Loads the `AR820` program into memory for execution.
   - The following `// FILE NAME-...` statements define the files to be used by the program, specifying their names, labels (with `?9?` as a library prefix), and disposition (`DISP-SHR` for shared access):
     - `ARCUST`: Customer master file.
     - `GSTABL`: General system table file.
     - `ARCONT`: Accounts receivable control file.
     - `ARCUSX`: Customer extension or auxiliary file.
     - `GSCNTR1`: Counter or control file.
     - `GSPROD`: Product master file.
   - `// RUN`: Initiates execution of the loaded program (`AR820`).

6. **Conditional Check on Switch 8**:
   - `// IF SWITCH8-0 LOCAL BLANK-*ALL`: If switch 8 is `0` (as set earlier), all local variables are reset to blank.
   - `// IF SWITCH8-0 CANCEL`: If switch 8 is `0`, the job is canceled, terminating execution.

7. **Conditional Check on Parameter**:
   - `// IF ?L'136,1'?/E GOTO END`: Checks if the parameter at position 136 (length 1) is equal to `E`. If true, the program jumps to the `END` tag, terminating the job. This likely checks for an error condition or specific input.

8. **A/R Pricing Inquiry Section**:
   - This section is labeled `** A/R PRICING INQUIRY **` and performs database file overrides to map logical files to physical files in the specified library (`?9?` prefix).
   - The `OVRDBF` (Override Database File) commands redirect file references for the RPG program:
     - `BBPRCY`: Pricing file.
     - `BICUA6`: Possibly a customer or pricing-related file.
     - `GSTABL`: General system table (already loaded).
     - `ARCUPR`: Customer pricing file.
     - `ARCUSP`: Customer special pricing file.
     - `ARCUST`: Customer master file (already loaded).
     - `SHIPTO`: Ship-to address file.
     - `GSCNTR1`: Counter/control file (already loaded).
     - `GSPROD`: Product master file (already loaded).
   - `CALL AR822R`: Calls the RPG program `AR822R`, which likely performs the core pricing inquiry logic using the overridden files.

9. **Conditional Check on Switch 7**:
   - `// IF SWITCH7-1 LOCAL BLANK-*ALL`: If switch 7 is `1`, all local variables are reset to blank.
   - `// IF SWITCH7-1 GOTO END`: If switch 7 is `1`, the program jumps to the `END` tag, terminating the job.

10. **Program Reset**:
    - `// RESET AR820 ,,,,,,,,?9?`: Resets the `AR820` program, clearing its state and parameters (with `?9?` as a placeholder).

11. **End of Program**:
    - `// TAG END`: Marks the `END` label, where the program jumps if certain conditions (e.g., switch 2, switch 7, or error parameter) are met.
    - `// SWITCH 00000000`: Resets all switches to `0`.
    - `// LOCAL BLANK-*ALL`: Clears all local variables again, ensuring a clean exit.

---

### External Programs Called

- **AR822R**: This is the primary RPG program called by the OCL script. It likely contains the business logic for the A/R pricing inquiry, using the files specified in the `OVRDBF` commands.

---

### Tables/Files Used

The program references the following files, which are likely database tables or physical files in the System/36 or AS/400 environment:

1. **ARCUST**: Customer master file, containing core customer data.
2. **GSTABL**: General system table, possibly holding configuration or reference data.
3. **ARCONT**: Accounts receivable control file, likely for A/R settings or control data.
4. **ARCUSX**: Customer extension file, possibly for additional customer attributes.
5. **GSCNTR1**: Counter or control file, used for tracking or control purposes.
6. **GSPROD**: Product master file, containing product details (e.g., pricing, descriptions).
7. **BBPRCY**: Pricing file, specific to billing or pricing calculations.
8. **BICUA6**: Likely a customer or pricing-related file (exact purpose unclear from context).
9. **ARCUPR**: Customer pricing file, for standard pricing data.
10. **ARCUSP**: Customer special pricing file, for customer-specific pricing agreements.
11. **SHIPTO**: Ship-to address file, for customer shipping information.

These files are accessed with shared disposition (`DISP-SHR`), allowing multiple processes to read them concurrently.

---

### Summary

The `AR820.ocl36.txt` OCL program orchestrates a customer search inquiry for product pricing by:
1. Initializing variables and switches.
2. Checking conditions to control program flow (e.g., switches 2, 7, 8, and parameter checks).
3. Loading necessary files and overriding database file references.
4. Calling the `AR822R` RPG program to perform the pricing inquiry.
5. Resetting the environment and terminating based on conditions.

The program ensures proper file access and conditional execution, with the core logic handled by `AR822R`. The files listed above support customer, product, and pricing data, critical for the inquiry process.