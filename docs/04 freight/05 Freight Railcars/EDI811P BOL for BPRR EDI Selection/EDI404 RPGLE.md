The provided document, `EDI404.rpgle.txt`, is a short RPGLE (Report Program Generator Language Enhanced) program for IBM midrange systems (e.g., AS/400 or IBM i). It is called by the `EDI404SD.ocl36.txt` OCL program as part of the EDI (Electronic Data Interchange) outbound Bill of Lading (BOL) tracking maintenance workflow. The `EDI404` program appears to be a simple interactive program that displays a screen to the user, handles user input, and sets a cancellation flag if certain conditions are met. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called, integrating context from the related `EDI811P`, `EDI811`, and `EDI404SD` programs.

---

### Process Steps of the EDI404 RPGLE Program

The `EDI404` RPGLE program is a straightforward interactive program that manages a workstation screen and processes user input to control program flow, particularly for cancellation. Here’s a detailed breakdown of the process steps:

1. **File and Data Structure Definitions**:
   - **File Definition**:
     - `EDI404SC` (Line 1): Defined as a workstation file (`CF`, commitment control not used, externally described) for interactive input/output. This file corresponds to a display file (likely a DDS-defined screen named `SCREEN1`) used to interact with the user.
   - **Data Structure**:
     - `UDS` (Unnamed Data Structure, Lines 2–3): Defines a data area with a field `CANCEL` (positions 100–105, 6 characters). This field is used to communicate status (e.g., cancellation) back to the calling OCL program.

2. **Display Screen and Process Input**:
   - `EXFMT SCREEN1` (Line 4): Executes the `EXFMT` (Execute Format) operation to write the `SCREEN1` format to the workstation (display) and read user input back. This displays the screen to the user and waits for a response (e.g., function key presses or data entry).
   - **Purpose**: Presents a user interface for validating or confirming EDI 404 data, likely allowing the user to proceed or cancel the process.

3. **Handle Function Key KE**:
   - `C KE MOVEL ' ' CANCEL` (Line 5): If function key `KE` is pressed (indicator not shown but implied), moves 6 blanks to the `CANCEL` field, indicating no cancellation (normal processing).
   - `C KE SETON LR` (Line 6): Sets the `LR` (Last Record) indicator, signaling the program to end after processing.
   - **Purpose**: `KE` likely represents a function key (e.g., Enter or a custom key) for normal completion, allowing the EDI process to continue.

4. **Handle Function Key KG**:
   - `C KG MOVEL 'CANCEL' CANCEL` (Line 7): If function key `KG` is pressed, moves the string `'CANCEL'` to the `CANCEL` field, indicating the user has canceled the operation.
   - `C KG SETON LR` (Line 8): Sets the `LR` indicator to end the program.
   - **Purpose**: `KG` likely represents a cancellation key (e.g., F12 or a custom key), triggering program termination and signaling cancellation to the calling OCL program.

5. **Program Termination**:
   - When the `LR` indicator is set (via `KE` or `KG`), the program ends, closing the workstation file and updating the `CANCEL` field in the data area.
   - The `CANCEL` field (positions 100–105) is checked by the calling OCL program (`EDI404SD.ocl36`) to determine whether to clear the `EDI404` file and terminate the workflow.

---

### Business Rules

The `EDI404` RPGLE program enforces the following business rules:
1. **Interactive Validation**:
   - Displays a screen (`SCREEN1`) to the user, likely to confirm or validate EDI 404 data (e.g., BOL records in `EDIOUT` or `EDIRSI` from `EDI811`).
   - The user can proceed (`KE`) or cancel (`KG`) the operation.
2. **Cancellation Handling**:
   - If the user presses the cancellation key (`KG`), the `CANCEL` field is set to `'CANCEL'`, signaling to the OCL program (`EDI404SD`) to clear the `EDI404` file (likely `EDIOUT`) and terminate the workflow.
   - If the user proceeds (`KE`), the `CANCEL` field is set to blanks, indicating normal completion.
3. **Program Termination**:
   - The program ends after processing user input (`KE` or `KG`), setting the `LR` indicator to close files and exit cleanly.
