### Process Steps of the RPG Program

This RPG program (BB170P) is designed for an IBM System/36 environment and serves as an interactive validation routine for creating new orders by copying an existing one. It uses a workstation display file (SCREEN) to prompt the user for inputs (company number, order number, and number of new order copies) and performs validations against disk files. The program is called and executed by the parent OCL procedure, which passes parameters and sets up the files. The flow is indicator-driven, with subroutines for one-time setup and main validation. If validations pass, it prepares data for further processing (e.g., by the subsequent BB170 OCL procedure); if not, it displays errors and may loop for corrections.

The program executes in a cycle typical of RPG: input, calculations, output. Here's a step-by-step breakdown:

1. **Initialization (Mainline Calculations)**:
   - Clear the error message field (MSG40) to blanks.
   - Set off indicators 50, 51, 52 (likely error highlights for individual fields: company, order, copies).
   - Set off indicators 81 (protect/control output), 44, 90 (general error/protect).
   - Clear KYTYPE (order type derivative field) to blanks.
   - If indicator KG is on (possibly set by OCL or prior logic for cancellation mode), move 'CANCEL' to KYCANC and set off indicators 01 and 09 (disabling main processing and one-time setup).

2. **One-Time Setup Subroutine (ONETIM)**:
   - Executed only if indicator 09 is on (likely a first-pass or initialization flag).
   - Sets on indicator 81 (enables protected output or initial screen display).
   - No other logic; subroutine ends.

3. **Main Validation Subroutine (S1)**:
   - Executed only if indicator 01 is on (likely the main processing flag, triggered after input).
   - **Validate Company Number (KYCO)**:
     - Chain (random read) to the BICONT file using KYCO as the key.
     - If record not found (indicator 10 on), set on indicators 81 (protect), 90 (error), 51 (company field error), move error message 1 ("INVALID COMPANY NUMBER ENTERED") to MSG40, and branch to ENDS1 (end subroutine).
   - **Validate Order Number (KYORDR)**:
     - If KYORDR is zeros, set on 81, 90, 52 (order field error), move message 2 ("INVALID ORDER NUMBER ENTERED") to MSG40, branch to ENDS1.
     - Build a composite key (KEY11): Move KYCO to first 2 positions, pad KYORDR to 9 digits with leading zeros and append '000', then chain to BBORDR file using KEY11.
     - If record not found (99 on), set on 81, 90, 52, move message 2 to MSG40, branch to ENDS1.
     - Check if the order is locked (BOLOCK not blanks, e.g., in a batch). If locked, set on 81, 90, 52, move message 4 ("ORDER IS CURRENTLY IN A BATCH PLEASE POST") to MSG40, branch to ENDS1.
   - **Validate Number of Copies (KYCPYS)**:
     - If KYCPYS is zeros, set on 81, 90, 53 (copies field error), move message 3 ("MUST ENTER NUMBER OF NEW ORDERS") to MSG40, branch to ENDS1.
   - **Derive Order Type (KYTYPE)**:
     - If BOTYPE (from BBORDR) is 'M' (possibly "master" or "main"), set KYTYPE to 'PM' (possibly "processed master" or a flag for copying).
   - End subroutine (ENDS1).

4. **Output to Screen**:
   - Output to SCREEN file only if indicator 81 is on (protected mode, e.g., for redisplay with errors).
   - Write format 'BB170PFM' (likely a display format name for the prompt screen).
   - Output fields: KYCO at positions 3-4 (input as 3-40, but output at 2? Wait, input spec is 3-40 for KYCO, but output at 2—possible typo or alignment).
   - KYORDR at 5-100 (output at 8? Similar alignment note).
   - KYCPYS at 11-120 (output at 10).
   - MSG40 (error message) at 50? (spec says 169 O MSG40 50, but line 0169—likely position 50 on screen).
   - The screen interaction allows user input, and the program likely loops (RPG cycle) until validations pass or user cancels (via KYCANC).

5. **Program End**:
   - If errors occur, the screen redisplays with fields protected/highlighted and error message shown, prompting corrections.
   - If all validations pass, the program ends successfully, returning control to the OCL, which checks for cancellation in local data area and proceeds to call BB170 if not canceled.

The program is interactive and error-tolerant, looping via the RPG cycle until inputs are valid or canceled.

### Business Rules

The program enforces rules for safely copying orders in a business/order management system:

- **Company Validation**: Company number (KYCO) must exist in the control file (BICONT). Ensures operations are tied to a valid company.
- **Order Validation**:
  - Order number (KYORDR) cannot be zero.
  - The order must exist in the order file (BBORDR) under the company.
  - The order cannot be locked (BOLOCK must be blanks), preventing copies of in-process/batched orders. User must post/complete the batch first.
- **Copies Validation**: Number of new orders to create (KYCPYS) must be greater than zero; enforces that at least one copy is requested.
- **Order Type Handling**: If the source order type is 'M', derive KYTYPE as 'PM'—possibly flags the copies as processed or special type.
- **Cancellation Handling**: If in cancel mode (KG on), set KYCANC to 'CANCEL', bypassing validations (sets off 01/09).
- **Error Handling**: All errors protect the screen (81 on), highlight specific fields (51-53), display a message, and prevent progression until fixed.
- **Data Integrity**: Uses chained reads for existence checks; composite keys ensure unique order identification (company + padded order number + '000' suffix, possibly for header records).
- **Security/Concurrency**: Files are opened in update mode (IF for input with update), but shared in OCL—allows reading locked status without altering unless needed.

These rules prevent invalid data entry, ensure referential integrity between company/control and order data, and avoid conflicts with ongoing processes.

### Tables Used

In RPG/System/36 context, "tables" refer to disk files used as databases:

- **BBORDR**: Order file (likely header records). Keyed (11AI—11-byte alphanumeric key, indexed), 512-byte records. Used for chaining to validate order existence and check lock status (BOLOCK). Fields include BODEL (delete flag?), BCOORD (company/order?), BSQ, BKEY, BOTYPE (order type), BOLOCK (lock code), BOWSID (lock workstation ID).
- **BICONT**: Control/company file. Keyed (2AI—2-byte alphanumeric key), 256-byte records. Used for chaining to validate company. Fields include BCDEL (delete flag), BCCO (company no), BCNAME (company name), BCORDN (next order number).

SCREEN is a workstation file (not a disk table) for interactive I/O.

### External Programs Called

- None directly called from this RPG program. It is self-contained, with all logic in subroutines. However:
  - It is invoked by the parent OCL procedure (via // LOAD and // RUN).
  - Outputs a format 'BB170PFM', which may reference a display file format, not a program.
  - Post-execution, the OCL calls BB170 (another OCL procedure) if no cancellation.