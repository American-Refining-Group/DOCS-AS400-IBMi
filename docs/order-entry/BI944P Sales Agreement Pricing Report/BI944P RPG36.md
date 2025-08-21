The RPG program `BI944P.rpg36.txt` is an RPG/400 program designed to prompt users for input parameters and validate them for generating a Customer Sales Agreement Master File Listing. It is called from the OCL program `BI944P.ocl36.txt` and interacts with a display file (`SCREEN`) and several database files to validate user inputs such as divisions, locations, customers, products, and units of measure. Below, I’ll explain the process steps, business rules, tables used, and any external programs called.

### Process Steps of the RPG Program

The RPG program is structured to handle user input through a workstation display file (`SCREEN`) with two screens (`BI944PS1` and `BI944PS2`) and validate the input data against database files. The program uses subroutines to perform one-time initialization, validate inputs, and handle date editing. Here’s a detailed breakdown of the process steps:

1. **Program Initialization** (Lines 0042–0050):
   - **Indicator Reset**: The program starts by resetting multiple indicators (31–90) used for error handling and control flow (`SETOF` statements).
   - **Blank Message**: Initializes the `MSG` field to blanks to clear any prior error messages (`MOVEL*BLANKS MSG`).
   - **KG Indicator Check**: If the `KG` indicator (keyboard error or cancel) is on, the program sets indicators `U1` and `LR` (last record), resets indicators 01, 09, 81, and 82, and branches to the `END` tag, effectively terminating the program.

2. **One-Time Setup Subroutine (`ONETIM`, Lines 0160–0200)**:
   - **Execution Condition**: Executed when indicator 09 is on, typically on the first program call.
   - **System Time and Date**: Captures the current system time and date (`TIME` operation) into `SYSTIM` and `SYSDAT`.
   - **Clear Arrays**: Initializes the `DCO` array to blanks.
   - **Set Counter**: Sets loop counter `X` to 1 and `BILIM` to 00 for file positioning.
   - **Read Control File (`BICONT`)**:
     - Positions the file pointer at the beginning (`SETLL BICONT`).
     - Reads records sequentially (`READ BICONT`).
     - Skips records marked as deleted (`BCDEL = 'D'`) by looping back to `AGNCO`.
     - Stops reading at end-of-file (indicator 20).
   - **Default Values**: Sets default selection parameters to `'ALL'` for `KYLOSL` (locations), `KYSMSL` (salesmen), `KYPDSL` (products), `KYPCSL` (product classes), `KYCSSL` (customers), and `KYUMSL` (units of measure). Sets `KYDIV` to `'R'`, `KYJOBQ`, `KYSPRD`, and `KYADDA` to `'N'`, and `KYCOPY` to 1.
   - **Indicator Setup**: Sets indicator 81 on to signal successful initialization.
   - **Exit**: Branches to `END` after execution.

