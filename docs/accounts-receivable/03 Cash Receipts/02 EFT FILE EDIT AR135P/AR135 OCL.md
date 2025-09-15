The provided document, `AR135.ocl36.txt`, is an **OCL (Operation Control Language)** program used on IBM System/36 or AS/400 systems to control the execution of an Accounts Receivable (AR) Electronic Funds Transfer (EFT) process, specifically for individual entry processing. This program is called from the main OCL program (`AR135P.ocl36.txt`). Below, I will explain the process steps, identify the business rules, list the tables/files used, and note any external programs called.

---

### Process Steps of the AR135 OCL Program

The OCL program `AR135` orchestrates the execution of an RPG program and related procedures for processing EFT transactions. Here’s a step-by-step breakdown of the process:

1. **Comment Block**:
   ```
   **
   ** AR EFT INDIVIDUAL ENTRY
   **
   ```
   - This is a descriptive comment indicating the purpose of the program, which is to handle "Accounts Receivable Electronic Funds Transfer (EFT) Individual Entry."

2. **Invoke SCPROCP Procedure**:
   ```
   ** SCPROCP ,,,,,,,,?9?
   **       ?9?E?L'110,6'?  AR EFT TRANSACTION WORKFILE
   ```
   - The `SCPROCP` procedure is called with eight commas (placeholders for optional parameters) and the substitution variable `?9?`.
   - The second line defines a parameter: `?9?E?L'110,6'?`, which is described as the "AR EFT TRANSACTION WORKFILE."
   - `?9?` is a substitution variable resolved at runtime (e.g., `PROD` for production).
   - `?L'110,6'?` refers to a 6-byte value at memory location 110, likely a dynamic file label or identifier.
   - `SCPROCP` is likely a system or custom procedure for setting up the environment or initializing the EFT workfile.

3. **GSY2K Command**:
   ```
   // GSY2K
   ```
   - This invokes the `GSY2K` command, likely a system utility for Year 2000 (Y2K) compliance or date handling.
   - It may set date-related parameters or validate date formats for the EFT process.

4. **Set Switch 1**:
   ```
   // SWITCH 1XXXXXXX
   ```
   - This sets Switch 1 to `1` (on) while leaving the other seven switches undefined (`X` indicates no change).
   - Switches are used to control program flow or indicate specific conditions (e.g., enabling a specific processing mode).

5. **Load Program `AR135`**:
   ```
   // LOAD AR135
   ```
   - This loads the RPG program `AR135` into memory for execution.
   - `AR135` is the main program responsible for processing EFT transactions.

6. **Define Files**:
   ```
   // FILE NAME-CRTRAN,LABEL-?9?E?L'110,6'?,DISP-SHR,EXTEND-100
   // FILE NAME-CRTRANR,LABEL-?9?E?L'110,6'?,DISP-SHR
   // FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR
   // FILE NAME-ARCUSTX,LABEL-?9?ARCUSX,DISP-SHR
   // FILE NAME-GLMAST,LABEL-?9?GLMAST,DISP-SHR
   // FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR
   // FILE NAME-ARDETL,LABEL-?9?ARDETL,DISP-SHR
   ```
   - These lines define the files used by the `AR135` program:
     - **CRTRAN**: EFT transaction workfile with a dynamic label (`?9?E?L'110,6'?`, e.g., `PRODE123456` if `?9?` is `PROD` and `?L'110,6'?` is `123456`). `DISP-SHR` allows shared access, and `EXTEND-100` reserves space for 100 additional records.
     - **CRTRANR**: Another EFT transaction file with the same label as `CRTRAN`, also shared.
     - **ARCUST**: Customer master file with label `?9?ARCUST` (e.g., `PRODARCUST`), shared.
     - **ARCUSTX**: Customer index or extension file with label `?9?ARCUSX`, shared.
     - **GLMAST**: General Ledger master file with label `?9?GLMAST`, shared.
     - **ARCONT**: Accounts Receivable control file with label `?9?ARCONT`, shared.
     - **ARDETL**: Accounts Receivable detail file with label `?9?ARDETL`, shared.
   - These files store transaction, customer, and financial data necessary for EFT processing.

7. **Run the Program**:
   ```
   // RUN
   ```
   - This executes the loaded `AR135` program, which processes EFT transactions using the defined files.

8. **Call Procedure `AR135B`**:
   ```
   // AR135B ,,,,,,,,?9?
   ```
   - This invokes the `AR135B` procedure with eight commas (placeholders) and the `?9?` variable.
   - `AR135B` is likely a follow-up procedure for post-processing, validation, or reporting related to EFT transactions.

9. **Call Procedure `AR135A`**:
   ```
   // AR135A ,,,,,,,,?9?
   ```
   - This invokes the `AR135A` procedure with the same parameter structure as `AR135B`.
   - `AR135A` may handle additional EFT processing steps, such as generating output files or updating records.

10. **Reset Switch 1**:
    ```
    // SWITCH 0XXXXXXX
    ```
    - This resets Switch 1 to `0` (off) while leaving the other switches unchanged.
    - This cleanup step ensures that the switch state is reset for subsequent runs or other programs.

---

### Business Rules

The OCL program enforces the following business rules, inferred from the structure and context:

