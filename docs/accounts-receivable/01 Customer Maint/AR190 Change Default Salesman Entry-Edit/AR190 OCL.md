The document outlines two related processes in an RPG .OCL (Operation Control Language) program for managing A/R (Accounts Receivable) Salesman Change Transactions: one for adding/updating transactions (AR190) and another for editing transactions (AR191). These processes appear to be part of a legacy system, likely running on an IBM AS/400 or similar platform, given the .OCL syntax and file structure references.

Below, I’ll explain the process steps for each section, list the external programs called, and identify the tables (files) used, based on the provided document.

---

### Process Steps of the RPG .OCL Program

#### 1. A/R Salesman Change Transaction Add/Update (AR190)
This section describes the process for adding or updating salesman change transactions in the Accounts Receivable system.

**Process Steps:**
1. **Initialization and File Setup (BLDFILE):**
   - The program uses the `BLDFILE` command to create or prepare a file named `?9?ARSLST` (likely `ARSLST` with a system-specific prefix `?9?`).
   - Parameters: `I` (input mode), `RECORDS,500` (allocates space for 500 records), `256` (record length), `2,8` (key field specifications, likely indicating a key starting at position 2 with a length of 8), `DFILE` (data file type), and `50` (possibly a file size or block factor).
   - This step ensures the `ARSLST` file is ready to store salesman change transaction data.

2. **File Definitions:**
   - The program specifies four files to be loaded with `DISP-SHRMM` (shared multiple access mode, allowing concurrent access):
     - `ARSLST`: The primary file for salesman change transactions.
     - `ARCONT`: Likely a control file containing configuration or control data for the A/R system.
     - `ARCUST`: The customer master file, containing customer-related data.
     - `GSTABL`: A general table file, possibly containing lookup or reference data (e.g., salesman codes, statuses).
   - These files are labeled with a `?9?` prefix, indicating a system or library-specific naming convention.

3. **Program Execution:**
   - The `LOAD AR190` command loads the RPG program `AR190`.
   - The `RUN` command executes the `AR190` program, which processes the add/update logic for salesman change transactions.
   - The program likely:
     - Reads input data (e.g., new or updated salesman assignments).
     - Validates the data against `ARCUST` (customer records) and `GSTABL` (reference data).
     - Updates or adds records to the `ARSLST` file, possibly using `ARCONT` for control parameters.
     - Ensures data integrity and consistency across the files.

**Purpose:**
- This process allows users to add new salesman change transactions or update existing ones, likely reassigning customers to different salesmen in the A/R system.

---

#### 2. A/R Salesman Change Transaction Edit (AR191)
This section describes the process for editing existing salesman change transactions.

**Process Steps:**
1. **File Definitions:**
   - The program specifies three files with `DISP-SHRMM`:
     - `ARSLST`: The salesman change transaction file (same as in AR190).
     - `ARCUST`: The customer master file.
     - `GSTABL`: The general table file.
   - Note: The `ARCONT` file is not used in this process, suggesting that editing transactions does not require control file data.

2. **Program Execution:**
   - The `LOAD AR191` command loads the RPG program `AR191`.
   - The `RUN` command executes the `AR191` program, which handles the editing of salesman change transactions.
   - The program likely:
     - Retrieves existing transaction records from `ARSLST`.
     - Allows users to modify transaction details (e.g., salesman codes, customer assignments).
     - Validates changes against `ARCUST` (to ensure valid customer data) and `GSTABL` (for valid reference data).
     - Updates the `ARSLST` file with the modified records.

**Purpose:**
- This process enables users to correct or modify existing salesman change transactions, ensuring the A/R system reflects accurate salesman-customer assignments.

---

### External Programs Called
The document explicitly references two RPG programs:
1. **AR190**: Handles the add/update process for salesman change transactions.
2. **AR191**: Handles the edit process for salesman change transactions.

No additional external programs are mentioned in the provided .OCL code.

---

### Tables (Files) Used
The following files are referenced in the .OCL program:
1. **ARSLST**: The primary file for storing salesman change transactions. Used in both AR190 and AR191.
2. **ARCUST**: The customer master file, containing customer data. Used in both AR190 and AR191.
3. **GSTABL**: A general table file, likely containing reference data such as salesman codes or statuses. Used in both AR190 and AR191.
4. **ARCONT**: A control file, likely containing configuration or control data for the A/R system. Used only in AR190.

---

### Summary
- **AR190 (Add/Update):**
  - Prepares the `ARSLST` file using `BLDFILE`.
  - Loads and uses `ARSLST`, `ARCUST`, `GSTABL`, and `ARCONT` files.
  - Executes the `AR190` program to add or update salesman change transactions.
- **AR191 (Edit):**
  - Loads and uses `ARSLST`, `ARCUST`, and `GSTABL` files.
  - Executes the `AR191` program to edit existing salesman change transactions.
- **External Programs**: `AR190`, `AR191`.
- **Tables Used**: `ARSLST`, `ARCUST`, `GSTABL`, `ARCONT` (only in AR190).

If you have additional details (e.g., the RPG source code for AR190/AR191 or specific transaction formats), I can provide a more detailed explanation of the processing logic. Let me know if you’d like me to search for related information or analyze any uploaded files!