4. **Integration with Workflow**:
   - The `CANCEL` field at offset 100, length 6, aligns with the OCL check (`IF ?L'100,6'?/CANCEL`) in `EDI404SD.ocl36`, ensuring the workflow responds appropriately to user actions.

---

### Tables (Files) Used

The RPGLE program uses the following file:
1. **EDI404SC**:
   - Type: Workstation file (`CF`, externally described).
   - Purpose: Handles interactive input/output via the `SCREEN1` format, likely a DDS-defined display file for user interaction (e.g., confirming or canceling EDI 404 processing).
   - Note: The program does not directly access the `EDIOUT` or `EDIRSI` files (used by `EDI811` and referenced in `EDI404SD.ocl36`). It likely relies on the OCL or prior programs to provide data context.

No disk files (e.g., `EDIOUT`, `EDIRSI`, `EDI404`) are directly referenced in the RPGLE code, but the program’s purpose suggests it validates data in these files indirectly through the screen interface.

---

### External Programs Called

The `EDI404` RPGLE program does not explicitly call any external programs via RPGLE operations (e.g., `CALL` or `PARM`). However:
- **Context from OCL**: The `EDI404SD.ocl36` program calls `EDI404` and subsequently `EDI404FTP`. The `EDI404` program’s role is to provide a user interface for validation or confirmation before the FTP transmission handled by `EDI404FTP`.
- **Display File**: The program uses the `EDI404SC` display file (with format `SCREEN1`), which is an externally described DDS file defining the screen layout.

---

### Integration with EDI811P, EDI811, and EDI404SD Programs

- **EDI811P (OCL and RPG)**:
  - `EDI811P.rpg36` validates company numbers (`KYCO`) and BOL numbers, writing them to `EDIBOL`.
  - `EDI811P.ocl36` sets up files (`BICONT`, `EDIBOL`, `EDIBOLTX`) and calls `EDI811`.
- **EDI811 (OCL and RPG)**:
  - `EDI811.rpg36` reads `EDIBOL` and `EDIBOLTX`, generating EDI 404 records in `EDIOUT` and routing/shipment data in `EDIRSI`.
  - `EDI811.ocl36` defines these files and calls `EDI404SD` after `EDI811`.
- **EDI404SD (OCL)**:
  - Checks if `EDIRSI` has records, calls `EDI404` for validation, calls `EDI404FTP` for transmission, archives `EDIRSI` to `EDIRSIH`, and clears `EDIRSI`.
  - Uses the `CANCEL` field (offset 100, length 6) set by `EDI404` to determine whether to clear the `EDI404` file (likely `EDIOUT`) and end the workflow.
- **EDI404 (RPGLE)**:
  - Provides a user interface (`SCREEN1`) to confirm or cancel the EDI 404 data processing.
  - Sets the `CANCEL` field to blanks (proceed) or `'CANCEL'` (cancel), influencing the OCL’s next steps.
- **Workflow Continuity**:
  - `EDI811P` prepares validated BOL data (`EDIBOL`).
  - `EDI811` formats it into EDI 404 records (`EDIOUT`, `EDIRSI`).
  - `EDI404` allows user validation via a screen.
  - `EDI404SD` handles transmission (`EDI404FTP`) and archiving, completing the outbound BOL process.

---

### Summary

- **Process Overview**: The `EDI404` RPGLE program displays a screen (`SCREEN1`) to the user for validating or confirming EDI 404 data. It processes user input via function keys (`KE` for proceed, `KG` for cancel), sets the `CANCEL` field in a data area (blanks or `'CANCEL'`), and ends by setting the `LR` indicator. The `CANCEL` field informs the calling OCL program (`EDI404SD`) whether to proceed with transmission or clear files and terminate.
- **Business Rules**: Provides an interactive interface for user validation, sets a cancellation flag if the user cancels, and ensures proper program termination. The `CANCEL` field drives the workflow’s next steps.
- **Tables/Files**: `EDI404SC` (workstation file, display format `SCREEN1`).
- **External Programs**: None called directly in RPGLE; interacts with the `EDI404SC` display file and is part of a workflow with `EDI404FTP` (called by `EDI404SD.ocl36`).

If you need further details (e.g., the `EDI404SC` DDS file or `EDI404FTP` code), please provide additional files or context.