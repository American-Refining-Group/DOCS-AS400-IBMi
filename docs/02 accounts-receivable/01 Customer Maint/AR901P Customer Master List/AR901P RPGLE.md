The provided document, `AR901P.rpgle.txt`, is an RPGLE (RPG IV) program for IBM System/36 or AS/400 (IBM i) that generates a "Customer Master Listing." It is called from the previously analyzed OCL program (`AR901P.ocl36.txt`). Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The RPGLE program `AR901P` is designed to prompt for user input, validate parameters, and prepare data for generating a customer master listing. It interacts with a workstation display file (`ar901pd`) and processes data from customer master files (`arcont` and `gscont`). Here’s a detailed breakdown of the process steps:

1. **Program Initialization**:
   - **Header Specifications**:
     - `H DFTACTGRP(*NO)`: Runs in a named activation group, ensuring modern RPG IV behavior.
     - `H FIXNBR(*ZONED:*INPUTPACKED)`: Ensures numeric fields are handled as zoned or packed decimal during input.
     - `H DFTNAME(AR901P)`: Sets the default program name to `AR901P`.
   - **File Declarations**:
     - `far901pd cf e workstn Handler('PROFOUNDUI(HANDLER)')`: Defines a workstation file for interactive display, using a Profound UI handler for modern UI support.
     - `farcont if f 256 2aidisk keyloc(2)`: Defines the customer master file `ARCONT` as input, with a record length of 256 bytes, indexed (keyed) on field position 2 (`acco`, company number).
     - `fgscont if f 512 2aidisk keyloc(2)`: Defines another file `GSCONT`, likely a global or system control file, also keyed on position 2 (`gxcono`, company number).
   - **Data Structures and Variables**:
     - `dco`: Array of 35-character fields to store company data (up to 3 companies).
     - `msg`: Array of 25-character messages for user feedback (e.g., "ENTER ALL OR CO", "INVALID COMPANY #").
     - `uds`: User data structure defining input fields like `kyalco` (all/company selection), `kyco1-kyco3` (company numbers), `kyalcs` (all/select customer selection), `kycs01-kycs10` (customer numbers), `addlst` (address list flag), `dollst` (dollar list flag), `kyjobq` (job queue flag), and `kycopy` (number of copies).

2. **Workstation File Processing**:
   - **Initial Screen Check**:
     - If `qsctl` (control field) is blank, set indicator `*in01` to '1' and `qsctl` to 'R' to display the input screen (`ar901pfm`).
     - Otherwise, read the workstation file (`ar901pfm`). If the last record is read (`lr`), the program terminates (`return`).
   - **Cancel Key Check**:
     - If `*inkg` (cancel key, e.g., F3) is pressed, set `*inu1` and `*inlr` to `*on`, clear `*in01`, and jump to the `end` tag to exit the program.

3. **Main Processing Logic**:
   - **Conditional Subroutine Calls**:
     - If `*in10` is `*on` (indicating prior initialization), call the `edit` subroutine if `*in11` is `*off`.
     - If `*in10` is `*off` (first run), call the `onetim` subroutine if `*in11` is `*off`.
   - **Screen Output**:
     - If `*in11` is `*off`, write to the workstation file (`ar901pfm`) to display the input screen.
     - If `*in11` is `*on`, set `*inlr` to `*on` to end the program.

4. **Edit Subroutine (`edit`)**:
   - Validates user input parameters for generating the customer master listing.
   - **Steps**:
     - Clear error indicators (`*in31` to `*in38`) and message output (`msgout`).
     - **Validate `kyalco` (All/Company Selection)**:
       - If `kyalco` is neither 'ALL' nor 'CO ', set `*in31` and display message "ENTER ALL OR CO".
     - **Validate Company Numbers (`kyco1`, `kyco2`, `kyco3`)**:
       - If `kyalco` is 'CO ' and all company numbers are zero, set `*in31` and display "ENTER COMPANY #".
       - For each non-zero `kyco1`, `kyco2`, `kyco3`:
         - Position the file pointer (`setll`) on `arcont` using the company number.
         - Read the record. If no record exists (`*in20` is `*on`) or the record is deleted (`acdel = 'D'`) or the company number doesn’t match, set `*in32`, `*in33`, or `*in34` and display "INVALID COMPANY #".
     - **Validate `kyalcs` (All/Select Customer Selection)**:
       - If `kyalcs` is neither 'ALL' nor 'SEL', set `*in35` and display "ENTER ALL OR SEL".
       - If `kyalcs` is 'SEL' and all customer numbers (`kycs01` to `kycs10`) are zero, set `*in39` and display "ENTER CUSTOMER #".
     - **Validate `addlst` (Address List Flag)**:
       - If `addlst` is not 'Y', 'N', or blank, set `*in36` and display "ENTER Y, N OR BLANK".
     - **Validate `dollst` (Dollar List Flag)**:
       - If `dollst` is not 'Y', 'N', or blank, set `*in37` and display "ENTER Y, N OR BLANK".
     - **Validate `kyjobq` (Job Queue Flag)**:
       - If `kyjobq` is not 'Y', 'N', or blank, set `*in38` and display "ENTER Y, N OR BLANK".
     - **Set Default for `kycopy`**:
       - If `kycopy` (number of copies) is zero, set it to 1.
     - **Set Default for `addlst` and `dollst`**:
       - If both `addlst` and `dollst` are not 'Y', set `addlst` to 'Y'.
     - **End of Validation**:
       - Set `*in11` to `*on` to indicate validation is complete and return to the main loop.

