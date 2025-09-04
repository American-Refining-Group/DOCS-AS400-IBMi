The provided document, `BB953B.rpg36.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `BB953B`. It is invoked by the OCL script `BB953B.ocl36.txt` as part of the pricing generation process for blended lubes, within the `PRICES.ocl36.txt` workflow, which is itself called by `PRICEGEN.clp`. The program appears to perform initial setup for rack pricing by processing contract data from the `BICONT` file and setting a key date (`KYDAT8`) for use in subsequent programs (`BB9531`, `BB9534`, `BB953`). Below, I will explain the process steps, business rules, tables used, and external programs called .

### Process Steps of the RPG Program

1. **File Definitions**:
   - The `F` specification defines one file:
     - `BICONT` (Input, `IF`, 256 bytes, indexed with 2-byte key, disk): Contract file, used for reading contract data, opened in shared read mode with record locking (`DISP-SHRRM` from `BB953B.ocl36.txt`).

2. **Input Specifications**:
   - **BICONT** (record type `NS`):
     - `BCDEL` (1): Delete code (likely `'D'` for deleted records).
     - `BCCO` (2–3): Company number.
     - `BCNAME` (4–33): Company name.
   - **User Data Structure (UDS)**:
     - `KYCO` (101–102): Company code.
     - `KYDIV` (103): Division filter ('ALL' or 'CO').
     - `KYLOSL` (104–106): Location selection ('SEL' or other).
     - `KYLOC1`–`KYLOC5` (107–121): Location codes (3 bytes each).
     - `KYPCSL` (122–124): Product class selection ('SEL' or other).
     - `KYPC01`–`KYPC10` (125–154): Product class codes (3 bytes each).
     - `KYCTSL` (155–157): Container selection ('SEL' or other).
     - `KYCT01`–`KYCT05` (158–172): Container codes (3 bytes each).
     - `KYPDSL` (173–175): Product code selection ('SEL' or other).
     - `KYPD01`–`KYPD10` (176–215): Product codes (4 bytes each).
     - `KYDTSL` (216–218): Date selection ('SEL' or other).
     - `KYFRDT` (219–224): From date (YYMMDD).
     - `KYTODT` (225–230): To date (YYMMDD).
     - `KYJOBQ` (231): Job queue indicator.
     - `KYCOPY` (232–233): Copy number.
     - `KYDVNO` (234–235): Division number.
     - `KYDAT8` (236–243): Key date (CYMD format, CCYYMMDD).
     - `Y2KCEN` (509–510): Century (e.g., `'19'`).
     - `Y2KCMP` (511–512): Comparison year (e.g., `'80'` for Y2K pivot).

3. **Calculation Specifications**:
   - **Initialization**:
     - `Z-ADD00 BILIM 20`: Initializes `BILIM` (2-digit field) to zero, used as a record limit or counter.
     - `BILIM SETLLBICONT`: Positions the file pointer at the start of `BICONT` using its index.
     - `READ BICONT 20`: Reads the first record from `BICONT`, setting indicator `20` if end-of-file or an error occurs.
   - **Date Processing (Y2K Logic)**:
     - `Z-ADD*ZERO KYDAT8`: Initializes `KYDAT8` to zero.
     - `UDATE MULT 10000.01 KYDAT6 60`: Multiplies the system date (`UDATE`, YYMMDD) by 10000.01 to create a 6-digit field (`KYDAT6`). This operation is unusual and may be a workaround to format the date, potentially resulting in `YYMMDD.00`.
     - `Z-ADDKYDAT6 KYDAT8`: Moves `KYDAT6` to `KYDAT8` (8 digits, CYMD format).
     - Y2K century logic:
       - `UYEAR IFGE Y2KCMP`: Checks if the system year (`UYEAR`, 2 digits) is greater than or equal to `Y2KCMP` (e.g., `'80'`).
         - If true: `Z-ADDY2KCEN UCN 20` sets `UCN` to `Y2KCEN` (e.g., `'19'` for 1900s).
         - If false: `1 ADD Y2KCEN UCN` increments `Y2KCEN` by 1 (e.g., `'20'` for 2000s).
       - `MOVELUCN KYDAT8 80`: Moves the century (`UCN`) into the first two digits of `KYDAT8`, forming a full CYMD date (e.g., `19YYMMDD` or `20YYMMDD`).
   - **Termination**:
     - `SETON LR`: Sets the Last Record indicator (`LR`), terminating the program after setting `KYDAT8`.

### Business Rules

1. **Purpose**: Initializes the rack pricing process by reading the `BICONT` file and setting a Y2K-compliant key date (`KYDAT8`) in the User Data Structure (UDS) for use by subsequent programs (`BB9531`, `BB9534`, `BB953`) in the `BB953B.ocl36.txt` workflow.
2. **Date Processing**:
   - Converts the system date (`UDATE`, YYMMDD) to a Y2K-compliant 8-digit date (`KYDAT8`, CCYYMMDD).
   - Uses a pivot year (`Y2KCMP = '80'`) to determine the century:
     - If `UYEAR ≥ 80`, sets century to `Y2KCEN` (e.g., `'19'`).
     - If `UYEAR < 80`, sets century to `Y2KCEN + 1` (e.g., `'20'`).
3. **File Access**: Reads the first record of `BICONT` to validate its existence or position the file pointer, but does not process further records, suggesting a setup role.
4. **Context**: As the first program in the `BB953B.ocl36.txt` sequence, it prepares parameters (e.g., `KYDAT8`, `KYLOC1`–`KYLOC5`, `KYPC01`–`KYPC10`, etc., set via `LOCAL OFFSET` in the OCL) for rack price reporting.
5. **Minimal Processing**: The program’s limited logic indicates it is a preparatory step, setting up `KYDAT8` and possibly validating `BICONT` for subsequent programs.

### Tables (Files) Used

1. **BICONT** (`?9?BICONT`, e.g., `ABICONT`):
   - **Access**: Input (`IF`), shared read with record locking (`DISP-SHRRM`).
   - **Purpose**: Reads contract data (company number, name, delete code) to validate or position the file.

### External Programs Called

- **None**: The `BB953B` RPG program does not call external programs or subroutines. It performs internal setup logic and terminates.

### Additional Notes

- **Context with BB953B.ocl36.txt**: Invoked as the first program in the `BB953B` workflow, followed by `BB9531`, `BB9534`, `#GSORT`, and `BB953`. It sets `KYDAT8` and uses `LOCAL OFFSET` parameters (e.g., `KYLOC1`–`KYLOC5`, `KYFRDT`, `KYTODT`) for filtering in later steps.
- **System/36 Environment**: Uses RPG II/III syntax in a System/36 environment, likely on AS/400.
- **Y2K Compliance**: The program ensures dates are Y2K-compliant, critical for rack price reporting within a date range (e.g., `KYFRDT` to `KYTODT`).
- **Potential Issue**: The `UDATE MULT 10000.01 KYDAT6` operation is unusual and may not produce the expected date format, potentially requiring verification.
- **Error Handling**: Lacks explicit error handling, relying on the System/36 environment or `BB953B.ocl36.txt` for error management.

If you have the RPG source code for `BB9531`, `BB9534`, `BB953`, or the `SA505*` programs, or need further analysis of the pricing generation workflow, please provide those details! Let me know if you have additional questions or files to share.