The provided OCL (Operation Control Language) program, `AR136.ocl36.txt`, is executed on IBM midrange systems (e.g., AS/400 or IBM i) and is called conditionally from the main OCL program `AR136P.ocl36.txt` as part of an Accounts Receivable (AR) Electronic Funds Transfer (EFT) process. This program is responsible for generating an EFT Customer Accounts Receivable Due Report, which prints details of EFT transactions for each EFT customer. Below, I will explain the process steps, business rules, tables/files used, and external programs called.

### Process Steps of the OCL Program

OCL is a scripting language used to control program execution and manage file operations on IBM midrange systems. The steps in the `AR136.ocl36.txt` program are as follows:

1. **Comment Block**:
   ```ocl
   **
   ** EFT CUSTOMER(S) ACCCOUNTS RECEIVABLE DUE REPORT
   ** PRINTS THE EFT REPORT (ELECTRONIC FUNDS TRANSFER FOR EACH EFT CUSTOMER)
   **
   ```
   - This is a comment describing the purpose of the program: it generates a report detailing Accounts Receivable amounts due for EFT customers, specifically for Electronic Funds Transfer processing.

2. **Load the Program**:
   ```ocl
   // LOAD AR136
   ```
   - The `LOAD` command loads the program `AR136` into memory for execution. This is likely an RPG program (or another executable) that contains the logic to process data and produce the EFT report.

3. **File Specifications**:
   ```ocl
   // FILE NAME-CRTRAN,LABEL-?9?E?L'110,6'?,DISP-SHR
   // FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR
   // FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR
   ```
   - Three files are defined for use by the `AR136` program:
     - **CRTRAN**:
       - **Name**: `CRTRAN` (likely a transaction file for Accounts Receivable).
       - **Label**: `?9?E?L'110,6'?` - The file label is dynamically constructed using substitution expressions. `?9?` likely represents a library or prefix, and `?L'110,6'?` refers to a 6-character value (the bank upload date `kyupdt`) from positions 110-115 in a data area (as defined in `AR136P.rpgle.txt`). The 'E' suggests a prefix for EFT-related files (e.g., `GE` as seen in `AR136P.rpgle.txt`).
       - **Disposition**: `DISP-SHR` (shared mode, allowing multiple processes to access the file simultaneously).
     - **ARCUST**:
       - **Name**: `ARCUST` (likely a customer master file for Accounts Receivable).
       - **Label**: `?9?ARCUST` - The label uses the `?9?` substitution for the library or prefix.
       - **Disposition**: `DISP-SHR`.
     - **ARCONT**:
       - **Name**: `ARCONT` (Accounts Receivable control file).
       - **Label**: `?9?ARCONT` - Uses the `?9?` substitution for the library or prefix.
       - **Disposition**: `DISP-SHR`.

4. **Run the Program**:
   ```ocl
   // RUN
   ```
   - The `RUN` command executes the loaded `AR136` program, which uses the specified files (`CRTRAN`, `ARCUST`, `ARCONT`) to generate the EFT report for each EFT customer.

### Business Rules

Based on the OCL program and its context within the main OCL program (`AR136P.ocl36.txt`) and the RPG program (`AR136P.rpgle.txt`), the following business rules can be inferred:
1. **Report Generation**:
   - The `AR136` program generates a report listing Accounts Receivable amounts due for EFT customers, specifically for Electronic Funds Transfer transactions.
   - The report likely includes details such as customer information (from `ARCUST`), transaction amounts (from `CRTRAN`), and control data (from `ARCONT`).

2. **Dynamic File Naming**:
   - The `CRTRAN` file uses a dynamic label (`?9?E?L'110,6'?`), where the date portion (`?L'110,6'?`) corresponds to the bank upload date (`kyupdt`) validated by the `AR136P` RPG program. This ensures the report processes transactions for the specific date selected by the user.

3. **File Access**:
   - All files (`CRTRAN`, `ARCUST`, `ARCONT`) are opened in shared mode (`DISP-SHR`), indicating that the report generation process does not modify these files but reads them to extract data for the report.
   - This allows concurrent access by other processes, ensuring the report can run without locking the files.

4. **Conditional Execution**:
   - The `AR136` OCL program is called from the main OCL program (`AR136P.ocl36.txt`) only if the `status` field (position 109) is not 'Y' after running `AR136P`. This suggests that `AR136` is executed when the user input (company number and bank upload date) is valid, and the corresponding EFT file exists (as verified by `AR135TC` in `AR136P.rpgle.txt`).