5. **One-Time Subroutine (`onetim`)**:
   - Performs initial setup for the program.
   - **Steps**:
     - Clear the `dco` array and initialize variables (`x = 1`, `arlim = 00`).
     - Position the file pointer (`setll`) on `arcont` using `arlim` (likely to read from the start).
     - Read `arcont` records in a loop (`agnco`):
       - Skip deleted records (`acdel = 'D'`).
       - Store company number (`acco`) and name (`acname`) in the `dco` array.
       - Increment `x` until 3 records are processed or end of file (`*in20` is `*on`).
     - Move `dco(1)`, `dco(2)`, and `dco(3)` to output fields `dco1`, `dco2`, and `dco3` for display.
     - **Check `gscont` File**:
       - Chain (lookup) to `gscont` using key '00'.
       - If found (`*in99` is `*off`) and `gxcono` is non-zero, set `kyalco` to 'CO ' and `kyco1` to `gxcono`.
       - Otherwise, set `kyalco` to 'ALL'.
     - **Set Defaults**:
       - Set `kyalcs` to 'ALL', `addlst` to 'Y', `dollst` to blank, `kyjobq` to 'N', and `kycopy` to 1.
     - Set `*in10` to `*on` to indicate initialization is complete and clear `*in20`.

6. **Output Specifications**:
   - Write to `ar901pfm` (workstation file) if `*in11` is `*off`, outputting fields like `kyalco`, `kyco1-kyco3`, `dco`, `kyalcs`, `kycs01-kycs10`, `addlst`, `dollst`, `kyjobq`, `kycopy`, and `msgout`.

---

### Business Rules

The program enforces the following business rules for generating the customer master listing:

1. **Company Selection (`kyalco`)**:
   - Must be 'ALL' (all companies) or 'CO ' (specific companies).
   - If 'CO ', at least one valid company number (`kyco1`, `kyco2`, or `kyco3`) must be provided and must exist in `arcont` without being deleted (`acdel ≠ 'D'`).

2. **Customer Selection (`kyalcs`)**:
   - Must be 'ALL' (all customers) or 'SEL' (selected customers).
   - If 'SEL', at least one customer number (`kycs01` to `kycs10`) must be non-zero.

3. **Address and Dollar List Flags (`addlst`, `dollst`)**:
   - Must be 'Y' (yes), 'N' (no), or blank.
   - If both are not 'Y', `addlst` defaults to 'Y' to ensure at least one report type is generated.

4. **Job Queue Flag (`kyjobq`)**:
   - Must be 'Y' (queue the job), 'N' (run interactively), or blank.

5. **Number of Copies (`kycopy`)**:
   - Defaults to 1 if zero is entered.

6. **Data Validation**:
   - Company numbers must exist in `arcont` and not be marked as deleted.
   - Invalid inputs result in error messages displayed to the user, prompting correction.

7. **Initialization**:
   - The `onetim` subroutine populates default values and retrieves up to three company records from `arcont` for display.
   - The `gscont` file provides a default company number if available; otherwise, 'ALL' is used.

---

### Tables (Files) Used

1. **AR901PD**:
   - Type: Workstation file (display file).
   - Purpose: Handles interactive user input/output via the `ar901pfm` format.
   - Handler: Uses Profound UI (`PROFOUNDUI(HANDLER)`) for modern UI rendering.

2. **ARCONT**:
   - Type: Input file (disk, indexed).
   - Record Length: 256 bytes.
   - Key: `acco` (company number, position 2).
   - Fields:
     - `acdel` (position 1, 1 byte): Deletion flag ('D' for deleted).
     - `acco` (position 2-3, 2 bytes, numeric): Company number.
     - `acname` (position 4-33, 30 bytes): Company name.
   - Purpose: Stores customer master data (company details).

3. **GSCONT**:
   - Type: Input file (disk, indexed).
   - Record Length: 512 bytes.
   - Key: Position 2 (likely `gxcono`, company number).
   - Fields:
     - `gxdel` (position 1, 1 byte): Deletion flag.
     - `gxcono` (position 77-78, 2 bytes, numeric): Company number.
   - Purpose: Likely a system control file storing default or global company settings.

---

### External Programs Called

No external RPG programs are explicitly called (e.g., via `CALL` operation) within `AR901P.rpgle`. The program interacts with the OCL program (`AR901P.ocl36.txt`) that loads and runs it, and it uses the Profound UI handler (`PROFOUNDUI(HANDLER)`) for workstation file processing, but this is a system-level interface, not a program call.

---

### Additional Notes

- **Conversion Notes**:
  - The program was converted on 04/08/23 using TARGET/400, with 161 lines added (`T4A`), 5 modified (`T4M`), and original lines (`T4O`) retained from an older RPG source.
  - Conversion likely modernized the program from RPG II/III to RPGLE, adding features like `EVAL` and modern file handling.

- **Indicators**:
  - Uses indicators (`*in01`, `*in10`, `*in11`, `*in20`, `*in31` to `*in39`, etc.) for flow control and error handling, typical in legacy RPG.
  - Modernized with `EVAL` for setting indicators (e.g., `eval *in31 = *on` instead of `SETON 31`).

- **OCL Integration**:
  - The OCL program (`AR901P.ocl36.txt`) sets up the environment, loads `AR901P`, and passes parameters via memory locations (e.g., `kyjobq` at position 120).
  - The RPGLE program uses these parameters (e.g., `kyjobq`, `kycopy`) to control job execution (interactive or queued).

- **No Chart Generation**:
  - The query does not request a chart, and no numerical data is provided for visualization, so no chart is generated.

If you need further details (e.g., specific field mappings, sample output format, or integration with other programs), please provide additional context or files.