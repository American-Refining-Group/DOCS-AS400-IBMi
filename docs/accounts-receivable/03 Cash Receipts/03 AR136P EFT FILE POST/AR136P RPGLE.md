The provided RPGLE program, `AR136P.rpgle.txt`, is an RPG IV program used on IBM midrange systems (e.g., AS/400 or IBM i) to prompt for and validate EFT (Electronic Funds Transfer) posting table date selection, likely as part of an Accounts Receivable (AR) process. It is called from the OCL program `AR136P.ocl36.txt` previously analyzed. Below, I will explain the process steps, business rules, tables/files used, and external programs called.

### Process Steps of the RPGLE Program

The program is structured to interact with a workstation (display file) for user input, validate the input against database files, and set a status flag for further processing. Here’s a step-by-step breakdown of the program’s execution:

1. **Program Initialization**:
   - **Header Specifications**:
     ```rpgle
     H DFTACTGRP(*NO)
     H fixnbr(*zoned:*inputpacked)
     H dftname(ar136p)
     ```
     - `DFTACTGRP(*NO)`: The program does not run in the default activation group, allowing for better control over resources.
     - `fixnbr(*zoned:*inputpacked)`: Automatically converts zoned and packed numeric fields to a valid format during input, handling potential data issues.
     - `dftname(ar136p)`: Sets the default program name to `AR136P`.

   - **File Definitions**:
     ```rpgle
     far136pd   cf   e             workstn Handler('PROFOUNDUI(HANDLER)')
     farcont    if   f  256    02aidisk    keyloc(2)
     fgscont    if   f  512     2aidisk    keyloc(2)
     ```
     - `ar136pd`: A workstation file (display file) for user interaction, using the `PROFOUNDUI(HANDLER)` for a modern UI interface (likely a web or graphical front-end).
     - `arcont`: An input file (Accounts Receivable control file) with a record length of 256 bytes, keyed starting at position 2.
     - `gscont`: Another input file (likely a system or general control file) with a record length of 512 bytes, also keyed starting at position 2.

   - **Data Definitions**:
     ```rpgle
     d msg             s             40    dim(3) ctdata perrcd(1)
     d filenm          ds
     d  file                   1      8
     d                uds
     d  kyco                 101    102  0
     d  kysldt               103    108  0
     d  status               109    109
     d  kyupdt               110    115  0
     d  y2kcen               509    510  0
     d  y2kcmp               511    512  0
     ```
     - `msg`: An array of three 40-character error messages loaded from compile-time data (see the end of the source).
     - `filenm`: A data structure with an 8-character subfield `file` used to pass a file name to an external program.
     - `uds`: A data structure for a data area, defining:
       - `kyco` (positions 101-102): Company number (numeric).
       - `kysldt` (positions 103-108): Selected date (numeric).
       - `status` (position 109): A single-character status flag.
       - `kyupdt` (positions 110-115): Bank upload date (numeric).
       - `y2kcen` (positions 509-510) and `y2kcmp` (positions 511-512): Likely used for Y2K date handling (century and company).

2. **Workstation File Handling**:
   ```rpgle
   c                   if        qsctl = ' '
   c                   move      '1'           *in09
   c                   move      '0'           *in01
   c                   move      'R'           qsctl             1
   c                   else
   c                   move      '0'           *in09
   c                   move      '1'           *in01
   c                   read      ar136pfm                               lr
   c   lr              return
   c                   end
   ```
   - The program checks the control variable `qsctl` (likely a screen control field).
   - If `qsctl` is blank, it sets indicator `*in09` on (for one-time processing), `*in01` off, and sets `qsctl` to 'R' (possibly indicating "read" mode).
   - If `qsctl` is not blank, it sets `*in09` off, `*in01` on, reads the workstation file `ar136pfm`, and checks for the last record (`lr`). If the last record is reached, the program returns (exits).
   - This handles the initial display and subsequent reads of the workstation screen.

