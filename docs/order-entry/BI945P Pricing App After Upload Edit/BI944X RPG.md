The provided document, `BI944X.rpg.txt`, is an RPG III program used within the context of the `BI9445.ocl36.txt` OCL program on an IBM midrange system (e.g., AS/400 or iSeries). The program updates the `BICUAG` file with salesman, product class, and division data, ensuring that only unexpired customer sales agreements are processed. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the RPG Program

The `BI944X` RPG program processes records from the `BICUAG` file, retrieves related data from `ARCUST` and `GSTABL`, and writes updated records to `BICUAGNW` (output as `BICUAGX` per the OCL). The program filters out expired agreements and appends salesman, product class, and division information. Here’s a detailed breakdown of the process steps:

1. **Program Initialization**:
   - **File Definitions**:
     - `FBICUAG IP F 256 DISK`: Defines `BICUAG` as the primary input file with a record length of 256 bytes.
     - `FGSTABL IF F 256 12AI 2 DISK`: Defines `GSTABL` as an input file with a 256-byte record length, indexed access (`12AI`), and key starting at position 2.
     - `FARCUST IF F 384 8AI 2 DISK`: Defines `ARCUST` as an input file with a 384-byte record length, indexed access (`8AI`), and key starting at position 2.
     - `FBICUAGNW O F 263 DISK`: Defines `BICUAGNW` as the output file with a 263-byte record length.
   - **Input Record Definitions**:
     - `BICUAG`:
       - `BACOCU` (positions 2–9): Company and customer number.
       - `BACONO` (positions 2–3): Company number.
       - `BACUST` (positions 4–9): Customer number.
       - `BAPR01` (positions 13–16): Product code.
       - `BAEND8` (positions 126–133, numeric): End date in CYMD format (e.g., CCYYMMDD).
       - `RECORD` (positions 1–256): Entire record.
     - `GSTABL`:
       - `TBDESC`, `TBDES2` (positions 14–43, 22–43): Table descriptions.
       - `TBPRCL` (positions 127–129): Product class code.
       - `TBABDS` (positions 145–154): Short description.
       - `TBCSRT` (positions 178–179, numeric): Inventory company sort (division).
     - `ARCUST`:
       - `ARDEL` (position 1): Delete code.
       - `ARSLS#` (positions 263–264, numeric): Salesman number.
     - `UDS` (User Data Structure):
       - `KYDAT8` (positions 433–440, numeric): System date from LDA, used for comparison.
   - Purpose: Sets up input and output files and defines fields for processing.

2. **Main Processing Loop**:
   - `C 01 DO`: Begins processing records from `BICUAG` (indicator `01` is set for each valid input record).
   - **Filter Expired Agreements**:
     - `C SETOF 50`: Turns off indicator `50`, which controls date field output.
     - `C BAEND8 IFNE *ZERO`: Checks if the agreement end date (`BAEND8`) is not zero.
       - `C BAEND8 IFLT KYDAT8`: Compares `BAEND8` to the system date (`KYDAT8`).
       - `C GOTO ENDIT`: If `BAEND8` is less than `KYDAT8` (expired), skips to the `ENDIT` tag, bypassing further processing for that record.
       - `C END`: Ends the inner condition.
     - `C END`: Ends the outer condition.
     - `C BAEND8 IFEQ *ZEROS`: If `BAEND8` is zero (no end date, i.e., perpetual agreement):
       - `C SETON 50`: Sets indicator `50` to force specific date values in the output.
       - `C END`: Ends the condition.
   - Purpose: Filters out expired agreements, including only those with an end date on or after today or with a zero end date (unexpired).

3. **Retrieve Salesman Number**:
   - `C BACOCU CHAINARCUST 99`: Chains (looks up) the `ARCUST` file using the company/customer key (`BACOCU`, positions 2–9).
   - `C N99 MOVELARSLS# SLSMAN 20`: If the record is found (`N99`, not indicator `99`), moves the salesman number (`ARSLS#`) to the `SLSMAN` field (20 characters, though defined as numeric in `ARCUST`).
   - Purpose: Retrieves the salesman number associated with the customer.

4. **Retrieve Product Class and Division**:
   - **Product Code Lookup**:
     - `C MOVEL'PRODCD' PRODKY 12`: Sets the key prefix to 'PRODCD' (product code).
     - `C MOVELBACONO COPROD 6`: Moves the company number (`BACONO`) to `COPROD`.
     - `C MOVE BAPR01 COPROD`: Appends the product code (`BAPR01`) to `COPROD`.
     - `C MOVE COPROD PRODKY`: Combines company and product code into `PRODKY`.
     - `C PRODKY CHAINGSTABL 65`: Chains `GSTABL` using `PRODKY`.
     - `C N65 MOVELTBPRCL PRCL 3`: If found (`N65`), moves the product class code (`TBPRCL`) to `PRCL`.
   - **Product Class Lookup (if necessary)**:
     - `C N65 DO`: If the product code lookup succeeds:
       - `C MOVEL'PRODCL' PRCLKY 12`: Sets the key prefix to 'PRODCL' (product class).
       - `C MOVE PRCL KEY5 5`: Moves the product class (`PRCL`) to `KEY5`.
       - `C MOVELBACONO KEY5`: Prepends the company number to `KEY5`.
       - `C MOVE KEY5 PRCLKY`: Combines into `PRCLKY`.
       - `C PRCLKY CHAINGSTABL 65`: Chains `GSTABL` using `PRCLKY`.
       - `C N65 MOVELTBCSRT DIV 20`: If found, moves the division code (`TBCSRT`) to `DIV`.
       - `C N65 EXCPTPRODCL`: Writes to the output file (`BICUAGNW`) if the product class lookup succeeds.
       - `C END`: Ends the inner `DO`.
     - `C END`: Ends the outer `DO`.
   - Purpose: Retrieves the product class (`PRCL`) and division (`DIV`) from `GSTABL` based on the company and product code.

