The provided document, `EDI811P.rpg36.txt`, is an RPG (Report Program Generator) program written for an IBM midrange system (e.g., AS/400 or IBM i). It serves as a subprogram called from the main OCL program (`EDI811P.ocl36.txt`) to handle the creation of an EDI (Electronic Data Interchange) Bill of Lading (BOL) file. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called, based on the RPG code and its integration with the OCL program.

---

### Process Steps of the EDI811P RPG Program

The RPG program `EDI811P` is designed to prompt users for input via a workstation screen and create records in the `EDIBOL` file based on validations against the `BICONT` and `EDIBOLTX` files. Here’s a detailed breakdown of the process steps:

1. **File and Screen Definitions** (Lines 0005–0007):
   - **SCREEN**: Defined as a workstation file (`CP`, 500 bytes) for interactive user input/output, likely a display file for the BOL selection screen.
   - **BICONT**: An input file (`IC`, 256 bytes, indexed with 2-byte keys) used to validate company numbers (`KYCO`). It’s a disk file with shared access.
   - **EDIBOLTX**: An input file (`IF`, 1340 bytes, indexed with 18-byte keys, externally described) used to validate BOL numbers.
   - **EDIBOL**: An output file (`O`, 12 bytes, indexed with 12-byte keys) where the program writes validated BOL records.

2. **Data Structure Definitions** (Lines 0008–0013, 0065–0066, 0106–0112):
   - **COM**: An array (5 elements, 40 characters each) for error messages (e.g., "INVALID COMPANY NUMBER ENTERED").
   - **BOL**: An array (10 elements, 10 digits each) to store up to 10 BOL numbers entered by the user.
   - **SCREEN Input (NS 01)**: Reads `KYCO` (company number, positions 3–4) and `BOL` array (positions 5–104) from the screen.
   - **EDIBOLTX Input (NS)**: Defines fields for BOL validation, including:
     - `KEY02` (positions 1–5): Likely a key field.
     - `BOL#` (positions 6–12): BOL number.
     - `ZERO3` (positions 13–15): Zeros (padding or unused).
     - `GS03` (positions 18–32): Bill-to purchase order.
     - `GS02` (positions 33–47): PO sequence number.
     - `SYSDT8` (positions 48–55): Record type (e.g., "02").
     - `SYSHM`, `GS06`, `GS07`, `GS08`: Routing and other EDI fields.
     - `SRN#` (positions 638–640): Serial number.

3. **Initialization** (Lines 0038–0040):
   - `SETOF 303132`, `SETOF 3334`, `SETOF 8190`: Clears indicators (30–34, 81, 90) used for controlling program flow and error handling.
   - `MOVEL*BLANKS MSG 40`: Clears the `MSG` field (40 characters) used for displaying error messages.

4. **Conditional Execution Based on Indicator 09** (Lines 0043–0046):
   - If indicator 09 is on (`C 09 DO`):
     - Executes the `ONETIM` subroutine (initializes `KYCO` to 10 and `BOL` array to zeros, sets indicator 81).
     - Jumps to `END` tag, terminating the program.
   - **Purpose**: Indicator 09 likely indicates a special condition (e.g., initialization or cancellation) that bypasses normal processing.

5. **Error Handling for Invalid Input (KG)** (Lines 0048–0052):
   - If indicator `KG` is on (`C KG DO`):
     - Clears indicators 01, 09, 81.
     - Sets indicators `U8` (user exit) and `LR` (last record, signaling program end).
     - Jumps to `END` tag.
   - **Purpose**: Handles invalid user input or a cancellation request, terminating the program.

6. **Main Screen Processing (S1 Subroutine)** (Lines 0054–0055, 0059–0115):
   - If indicator 01 is on (`C 01 EXSR S1`):
     - Executes the `S1` subroutine, which handles the core logic for validating user input and creating BOL records.
     - Jumps to `END` tag after execution.
   - **S1 Subroutine Steps** (Lines 0059–0115):
     - **Validate Company Number (KYCO)**:
       - `KYCO CHAINBICONT 30`: Chains (looks up) the `KYCO` value in the `BICONT` file to verify the company number.
       - If not found (`N30`) or if the record is marked as deleted (`BCDEL = 'D'`), sets indicator 30, moves error message `COM,1` ("INVALID COMPANY NUMBER ENTERED") to `MSG`, sets indicators 81 and 90, and jumps to `ENDS1`.
     - **Validate BOL Numbers**:
       - Loops through the `BOL` array (up to 10 entries, indexed by `Y`):
         - If `BOL,Y` is zero, increments `Y` and continues until `Y > 10`, then jumps to `ENDS1`.
         - Constructs a key (`KEYORD`) by combining:
           - Hardcoded prefix `"40402"`.
           - `BOL,Y` (BOL number, padded to 7 digits in `KEY7`).
           - Suffix `"000"` (in `KEY10`).
         - Chains `KEYORD` against `EDIBOLTX` to validate the BOL number.
         - If not found (indicator 99 on), sets error message `COM,3` ("INVALID ORDER NUMBER ENTERED AT LINE XX") with `Y` appended, sets indicators 81 and 90, and jumps to `ENDS1`.
         - If found (`N99`), increments `Y` and continues the loop.
     - **Purpose**: Ensures the company number exists and is not deleted, and validates each non-zero BOL number against `EDIBOLTX`.

