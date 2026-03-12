The provided document is an RPGLE program (`AR135P.rpgle.txt`), which is part of the Accounts Receivable (AR) Electronic Funds Transfer (EFT) draft automation process. It is called from the main OCL program (`AR135P.ocl36.txt`) and serves as an interactive program for entering and validating initial EFT parameters, such as company number and bank upload date. Below, I will explain the process steps, business rules, tables/files used, and external programs called.

---

### Process Steps of the AR135P RPG Program

The `AR135P` RPGLE program is a converted System/36 RPG program (via TARGET/400, dated 08/30/23) that uses a workstation file for user interaction and validates input against database files. It is designed to capture and validate key parameters before proceeding with EFT processing. Here’s a step-by-step breakdown of the program’s logic:

1. **Program Initialization**:
   - **Header Specifications**:
     - `DFTACTGRP(*NO)`: Runs in a named activation group, typical for ILE RPG.
     - `fixnbr(*zoned:*inputpacked)`: Ensures zoned and packed decimal fields are handled correctly.
     - `dftname(ar135p)`: Sets the default program name.
   - **File Definitions**:
     - `ARCONT`: Input file, 256 bytes, keyed on position 2 (company number).
     - `GSCONT`: Input file, 512 bytes, keyed on position 2 (company number).
     - `AR135PD`: Workstation file (CF), used for interactive display with Profound UI handler.
   - **Data Definitions**:
     - `MSG`: Array of 3 error messages (40 bytes each).
     - `FILENM`: Data structure for file name, used in external call.
     - `UDS`: User Data Structure with fields `KYCO` (company), `KYSLDT` (select date), `STATUS`, `KYUPDT` (update date), `TSTPRD` (test/production flag), `Y2KCEN`, `Y2KCMP`.
   - **Initialization Subroutine (`*INZSR`)**: Not explicitly defined but implied to set initial states.

2. **Read Workstation File**:
   ```
   c                   if        qsctl = ' '
   c                   move      '1'           *in09
   c                   move      '0'           *in01
   c                   move      'R'           qsctl             1
   c                   else
   c                   move      '0'           *in09
   c                   move      '1'           *in01
   c                   read      ar135pfm                               lr
   c   lr              return
   c                   end
   ```
   - Checks control field `QSCTL`:
     - If blank, sets read mode (`QSCTL = 'R'`), `*IN09` on (initial display), `*IN01` off.
     - Otherwise, sets `*IN01` on, `*IN09` off, and reads `AR135PFM` (workstation file).
     - If last record (`LR`), exits the program.
   - Resets indicators (`*IN50-*IN55`, `*IN20`, `*IN81`) and clears `MSG35`.

3. **Handle Exit (F23, `*INKG`)**:
   ```
   c                   if        (*inkg = *on)
   c                   eval      *in01 = *off
   c                   eval      *inu8 = *on
   c                   eval      *inlr = *on
   c                   goto      end
   c                   endif
   ```
   - If F23 is pressed (`*INKG`), turns off `*IN01`, sets `*INU8` and `*INLR` on, and jumps to `END` to exit.

4. **Initial Display (`*IN09`)**:
   ```
   c                   if        (*in09 = *on)
   c                   exsr      onetim
   c                   goto      end
   c                   endif
   ```
   - If `*IN09` is on (initial display), executes `ONETIM` subroutine and jumps to `END`.

5. **Validate Input (`*IN01`)**:
   ```
   c                   if        (*in01 = *on)
   c                   exsr      edit
   c                   endif
   ```
   - If `*IN01` is on (user submitted data), executes `EDIT` subroutine to validate input.

6. **Output Screen**:
   ```
   c  n81              eval      *inLR = *on
   c   81              write     ar135pfm
   ```
   - If `*IN81` is off, sets `*INLR` to exit after processing.
   - If `*IN81` is on (error condition), writes to `AR135PFM` to redisplay the screen with error messages.

7. **ONETIM Subroutine**:
   ```
   c     onetim        begsr
   C     '00'          chain     gscont                             99
   c  N99gxcono        ifne      *zeros
   c  N99              z-add     gxcono        kyco
   c                   endif
   c                   eval      *in81 = *on
   c                   movel     *blanks       msg40            40
   c                   endsr
   ```
   - Chains to `GSCONT` with key `'00'` to retrieve default company number (`GXCONO`).
   - If found and non-zero, sets `KYCO = GXCONO`.
   - Sets `*IN81` to display the screen and clears `MSG40`.

