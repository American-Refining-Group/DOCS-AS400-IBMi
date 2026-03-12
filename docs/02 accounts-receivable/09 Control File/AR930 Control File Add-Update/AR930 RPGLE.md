The provided `AR930.rpgle.txt` is an RPGLE (Report Program Generator Language Extended) program for IBM i (AS/400) systems, used for Accounts Receivable (A/R) control file maintenance. It is called by the OCL script `AR930.ocl36.txt` previously discussed. Below, I will explain the process steps, business rules, tables (files) used, and external programs called by the `AR930` RPGLE program.

---

### Process Steps of the RPGLE Program

The `AR930` program is designed to manage the A/R control file (`ARCONT`) through a workstation interface, allowing users to add, update, delete, or reactivate A/R company records. It includes screen handling, data validation, and file operations. Below is a step-by-step explanation of the program's process flow, based on the code structure and logic:

1. **Program Initialization**:
   - **Header Specifications**:
     - `DFTACTGRP(*NO)`: The program runs in a non-default activation group, allowing better control over resources.
     - `fixnbr(*zoned:*inputpacked)`: Ensures zoned and packed numeric fields are handled correctly during input.
     - `dftname(ar930)`: Sets the default program name to `AR930`.
   - **File Declarations**:
     - `arcont`: A/R control file, used for update and add operations (`uf a`), keyed by company number (2 bytes, starting at position 2).
     - `arcust`: A/R customer file, input-only (`if`), keyed by an 8-byte key starting at position 2.
     - `glmast`: General Ledger master file, input-only (`if`), keyed by an 11-byte key starting at position 2.
     - `ar930d`: Workstation file, used for interactive screen handling with Profound UI (`Handler('PROFOUNDUI(HANDLER)')`).
   - **Data Structures and Variables**:
     - `infds`: Information data structure to capture workstation status (e.g., function key presses).
     - `msg`: Array of 22 error messages, each 35 characters, defined in compile-time data (`CTDATA`).
     - Input specifications define fields for `arcont`, `arcust`, and `glmast`, mapping record layouts (e.g., `acco` for company number, `acname` for company name).

2. **Main Processing Loop**:
   - The program operates in a loop, displaying screens (`AR930S1` for inquiry/update, `AR930S2` for data entry) and processing user inputs based on function keys and data validation.
   - **Initial Screen Setup**:
     - If `qsctl` (control field) is blank, the program sets indicator `*in09` to '1', `*in01` to '1' (for `AR930S1`), and `qsctl` to 'R' to initialize the process.
     - Otherwise, it reads the workstation file (`ar930s1` or `ar930s2`) based on indicators `*in81` or `*in82`. If the read fails (indicator `*in09` is on), the `rollsr` subroutine handles roll-up/roll-down actions.
   - **Indicator Management**:
     - Clears indicators (e.g., `*in62` to `*in67`, `*in44`, `*in71`, `*in81`, `*in82`, `*in90`) and variables (`z2`, `z3`, `z8`, `z6`) to reset the state.
     - Handles roll keys (`*in54` for roll-up, `*in55` for roll-down) by calling `rollfw` or `rollbw` subroutines.

3. **Function Key Processing**:
   - The program responds to specific function keys, each triggering a subroutine or action:
     - **KA (F1 - Bypass Entry)**: Clears fields, sets `*in81` (update mode) and `*in11` (update mode), and jumps to the end of the loop.
     - **KD (F4 - Delete/Reactivate)**: In update mode (`*in11`), checks if the record is not deleted (`*in21` off) and calls `delete` to mark the record as deleted (`acdel = 'D'`). If deleted, calls `reacti` to reactivate (`acdel = 'A'`).
     - **KG (End of Screen)**: Sets `*inlr` (last record) to terminate the program.
     - **KJ (F10 - Entry Mode)**: Clears fields, sets `*in82` and `*in10` (entry mode), and resets other indicators.
     - **KK (F11 - Update Mode)**: Clears fields, sets `*in81` and `*in11` (update mode), and resets other indicators.

