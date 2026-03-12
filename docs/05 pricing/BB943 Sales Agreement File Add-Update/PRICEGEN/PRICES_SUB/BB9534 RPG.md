The provided document, `BB9534.rpg.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `BB9534`. It is invoked by the OCL script `BB953B.ocl36.txt` as part of the rack pricing process within the `PRICES.ocl36.txt` workflow, which is itself called by `PRICEGEN.clp`. The program filters records from the temporary file `BB9531` (produced by `BB9531`) based on selection criteria (division, location, product class, container, and product code) and writes valid records to another temporary file, `BB9532`, for further processing in the rack price reporting sequence. Below, I will explain the process steps, business rules, tables used, and external programs called.

### Process Steps of the RPG Program

1. **File Definitions**:
   - **Input File**:
     - `BB9531` (Input Primary, `IP`, 169 bytes, disk): Temporary file, mapped to `?9?BB9531` (e.g., `ABB9531`), containing preprocessed pricing data from `BB9531`.
   - **Output File**:
     - `BB9532` (Output, `O`, 169 bytes, disk): Temporary file, mapped to `?9?BB9532` (e.g., `ABB9532`), for filtered pricing data.
   - **Note**: The file definitions were updated in revision JB01 (09/19/12) to reflect a record length increase from 167 to 169 bytes to accommodate new fields (`RKRKRQ` and `RKINAC`).

2. **Extension Specifications**:
   - Arrays for filtering:
     - `LOC` (5 elements, 3 bytes): Stores location codes (`KYLOC1` to `KYLOC5`).
     - `CLS` (10 elements, 3 bytes): Stores product class codes (`KYPC01` to `KYPC10`).
     - `CTR` (5 elements, 3 bytes): Stores container codes (`KYCT01` to `KYCT05`).
     - `PROD` (10 elements, 4 bytes): Stores product codes (`KYPD01` to `KYPD10`).

3. **Input Specifications**:
   - **BB9531** (record type `NS 01`):
     - `RKCONO` (1–2): Company number.
     - `RKLOC` (3–5): Location.
     - `RKPROD` (8–11): Product code.
     - `RKCNTR` (12–14): Container code.
     - `RKDIV` (163–164): Division code.
     - `RKPRCL` (165–167): Product class code.
     - `RKRKRQ` (168): No rack price required flag (added per JB01).
     - `RKINAC` (169): Inactive flag (`b`, `I`, or `B`, added per JB01).
     - `RECORD` (1–169): Entire 169-byte record.
   - **User Data Structure (UDS)**:
     - `KYDIV` (103): Division filter ('ALL' or 'CO').
     - `KYLOSL` (104–106): Location selection ('SEL' for specific locations).
     - `KYLOC1`–`KYLOC5` (107–121): Location codes (3 bytes each).
     - `LOC` (107–121): Array of location codes.
     - `KYPCSL` (122–124): Product class selection ('SEL' for specific classes).
     - `KYPC01`–`KYPC10` (125–154): Product class codes (3 bytes each).
     - `CLS` (125–154): Array of product class codes.
     - `KYCTSL` (155–157): Container selection ('SEL' for specific containers).
     - `KYCT01`–`KYCT05` (158–172): Container codes (3 bytes each).
     - `CTR` (158–172): Array of container codes.
     - `KYPDSL` (173–175): Product code selection ('SEL' for specific products).
     - `KYPD01`–`KYPD10` (176–215): Product codes (4 bytes each).
     - `PROD` (176–215): Array of product codes.
     - `KYDTSL` (216–218): Date selection ('SEL' for date range).
     - `KYFRDT` (219–224): From date (YYMMDD).
     - `KYTODT` (225–230): To date (YYMMDD).
     - `KYJOBQ` (231): Job queue indicator.
     - `KYCOPY` (232–233): Copy number.
     - `KYDVNO` (234–235): Division number.
     - `KYDAT8` (236–243): Key date (CYMD, set by `BB953B`).

4. **Calculation Specifications**:
   - **Main Loop**:
     - Processes each `BB9531` record (`01 DO`, implied).
   - **Indicator Setup**:
     - `SETOF 5051`: Clears indicators `50` and `51`.
     - `SETON 52`: Sets indicator `52` (assume record is valid unless filtered out).
   - **Division Filter**:
     - If `KYDIV ≠ 'ALL'`, checks if `RKDIV` matches the specified division (logic not shown due to truncation, likely sets `51` if no match).
   - **Location Filter** (`LOCSR` subroutine):
     - If `KYLOSL = 'SEL'`:
       - Loops through `LOC` array (5 elements).
       - If `RKLOC` matches any `KYLOC1`–`KYLOC5`, sets `50`; otherwise, sets `51` (invalid record).
     - If `KYLOSL ≠ 'SEL'`, skips location check.
   - **Product Class Filter** (`PRCLSR` subroutine):
     - If `KYPCSL = 'SEL'`:
       - Loops through `CLS` array (10 elements).
       - If `RKPRCL` matches any `KYPC01`–`KYPC10`, sets `50`; otherwise, sets `51`.
     - If `KYPCSL ≠ 'SEL'`, skips product class check.
   - **Container Filter** (`CNTRSR` subroutine):
     - If `KYCTSL = 'SEL'`:
       - Loops through `CTR` array (5 elements).
       - If `RKCNTR` matches any `KYCT01`–`KYCT05`, sets `50`; otherwise, sets `51`.
     - If `KYCTSL ≠ 'SEL'`, skips container check.
   - **Product Code Filter** (`PRODSR` subroutine):
     - If `KYPDSL = 'SEL'`:
       - Loops through `PROD` array (10 elements).
       - If `RKPROD` matches any `KYPD01`–`KYPD10`, sets `50`; otherwise, sets `51`.
     - If `KYPDSL ≠ 'SEL'`, skips product code check.
   - **Output Decision**:
     - If `50` (valid record) or `52` (no filtering applied), executes `EXCPTGOOD` to write the entire `RECORD` (169 bytes) to `BB9532`.
     - If `51` (invalid record), skips to `END` tag, omitting the record.
   - **Subroutines**:
     - `LOCSR`, `PRCLSR`, `CNTRSR`, `PRODSR`: Validate location, product class, container, and product code, respectively, setting indicators to control output.

5. **Output Specifications**:
   - `BB9532` (`EADD`, `GOOD`):
     - `RECORD` (1–169): Writes the entire 169-byte input record, including company, location, product, container, division, product class, and flags (`RKRKRQ`, `RKINAC`).

### Business Rules

1. **Purpose**: Filters pricing records from `BB9531` based on division, location, product class, container, and product code criteria, writing valid records to `BB9532` for sorting and reporting in the rack pricing process.
2. **Filtering Logic**:
   - **Division**: If `KYDIV ≠ 'ALL'`, includes records matching the specified division (`RKDIV`).
   - **Location**: If `KYLOSL = 'SEL'`, includes records where `RKLOC` matches one of `KYLOC1`–`KYLOC5`.
   - **Product Class**: If `KYPCSL = 'SEL'`, includes records where `RKPRCL` matches one of `KYPC01`–`KYPC10`.
   - **Container**: If `KYCTSL = 'SEL'`, includes records where `RKCNTR` matches one of `KYCT01`–`KYCT05`.
   - **Product Code**: If `KYPDSL = 'SEL'`, includes records where `RKPROD` matches one of `KYPD01`–`KYPD10`.
   - A record is written to `BB9532` only if all applied filters pass (`50` on) or no filters are applied (`52` on).
3. **Data Preservation**: Valid records are copied unchanged (169 bytes, including `RKRKRQ` and `RKINAC` flags added per JB01).
4. **Context**: Follows `BB9531` in the `BB953B.ocl36.txt` workflow, producing `?9?BB9532` for sorting by `#GSORT` and final reporting by `BB953`.
5. **Default Behavior**: If no selection criteria are specified (`KYDIV = 'ALL'`, `KYLOSL`, `KYPCSL`, `KYCTSL`, `KYPDSL ≠ 'SEL'`), all records are written (`52` on).