8. **EDIT Subroutine**:
   ```
   c     edit          begsr
   c     kyco          chain     arcont                             20
   c                   if        (*in20 = *on)
   c                   eval      *in81 = *on
   c                   movel     msg(1)        msg40
   c                   goto      endedt
   c                   endif
   c     kyupdt        comp      *zeros                                 20
   c                   if        (*in20 = *on)
   c                   eval      *in81 = *on
   c                   movel     msg(2)        msg40
   c                   goto      endedt
   c                   endif
   c                   movel     'GE'          file
   c                   movel     tstprd        wrk2              2
   c                   move      'E'           wrk2
   c                   movel     wrk2          file
   c                   move      kyupdt        file
   c                   call      'AR135TC'
   c                   parm                    filenm
   c                   parm                    status
   c                   if        status = 'N'
   c                   movel     msg(3)        msg40
   c                   eval      *in81 = *on
   c                   eval      *in20 = *on
   c                   goto      endedt
   c                   end
   c     endedt        endsr
   ```
   - **Validate Company**:
     - Chains to `ARCONT` using `KYCO`.
     - If not found (`*IN20`), sets `*IN81`, loads error message ("INVALID COMPANY #"), and jumps to `ENDEDT`.
   - **Validate Bank Upload Date**:
     - Compares `KYUPDT` to zero.
     - If zero (`*IN20`), sets `*IN81`, loads error message ("MUST ENTER BANK UPLOAD DATE"), and jumps to `ENDEDT`.
   - **Validate File Existence**:
     - Builds file name: `GE + TSTPRD + E + KYUPDT` (e.g., `GEPRODE123456`).
     - Calls `AR135TC` with `FILENM` and `STATUS` parameters to check if the file exists.
     - If `STATUS = 'N'` (file not found), sets `*IN81` and `*IN20`, loads error message ("FILE WITH DATE SELECTED DOES NOT EXIST"), and jumps to `ENDEDT`.
   - Ends with `ENDEDT`.

9. **End Processing**:
   ```
   c     end           tag
   ```
   - Jumps to `END` tag to redisplay the screen or exit.

---

### Business Rules

The program enforces the following business rules:

1. **Interactive Input**:
   - Displays a screen (`AR135PFM`) for entering company number (`KYCO`) and bank upload date (`KYUPDT`).
   - Initial display (`*IN09`) populates default company from `GSCONT`.

2. **Validation**:
   - **Company Number (`KYCO`)**:
     - Must exist in `ARCONT` (error: "INVALID COMPANY #").
   - **Bank Upload Date (`KYUPDT`)**:
     - Must be non-zero (error: "MUST ENTER BANK UPLOAD DATE").
   - **File Existence**:
     - Checks if the EFT transaction file (e.g., `GEPRODE123456`) exists via `AR135TC` (error: "FILE WITH DATE SELECTED DOES NOT EXIST").
   - Errors trigger `*IN81` to redisplay the screen with `MSG40`.

3. **Environment Flexibility**:
   - Uses `TSTPRD` (test/production flag) to construct file names, supporting multiple environments.
   - `KYCO` defaults from `GSCONT` if available.

4. **Y2K Compliance**:
   - Uses `Y2KCEN` and `Y2KCMP` for date handling, ensuring correct century for `KYUPDT`.

5. **Exit Handling**:
   - F23 (`*INKG`) exits the program cleanly, setting `*INU8` and `*INLR`.

---

### Tables/Files Used

| File Name | Type | Record Length | Key Location | Usage/Description |
|-----------|------|---------------|--------------|-------------------|
| **ARCONT** | Input (IF) | 256 | 2 | AR control file; validates company number (`KYCO`) and provides company data. |
| **GSCONT** | Input (IF) | 512 | 2 | System control file; provides default company number (`GXCONO`). |
| **AR135PD** | Workstation (CF) | N/A | N/A | Display file for user input (company, bank upload date) and error messages. |

- **OCL Context**: Only `ARCONT` is explicitly opened in `AR135P.ocl36.txt` (`?9?ARCONT`, shared mode). `GSCONT` and `AR135PD` are managed by the RPG program.
- **RPG Context**: `ARCONT` and `GSCONT` are chained for validation; `AR135PD` handles user interaction.

---

### External Programs Called

- **AR135TC**:
  - Called in the `EDIT` subroutine to check the existence of the EFT transaction file.
  - Parameters: `FILENM` (file name, e.g., `GEPRODE123456`) and `STATUS` (returns 'N' if file not found).
  - Purpose: Validates that the transaction file for the selected date exists.

No other external programs are called. The program is invoked by the OCL (`AR135P.ocl36.txt`) via `LOAD` and `RUN`.

---

### Summary

**RPG Program (`AR135P.rpgle.txt`)**:
- **Process Steps**: Initializes with default company from `GSCONT`, displays a screen (`AR135PD`) for entering company number (`KYCO`) and bank upload date (`KYUPDT`), validates input against `ARCONT` and file existence via `AR135TC`, and redisplays on errors or exits on F23.
- **Business Rules**: Ensures valid company and non-zero bank upload date, checks transaction file existence, supports multiple environments via `TSTPRD`, and handles Y2K dates. Errors trigger screen redisplay with messages.
- **Files Used**: `ARCONT`, `GSCONT`, `AR135PD`.
- **External Programs**: Calls `AR135TC` for file validation.

The `AR135P` program is the entry point for the EFT process, capturing and validating initial parameters before other programs (`AR135`, `AR135B`, `AR135A`) process transactions and generate reports. If you need details on `AR135TC` or further integration, additional source files would be required.