3. **Screen 1 Processing Subroutine (`S1`, Lines 0062–0156)**:
   - **Execution Condition**: Triggered when indicator 01 is on, corresponding to the first screen (`BI944PS1`) submission.
   - **Validate Selection Fields**:
     - Checks fields like `KYLOSL`, `KYPCSL`, `KYCSSL`, `KYPDSL`, `KYSMSL`, and `KYUMSL` for values `'ALL'` or `'SEL'`. If neither, sets error indicators 81 and 90, moves an error message from the `COM` array (e.g., `COM,1` for invalid selection), and branches to `ENDS1`.
     - For `'ALL'`, ensures no specific selections are entered (e.g., `LOC`, `CLS`, `CS`, `PRD`) by comparing them to blanks. If non-blank, sets error indicators and assigns message `COM,10`.
     - For `'SEL'`, ensures specific selections are provided (non-blank). If blank, sets error indicators and assigns message `COM,11`.
   - **Division Validation**:
     - Checks `KYDIV` for valid values (`'R'`, `'L'`, or `'B'`). If invalid, sets error indicators and assigns `COM,2`.
     - Sets `KYDVNO` to `'00'` for `'R'` or `'50'` for `'L'`.
   - **Location Validation (`KYLOSL = 'SEL'`)**:
     - Loops through the `LOC` array (up to 5 locations, `KYLOC1`–`KYLOC5`).
     - For non-blank/non-zero entries, chains to `INLOC` using `LOCKEY`. If not found (indicator 41), sets error indicators, calls `SETIND` to set error flags, and assigns `COM,3`.
   - **Product Class Validation (`KYPCSL = 'SEL'`)**:
     - Loops through the `CLS` array (up to 10 classes, `KYPC01`–`KYPC10`).
     - Chains to `GSTABL` using `PRCLKY`. If not found (indicator 32), sets error indicators, calls `SETIND`, and assigns `COM,5`.
   - **Customer Validation (`KYCSSL = 'SEL'`)**:
     - Loops through the `CS` array (up to 20 customers, `KYCS01`–`KYCS20`).
     - Chains to `ARCUST` using `KEY8`. If not found (indicator 33), sets error indicators, calls `SETIND`, and assigns `COM,6`.
   - **Product Validation (`KYPDSL = 'SEL'`)**:
     - Loops through the `PRD` array (up to 10 products, `KYPD01`–`KYPD10`).
     - Chains to `GSPROD` using `KLPROD`. If not found (indicator 31), sets error indicators, calls `SETIND`, and assigns `COM,4`.
   - **Salesman Validation (`KYSMSL = 'SEL'`)**:
     - Chains to `GSTABL` using `PRSLKY` for `KYFRSM` and `KYTOSM`. If either is invalid (indicators 47 or 48), sets error indicators and assigns `COM,7` or `COM,8`.
   - **Unit of Measure Validation (`KYUMSL = 'SEL'`)**:
     - Checks `KYUMYP` for `'INCLUDE'` or `'EXCLUDE'`. If invalid, sets error indicators and assigns `COM,18`.
     - If `'ALL'`, ensures `KYUMYP` is blank (`COM,20` if not).
     - Chains to `GSTABL` for `KYUM01` using `UMKEY`. If not found (indicator 18), sets error indicators and assigns `COM,14`.
   - **Additional Parameters**:
     - Validates `KYADDA` and `KYJOBQ` for blank, `'N'`, or `'Y'`. If invalid, sets error indicators and assigns `COM,9`.
     - If `KYADDA = 'Y'` and `KYSPRD = 'Y'`, sets error indicators and assigns `COM,13`.
   - **Date Handling**:
     - Converts `UDATE` to an 8-digit format (`KYDAT8`) with century adjustment based on `Y2KCMP` and `Y2KCEN`.
   - **Output**: Writes validated or error data to `SCREEN` with record format `BI944PS1`, including error message `MSG`.

4. **Screen 2 Processing Subroutine (`S2`, Lines 0157–0150)**:
   - **Execution Condition**: Triggered when indicator 02 is on, corresponding to the second screen (`BI944PS2`) submission.
   - **Start Date Conversion**:
     - Converts `KYSTDT` to an 8-digit format (`KYSTD8`) with century adjustment, similar to `KYDAT8`.
   - **Industry Code Validation**:
     - Checks `KYINDC` for `'INCREASE'` or `'DECREASE'`. If invalid, sets indicators 82 and 90 and assigns `COM,12`.
   - **Unit of Measure Validation**:
     - Chains to `GSTABL` for `KYUOFM` using `UMKEY`. If not found (indicator 88), sets indicators 82 and 90 and assigns `COM,14`.
   - **Start Date Validation**:
     - If `KYUOFM` is non-blank and `KYSTDT` is zero, sets indicators 82, 90, and 89 and assigns `COM,15`.
     - Calls `@DTEDT` to validate `KYSTDT`. If invalid (indicator 86), sets error indicators and assigns `COM,16`.
     - Compares `KYSTDT` to system date (`SYSDAT`). If `KYSTDT` is not after today, sets error indicators and assigns `COM,17`.
   - **Output**: Writes validated or error data to `SCREEN` with record format `BI944PS2`, including error message `MSG`.

5. **Indicator Setting Subroutine (`SETIND`, Lines 0160–0156)**:
   - Sets specific indicators (21–80) based on the value of `X` and the error condition (indicators 31, 32, 33, or 41) to flag which specific selection (location, product, class, or customer) caused the error.

6. **Date Editing Subroutine (`@DTEDT`, Lines 0014–0125)**:
   - Validates `KYSTDT` by breaking it into month, day, and year components.
   - Checks month (1–12) and day validity, accounting for leap years:
     - For February, checks for leap year using year divisibility by 4 or 400 for century years.
     - For other months, checks day limits (30 or 31).
   - Sets indicator 86 if the date is invalid.