### Tables (Files) Used

1. **BB9531** (`?9?BB9531`):
   - **Access**: Input Primary (`IP`), shared mode (`DISP-SHR` from OCL).
   - **Purpose**: Source of preprocessed pricing data from `BB9531`.
2. **BB9532** (`?9?BB9532`):
   - **Access**: Output (`O`), shared mode (`DISP-SHR`).
   - **Purpose**: Destination for filtered pricing records.

### External Programs Called

- **None**: The `BB9534` RPG program does not call external programs or subroutines. It performs internal filtering using subroutines (`LOCSR`, `PRCLSR`, `CNTRSR`, `PRODSR`).

### Additional Notes

- **Context with BB953B.ocl36.txt**: Invoked after `BB9531`, processing `?9?BB9531` to produce `?9?BB9532`, which is sorted by `#GSORT` into `?9?BB953S` for `BB953`.
- **Revisions**:
  - JB01 (09/19/12): Increased record length to 169 bytes, added `RKRKRQ` and `RKINAC` fields.
- **System/36 Environment**: Uses RPG II/III syntax, likely on AS/400.
- **Truncation**: The document is truncated, but the provided code includes all critical sections (`F`, `E`, `I`, `C`, `O`), sufficient to understand the filtering logic.
- **Error Handling**: Relies on indicators (`50`, `51`, `52`) and System/36 environment for error handling.
- **Similarity to BI9444**: The program’s structure mirrors `BI9444.rpg.txt`, indicating a common pattern for filtering temporary files in the pricing workflow.

If you have the RPG source code for `BB953`, `SA505C`, or other `SA505*` programs, or need further analysis, please provide those details! Let me know if you have additional questions or files to share.