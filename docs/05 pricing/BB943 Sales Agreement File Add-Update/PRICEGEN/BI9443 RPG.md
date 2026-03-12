The provided document, `BI9443.rpg.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `BI9443`. It is invoked by the OCL script `BI944B.ocl36.txt` as part of the pricing generation process for blended lubes, initiated by `PRICEGEN.clp`. The program updates the `BICUAG` file with salesman, product class, and division data, incorporating logic to include only unexpired agreements based on end dates. Below, I will explain the process steps, business rules, tables used, and external programs called.

### Process Steps of the RPG Program

1. **File Definitions**:
   - The `F` specifications define the following files:
     - `BICUAG` (Input Primary, `IP`, 256 bytes, disk): Primary input file for pricing agreements.
     - `GSTABL` (Input, `IF`, 256 bytes, indexed with 12-byte key, disk): Table file for product class and division data.
     - `ARCUST` (Input, `IF`, 384 bytes, indexed with 8-byte key, disk): Customer file for salesman data.
     - `GSPROD` (Input, `IF`, 512 bytes, indexed with 6-byte key, disk): Product file for product class data (added per revision JK01).
     - `BICUAGNW` (Output, `O`, 263 bytes, disk): Output file for updated pricing records.

2. **Input Specifications**:
   - `BICUAG` (record type `NS 01`):
     - `BACOCU` (2–9): Concatenation of company (`BACONO`, 2–3) and customer number (`BACUST`, 4–9).
     - `BACONO` (2–3): Company number.
     - `BACUST` (4–9): Customer number.
     - `BAPR01` (13–16): Product code 1.
     - `BAEND8` (126–133): End date in CYMD format (CCYYMMDD).
     - `RECORD` (1–256): Entire record for output.
   - `GSTABL` (record type `NS`):
     - `TBDESC` (14–43): Table description.
     - `TBDES2` (22–43): Alternate table description.
     - `TBPRCL` (127–129): Product class code.
     - `TBABDS` (145–154): Short description.
     - `TBCSRT` (178–179): Inventory company sort (division code).
   - `ARCUST` (record type `NS`):
     - `ARDEL` (1): Delete code.
     - `ARSLS#` (263–264): Salesman number.
   - `GSPROD` (record type `NS`, added per JK01):
     - `TPDEL` (1): Delete code.
     - `TPDESC` (14–43): Complete description.
     - `TPDES1` (14–33): Partial description.
     - `TPPRGP` (89–90): Product group code.
     - `TPPRCL` (127–129): Product class code.
     - `TPABDS` (145–154): Short description.
     - `TPFLCD` (252): Field code.
   - User Data Structure (`UDS`):
     - `KYDAT8` (333–340): 8-digit key date (CYMD format, likely set by `BI944A`).

3. **Calculation Specifications (Processing Logic)**:
   - **Main Loop**:
     - `01 DO`: Processes each record in `BICUAG` (primary file).
   - **Filter Expired Agreements** (revision JB01):
     - `BAEND8 IFNE *ZERO`: Checks if the end date is non-zero.
     - `BAEND8 IFLT KYDAT8`: Checks if the end date is earlier than the key date (`KYDAT8`, today’s date in CYMD format).
     - If both conditions are true (expired agreement), `GOTO ENDIT` skips the record.
     - `BAEND8 IFEQ *ZEROS`: If the end date is zero (no expiration), sets indicator `50` to mark the record as unexpired.
   - **Salesman Lookup**:
     - `BACOCU CHAINARCUST 99`: Chains (looks up) the customer record in `ARCUST` using `BACOCU` (company + customer number).
     - `N99 MOVELARSLS# SLSMAN 20`: If found (`N99`, not in error), moves the salesman number (`ARSLS#`) to `SLSMAN` (2 digits).
   - **Product Class and Division Lookup** (revision JK01):
     - `MOVELBACONO COPROD 6`: Moves company number to `COPROD`.
     - `MOVE BAPR01 COPROD`: Appends product code 1 to `COPROD`, forming a 6-byte key (company + product code).
     - `KLPROD CHAINGSPROD 65`: Chains `GSPROD` using `KLPROD` (same as `COPROD`) to find product class.
     - `N65 MOVELTPPRCL PRCL 3`: If found (`N65`), moves product class code (`TPPRCL`) to `PRCL` (3 digits).
     - If not found, enters a secondary lookup:
       - `MOVEL'PRODCL' PRCLKY 12`: Sets `PRCLKY` to `'PRODCL'`.
       - `MOVE PRCL KEY5 5`: Moves `PRCL` to `KEY5`.
       - `MOVELBACONO KEY5`: Prepends company number to `KEY5`.
       - `MOVE KEY5 PRCLKY`: Updates `PRCLKY` with the new key.
       - `PRCLKY CHAINGSTABL 65`: Chains `GSTABL` to find division data.
       - `N65 MOVELTBCSRT DIV 20`: If found, moves division code (`TBCSRT`) to `DIV` (2 digits).
       - `N65 EXCPTPRODCL`: If found, writes the updated record (via exception output).
   - **End of Record Processing**:
     - `ENDIT TAG`: Marks the end of processing for each record, skipping expired agreements.

