The provided **AR932.rpgle.txt** is an RPGLE (RPG IV) program, converted from an earlier RPG version (as indicated by the TARGET/400 source conversion comments), used for **Accounts Receivable Fuels Customer Maintenance**. This program is called from the OCL program (`AR932.ocl36.txt`) and provides an interactive screen-based interface for maintaining customer records in an A/R fuels system on an IBM midrange platform (likely IBM System/36 or AS/400). Below, I’ll explain the **process steps**, **business rules**, **tables (files) used**, and **external programs called**.

---

### Process Steps of the RPGLE Program

The RPGLE program (`AR932`) is designed to manage customer records for fuel-related accounts receivable, allowing users to add, update, delete, or reactivate records through a workstation interface. The program uses multiple files and handles user interactions via screens (`AR932S1` and `AR932S2`). Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - **Header Specifications (`H` specs)**:
     - `DFTACTGRP(*NO)`: Runs in a named activation group, typical for ILE programs.
     - `DFTNAME(AR932)`: Specifies the default program name.
     - `FIXNBR(*ZONED:*INPUTPACKED)`: Ensures zoned and packed numeric fields are handled correctly during input.
   - **File Declarations (`F` specs)**:
     - Defines files (`ARFUEL`, `ARCUST`, `ARCONT`, `GSCONT`, `PRINT`, and workstation file `AR932D`).
     - Initializes the workstation file with a handler (`PROFOUNDUI(HANDLER)`) for modern UI integration and an information data structure (`INFDS`) for screen status.
   - **Data Structures and Arrays**:
     - Defines arrays `SEP` and `SEP2` for formatting output (likely separators for reports).
     - Defines `MSG` array with 12 predefined messages for user feedback (e.g., "THIS RECORD PREVIOUSLY DELETED", "RECORD NOT FOUND").
     - Defines `INFDS` to capture workstation status (e.g., function key presses).

2. **Initial Setup and Screen Control**:
   - Checks `QSCTL` (likely a control variable) to determine the initial screen mode:
     - If `QSCTL` is blank, sets `*IN09` and `*IN01` to '1' (on), indicating initial screen display (`AR932S1`), and sets `QSCTL` to 'R'.
     - Otherwise, clears `*IN09`, `*IN01`, and `*IN02`, and attempts to read from screen formats `AR932S1` (indicator 81) or `AR932S2` (indicator 82).
   - If `*IN09` is on (error or initial state), calls the `ROLLSR` subroutine to handle screen navigation and clears function key indicators (`*INKA`, `*INKD`, etc.).
   - Initializes flags (`*IN15`, `*IN75`) and sets up time/date fields (`TIMDAT`, `TIME`, `DATE`) for display or reporting.
   - Chains to `ARCONT` to retrieve control data, setting `*IN75` and `*IN15` if successful.

3. **Main Processing Loop**:
   - The program operates in a loop, handling user inputs via function keys (`*INKA`, `*INKD`, `*INKG`, `*INKJ`, `*INKK`) and screen formats (`AR932S1`, `AR932S2`).
   - **Function Key Handling**:
     - `*INKA` (Bypass Entry): Clears fields (`CLEAR` subroutine), sets `*IN81` and `*IN11` (update mode), and skips to the end of the loop.
     - `*INKD` (Delete Record): Checks mode (`*IN10` for entry, `*IN11` for update) and deletion status (`*IN21`):
       - If `*IN21` is off, calls `DELETE` subroutine to mark a record for deletion.
       - If `*IN21` is on, calls `REACTI` subroutine to reactivate a deleted record.
     - `*INKG` (End of Screen): Clears screen indicators (`*IN81`, `*IN82`) and sets `*INLR` (last record) to exit the program.
     - `*INKJ` (Switch to Entry Mode): Clears fields, sets `*IN82` and `*IN10` (entry mode), and clears other indicators.
     - `*INKK` (Switch to Update Mode): Clears fields, sets `*IN81` and `*IN11` (update mode), and clears other indicators.
   - **Screen Processing**:
     - If `*IN01` is on, executes `S1` subroutine (processes `AR932S1` screen).
     - If `*IN02` is on, executes `S2` subroutine (processes `AR932S2` screen).
   - Writes to screen formats `AR932S1` or `AR932S2` based on indicators `*IN81` or `*IN82`.