4. **Screen Processing**:
   - **AR930S1 (Inquiry/Update Screen)**:
     - Triggered by `*in01`. Calls the `s1` subroutine:
       - Reads `arcont` by company number (`maskey = co`).
       - If not found (`*in92` on), displays error messages (msg 6, 7) and sets `*in81` and `*in90`.
       - If found, calls `move` to populate screen fields and sets `*in11`, `*in62`, `*in82` for update mode.
     - Writes the `ar930s1` screen to display company data.
   - **AR930S2 (Entry Screen)**:
     - Triggered by `*in02`. Calls the `s2` subroutine:
       - Validates input data (see **Business Rules** below).
       - If in entry mode (`*in10`), checks if the company number is non-zero and does not exist in `arcont`. If it exists, displays error messages (msg 8, 9).
       - Writes the `ar930s2` screen to allow data entry.

5. **Data Validation and File Updates**:
   - The `s2` subroutine performs extensive validation (detailed in **Business Rules**).
   - If validation passes, the program updates or adds records to `arcont`:
     - In entry mode (`*in10`), adds a new record with `except` (opcode `eadd 70 92`).
     - In update mode (`*in11`), updates the existing record (`except` with conditions `70n92n25` for updates, or `88` for delete/reactivate).

6. **Subroutines**:
   - **s1**: Handles inquiry/update screen logic, retrieves `arcont` data, and populates screen fields.
   - **s2**: Validates input data and prepares for add/update operations.
   - **clear**: Resets all screen fields to blanks or zeros.
   - **move**: Transfers `arcont` fields to screen fields for display.
   - **rollsr**: Handles roll-up/roll-down key presses, resetting function key indicators.
   - **rollfw**: Reads the next `arcont` record for navigation.
   - **rollbw**: Reads the previous `arcont` record for navigation.
   - **delete**: Marks an `arcont` record as deleted (`acdel = 'D'`) after checking `arcust` for active records.
   - **reacti**: Reactivates a deleted record (`acdel = 'A'`).

7. **Termination**:
   - The program exits when `*inlr` is set (via `KG` key or end of processing), writing final screen output and closing files.

---

### Business Rules

The program enforces the following business rules during data entry and validation in the `s2` subroutine:

1. **Company Number Validation**:
   - The company number (`co`) cannot be zero in entry mode (msg 4: "COMPANY CANNOT BE ZERO - ENTER OR").
   - In entry mode, the company number must not already exist in `arcont` (msg 8: "CANNOT ADD - THIS RECORD EXISTS", msg 9: "PRESS CMD KEY 11 TO UPDATE").

2. **Company Name**:
   - The company name (`name`) cannot be blank (msg 13: "COMPANY NAME CANNOT BE BLANK").

3. **General Ledger Account Validation**:
   - For A/R GL number (`argl`), sales GL number (`slgl`), discount GL number (`dsgl`), cash GL number (`csgl`), EFT cash GL number (`efcg`), intercompany GL number (`icgl`), and finance charge GL number (`fcgl`):
     - If non-zero, the GL number must exist in `glmast` and not be marked as deleted (`gldel = 'D'`) or inactive (`gldel = 'I'`).
     - Errors trigger specific messages:
       - Msg 10: "INVALID ACCOUNTS RECEIVABLE GL NO"
       - Msg 11: "INVALID SALES GL NUMBER"
       - Msg 14: "INVALID DISCOUNT GL NUMBER"
       - Msg 15: "INVALID CASH GL NUMBER"
       - Msg 16: "INVALID INTER-CO G/L NUMBER"
       - Msg 18: "INVALID FINANCE CHARGE GL NUMBER"
       - Msg 22: "INVALID EFT CASH GL NUMBER"

4. **Security Code**:
   - The security code (`secr`) cannot be blank (msg 17: "SECURITY CODE CAN NOT BE BLANK").