4. **Output Specifications**:
   - `BICUAGNW` (exception output `EADD`, `PRODCL`):
     - `RECORD 256`: Writes the entire input record (1–256).
     - If indicator `50` is on (unexpired agreement with zero end date):
       - Position 70: Writes `'791231'` (December 31, 1979, likely a default start date).
       - Position 133: Writes `'20791231'` (December 31, 2079, likely a default end date).
     - `DIV 258`: Writes division code (2 digits) to positions 257–258.
     - `PRCL 261`: Writes product class code (3 digits) to positions 259–261.
     - `SLSMAN B 263`: Writes salesman number (2 digits) to positions 262–263, only if non-blank.

### Business Rules

1. **Purpose**: The program updates the `BICUAGNW` file (mapped to `?9?BICUAGX`) with salesman number, product class, and division for unexpired pricing agreements from `BICUAG`, ensuring only valid agreements are processed.
2. **Agreement Filtering** (JB01 revision):
   - Include agreements where:
     - `BAEND8` = 0 (no expiration date).
     - `BAEND8` ≥ `KYDAT8` (end date is today or later).
   - Exclude agreements where `BAEND8` < `KYDAT8` (expired).
   - For agreements with `BAEND8` = 0, set default dates:
     - Start date: `791231` (1979-12-31).
     - End date: `20791231` (2079-12-31).
3. **Salesman Assignment**:
   - Use `BACOCU` (company + customer number) to look up the salesman number (`ARSLS#`) in `ARCUST`.
   - If found, store in `SLSMAN` and write to `BICUAGNW` (positions 262–263).
4. **Product Class and Division** (JK01 revision):
   - Use company number (`BACONO`) and product code (`BAPR01`) to look up product class (`TPPRCL`) in `GSPROD`.
   - If not found in `GSPROD`, use product class (`PRCL`) and company number to look up division (`TBCSRT`) in `GSTABL`.
   - Write product class to positions 259–261 and division to positions 257–258 in `BICUAGNW`.
5. **Output Record Structure**:
   - Copy the entire `BICUAG` record (256 bytes).
   - Append division (2 bytes), product class (3 bytes), and salesman (2 bytes) to create a 263-byte record.
6. **Context in Pricing Generation**: As part of the `BI944B` process, `BI9443` enriches `?9?BICUAGX` with salesman, product class, and division data, filtering out expired agreements to prepare data for subsequent sorting and processing by `BI9444` and `BI944`.

### Tables (Files) Used

1. **BICUAG**: Primary input file for pricing agreements.
   - **Access**: Input Primary (`IP`).
   - **Label**: `?9?BICUAG` (e.g., `ABICUAG` if `&P$GRP` = `'A'`).
   - **Purpose**: Read pricing agreement records to filter and process.
2. **GSTABL**: Input file for product class and division data.
   - **Access**: Input (`IF`), indexed, shared read mode with record locking (`DISP-SHRRM`).
   - **Label**: `?9?GSTABL` (e.g., `AGSTABL`).
   - **Purpose**: Look up division code (`TBCSRT`) for product class.
3. **ARCUST**: Input file for customer data.
   - **Access**: Input (`IF`), indexed, shared read mode with record locking (`DISP-SHRRM`).
   - **Label**: `?9?ARCUST` (e.g., `AARCUST`).
   - **Purpose**: Look up salesman number (`ARSLS#`) for customer.
4. **GSPROD**: Input file for product data (JK01 revision).
   - **Access**: Input (`IF`), indexed, shared read mode with record locking (`DISP-SHRRM`).
   - **Label**: `?9?GSPROD` (e.g., `AGSPROD`).
   - **Purpose**: Look up product class code (`TPPRCL`) for product code.
5. **BICUAGNW**: Output file for updated records.
   - **Access**: Output (`O`).
   - **Label**: `?9?BICUAGX` (e.g., `ABICUAGX`).
   - **Purpose**: Write enriched records with salesman, product class, and division.

### External Programs Called

- **None**: The `BI9443` RPG program does not explicitly call any external programs or subroutines. It performs internal logic to process and update records.

### Additional Notes

- **Context with BI944B.ocl36**: Invoked by `BI944B.ocl36` after `BI944A`, which sets `KYDAT8`. `BI9443` filters and enriches pricing agreements, writing to `?9?BICUAGX` for sorting and further processing by `BI9444` and `BI944`.
- **Revisions**:
  - **JB01 (12/04/2012)**: Added logic to filter out expired agreements, ensuring only unexpired ones (end date ≥ today or zero) are processed.
  - **JK01 (02/02/23)**: Replaced `GSTABL` with `GSPROD` for product class lookup, improving accuracy for product data.
- **System/36 Environment**: The program uses RPG II/III syntax and operates in a System/36 environment, likely on an AS/400 in compatibility mode.
- **Error Handling**: The program uses indicators (`99`, `65`) to handle lookup failures but lacks explicit error handling for file access issues, relying on the System/36 environment or `BI944B.ocl36` for error management.
- **Data Integrity**: The program ensures only unexpired agreements are processed and enriches records with critical data (salesman, product class, division), maintaining consistency for the pricing workflow.

If you have the RPG source code for `BI9444`, `BI944`, `BI942E`, or `PRICES`, or need further analysis of the pricing generation workflow, please provide those details! Let me know if you have additional questions or files to share.