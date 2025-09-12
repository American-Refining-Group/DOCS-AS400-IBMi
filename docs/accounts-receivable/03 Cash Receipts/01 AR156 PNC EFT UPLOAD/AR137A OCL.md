The provided document is an OCL (Operation Control Language) script named `AR137A.ocl36`, which is called by the main `AR156.ocl36` script. This script is responsible for creating a bank upload table for EFT (Electronic Funds Transfer) processing and generating a report for EFT transactions for each customer. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the AR137A.ocl36 Program

1. **Clear the AREFTS File**:
   - The script begins with the `CLRPFM` command to clear the file `?9?AREFTS`.
   - `?9?` is a placeholder for a dynamically assigned library prefix (set earlier in the main `AR156.ocl36` script, likely via the `LOCAL OFFSET-501,DATA-'?9?'` command).
   - This step ensures that the `AREFTS` file, which serves as a staging file for EFT data, is empty before new data is written to it.

2. **Load and Run AR137A Program**:
   - The script loads the program `AR137A` (likely an RPG or similar program designed for EFT processing).
   - The following files are opened in shared mode (`DISP-SHR`):
     - `AREFTD` with label `?9?E?L'110,6'?`: An input file containing EFT data, where `?L'110,6'?` likely represents a date or batch identifier (from position 110, length 6, as set in `AR156P`).
     - `AREFTD` with label `?9?AREFTX`: Another EFT data file, possibly an alternate or temporary file used for processing.
     - `AREFTS` with label `?9?AREFTS`: The output staging file for EFT data, which was cleared in the previous step.
     - `ARCONT` with label `?9?ARCONT`: A control or configuration file, likely containing company or system settings.
     - `ARCUST` with label `?9?ARCUST`: A customer master file, containing customer details relevant to EFT processing.
   - The `RUN` command executes the `AR137A` program, which processes the input EFT data (`AREFTD`), retrieves necessary information from `ARCONT` and `ARCUST`, and populates the `AREFTS` file with processed EFT data. Additionally, it generates a report detailing EFT transactions for each EFT customer.

---

### Business Rules

1. **Clear Output File Before Processing**:
   - The `AREFTS` file must be cleared before processing to ensure no residual data from previous runs affects the current EFT data generation.
   - This ensures data integrity and prevents duplicate or incorrect transactions in the output.

2. **Shared File Access**:
   - All files (`AREFTD`, `AREFTS`, `ARCONT`, `ARCUST`) are opened in shared mode (`DISP-SHR`), allowing concurrent access by other processes or programs, which is critical in a multi-user environment.

3. **EFT Data Processing**:
   - The `AR137A` program processes EFT data from the input file `AREFTD` (labeled `?9?E?L'110,6'?` or `?9?AREFTX`) to create a bank upload table in `AREFTS`.
   - It uses customer data from `ARCUST` and configuration data from `ARCONT` to ensure accurate EFT transaction details.

4. **Report Generation**:
   - The program generates an EFT report listing transactions for each EFT customer, providing a record of the funds transfer details for auditing or verification purposes.

5. **Dynamic File Naming**:
   - The use of `?9?` as a library prefix and `?L'110,6'?` as a dynamic label indicates that the script is designed to work with dynamically assigned libraries and date-specific files, ensuring flexibility across different environments or batch dates.

---

### Tables (Files) Used

1. **AREFTD** (`?9?E?L'110,6'?`, `?9?AREFTX`):
   - Input file(s) containing EFT data to be processed.
   - Two labels are specified, suggesting either multiple sources or alternate files for EFT data (e.g., `?9?AREFTX` might be a temporary or alternate dataset).
   - Opened in shared mode (`DISP-SHR`).

2. **AREFTS** (`?9?AREFTS`):
   - Output staging file for processed EFT data, cleared at the start of the script.
   - Used to store the bank upload table created by the `AR137A` program.
   - Opened in shared mode (`DISP-SHR`).

3. **ARCONT** (`?9?ARCONT`):
   - Control or configuration file, likely containing company or system-level settings for EFT processing.
   - Opened in shared mode (`DISP-SHR`).

4. **ARCUST** (`?9?ARCUST`):
   - Customer master file, containing customer details such as account numbers or EFT preferences.
   - Opened in shared mode (`DISP-SHR`).

---

### External Programs Called

1. **AR137A**:
   - The main program loaded and executed by the script.
   - Responsible for processing EFT data from `AREFTD`, using `ARCONT` and `ARCUST` for reference, populating `AREFTS`, and generating an EFT report for each customer.

---

### Summary

The `AR137A.ocl36` script is a component of the EFT process that prepares a bank upload table and generates a report for EFT transactions. It starts by clearing the `AREFTS` staging file to ensure a clean slate, then loads and runs the `AR137A` program, which processes EFT data from `AREFTD`, references `ARCONT` and `ARCUST`, and writes the results to `AREFTS`. The program also produces a report detailing EFT transactions for each customer. The script uses shared file access and dynamic file naming for flexibility, ensuring robust data handling in a multi-user environment.