5. **Output Record**:
   - `OBICUAGNWEADD PRODCL`:
     - Writes a record to `BICUAGNW` with the `PRODCL` exception output.
     - Fields:
       - `RECORD 256`: Writes the entire input record (positions 1–256).
       - `70 '791231'`: If indicator `50` is on (zero end date), writes '791231' (December 31, 1979) to position 70.
       - `133 '20791231'`: If indicator `50` is on, writes '20791231' (December 31, 2079) to position 133.
       - `DIV 258`: Writes the division code to positions 257–258.
       - `PRCL 261`: Writes the product class code to positions 259–261.
       - `SLSMAN B 263`: Writes the salesman number to positions 262–263 (binary format).
   - Purpose: Outputs the updated record with salesman, product class, and division, adjusting dates for perpetual agreements.

6. **End Processing**:
   - `C ENDIT TAG`: Marks the end of processing for each record.
   - The program loops back to process the next `BICUAG` record until the file is exhausted.

---

### Business Rules

1. **Filter Unexpired Agreements**:
   - Only process agreements with:
     - An end date (`BAEND8`) on or after the system date (`KYDAT8`).
     - A zero end date (perpetual agreements).
   - Expired agreements (end date before today) are skipped.
2. **Salesman Assignment**:
   - Retrieve the salesman number (`ARSLS#`) from `ARCUST` using the company/customer key (`BACOCU`).
   - If no customer record is found, the salesman field remains blank.
3. **Product Class and Division Assignment**:
   - Use the company (`BACONO`) and product code (`BAPR01`) to look up the product class (`TBPRCL`) in `GSTABL`.
   - Use the product class and company to look up the division (`TBCSRT`) in `GSTABL`.
   - If either lookup fails, the corresponding fields (`PRCL`, `DIV`) remain blank.
4. **Date Handling for Perpetual Agreements**:
   - For agreements with a zero end date (`BAEND8 = *ZEROS`), set indicator `50` to write default dates:
     - Position 70: '791231' (December 31, 1979).
     - Position 133: '20791231' (December 31, 2079).
5. **Output Record Structure**:
   - Retain the original 256-byte record.
   - Append division (2 bytes), product class (3 bytes), and salesman (2 bytes, binary).

---

### Tables Used

The program uses the following files/tables:
1. **BICUAG**:
   - Primary input file containing customer sales agreement data.
   - Fields:
     - `BACOCU` (2–9): Company/customer key.
     - `BACONO` (2–3): Company number.
     - `BACUST` (4–9): Customer number.
     - `BAPR01` (13–16): Product code.
     - `BAEND8` (126–133, numeric): End date (CYMD).
     - `RECORD` (1–256): Entire record.
2. **GSTABL**:
   - Reference table for product and product class data.
   - Fields:
     - `TBDESC`, `TBDES2` (14–43, 22–43): Descriptions.
     - `TBPRCL` (127–129): Product class code.
     - `TBABDS` (145–154): Short description.
     - `TBCSRT` (178–179, numeric): Division code.
3. **ARCUST**:
   - Customer master file.
   - Fields:
     - `ARDEL` (1): Delete code.
     - `ARSLS#` (263–264, numeric): Salesman number.
4. **BICUAGNW**:
   - Output file (aliased as `BICUAGX` in the OCL).
   - Contains the input record (256 bytes) plus:
     - Division (257–258).
     - Product class (259–261).
     - Salesman (262–263, binary).
   - Record length: 263 bytes.

---

### External Programs Called

The `BI944X` RPG program does not call any external programs (no `CALL` or `CALLP` operations). It operates solely on the defined input and output files, performing lookups (`CHAIN`) and writing output records.

---

### Summary

The `BI944X` RPG program updates the `BICUAG` file by adding salesman, product class, and division data, writing the results to `BICUAGNW` (aliased as `BICUAGX`). It filters out expired agreements, retrieves salesman numbers from `ARCUST`, and product class/division from `GSTABL`, and handles perpetual agreements with default dates.

**Process Steps**:
1. Initialize files (`BICUAG`, `GSTABL`, `ARCUST`, `BICUAGNW`) and fields.
2. For each `BICUAG` record:
   - Skip if the end date is before today and non-zero.
   - Set indicator `50` for zero end dates.
   - Retrieve salesman number from `ARCUST` using `BACOCU`.
   - Retrieve product class and division from `GSTABL` using company/product code and product class.
   - Write the updated record to `BICUAGNW` with division, product class, salesman, and adjusted dates (if applicable).
3. Loop until all records are processed.

**Business Rules**:
1. Include only unexpired agreements (end date ≥ today or zero).
2. Assign salesman from `ARCUST` based on company/customer.
3. Assign product class and division from `GSTABL` based on company/product code.
4. For perpetual agreements, use default dates ('791231', '20791231').
5. Output the original record plus division (2 bytes), product class (3 bytes), and salesman (2 bytes).

**Tables Used**:
- Input: `BICUAG`, `GSTABL`, `ARCUST`.
- Output: `BICUAGNW` (aliased as `BICUAGX`).

**External Programs Called**:
- None.

If you provide the RPG source for `BI9445` or additional details about the file structures, I can further clarify the integration with the overall process described in `BI9445.ocl36.txt`.