7. **Program Termination**:
   - The program branches to `END` after processing each screen or if errors occur (via `KG` or validation failures).
   - Outputs to `SCREEN` are used to display errors or prompt for corrections.

### Business Rules

The program enforces the following business rules for validating user inputs:

1. **Selection Fields (`KYLOSL`, `KYPCSL`, `KYCSSL`, `KYPDSL`, `KYSMSL`, `KYUMSL`)**:
   - Must be `'ALL'` or `'SEL'`. Otherwise, error message `COM,1` is displayed.
   - If `'ALL'`, corresponding selection arrays (`LOC`, `CLS`, `CS`, `PRD`) must be blank (`COM,10` if not).
   - If `'SEL'`, at least one entry in the corresponding array must be non-blank (`COM,11` if all blank).

2. **Division (`KYDIV`)**:
   - Must be `'R'`, `'L'`, or `'B'`. Otherwise, error message `COM,2` is displayed.
   - Sets `KYDVNO` to `'00'` for `'R'` or `'50'` for `'L'`.

3. **Locations (`KYLOC1`–`KYLOC5`)**:
   - If `KYLOSL = 'SEL'`, each non-blank/non-zero entry must exist in `INLOC` (`COM,3` if not found).

4. **Product Classes (`KYPC01`–`KYPC10`)**:
   - If `KYPCSL = 'SEL'`, each non-blank entry must exist in `GSTABL` (`COM,5` if not found).

5. **Customers (`KYCS01`–`KYCS20`)**:
   - If `KYCSSL = 'SEL'`, each non-blank/non-zero entry must exist in `ARCUST` (`COM,6` if not found).
   - If `KYCSSL = 'SEL'`, `KYSTYP` must be `'INCLUDE'` or `'EXCLUDE'` (`COM,18` if invalid).
   - If `KYCSSL = 'ALL'`, `KYSTYP` must be blank (`COM,19` if not).

6. **Products (`KYPD01`–`KYPD10`)**:
   - If `KYPDSL = 'SEL'`, each non-blank entry must exist in `GSPROD` (`COM,4` if not found).

7. **Salesmen (`KYFRSM`, `KYTOSM`)**:
   - If `KYSMSL = 'SEL'`, both must exist in `GSTABL` (`COM,7` or `COM,8` if not found).

8. **Units of Measure (`KYUMSL`, `KYUMYP`, `KYUM01`)**:
   - If `KYUMSL = 'SEL'`, `KYUMYP` must be `'INCLUDE'` or `'EXCLUDE'` (`COM,18` if invalid).
   - If `KYUMSL = 'ALL'`, `KYUMYP` must be blank (`COM,20` if not).
   - If `KYUMSL = 'SEL'`, `KYUM01` must exist in `GSTABL` (`COM,14` if not found).

9. **Additional Parameters**:
   - `KYADDA` and `KYJOBQ` must be blank, `'N'`, or `'Y'` (`COM,9` if invalid).
   - If `KYADDA = 'Y'`, `KYSPRD` cannot be `'Y'` (`COM,13` if both are `'Y'`).

10. **Industry Code (`KYINDC`)**:
    - Must be `'INCREASE'` or `'DECREASE'` if non-blank (`COM,12` if invalid).

11. **Unit of Measure and Start Date (`KYUOFM`, `KYSTDT`)**:
    - If `KYUOFM` is non-blank, `KYSTDT` must be non-zero (`COM,15` if zero).
    - `KYSTDT` must be a valid date (`COM,16` if invalid, checked via `@DTEDT`).
    - `KYSTDT` must be after the system date (`COM,17` if not).

12. **Copy Count (`KYCOPY`)**:
    - Defaults to 1 if zero.

### Tables (Files) Used

The program uses the following files, as defined in the File Specification (F-spec) and Input Specification (I-spec):

1. **SCREEN** (Workstation File):
   - Type: Combined (`CP`), used for interactive input/output.
   - Record formats: `BI944PS1` (screen 1) and `BI944PS2` (screen 2).
   - Fields: Input fields like `KYDIV`, `KYLOSL`, `KYLOC1`–`KYLOC5`, `KYPCSL`, `KYPC01`–`KYPC10`, `KYCSSL`, `KYCS01`–`KYCS20`, `KYPDSL`, `KYPD01`–`KYPD10`, `KYSMSL`, `KYFRSM`, `KYTOSM`, `KYSPRD`, `KYJOBQ`, `KYCOPY`, `KYADDA`, `KYUMSL`, `KYUMYP`, `KYUM01`, `KYINDC`, `KYDLCH`, `KYUOFM`, `KYSTDT`, and output field `MSG`.

