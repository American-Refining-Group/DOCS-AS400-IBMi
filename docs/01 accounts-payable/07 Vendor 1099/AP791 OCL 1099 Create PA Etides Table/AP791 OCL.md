The provided document is an Operation Control Language (OCL) program, specifically `AP791.ocl36.txt`, used in IBM AS/400 or System/36 environments to handle operations related to printing 1099 forms. Below is an explanation of the process steps, the external programs called, and the tables/files used in the program.

### Process Steps of the AP791 OCL Program

1. **Prompt for the Year**:
   - The program starts by prompting the user to input a four-digit year for the 1099 forms. This is indicated by the comment:
     ```
     // * 'WHAT YEAR ARE THESE 1099''S FOR?  PLEASE ENTER THE "FOUR DIGIT YEAR"...'
     ```
   - The `LOCAL OFFSET-100,DATA-'?10R?'` statement suggests that the program reserves a local variable at offset 100 to store the user-entered year, likely as a 4-character input (e.g., "2025").

2. **Evaluate Parameter**:
   - The statement `EVALUATE P13='?9?P1099?L'102,2'?` evaluates a parameter (`P13`) to construct a file or label name dynamically. The `?9?` is a placeholder for a library or prefix, and `P1099` combined with `?L'102,2'?` likely incorporates the last two digits of the entered year (e.g., "25" for 2025) to form a specific file name or label (e.g., `P109925`).

3. **Clear Physical File**:
   - The `CLRPFM FILE(?9?PATAX)` command clears the physical file `PATAX` in the library specified by `?9?`. This ensures that the file is empty before new data is loaded or processed, likely to store 1099-related data.

4. **Delete File (Conditional)**:
   - The commented-out `DLTF FILE(?13?)` and the conditional `IF DATAF1-?13? DLTF FILE(?13?)` suggest that the program checks if a file (referenced by `?13?`, the evaluated file name from step 2) exists. If it does, the file is deleted (`DLTF`). This step ensures that any existing file with the same name is removed to avoid conflicts when creating a new file.

5. **GSY2K Execution**:
   - The `GSY2K` command calls an external program or procedure, likely a system utility or a custom program to handle year 2000 compliance or date-related processing for the 1099 forms. This could involve formatting or validating the year entered by the user.

6. **Load and Run AP791**:
   - The `LOAD AP791` statement loads the main program (likely an RPG program named `AP791`) into memory for execution.
   - The `RUN` command executes the loaded `AP791` program.

7. **File Definitions**:
   - The program specifies two files:
     - `FILE NAME-IRSTAX,LABEL-IRSTAX,DISP-SHR`: Opens the `IRSTAX` file (likely a master file containing tax-related data) in shared mode (`DISP-SHR`), allowing multiple processes to access it.
     - `FILE NAME-PA1099,LABEL-?9?PATAX,DISP-SHR`: Opens the `PATAX` file in the library `?9?` (with the label `PATAX`) in shared mode. This file is likely used to store processed 1099 data.
   - The commented-out line `** FILE NAME-IRSTAX,LABEL-IRSTAX,DISP-SHR` suggests a redundant or alternative definition for the `IRSTAX` file, possibly for documentation or historical purposes.

8. **Printer Configuration**:
   - The `PRINTER NAME-PRINT,CPI-15` statement configures the printer output for the program, specifying a printer named `PRINT` with a character-per-inch (CPI) setting of 15, suitable for condensed printing of 1099 forms.

9. **Copy File**:
   - The `CPYF FROMFILE(QS36F/?9?PATAX) TOFILE(QS36F/?13?) CRTFILE(*YES)` command copies data from the `PATAX` file (in library `QS36F/?9?`) to a target file specified by `?13?` (the dynamically evaluated file name, e.g., `P109925`). The `CRTFILE(*YES)` parameter indicates that the target file will be created if it does not already exist.

### Summary of Process Flow
The program:
- Prompts the user for the 1099 year.
- Constructs a dynamic file name based on the year.
- Clears the `PATAX` file to prepare for new data.
- Optionally deletes an existing file with the dynamically generated name.
- Calls the `GSY2K` utility for year-related processing.
- Loads and runs the `AP791` program, which processes data using the `IRSTAX` and `PATAX` files.
- Configures the printer for output.
- Copies the processed data from `PATAX` to a new file (e.g., `P109925`) for storage or further use.

### External Programs Called
1. **GSY2K**: A utility or program likely used for year 2000 compliance or date processing.
2. **AP791**: The main RPG program loaded and executed to process the 1099 data.

### Tables/Files Used
1. **IRSTAX**:
   - Label: `IRSTAX`
   - Description: Likely a master file containing tax-related data (e.g., vendor or tax information for 1099 forms).
   - Access: Shared mode (`DISP-SHR`).
2. **PATAX**:
   - Label: `?9?PATAX` (in library specified by `?9?`).
   - Description: A file used to store processed 1099 data, cleared before processing.
   - Access: Shared mode (`DISP-SHR`).
3. **Dynamic File (e.g., P109925)**:
   - Label: `?13?` (e.g., `P109925`, based on the evaluated year).
   - Description: A dynamically named file created to store the output of the `PATAX` file after processing.
   - Access: Created via `CPYF` with `CRTFILE(*YES)`.

### Notes
- The placeholders `?9?` and `?13?` are substituted at runtime, typically representing a library name and a dynamically constructed file name, respectively.
- The program assumes an AS/400 or System/36 environment, where OCL is used to control job execution and file operations.
- The `AP791` RPG program likely contains the core logic for processing 1099 data, while the OCL script handles setup, file management, and output configuration.

If you need further clarification or analysis (e.g., the logic inside `AP791` or specific file structures), please provide additional details or related files!