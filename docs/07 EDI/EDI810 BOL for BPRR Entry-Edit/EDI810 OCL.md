The provided document is an **OCL (Operation Control Language)** program snippet, typically used on IBM midrange systems like the AS/400 (now IBM i) to control job execution and file operations. This specific program, named **EDI810.ocl36**, appears to handle **EDI Outbound Bill of Lading Tracking Maintenance**. Below, I’ll explain the process steps, identify any external programs called, and list the tables (files) used, based on the provided OCL code.

### Process Steps of the EDI810.ocl36 Program

OCL is a scripting language used to automate tasks and control program execution on IBM systems. The program defines a sequence of operations, including setting switches, loading files, and running a program. Here’s a step-by-step breakdown of the process:

1. **Program Initialization and Comments**:
   - The program starts with a header indicating it is an **EDI Outbound Bill of Lading Tracking Maintenance** program.
   - `// SCPROCP ,,,,,,,,?9?` likely specifies a procedure name or parameters, with `?9?` being a placeholder for a variable or specific value (common in OCL for dynamic substitution).
   - `// GSY2K` might be a comment or a directive related to system configuration (possibly Year 2000 compliance or a specific system module).

2. **Local Variable Initialization**:
   - `// LOCAL OFFSET-200,DATA-'        '` initializes a local variable at offset 200 with 8 blank spaces. This could be used to clear or set a specific data area or parameter.
   - `// LOCAL BLANK-*ALL` (conditional on `SWITCH2-1`) clears all local variables to blanks if the condition is met.

3. **Conditional Logic Based on Switch**:
   - `// IF SWITCH2-1 GOTO END` checks if the second switch (bit 1) is set to 1. If true, the program jumps to the `END` tag, effectively terminating execution without performing further steps. Switches are used in OCL to control program flow based on system or job conditions.
   - If `SWITCH2-1` is not set, execution continues.

4. **Set Additional Local Variable**:
   - `// LOCAL OFFSET-202,DATA-'      '` initializes another local variable at offset 202 with 6 blank spaces. This might be used to pass parameters or clear a data area.

5. **Set Switches**:
   - `// SWITCH XX00XXX0` sets a switch configuration. In OCL, switches are 8-bit flags (0 or 1) used to control program behavior. Here, `XX00XXX0` indicates:
     - Bits 3 and 4 are explicitly set to 0.
     - Bit 8 is set to 0.
     - `X` represents bits that are unchanged or undefined (likely retaining their previous values).
     - This configuration likely prepares the environment for the subsequent program execution.

6. **Load the EDI810 Program**:
   - `// LOAD EDI810` loads a program named **EDI810**, which is likely an RPG (Report Program Generator) or another high-level program that performs the core logic of the Bill of Lading Tracking Maintenance.

7. **File Definitions**:
   - The program specifies several files to be used by the loaded **EDI810** program. Each file is defined with a name, label, and disposition:
     - `FILE NAME-EDIBOLTX,LABEL-?9?EDIBOLX,DISP-SHR`: Opens the file `EDIBOLTX` (likely the EDI Bill of Lading transaction file) with a label that includes a dynamic value `?9?`. `DISP-SHR` indicates shared access, allowing multiple jobs to read the file simultaneously.
     - `FILE NAME-BOLEDIY,LABEL-?9?BOLEDIY,DISP-SHR`: Opens the file `BOLEDIY`, likely a Bill of Lading history or detail file.
     - `FILE NAME-TRRTCD,LABEL-?9?TRRTCD,DISP-SHR`: Opens the file `TRRTCD`, possibly a transportation or routing code file.
     - `FILE NAME-GSHAZM,LABEL-?9?GSHAZM,DISP-SHR`: Opens the file `GSHAZM`, likely a hazardous materials file.
     - `FILE NAME-GSPROD,LABEL-?9?GSPROD,DISP-SHR`: Opens the file `GSPROD`, likely a product or inventory file.
     - `FILE NAME-SHIPTO,LABEL-?9?SHIPTO,DISP-SHR`: Opens the file `SHIPTO`, likely containing ship-to customer information.
     - `FILE NAME-SHPADR,LABEL-?9?SHPADR,DISP-SHR`: Opens the file `SHPADR`, likely containing shipping address details.
     - `FILE NAME-BBBOLY,LABEL-?9?BBBOLY,DISP-SHR`: Opens the file `BBBOLY`, possibly another Bill of Lading-related file (e.g., a backup or secondary file).
   - The `?9?` in the `LABEL` parameter suggests a dynamic substitution, possibly a library name or a specific identifier passed to the program.

8. **Run the Program**:
   - `// RUN` executes the loaded **EDI810** program with the specified files and switch settings. The actual processing (e.g., updating Bill of Lading records, generating EDI transactions) occurs within the **EDI810** program.

9. **End of Program**:
   - `// TAG END` marks a label named `END`, which is the target of the conditional `GOTO END` statement earlier.
   - `// SWITCH 00000000` resets all switches to 0, clearing any previous settings.
   - `// LOCAL BLANK-*ALL` clears all local variables again, ensuring a clean state before the program terminates.

### External Programs Called

- **EDI810**: This is the primary program loaded and executed by the OCL script. It is likely an RPG program (or possibly another language like COBOL) responsible for the core logic of processing EDI Bill of Lading transactions. No other external programs are explicitly called in the OCL code.

### Tables (Files) Used

The program references the following files, which are likely physical or logical files on the IBM i system used by the **EDI810** program:

1. **EDIBOLTX**: EDI Bill of Lading transaction file.
2. **BOLEDIY**: Bill of Lading history or detail file.
3. **TRRTCD**: Transportation or routing code file.
4. **GSHAZM**: Hazardous materials file.
5. **GSPROD**: Product or inventory file.
6. **SHIPTO**: Ship-to customer information file.
7. **SHPADR**: Shipping address file.
8. **BBBOLY**: Secondary Bill of Lading-related file (possibly a backup or archive).

All files are opened with `DISP-SHR` (shared disposition), indicating they can be accessed concurrently by other jobs.

### Summary

The **EDI810.ocl36** OCL program serves as a control script to set up and execute the **EDI810** program for EDI Outbound Bill of Lading Tracking Maintenance. It initializes variables, sets switches, defines necessary files, and runs the core program. The process includes conditional logic to skip execution if a specific switch is set, ensuring flexibility in job control. The **EDI810** program handles the actual business logic, using the specified files for data processing.

- **External Program Called**: EDI810
- **Files Used**: EDIBOLTX, BOLEDIY, TRRTCD, GSHAZM, GSPROD, SHIPTO, SHPADR, BBBOLY

If you need further analysis of the **EDI810** RPG program’s logic or specific details about the files, please provide additional context or the RPG source code, and I can dive deeper!