4. **Subroutine Execution**:
   - **S1 Subroutine (Screen 1 Processing)**:
     - Chains to `ARCONT` using company number (`CO`) to validate the company.
     - If not found (`*IN92` on), displays error message 6 ("RECORD NOT FOUND").
     - Chains to `ARCUST` using `CO` and `CUST` (customer number) to validate the customer.
     - If not found, displays error message 11 ("INVALID CUSTOMER NUMBER ENTERED").
     - Chains to `ARFUEL` to check if a fuel record exists.
     - If not found, displays messages 6 and 7 ("RECORD NOT FOUND", "PRESS F10 TO ADD").
     - If the fuel record is marked deleted (`AFDEL = 'D'`), sets `*IN21` and displays messages 1 and 10 ("THIS RECORD PREVIOUSLY DELETED", "PRESS F4 TO REACTIVATE").
     - If valid, moves data to screen fields (`MOVE` subroutine) and sets update mode (`*IN11`, `*IN62`, `*IN82`).
   - **S2 Subroutine (Screen 2 Processing)**:
     - Validates company and customer numbers similar to `S1`.
     - In entry mode (`*IN10`):
       - Checks if `CO` is zero (displays message 4: "COMPANY CAN'T BE ZERO").
       - Checks if a fuel record already exists (displays messages 8 and 9: "CANNOT ADD - THIS RECORD EXISTS", "PRES F11 TO UPDATE").
     - Sets appropriate mode indicators (`*IN82` for entry, `*IN81` for update) and writes to the output file if needed.
   - **CLEAR Subroutine**:
     - Resets `CUST` and `NAME` fields to zeros or blanks.
   - **MOVE Subroutine**:
     - Moves `AFCO` (company number) and `AFCUST` (customer number) from `ARFUEL` to screen fields `CO` and `CUST`.
   - **ROLLSR Subroutine**:
     - Clears function key indicators (`*INKA`, `*INKD`, `*INKG`, `*INKJ`, `*INKK`).
     - Checks workstation status for roll-up (`01122`, `*IN54`) or roll-down (`01123`, `*IN55`) events, triggering `ROLLFW` or `ROLLBW` subroutines.
   - **ROLLFW Subroutine (Roll Forward)**:
     - Chains to `ARFUEL` using `CO` and `CUST`, then reads the next record.
     - Updates screen fields with new `AFCO` and `AFCUST` values or displays message 2 ("END OF FILE HAS BEEN REACHED") if no more records.
   - **ROLLBW Subroutine (Roll Backward)**:
     - Chains to `ARFUEL`, then reads the previous record (`READP`).
     - Updates screen fields or displays message 3 ("BEGINNING OF FILE REACHED") if at the start.
   - **DELETE Subroutine**:
     - Sets deletion message 5 ("PREVIOUS RECORD DELETED") and indicators (`*IN88`, `*IN90`, `*IN81`, `*IN72`).
     - Writes to the output file and clears screen indicators.
   - **REACTI Subroutine**:
     - Sets reactivation message 12 ("PREVIOUS RECORD WAS REACTIVATED") and indicators (`*IN88`, `*IN90`, `*IN81`, `*IN73`).
     - Writes to the output file and clears screen indicators.

5. **Output and Reporting**:
   - Writes to the `PRINT` file for a customer maintenance listing, including:
     - Company name (`ACNAME`), page number, date, time, and headers ("FUELS CUSTOMER MAINTENANCE LISTING").
     - Details for each record (`AFCO`, `AFCUST`, `NAME`, and action: "NO ACTION TAKEN", "RECORD ADDED", "RECORD DELETED", "RECORD REACTIVATED").
   - Uses indicators `*IN15` (report header) and `*INOF` (overflow) to control report formatting.

6. **Program Termination**:
   - If neither `*IN81` nor `*IN82` is on, sets `*INLR` (last record) to exit the program.
   - The program loops back to handle additional user inputs until `*INKG` or another exit condition is met.

---

### Business Rules

The program enforces the following business rules for A/R fuels customer maintenance:

1. **Company and Customer Validation**:
   - A valid company number (`CO`) must exist in `ARCONT` before processing.
   - A valid customer number (`CUST`) must exist in `ARCUST` for updates or deletions.
   - Company number cannot be zero in entry mode (message 4: "COMPANY CAN'T BE ZERO").

2. **Record Existence**:
   - In entry mode (`*IN10`), a new fuel record cannot be added if it already exists in `ARFUEL` (messages 8 and 9).
   - In update mode (`*IN11`), the record must exist in `ARFUEL` for updates or deletions (message 6: "RECORD NOT FOUND").

3. **Deletion and Reactivation**:
   - Records marked as deleted (`AFDEL = 'D'`) can be reactivated using F4 (`*INKD` with `*IN21` on, message 10: "PRESS F4 TO REACTIVATE").
   - Deletion is only allowed if the record exists and is not already deleted (`*IN21` off, message 5: "PREVIOUS RECORD DELETED").
   - Reactivation updates the record to remove the deletion flag (message 12: "PREVIOUS RECORD WAS REACTIVATED").

4. **Screen Modes**:
   - **Entry Mode (`*IN10`)**: Allows adding new records (F10 key, `*INKJ`).
   - **Update Mode (`*IN11`)**: Allows updating or deleting existing records (F11 key, `*INKK`).
   - **Bypass Entry (`*INKA`)**: Skips data entry and returns to update mode.
   - **End of Screen (`*INKG`)**: Exits the program.