2. **BICONT** (Control File):
   - Type: Input (`IF`), disk file, 256 bytes, logical access.
   - Fields: `BCDEL` (delete flag), `BCCO` (company number), `BCNAME` (company name).
   - Usage: Read to check for non-deleted company records.

3. **GSTABL** (General System Table):
   - Type: Input (`IF`), disk file, 256 bytes, indexed.
   - Fields: `TBDESC` (table description), `TBDES2` (alternate description), `TBABDS` (short description).
   - Usage: Chained to validate product classes (`PRCLKY`), salesmen (`PRSLKY`), and units of measure (`UMKEY`).

4. **ARCUST** (Customer Master File):
   - Type: Input (`IF`), disk file, 384 bytes, indexed.
   - Fields: `ARNAME` (customer name).
   - Usage: Chained to validate customer numbers (`KEY8`).

5. **INLOC** (Location File):
   - Type: Input (`IF`), disk file, 512 bytes, indexed.
   - Fields: `ILDEL` (delete flag), `ILCONO` (company), `ILLOC` (location), `ILNAME` (location name).
   - Usage: Chained to validate location codes (`LOCKEY`).

6. **GSPROD** (Product Master File):
   - Type: Input (`IF`), disk file, 512 bytes, indexed.
   - Fields: `TPDEL` (delete flag), `TPDESC` (description), `TPDES1` (complete description), `TPPRGP` (product group code), `TPPRCL` (product class code), `TPABDS` (short description), `TPFLCD`.
   - Usage: Chained to validate product codes (`KLPROD`).

### External Programs Called

The RPG program does not explicitly call any external programs via `CALL` operations. It interacts with the display file `SCREEN` and database files but relies on the OCL program (`BI944P.ocl36.txt`) to initiate its execution and potentially call a related program (`BI944`) based on the `KYJOBQ` parameter. The reference to `BI944` in the OCL suggests it may be a follow-up program, but no direct calls are made within the RPG code.

### Summary

- **Process Steps**:
  1. Initialize indicators and message field; exit if `KG` is on.
  2. Perform one-time setup (`ONETIM`): read `BICONT`, set defaults, and initialize parameters.
  3. Process screen 1 (`S1`): validate division, locations, product classes, customers, products, salesmen, units of measure, and additional parameters; output to `BI944PS1`.
  4. Process screen 2 (`S2`): validate industry code, unit of measure, and start date; output to `BI944PS2`.
  5. Set error indicators (`SETIND`) for specific invalid selections.
  6. Validate dates (`@DTEDT`) for `KYSTDT`.
  7. Terminate after processing or errors, writing results to `SCREEN`.

- **Business Rules**:
  - Enforce valid values for selection fields (`'ALL'` or `'SEL'`), division (`'R'`, `'L'`, `'B'`), and other parameters.
  - Validate specific selections against database files (`INLOC`, `GSTABL`, `ARCUST`, `GSPROD`).
  - Ensure `KYSTYP` and `KYUMYP` are `'INCLUDE'` or `'EXCLUDE'` when applicable, or blank for `'ALL'`.
  - Validate `KYADDA`, `KYJOBQ`, and `KYSPRD` for correct values.
  - Ensure `KYSTDT` is valid and future-dated if `KYUOFM` is non-blank.
  - Default `KYCOPY` to 1.

- **Tables Used**:
  - `SCREEN` (workstation file)
  - `BICONT` (control file)
  - `GSTABL` (general system table)
  - `ARCUST` (customer master file)
  - `INLOC` (location file)
  - `GSPROD` (product master file)

- **External Programs Called**:
  - None directly called within the RPG program. The OCL may call `BI944` based on `KYJOBQ`.

This RPG program serves as a front-end validation step for the Customer Sales Agreement Master File Listing, ensuring all user inputs are correct before proceeding to further processing, likely handled by `BI944` as indicated in the OCL. If you need further analysis of the `BI944` program or specific file layouts, please provide additional details or files.