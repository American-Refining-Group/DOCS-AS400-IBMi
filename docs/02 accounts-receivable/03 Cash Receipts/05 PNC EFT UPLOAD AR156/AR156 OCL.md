The provided document is an OCL (Operation Control Language) script used in IBM System/36 or AS/400 environments to manage the creation of an ACH (Automated Clearing House) file for EFT (Electronic Funds Transfer) withdrawals through PNC Bank. Below is a detailed explanation of the process steps, followed by a list of external programs called and tables (files) used.

---

### Process Steps of the AR156.ocl36 Program

1. **Initial Setup and Environment Check**:
   - The script begins with a call to the program `GSGENIEC`, likely a utility or initialization program to set up the environment or validate conditions.
   - A condition checks if the value at position 506 (length 3) in a local variable is 'YES'. If true, the program terminates immediately (`RETURN`).
   - Local variables are initialized with `LOCAL BLANK-*ALL`, clearing all local storage.
   - The `GSY2K` command is executed, possibly to handle Year 2000 date compliance or set date-related parameters.
   - A local variable at offset 501 is set to the value `?9?`, which is likely a library or prefix identifier used dynamically throughout the script.

2. **Load and Run AR156P Program**:
   - The program `AR156P` is loaded.
   - Two files are opened:
     - `ARCONT` (likely a control or configuration file) with the label `?9?ARCONT` in shared mode (`DISP-SHR`).
     - `ARCUST` (likely a customer master file) with the label `?9?ARCUST` in shared mode.
   - The `RUN` command executes `AR156P`.
   - A condition checks if the value at position 109 (length 1) is 'Y'. If true, the program jumps to the `END` tag, terminating the process early.

3. **NACHA File Creation for EFT Withdrawals**:
   - The `GSY2K` command is called again, possibly to ensure date compliance for subsequent operations.
   - If a file named `?9?AR156S` exists in `DATAF1`, it is deleted (`DLTF ?9?AR156S`).
   - A conditional copy operation (`CPYF`) is performed based on the value of `?9?` (the library prefix):
     - If `?9?` is 'G', the file `?9?E?L'110,6'?` (likely an EFT data file) is copied from `QS36F` to `?9?AREFTD` in `QS36F`, replacing the target file (`MBROPT(*REPLACE)`), without creating a new file (`CRTFILE(*NO)`) and without format checking (`FMTOPT(*NOCHK)`).
     - An additional copy is made to `QS36FTEST/?9?AREFTD` under the same conditions.

4. **Clear AREFTS File**:
   - The file `?9?AREFTS` (likely a temporary or staging file for EFT data) is cleared using `CLRPFM`.

5. **Load and Run AR137A Program**:
   - The program `AR137A` is loaded.
   - Files opened include:
     - `AREFTD` (input EFT data file) with labels `?9?AR156S` and `?9?AREFTX` in shared mode.
     - `AREFTS` (output EFT staging file) with label `?9?AREFTS` in shared mode.
     - `ARCONT` and `ARCUST` (same as before) in shared mode.
   - The `RUN` command executes `AR137A`, which likely processes EFT data and populates the `AREFTS` file.

6. **Build ARDTGGC File (if needed)**:
   - If the file `?9?ARDTGGC` exists in `DATAF1`, a new file is built (`BLDFILE`) with the following parameters:
     - File name: `?9?ARDTGGC`
     - Type: Input file (`I`)
     - Record size: 500 bytes
     - Key length: 11 bytes
     - Other parameters specify file structure and defaults.
   - This step creates a file for additional EFT or bank-related data processing.

7. **Clear EFTFIL File**:
   - The file `?9?EFTFIL` (likely the final ACH output file) is cleared using `CLRPFM`.

8. **Load and Run AR156 Program**:
   - The program `AR156` is loaded.
   - Files opened include:
     - `AREFTS` (EFT staging file) with label `?9?AREFTS` in shared mode.
     - `ARCONT` and `ARCUST` (same as before) in shared mode.
     - `ARCUSP` (likely a customer-specific EFT or payment file) with label `?9?ARCUSP` in shared mode.
     - `EFTFILE` (the final ACH output file) with label `?9?EFTFIL` in shared mode.
   - The `RUN` command executes `AR156`, which likely formats the EFT data into a NACHA-compliant ACH file for upload to PNC Bank.

9. **Cleanup and Termination**:
   - If the file `?9?AR156S` exists in `DATAF1`, it is deleted (`DLTF ?9?AR156S`).
   - Local variables are cleared again with `LOCAL BLANK-*ALL`.
   - The program ends at the `END` tag.

---

### External Programs Called
The script calls the following external programs:
1. **GSGENIEC**: Likely an initialization or environment setup utility.
2. **AR156P**: A program that performs preliminary processing or validation for EFT data.
3. **AR137A**: A program that processes EFT data and populates the `AREFTS` file.
4. **AR156**: The main program that generates the NACHA-compliant ACH file.

---

### Tables (Files) Used
The script references the following files (tables):
1. **ARCONT** (`?9?ARCONT`): A control or configuration file, used in `AR156P`, `AR137A`, and `AR156`.
2. **ARCUST** (`?9?ARCUST`): A customer master file, used in `AR156P`, `AR137A`, and `AR156`.
3. **AREFTD** (`?9?AR156S`, `?9?AREFTX`): An EFT data file, used as input in `AR137A` and copied to/from in earlier steps.
4. **AREFTS** (`?9?AREFTS`): A staging file for EFT data, used in `AR137A` and `AR156`.
5. **ARCUSP** (`?9?ARCUSP`): A customer-specific EFT or payment file, used in `AR156`.
6. **EFTFILE** (`?9?EFTFIL`): The final ACH output file, used in `AR156`.
7. **ARDTGGC** (`?9?ARDTGGC`): A dynamically built file for additional EFT or bank data, created if needed.
8. **?9?E?L'110,6'?**: An EFT source file in `QS36F` or `*LIBL`, copied to `AREFTD`.

---

### Summary
The OCL script orchestrates the creation of an ACH file for EFT withdrawals through PNC Bank. It involves initializing the environment, copying and processing EFT data, clearing and building temporary files, and generating the final NACHA-compliant file. The process is modular, relying on three main programs (`AR156P`, `AR137A`, `AR156`) and a utility (`GSGENIEC`), with multiple files used for data storage and transfer. The script uses dynamic library prefixes (`?9?`) and conditional logic to ensure flexibility and proper execution.