5. **Aging Limits**:
   - Aging limits (`lmt1`, `lmt2`, `lmt3`, `lmt4`) correspond to aging buckets (0-30, 31-60, 61-90, 91-120/over 120 days from invoice date, as per revision JB01).
   - Rules:
     - None can be zero (msg 21: "INVALID AGING LIMITS").
     - Must be in ascending order: `lmt1 < lmt2 < lmt3 < lmt4` (msg 21 if violated).

6. **Deletion Rules**:
   - A company cannot be deleted if active customer records exist in `arcust` (msg 19: "COMPANY MASTER RECORDS EXIST", msg 20: "DELETION OF COMPANY NOT ALLOWED").
   - Deletion sets `acdel = 'D'`; reactivation sets `acdel = 'A'`.

7. **Aging by Invoice Date**:
   - Per revision JB01 (04/13/05), aging is based on invoice date rather than due date, with buckets redefined as 0-30, 31-60, 61-90, 91-120, and over 120 days.

8. **Inactive GL Accounts**:
   - Per revision JB02 (08/24/16), inactive GL accounts (`gldel = 'I'`) are treated the same as deleted accounts (`gldel = 'D'`) during validation.

9. **Numeric Field Handling**:
   - Per revision JB03 (09/27/22), the filler at positions 113-114 in `arcont` is defined as a 2-digit numeric field (N2.0).

---

### Tables (Files) Used

The program uses the following files:
1. **ARCONT**: A/R control file (update/add, keyed by company number).
   - Fields include `acdel` (delete flag), `acco` (company number), `acname` (company name), `acarj#` (next A/R journal number), `acslj#` (next sales journal number), `acargl` (A/R GL number), `acslgl` (sales GL number), `acdsgl` (discount GL number), `accsgl` (cash GL number), `actrgl` (intercompany GL number), `acfngl` (finance GL number), `acsecr` (security code), `acfinc` (finance charge %), `acffmo` (first fiscal month), `aclmt1` to `aclmt4` (aging limits), `acefcg` (EFT cash GL number), `acdtcl` (last close date), `acdtcr` (cash receipts date).
2. **ARCUST**: A/R customer file (input-only, keyed).
   - Used to check for active customer records during deletion.
   - Fields include `ardel` (delete flag) and `arco` (company number).
3. **GLMAST**: General Ledger master file (input-only, keyed).
   - Used to validate GL account numbers.
   - Fields include `gldel` (delete/inactive flag).
4. **AR930D**: Workstation file for screen interaction (Profound UI handler).
   - Defines two formats: `AR930S1` (inquiry/update) and `AR930S2` (data entry).

---

### External Programs Called

The RPGLE program **does not explicitly call any external programs** (e.g., via `CALL` operation). It relies entirely on internal subroutines (`s1`, `s2`, `clear`, `move`, `rollsr`, `rollfw`, `rollbw`, `delete`, `reacti`) and file operations. However, the program is invoked by the OCL script (`AR930.ocl36.txt`), which calls `GSGENIEC` and `GSY2K` before loading `AR930`. These are not called from within the RPGLE code but are part of the job setup.

---

### Summary

The `AR930` RPGLE program manages A/R control file maintenance through an interactive workstation interface. It supports adding, updating, deleting, or reactivating company records in `ARCONT`, with extensive validation against `ARCUST` and `GLMAST`. Key features include:
- **Process Flow**: Initializes, displays screens (`AR930S1`, `AR930S2`), processes function keys (KA, KD, KG, KJ, KK), validates input, and updates files.
- **Business Rules**: Enforces non-zero company numbers, non-blank names and security codes, valid GL accounts, ascending aging limits, and deletion restrictions.
- **Files**: Uses `ARCONT` (A/R control), `ARCUST` (A/R customers), `GLMAST` (GL master), and `AR930D` (workstation).
- **External Programs**: None called directly; relies on OCL setup (`GSGENIEC`, `GSY2K`).

The program is modernized with Profound UI for screen handling and includes Y2K compliance and aging revisions, ensuring robust A/R control file management.