5. **Navigation**:
   - Roll-up (`*IN54`) and roll-down (`*IN55`) allow browsing through `ARFUEL` records, with messages for end-of-file (message 2) or beginning-of-file (message 3).

6. **Error Handling**:
   - Displays appropriate error messages for invalid inputs (e.g., message 11: "INVALID CUSTOMER NUMBER ENTERED").
   - Prevents actions on deleted records unless reactivating.

7. **Reporting**:
   - Generates a printed report listing actions taken (add, delete, reactivate, or no action) for each customer record, including company and customer numbers, names, and timestamps.

---

### Tables (Files) Used

The program uses the following files, as defined in the `F` specifications:

1. **AR932D**:
   - Type: Workstation file (CF, combined file).
   - Purpose: Handles interactive screen input/output for formats `AR932S1` and `AR932S2`.
   - Handler: `PROFOUNDUI(HANDLER)` for modern UI integration.
   - Information Data Structure: `INFDS` captures screen status (e.g., function key codes).

2. **ARCONT**:
   - Type: Input file (IF, fixed-length 256 bytes, key length 2).
   - Purpose: Stores control data (e.g., company details).
   - Fields: `ACDEL` (delete flag), `ACCO` (company number), `ACNAME` (company name).
   - Access: Keyed access starting at position 2.

3. **ARFUEL**:
   - Type: Update file (UF, fixed-length 9 bytes, key length 8).
   - Purpose: Stores fuel-specific A/R data (e.g., customer fuel transactions).
   - Fields: `AFDEL` (delete flag), `AFCO` (company number), `AFCUST` (customer number).
   - Access: Keyed access starting at position 2, supports add (`EADD`) and delete (`EDEL`) operations.

4. **ARCUST**:
   - Type: Input file (IF, fixed-length 384 bytes, key length 8).
   - Purpose: Stores customer master data.
   - Fields: `ARDEL` (delete flag), `ARCO` (company number), `ARNAME` (customer name).
   - Access: Keyed access starting at position 2.

5. **GSCONT**:
   - Type: Input file (IF, fixed-length 512 bytes, key length 2).
   - Purpose: Likely a general system control file, possibly for cross-module data.
   - Fields: `GXDEL` (delete flag), `GXCONO` (company number).
   - Access: Keyed access starting at position 2.

6. **PRINT**:
   - Type: Output file (O, fixed-length 160 bytes).
   - Purpose: Generates a printed report for customer maintenance actions.
   - Fields: Includes `ACNAME`, `AFCO`, `AFCUST`, `NAME`, `PAGE`, `DATE`, `TIME`, and action descriptions.
   - Overflow Indicator: `*INOF`.

---

### External Programs Called

The RPGLE program does **not** explicitly call any external programs via `CALL` operations. All processing is handled within the program using subroutines (`S1`, `S2`, `CLEAR`, `MOVE`, `ROLLSR`, `ROLLFW`, `ROLLBW`, `DELETE`, `REACTI`). However, the program is called from the OCL program (`AR932.ocl36.txt`), which also invokes `GSGENIEC` as part of the job setup. Thus, no additional external programs are called directly by `AR932`.

---

### Summary

- **Process Steps**:
  1. Initializes the program, files, and screen environment.
  2. Sets up initial screen mode based on `QSCTL` and reads screen input (`AR932S1` or `AR932S2`).
  3. Processes user inputs via function keys (`*INKA`, `*INKD`, `*INKG`, `*INKJ`, `*INKK`) to manage modes (entry/update), delete, or reactivate records.
  4. Executes subroutines for screen processing (`S1`, `S2`), navigation (`ROLLFW`, `ROLLBW`), and record actions (`DELETE`, `REACTI`).
  5. Generates a printed report of customer maintenance actions.
  6. Loops until an exit condition (`*INKG` or no active screen indicators) sets `*INLR` to terminate.

- **Business Rules**:
  - Validates company and customer numbers.
  - Prevents adding duplicate records or invalid actions (e.g., zero company number).
  - Manages deletion and reactivation with appropriate flags and messages.
  - Supports entry and update modes with distinct screen behaviors.
  - Provides navigation for browsing records and detailed error messaging.
  - Produces a formatted report of actions taken.

- **Tables (Files) Used**:
  1. `AR932D` (workstation, screen I/O).
  2. `ARCONT` (control data).
  3. `ARFUEL` (fuel A/R data).
  4. `ARCUST` (customer master data).
  5. `GSCONT` (system control data).
  6. `PRINT` (report output).

- **External Programs Called**:
  - None directly called by `AR932.rpgle`. The OCL program calls `AR932` and `GSGENIEC`.

If you have additional details (e.g., the content of `AR932S1`/`AR932S2` screen formats, the purpose of `GSCONT`, or specific business requirements), I can provide a more detailed analysis. Let me know if you need further clarification or specific aspects explored!