7. **Write BOL Records** (Lines starting with `CLR`):
   - Loops through the `BOL` array (indexed by `X`, up to 10):
     - If `BOL,X` is non-zero, writes a record to `EDIBOL` using the `ADDREC` exception output (`EXCPTADDREC`), including `KYCO` and `BOL,X`.
     - Increments `X` and continues until `X > 10`, then jumps to `DONE`.
   - **Purpose**: Writes validated BOL records to the `EDIBOL` output file.

8. **Screen Output** (Lines 0132–0140):
   - If indicator 81 is on, displays the screen (`OSCREEN D 81`):
     - Outputs the display file `EDI811FM` (via `K8` keyword).
     - Displays `KYCO` (company number), `BOL` array (BOL numbers), and `MSG` (error message, if any).
   - **Purpose**: Updates the user interface with input fields and any error messages.

9. **Program Termination** (Line 0057):
   - `END TAG`: Marks the end of the program, reached after processing, cancellation, or errors.

---

### Business Rules

The program enforces the following business rules:
1. **Company Number Validation**:
   - The company number (`KYCO`) must exist in the `BICONT` file and must not be marked as deleted (`BCDEL ≠ 'D'`).
   - If invalid or deleted, the program displays "INVALID COMPANY NUMBER ENTERED" and halts further processing.
2. **BOL Number Validation**:
   - Up to 10 BOL numbers can be entered via the screen.
   - Each non-zero BOL number is validated against the `EDIBOLTX` file using a constructed key (`40402` + BOL + `000`).
   - If a BOL number is invalid, the program displays "INVALID ORDER NUMBER ENTERED AT LINE XX" (where XX is the line number) and halts further processing.
3. **Output File Creation**:
   - Only valid, non-zero BOL numbers are written to the `EDIBOL` file, along with the company number (`KYCO`).
   - The output records are 12 bytes, containing `KYCO` (2 bytes) and `BOL` (10 bytes).
4. **Error Handling**:
   - Errors (invalid company number or BOL) set indicators 81 and 90, display an error message, and stop processing.
   - Cancellation (via indicator `KG`) sets `U8` and `LR`, terminating the program.
5. **Initialization**:
   - The `ONETIM` subroutine initializes `KYCO` to 10 and clears the `BOL` array, used when indicator 09 is on (likely a special case or reset condition).

---

### Tables (Files) Used

The RPG program uses the following files:
1. **SCREEN**:
   - Type: Workstation file (`CP`, 500 bytes).
   - Purpose: Handles user input/output for the BOL selection screen (display file `EDI811FM`).
2. **BICONT**:
   - Type: Input file (`IC`, 256 bytes, 2-byte key).
   - Purpose: Validates the company number (`KYCO`). Contains a deletion flag (`BCDEL`).
3. **EDIBOLTX**:
   - Type: Input file (`IF`, 1340 bytes, 18-byte key, externally described).
   - Purpose: Validates BOL numbers using a constructed key. Contains fields like `BOL#`, `GS03` (purchase order), and `SRN#` (serial number).
4. **EDIBOL**:
   - Type: Output file (`O`, 12 bytes, 12-byte key).
   - Purpose: Stores validated BOL records with `KYCO` and `BOL`.

---

### External Programs Called

The RPG program does not explicitly call other programs via RPG operations (e.g., `CALL` or `PARM`). However:
- It interacts with the display file `EDI811FM` (line 0133, `K8 'EDI811FM'`), which is an external DDS (Data Description Specifications) file defining the screen layout.
- The OCL program (`EDI811P.ocl36.txt`) that calls this RPG program also references a subsequent program/procedure `EDI811`. This suggests `EDI811P` is part of a larger workflow, but within the RPG code itself, no external RPG programs are directly called.

---

### Integration with OCL Program

The OCL program (`EDI811P.ocl36.txt`) provides the setup for `EDI811P`:
- **File Setup**: The OCL defines `BICONT`, `EDIBOL`, and `EDIBOLTX` with specific labels and attributes, which the RPG program uses.
- **Cancellation Check**: The OCL checks switch 8 (`SWITCH8-1`) to detect cancellation, which may correspond to the RPG’s `U8` indicator.
- **Subsequent Processing**: After `EDI811P` completes, the OCL calls `EDI811`, likely to process the `EDIBOL` file further (e.g., for EDI transmission).

---

### Summary

- **Process Overview**: The `EDI811P` RPG program prompts users for a company number (`KYCO`) and up to 10 BOL numbers via a workstation screen. It validates `KYCO` against `BICONT` and each BOL against `EDIBOLTX`. Valid BOLs are written to `EDIBOL`. Errors trigger messages and halt processing. The program supports cancellation and initialization modes.
- **Business Rules**: Validates company and BOL numbers, writes only valid data to `EDIBOL`, and handles errors with specific messages.
- **Tables/Files**: `SCREEN` (workstation), `BICONT` (input), `EDIBOLTX` (input), `EDIBOL` (output).
- **External Programs**: None called directly in RPG, but interacts with display file `EDI811FM`. The OCL calls `EDI811` afterward.

If you need further analysis (e.g., details of `EDI811FM` or `EDI811`), please provide additional files or context.