3. **Indicator Initialization**:
   ```rpgle
   c                   eval      *in50 = *off
   c                   eval      *in51 = *off
   c                   eval      *in52 = *off
   c                   eval      *in53 = *off
   c                   eval      *in54 = *off
   c                   eval      *in55 = *off
   c                   eval      *in20 = *off
   c                   eval      *in81 = *off
   c                   movel     *blanks       msg35            35
   ```
   - Initializes indicators 50-55, 20, and 81 to off, ensuring a clean state for error handling and screen control.
   - Clears `msg35` (a 35-character message field) to blanks.

4. **Function Key Check (Exit)**:
   ```rpgle
   c                   if        (*inkg = *on)
   c                   eval      *in01 = *off
   c                   eval      *inu8 = *on
   c                   eval      *inlr = *on
   c                   goto      end
   c                   endif
   ```
   - Checks if function key indicator `*inkg` (likely F3 for exit) is on.
   - If true, sets `*in01` off, `*inu8` on (possibly for user exit), `*inlr` on (last record, to end the program), and branches to the `end` tag to terminate.

5. **One-Time Processing**:
   ```rpgle
   c                   if        (*in09 = *on)
   c                   exsr      onetim
   c                   endif
   c                   if        (*in09 = *on)
   c                   goto      end
   c                   endif
   ```
   - If indicator `*in09` is on (set during initial screen load), the `onetim` subroutine is executed to perform one-time setup.
   - After `onetim`, the program branches to the `end` tag, skipping further processing for the initial screen load.

6. **Edit Processing**:
   ```rpgle
   c                   if        (*in01 = *on)
   c                   exsr      edit
   c                   endif
   ```
   - If indicator `*in01` is on (set after the first screen read), the `edit` subroutine is executed to validate user input.

7. **Program Termination**:
   ```rpgle
   c     end           tag
   c  n81              eval      *inLR = *on
   c   81              write     ar136pfm
   ```
   - The `end` tag is the termination point.
   - If indicator `*in81` is off (no errors), sets `*inLR` on to end the program.
   - If `*in81` is on (error condition), writes to the `ar136pfm` display file to show the error message to the user.

8. **Edit Subroutine (`edit`)**:
   ```rpgle
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
   c                   movel     'Y'           status
   c     endedt        endsr
   ```
   - **Validate Company Number**:
     - Performs a `CHAIN` operation on the `arcont` file using `kyco` (company number).
     - If no record is found (`*in20` on), sets `*in81` on, moves the first error message ("INVALID COMPANY #") to `msg40`, and branches to `endedt`.
   - **Validate Bank Upload Date**:
     - Compares `kyupdt` (bank upload date) to zero.
     - If zero (`*in20` on), sets `*in81` on, moves the second error message ("MUST ENTER BANK UPLOAD DATE") to `msg40`, and branches to `endedt`.
   - **Check Table Existence**:
     - Constructs a file name by concatenating 'GE' with `kyupdt` into the `file` field.
     - Calls the external program `AR135TC`, passing `filenm` (file name) and `status` as parameters.
     - If `status` returns 'N' (file does not exist), sets `*in81` and `*in20` on, moves the third error message ("FILE WITH DATE SELECTED DOES NOT EXIST") to `msg40`, and branches to `endedt`.
   - **Success**:
     - If all validations pass, sets `status` to 'Y' (indicating a valid record).
   - **End of Subroutine**:
     - The `endedt` tag marks the end of the `edit` subroutine.

9. **One-Time Subroutine (`onetim`)**:
   ```rpgle
   c     onetim        begsr
   c     '00'          chain     gscont                             99
   c  n99              if        gxcono <> *zero
   c                   if        (*in99 = *off)
   c                   z-add     gxcono        kyco
   c                   endif
   c                   end
   c                   eval      *in81 = *on
   c                   movel     *blanks       msg40
   c                   endsr
   ```
   - Performs a `CHAIN` operation on the `gscont` file with key '00' to retrieve a default company number.
   - If a record is found (`*in99` off) and the company number (`gxcono`) is non-zero, sets `kyco` to `gxcono`.
   - Sets `*in81` on to trigger a screen write and clears `msg40` to blanks.

