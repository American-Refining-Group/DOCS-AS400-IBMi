The provided `BB714A.rpg36.txt` is an RPG program (for IBM System/36 or AS/400) called by the `BB715.ocl36.txt` OCL program as part of the preprocessing for the "Daily Requirements Report." Its primary function is to enhance the `BB713M` file by adding sort codes (`CNTY` and `CSRT`) based on container and product data from `GSCNTR1` and `GSPROD`, using `GSTABL` for additional validation. Below, I’ll explain the process steps, business rules, tables (files) used, and external programs called.

### Process Steps of the RPG Program

The `BB714A` RPG program updates records in `BB713M` by retrieving container type (`TCCNTY`) from `GSCNTR1` and sort code (`TBCSRT`) from `GSTABL`, based on product class (`TPPRCL` from `GSPROD`). Here’s a step-by-step breakdown:

1. **Program Initialization**:
   - **Header (H-spec)**: Line 0002 defines the program name (`BB714A`) and page length (`P064`).
   - **File Definitions (F-spec)**:
     - `BB713M`: Update file (input/output, 530 bytes, disk, Line 0008).
     - `GSTABL`: Input file (256 bytes, indexed with 12 keys, disk, Line 0008).
     - `GSCNTR1`: Input file (512 bytes, indexed with 3 keys, disk, Line JK01).
     - `GSPROD`: Input file (512 bytes, indexed with 6 keys, disk, Line JK02).
   - **Input Definitions (I-spec)**:
     - `BB713M` (Lines 0017–0029):
       - `BDDEL` (1): Delete flag ('D' for deleted).
       - `BDCO` (2–3): Company number.
       - `BDORD#` (4–9): Order number.
       - `BDSEQ` (10–12): Detail line sequence.
       - `BDLOC` (22–24): Location.
       - `BDPROD` (25–28): Product code.
       - `BDTANK` (29–32): Tank code.
       - `BDQTY` (35–41): Container quantity.
       - `BDPRCE` (42–50): Price.
       - `BDUM` (51–53): Unit of measure.
       - `BDCSTK` (54–73): Customer stock number.
       - `BDDESC` (75–104): Description.
       - `BDCNTR` (121–123): Container code.
     - `GSTABL` (Lines 0035–0041):
       - `TBDEL` (1): Delete flag.
       - `TBTYPE` (2–7): Type code.
       - `TBCODE` (8–13): Code.
       - `TBPRGP` (89–90): Product group.
       - `TBICOL` (94–95): Item color.
       - `TBPRCL` (127–129): Product class.
       - `TBSELL` (155): Sell product indicator ('Y').
       - `TBCSRT` (178–179): Customer sort code.
     - `GSCNTR1` (Line JK01):
       - `TCCNTY` (180): Container type ('B' for bulk, 'P' for packaged).
     - `GSPROD` (Line JK02):
       - `TPDEL` (1): Delete flag.
       - `TPPRCL` (127–129): Product class.
   - **Indicators**:
     - `01`: Triggers record processing.
     - `02`: Skips to program end.
     - `97`: Indicates chain failure on `GSPROD`.
     - `98`: Indicates chain failure on `GSTABL`.
     - `99`: Indicates chain failure on `GSCNTR1`.

2. **One-Time Setup (ONETIM Subroutine)**:
   - **Condition**: `ONCE = 0` (Line 0043).
   - **Steps** (Lines post-0043):
     - Increments `ONCE` to 1 to prevent re-execution (Line in `ONETIM`).
     - Sets `PRODCL` to 'PRODCL' (Line in `ONETIM`, used as a key type for `GSTABL`).
     - The commented `PRODCD` line suggests a prior version used a different key type.

3. **Main Processing Loop** (Lines 0043–end of C-specs):
   - **Condition**: Indicator 02 on skips to `END` tag, terminating the program (Line 0046).
   - **Processing for Indicator 01** (Line 0047):
     - **Container Type Lookup** (Lines JK01):
       - Chains `GSCNTR1` using `BDCNTR` (container code, Line JK01).
       - If found (`N99`), moves `TCCNTY` (container type, 'B' or 'P') to `CNTY` (Line JK01).
     - **Product Class and Sort Code Lookup**:
       - Builds a 6-character key (`PRK6`) combining `BDCO` (company number) and `BDPROD` (product code, Lines post-JK01).
       - Moves `PRK6` to `KLPROD` (Line JK02).
       - Chains `GSPROD` using `KLPROD` to get `TPPRCL` (product class, Line JK02).
       - If found (`N97`):
         - Builds a 5-character key (`KEY5`) combining `BDCO` and `TPPRCL` (Lines post-JK02).
         - Chains `GSTABL` using `KEY5` with `PRODCL` as the type to get `TBCSRT` (customer sort code, Lines post-JK02).
         - If found (`N98`), moves `TBCSRT` to `CSRT` (Line post-JK02).
     - **Output Update**:
       - Updates the `BB713M` record with:
         - `CNTY` (position 521, bulk/packaged indicator).
         - `CSRT` (position 523, customer sort code).
       - If either chain fails (`97` or `98`), no sort code is updated, but `CNTY` may still be set if `GSCNTR1` chain succeeds.