5. **Data Dependency**:
   - The report relies on:
     - **CRTRAN**: Contains transaction data for the specified bank upload date, likely including EFT amounts due.
     - **ARCUST**: Provides customer details (e.g., customer name, EFT account information).
     - **ARCONT**: Supplies control information (e.g., company-specific settings or validation data).

### Tables/Files Used

1. **CRTRAN**:
   - **Purpose**: Likely an Accounts Receivable transaction file containing EFT transaction data for the selected bank upload date.
   - **Label**: `?9?E?L'110,6'?` (e.g., a file like `GExxxxxx` where `xxxxxx` is the bank upload date from `kyupdt`).
   - **Disposition**: Shared (`DISP-SHR`).
   - **Role**: Provides the transaction data (e.g., amounts due) for the EFT report.

2. **ARCUST**:
   - **Purpose**: Accounts Receivable customer master file, containing customer details such as names, account numbers, or EFT banking information.
   - **Label**: `?9?ARCUST` (library/prefix dynamically substituted via `?9?`).
   - **Disposition**: Shared (`DISP-SHR`).
   - **Role**: Supplies customer-specific information for the report.

3. **ARCONT**:
   - **Purpose**: Accounts Receivable control file, containing company-specific control data or configuration settings.
   - **Label**: `?9?ARCONT` (library/prefix dynamically substituted via `?9?`).
   - **Disposition**: Shared (`DISP-SHR`).
   - **Role**: Provides control or validation data (e.g., company settings) for report generation.

### External Programs Called

1. **AR136**:
   - **Purpose**: The main program loaded and executed by the OCL script. It is likely an RPG program (or similar executable) that processes the `CRTRAN`, `ARCUST`, and `ARCONT` files to generate the EFT Customer Accounts Receivable Due Report.
   - **Parameters**: The OCL does not explicitly list parameters, but the main OCL program (`AR136P.ocl36.txt`) calls `AR136` with nine parameters, including a dynamic value (`?9?`). These parameters likely include the company number (`kyco`), bank upload date (`kyupdt`), or other control values passed from the data area.

### Integration with Main OCL and RPG Programs

The `AR136` OCL program is called from the main OCL program (`AR136P.ocl36.txt`) as a conditional step after the RPG program `AR136P` validates user input. The integration works as follows:
- **Main OCL (`AR136P.ocl36.txt`)**:
  - Calls `GSGENIEC` for setup, runs `GSY2K` for date handling, and executes `AR136P` to prompt for and validate the company number (`kyco`) and bank upload date (`kyupdt`).
  - Checks the `status` field (position 109). If `status` is not 'Y', it calls `AR136` with the dynamic parameter `?9?` (likely a library or prefix).
  - The `ARCONT` file is shared between `AR136P` and `AR136`, ensuring consistent access to control data.

- **RPG Program (`AR136P.rpgle.txt`)**:
  - Validates the company number (`kyco`) against `arcont`, ensures `kyupdt` is non-zero, and checks if the EFT file (`GE` + `kyupdt`) exists via `AR135TC`.
  - Sets `status` to 'Y' if validations pass, which influences whether `AR136` is called in the main OCL.

- **AR136 OCL**:
  - Uses the validated `kyupdt` to construct the `CRTRAN` file name (e.g., `GExxxxxx`), ensuring the report processes the correct EFT transactions.
  - Reads customer data from `ARCUST` and control data from `ARCONT` to produce a comprehensive report.

### Notes
- The dynamic file label for `CRTRAN` (`?9?E?L'110,6'?`) aligns with the `AR136P` RPG program’s logic, where the file name is constructed as 'GE' + `kyupdt`. This indicates that `CRTRAN` is the EFT transaction file for the selected date.
- The report is likely formatted to include customer details, transaction amounts, and EFT-specific information (e.g., bank details) for each EFT customer.
- The shared disposition (`DISP-SHR`) ensures that the report generation does not interfere with other processes accessing these files.
- Without the source code for `AR136`, the exact report format and logic are inferred, but it likely involves reading `CRTRAN` for transactions, joining with `ARCUST` for customer details, and using `ARCONT` for control settings.

If you have additional details (e.g., the `AR136` program source, the structure of `CRTRAN`, `ARCUST`, or `ARCONT`, or the report layout), I can provide a more detailed analysis of the report’s content and processing logic.