### Business Rules

The program enforces the following business rules for EFT posting table date selection:
1. **Company Number Validation**:
   - The user-entered company number (`kyco`) must exist in the `arcont` file. If not, the error message "INVALID COMPANY #" is displayed.
2. **Bank Upload Date Validation**:
   - The bank upload date (`kyupdt`) must be non-zero. If zero, the error message "MUST ENTER BANK UPLOAD DATE" is displayed.
3. **Table Existence Check**:
   - A file corresponding to the selected date (`kyupdt`) prefixed with 'GE' must exist, as verified by the `AR135TC` program. If it does not exist, the error message "FILE WITH DATE SELECTED DOES NOT EXIST" is displayed.
4. **Default Company Number**:
   - During one-time processing, the program retrieves a default company number from the `gscont` file (key '00') and populates `kyco` if a valid non-zero company number is found.
5. **Status Flag**:
   - If all validations pass, the `status` field is set to 'Y' to indicate a valid selection, which is likely used by the calling OCL program (`AR136P.ocl36.txt`) to proceed with further processing.
6. **Error Handling**:
   - If any validation fails, the program sets `*in81` on, displays the appropriate error message via the `ar136pfm` display file, and waits for user correction.
7. **Exit Handling**:
   - If the user presses the exit function key (`*inkg`, likely F3), the program terminates immediately.

### Tables/Files Used

1. **ar136pd**:
   - Type: Workstation (display) file
   - Purpose: Used for user interaction to input and display company number (`kyco`), bank upload date (`kyupdt`), and error messages (`msg40`).
   - Handler: `PROFOUNDUI(HANDLER)` (indicates a modern UI interface, possibly web-based).

2. **arcont**:
   - Type: Input file (disk)
   - Record Length: 256 bytes
   - Key: Starts at position 2 (`keyloc(2)`)
   - Purpose: Accounts Receivable control file, used to validate the company number (`kyco`).

3. **gscont**:
   - Type: Input file (disk)
   - Record Length: 512 bytes
   - Key: Starts at position 2 (`keyloc(2)`)
   - Purpose: General system control file, used to retrieve a default company number (`gxcono`) during one-time processing.

### External Programs Called

1. **AR135TC**:
   - Purpose: Called to check if a file with the name 'GE' + `kyupdt` exists.
   - Parameters:
     - `filenm`: An 8-character file name (input).
     - `status`: A single-character field (output, returns 'Y' or 'N' to indicate file existence).
   - Usage: Validates whether the EFT table for the selected date exists.

### Integration with OCL Program

The RPG program `AR136P` is called by the OCL program `AR136P.ocl36.txt`. The OCL program:
- Loads `AR136P` and specifies the `ARCONT` file (matching `arcont` in the RPG program).
- Checks the `status` field (position 109) after `AR136P` execution to determine if further processing (e.g., calling `AR136`) is needed.
- The `status` field set to 'Y' by `AR136P` indicates successful validation, allowing the OCL to proceed with the `AR136` procedure.

### Notes
- The program uses indicators (`*in01`, `*in09`, `*in20`, `*in81`, etc.) extensively for flow control and error handling, typical of legacy RPG programming.
- The `PROFOUNDUI(HANDLER)` suggests a modernized UI, likely replacing a traditional green-screen interface.
- The conversion comments (T4A, T4M, T4O) indicate this program was converted or modernized on 08/31/23, possibly from RPG III to RPG IV, with 43 added lines, 7 modified lines, and 93 total lines processed.
- The Y2K fields (`y2kcen`, `y2kcmp`) and the `GSY2K` utility in the OCL suggest historical Y2K compliance efforts, though they may not be actively used in modern executions.

If you have additional details (e.g., the display file `ar136pfm` layout, the `AR135TC` program logic, or the `AR136` procedure), I can provide further insights into the system’s functionality.