The provided document is an RPGLE program named `AR050P.rpgle.txt`, which is part of a cash receipts entry prompt system for an IBM System/36 or AS/400 environment. This program, converted from RPG II to RPGLE (as indicated by the TARGET/400 conversion comments), is called by the `AR050P.ocl36` OCL script previously discussed. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called.

### Process Steps of the AR050P RPGLE Program

The `AR050P` program is designed to handle the entry and validation of cash receipts data, including General Ledger (GL) account assignments for debit, credit, and discount accounts. It interacts with a workstation display file and several disk files to manage user input and validate GL accounts. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - **Header Specifications**: The program uses `DFTACTGRP(*NO)` to run in a named activation group and `fixnbr(*zoned:*inputpacked)` to handle numeric data formats. The default program name is set to `AR050P`.
   - **Initialization Subroutine (`*inzsr`)**:
     - Initializes `glco` (GL company number) and `glacct` (GL account number) to zero.

2. **Workstation File Processing**:
   - The program reads the workstation file `ar050pd` (display file) to handle user input.
   - **Logic for `qsctl` (control field)**:
     - If `qsctl` is blank:
       - Sets indicator `*in09` to `1` (indicating first-time processing).
       - Sets indicator `*in01` to `0`.
       - Sets `qsctl` to `R` (read mode).
     - Otherwise:
       - Sets `*in09` to `0`.
       - Sets `*in01` to `1`.
       - Reads the `ar050pfm` format from the display file.
       - If the last record is read (`lr` indicator), the program terminates (`return`).
   - Clears indicator `*in21` (used for screen redisplay).

3. **Main Processing Loop (`kg` Do Loop)**:
   - Sets indicator `*inu1` (likely a control indicator for user interaction).
   - Attempts to chain (retrieve) a record from the `crgldf` file using key `'00000'`.
   - Writes or updates the `crgldf` file (`gldflt` exception output).
   - Sets `*inlr` (last record indicator) and exits to the `end` tag.
   - If the chain fails, proceeds to further processing.

4. **Conditional Subroutine Execution**:
   - If `*in01` is on (subsequent screen read), executes the `s1edt` subroutine (screen edit).
   - If `*in09` is on (first-time processing), executes the `onetim` subroutine (initial setup).

5. **Screen Edit Subroutine (`s1edt`)**:
   - **Input Validation**:
     - Clears `*in50` (error indicator) and `msgout` (error message field).
     - Validates the `select` field (user selection, expected to be 1 or 2):
       - If `select` is not 1 or 2, sets `*in21` and `*in50`, displays error message `msg(1)` ("SELECTION INVALID"), and jumps to `endedt`.
   - **Default Company Values**:
     - Copies `drco` (debit company) to `crco` (credit company) and `dsco` (discount company).
     - Chains to `arcont` using `drco` to retrieve default GL accounts:
       - If `drgl` (debit GL account) is zero, sets it to `accsgl` (cash GL account from `arcont`).
       - If `crgl` (credit GL account) is zero and `crco` equals `drco`, sets `crgl` to `acargl` (AR GL account from `arcont`).
       - If `dsgl` (discount GL account) is zero and `dsco` equals `drco`, sets `dsgl` to `acdsgl` (discount GL account from `arcont`).
   - Executes the `subrgl` subroutine to validate GL accounts.
   - If `*in50` is on (error during GL validation), jumps to `endedt`.
   - **Screen Redisplay Check**:
     - Compares saved GL accounts (`svdrgl`, `svcrgl`, `svdsgl`) with current values (`drgl`, `crgl`, `dsgl`).
     - If any differ (`*in99` off), updates `crgldf` with new values and jumps to `endedt`.
     - Updates saved values (`svdrgl`, `svcrgl`, `svdsgl`) with current GL accounts.
   - Sets `*in21` to trigger screen redisplay.

6. **One-Time Setup Subroutine (`onetim`)**:
   - Sets `select` to 1 and `z5` (a counter or temporary field) to zero.
   - Sets `*in21` for screen display.
   - Attempts to chain to `crgldf` with key `'00000'`:
     - If successful (`*in90` off), loads existing values from `crgldf` into `drco`, `drgl`, `crco`, `crgl`, `dsco`, `dsgl`, and their saved counterparts.
     - If unsuccessful, chains to `arcont` with key `'01'` to load default values:
       - Sets `drgl` to `accsgl` if zero.
       - Sets `crgl` to `acargl` if zero.
       - Sets `dsgl` to `acdsgl` if zero.
       - Sets `drco`, `crco`, `dsco` to `01` if zero.
   - Executes `subrgl` to validate GL accounts.

7. **GL Account Validation Subroutine (`subrgl`)**:
   - Validates debit, credit, and discount GL accounts against the `glmast` file:
     - For `drgl`:
       - Constructs `glkey` using `drco` and `drgl` with `gltype = 'C'`.
       - Chains to `glmast` using `glkey`.
       - Checks if the account is not found (`*in38` on) or marked as deleted (`gldel = 'D'`) or inactive (`gldel = 'I'`).
       - If valid, sets `drdesc` to `gldesc` (account description).
       - If invalid, sets `*in21` and `*in50`, displays `msg(2)` ("INVALID DR G/L ACCOUNT"), and jumps to `endgl`.
     - For `crgl`:
       - Similar process using `crco` and `crgl`, with error message `msg(3)` ("INVALID CR G/L ACCOUNT") and indicator `*in39`.
     - For `dsgl`:
       - Similar process using `dsco` and `dsgl`, with error message `msg(4)` ("INVALID DISCOUNT ACCT") and indicator `*in43`.
   - If all validations pass, the subroutine completes without errors.