4. **Program Termination**:
   - The `END` tag is reached when indicator 02 is set or all records are processed, ending the program.

### Business Rules

1. **Container Type Assignment**:
   - For each `BB713M` record, the container code (`BDCNTR`) is used to look up the container type (`TCCNTY`, 'B' for bulk or 'P' for packaged) in `GSCNTR1`.
   - If found, `TCCNTY` is written to position 521 (`CNTY`) in `BB713M`.

2. **Sort Code Assignment**:
   - The product code (`BDPROD`) and company number (`BDCO`) are used to look up the product class (`TPPRCL`) in `GSPROD`.
   - If found, `TPPRCL` and `BDCO` are used to look up the customer sort code (`TBCSRT`) in `GSTABL` with type 'PRODCL'.
   - If found, `TBCSRT` is written to position 523 (`CSRT`) in `BB713M`.
   - If either lookup fails, no sort code is assigned.

3. **Record Processing**:
   - Only non-deleted records (`BDDEL ≠ 'D'`) are processed (implied by `10NC9` in `BB713M` input spec).
   - Records with missing `GSCNTR1` or `GSPROD`/`GSTABL` entries may receive partial updates (`CNTY` but not `CSRT`).

4. **One-Time Initialization**:
   - The `ONETIM` subroutine runs once to set `PRODCL = 'PRODCL'`, ensuring the correct key type for `GSTABL` lookups.

### Tables (Files) Used

- **Input Files**:
  - `BB713M`: Update file (530 bytes), contains order data to be enhanced.
    - Fields: `BDDEL` (1), `BDCO` (2–3), `BDORD#` (4–9), `BDSEQ` (10–12), `BDLOC` (22–24), `BDPROD` (25–28), `BDTANK` (29–32), `BDQTY` (35–41), `BDPRCE` (42–50), `BDUM` (51–53), `BDCSTK` (54–73), `BDDESC` (75–104), `BDCNTR` (121–123).
  - `GSTABL`: Input file (256 bytes, indexed with 12 keys), contains table data for sort codes.
    - Fields: `TBDEL` (1), `TBTYPE` (2–7), `TBCODE` (8–13), `TBPRGP` (89–90), `TBICOL` (94–95), `TBPRCL` (127–129), `TBSELL` (155), `TBCSRT` (178–179).
  - `GSCNTR1`: Input file (512 bytes, indexed with 3 keys), contains container type data.
    - Field: `TCCNTY` (180, 'B' or 'P').
  - `GSPROD`: Input file (512 bytes, indexed with 6 keys), contains product data.
    - Fields: `TPDEL` (1), `TPPRCL` (127–129).

- **Output File**:
  - `BB713M`: Updated with:
    - `CNTY` (position 521, container type 'B' or 'P').
    - `CSRT` (position 523, customer sort code).

### External Programs Called

- **None**: The `BB714A` program does not explicitly call other programs. It performs all processing internally using file I/O and exception outputs.

### Notes

- **Integration with OCL**: The `BB715.ocl36.txt` calls `BB714A` after `BB714` to add sort codes to `BB713M`, preparing it for sorting by `#GSORT` in the next step.
- **Change History**:
  - 04/18/16 (JK01): Replaced `GSTABL` for container type with `GSCNTR1` (alpha key).
  - 01/18/23 (JK02): Replaced `GSTABL` for product code with `GSPROD` for product class lookup.
- **Error Handling**: Uses indicators 97, 98, and 99 to handle chain failures. Missing records in `GSCNTR1` or `GSPROD`/`GSTABL` result in partial or no updates for the affected fields.
- **Sort Code Purpose**: The `CNTY` (bulk/packaged) and `CSRT` (customer sort code) fields are added to support the sorting criteria in `#GSORT` (e.g., positions 521 and 522–523).

### Summary

The `BB714A` RPG program enhances `BB713M` records by adding container type (`CNTY`) from `GSCNTR1` and customer sort code (`CSRT`) from `GSTABL`, based on product class (`TPPRCL`) from `GSPROD`. It processes non-deleted records, updates positions 521 and 523, and relies on chained lookups. The program uses `BB713M`, `GSTABL`, `GSCNTR1`, and `GSPROD` files, and calls no external programs.

**Files Used**: `BB713M`, `GSTABL`, `GSCNTR1`, `GSPROD`.
**External Programs**: None.

If you need further details (e.g., field formats, subsequent programs like `#GSORT`), or additional files, let me know!