1. **Initialization with SCPROCP**:
   - The `SCPROCP` procedure is called to initialize the EFT process, likely setting up the transaction workfile (`CRTRAN`) or validating the environment.
   - The use of `?9?` and `?L'110,6'?` suggests that the system supports multiple environments (e.g., production, test) and dynamically resolves file labels.

2. **Y2K Compliance**:
   - The `GSY2K` command ensures that date-related processing is Y2K-compliant, critical for financial transactions like EFTs where accurate date handling is essential.

3. **Switch-Based Control**:
   - Setting Switch 1 (`1XXXXXXX`) enables a specific processing mode or condition for `AR135`. For example, it might indicate that EFT transactions are being processed in a particular context (e.g., individual entry mode).
   - Resetting Switch 1 (`0XXXXXXX`) ensures that the mode is cleared after processing, preventing unintended effects in subsequent jobs.

4. **File Access and Sharing**:
   - All files are opened in shared mode (`DISP-SHR`), allowing concurrent access by other programs or jobs. This is typical in multi-user environments where EFT processing may occur alongside other AR or GL operations.
   - The `EXTEND-100` parameter for `CRTRAN` ensures sufficient space for additional transaction records, indicating that the system anticipates dynamic growth in transaction volume.

5. **Sequential Processing**:
   - The program follows a strict sequence: initialize (`SCPROCP`), load and run `AR135`, then execute `AR135B` and `AR135A`. This suggests a multi-step process, such as:
     - `AR135`: Core EFT transaction processing (e.g., validating and posting transactions).
     - `AR135B`: Post-processing (e.g., generating reports or updating ledgers).
     - `AR135A`: Finalization or cleanup (e.g., creating output files or logging results).

6. **Dynamic File Labeling**:
   - The use of `?9?` and `?L'110,6'?` for file labels ensures flexibility, allowing the program to operate in different environments (e.g., production, test) without hardcoding file names.

7. **Data Integrity**:
   - The use of multiple files (`CRTRAN`, `CRTRANR`, `ARCUST`, etc.) suggests a robust data model where transactions are validated against customer, control, and ledger data to ensure accuracy and consistency.

---

### Tables/Files Used

The OCL program references the following files, all opened in shared mode (`DISP-SHR`):
1. **CRTRAN**:
   - Label: `?9?E?L'110,6'?` (e.g., `PRODE123456`)
   - Description: EFT transaction workfile
   - Additional: `EXTEND-100` reserves space for 100 additional records
2. **CRTRANR**:
   - Label: `?9?E?L'110,6'?` (same as `CRTRAN`)
   - Description: Likely a related transaction file, possibly for reference or redundant storage
3. **ARCUST**:
   - Label: `?9?ARCUST` (e.g., `PRODARCUST`)
   - Description: Customer master file, containing customer details
4. **ARCUSTX**:
   - Label: `?9?ARCUSX` (e.g., `PRODARCUSX`)
   - Description: Customer index or extension file, possibly for quick lookups or additional customer data
5. **GLMAST**:
   - Label: `?9?GLMAST` (e.g., `PRODGLMAST`)
   - Description: General Ledger master file, containing account information
6. **ARCONT**:
   - Label: `?9?ARCONT` (e.g., `PRODARCONT`)
   - Description: Accounts Receivable control file, likely containing configuration or summary data
7. **ARDETL**:
   - Label: `?9?ARDETL` (e.g., `PRODARDETL`)
   - Description: Accounts Receivable detail file, storing transaction details

---

### External Programs Called

The OCL program explicitly calls or references the following external programs or procedures:
1. **SCPROCP**:
   - A procedure for initializing the EFT process, likely setting up the transaction workfile or environment.
2. **AR135**:
   - The main RPG program loaded and executed to process EFT transactions.
3. **AR135B**:
   - A procedure called after `AR135`, likely for post-processing or reporting.
4. **AR135A**:
   - A procedure called after `AR135B`, possibly for finalization or additional processing.

Additionally, the `GSY2K` command is invoked, which may be a system utility rather than a standalone program.

---

### Summary

The `AR135` OCL program, called from the main `AR135P` OCL program, handles the core processing of Accounts Receivable EFT individual entries. It follows these key steps:
- Initializes the environment with `SCPROCP` and `GSY2K`.
- Sets Switch 1 to control processing mode.
- Loads and runs the `AR135` RPG program, using multiple files (`CRTRAN`, `CRTRANR`, `ARCUST`, etc.) for transaction processing.
- Executes `AR135B` and `AR135A` for post-processing and finalization.
- Resets Switch 1 for cleanup.

**Business Rules**:
- Ensures Y2K compliance and dynamic file labeling for flexibility.
- Uses shared file access for concurrent processing.
- Follows a structured, multi-step process with initialization, core processing, and post-processing.
- Maintains data integrity through multiple file validations.

**Files Used**:
- `CRTRAN`, `CRTRANR`, `ARCUST`, `ARCUSTX`, `GLMAST`, `ARCONT`, `ARDETL`

**External Programs Called**:
- `SCPROCP`, `AR135`, `AR135B`, `AR135A`

If you need further details about the RPG program `AR135` or the specific logic of `AR135A`/`AR135B`, additional information (e.g., RPG source code or procedure definitions) would be required, as these are not included in the OCL script.