8. **Output to Files**:
   - **Display File (`ar050pfm`)**:
     - If `*in21` is on, writes the `ar050pfm` format to the screen, displaying fields like `select`, `drco`, `drgl`, `drdesc`, `crco`, `crgl`, `crdesc`, `dsco`, `dsgl`, `dsdesc`, and `msgout` (error message).
   - **Disk File (`crgldf`)**:
     - If `*in90` is on (no existing record), adds a new record to `crgldf` with fields `z5`, `drco`, `drgl`, `crco`, `crgl`, `dsco`, `dsgl`.
     - If `*in90` is off (existing record), updates the record with the same fields.

9. **Program Termination**:
   - If `*in21` is off, sets `*inLR` to terminate the program.
   - The program loops back to read the next user input unless terminated.

### Business Rules

The program enforces the following business rules for cash receipts entry:

1. **Selection Validation**:
   - The `select` field must be either 1 or 2. Any other value triggers an error ("SELECTION INVALID") and redisplays the screen.

2. **Default GL Account Assignment**:
   - If `drgl`, `crgl`, or `dsgl` are zero, the program retrieves default values from the `arcont` file:
     - `drgl` defaults to `accsgl` (cash GL account).
     - `crgl` defaults to `acargl` (AR GL account) if `crco` matches `drco`.
     - `dsgl` defaults to `acdsgl` (discount GL account) if `dsco` matches `drco`.
   - Company numbers (`drco`, `crco`, `dsco`) default to `01` if zero during first-time processing.

3. **GL Account Validation**:
   - All GL accounts (`drgl`, `crgl`, `dsgl`) must exist in the `glmast` file and not be marked as deleted (`gldel = 'D'`) or inactive (`gldel = 'I'`).
   - If any account is invalid, an error message is displayed ("INVALID DR/CR/DISCOUNT G/L ACCOUNT"), and the screen is redisplayed for correction.

4. **First-Time vs. Subsequent Processing**:
   - On first-time execution (`*in09` on), the program loads default values from `arcont` or existing values from `crgldf`.
   - On subsequent executions (`*in01` on), it validates user-entered values and updates `crgldf` if GL accounts change.

5. **Screen Redisplay**:
   - The screen is redisplayed if:
     - An invalid selection or GL account is entered.
     - GL accounts change from their saved values (`svdrgl`, `svcrgl`, `svdsgl`).

6. **Data Persistence**:
   - Validated GL account data is saved to the `crgldf` file, either as a new record or an update.
   - Account descriptions (`drdesc`, `crdesc`, `dsdesc`) are retrieved from `glmast` for display.

7. **Inactive GL Accounts** (per JB01 comment, 08/24/2016):
   - Inactive GL accounts (`gldel = 'I'`) are treated the same as deleted accounts (`gldel = 'D'`), triggering an error and preventing their use.

### Tables (Files) Used

The program interacts with the following files:
1. **ar050pd**:
   - Type: Workstation (display) file.
   - Format: `ar050pfm`.
   - Usage: Input/output for user interaction, displaying fields like `select`, `drco`, `drgl`, `drdesc`, `crco`, `crgl`, `crdesc`, `dsco`, `dsgl`, `dsdesc`, and `msgout`.
   - Handler: Uses `PROFOUNDUI(HANDLER)` for modern UI integration.

2. **arcont**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 256 bytes.
   - Key: 2-byte company number (keyloc(2)).
   - Fields Used:
     - `acargl` (positions 38-45): Accounts Receivable GL account number.
     - `acdsgl` (positions 54-61): Discount GL account number.
     - `accsgl` (positions 62-69): Cash GL account number.
   - Usage: Provides default GL account numbers for cash receipts.

3. **glmast**:
   - Type: Input file (`if`), disk-based.
   - Record Length: 256 bytes.
   - Key: 11-byte key (company number + account number + type, keyloc(2)).
   - Fields Used:
     - `gldel` (position 1): Deletion/inactive flag (`'D'` for deleted, `'I'` for inactive).
     - `gldesc` (positions 13-37): GL account description.
   - Usage: Validates GL accounts and retrieves descriptions.

4. **crgldf**:
   - Type: Update file (`uf a`), disk-based, allows additions.
   - Record Length: 48 bytes.
   - Key: 5-byte key (keyloc(2)).
   - Fields Used:
     - `cgdrco` (positions 7-8): Debit company number.
     - `cgdrgl` (positions 9-16): Debit GL account number.
     - `cgcrco` (positions 17-18): Credit company number.
     - `cgcrgl` (positions 19-26): Credit GL account number.
     - `cgdsco` (positions 27-28): Discount company number.
     - `cgdsgl` (positions 29-36): Discount GL account number.
   - Usage: Stores validated GL account assignments for cash receipts.

### External Programs Called

The `AR050P` RPGLE program does not explicitly call any external programs. It is a standalone program that handles user input, validates data, and updates files within its own logic. It is invoked by the `AR050P.ocl36` script, which may call other programs (`AR050`, `AR100`, `AR105`, `AR110`, `GSGENIEC`, `SCPROCP`), but `AR050P` itself does not initiate calls to other programs.

### Summary

The `AR050P` RPGLE program is a cash receipts entry prompt that:
- Facilitates user input via a workstation display file (`ar050pd`).
- Validates user-entered or default GL accounts (debit, credit, discount) against the `glmast` file, ensuring they are not deleted or inactive.
- Retrieves default GL accounts from `arcont` for first-time processing or when fields are zero.
- Saves validated data to `crgldf` and updates the display with account descriptions or error messages.
- Enforces business rules such as valid selection values, GL account validation, and default company assignments.
- Handles first-time and subsequent processing differently, with screen redisplay for errors or changes.

The program is tightly integrated with the OCL script, which manages the broader cash receipts workflow, while `AR050P` focuses on user interaction and GL account validation.