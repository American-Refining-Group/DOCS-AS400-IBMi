The provided document, `BB953B.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is part of the pricing generation process for blended lubes, invoked by `PRICES.ocl36.txt` within the `PRICEGEN.clp` workflow, following `BI942E`. The script orchestrates the generation of a rack price report for the latest product prices within a selected date range, involving multiple programs, file operations, and a sort. Below, I will explain the process steps, business rules, external programs called, and tables (files) used.

### Process Steps of the OCL Script

1. **Set System Switch**:
   - `// SWITCH 00100000`: Sets the System/36 switch settings, specifically setting switch 3 to `1` (bit 3 in an 8-bit switch register). This controls conditional logic for subsequent `LOCAL OFFSET` commands.

2. **Conditional Local Data Setup (if SWITCH3-1)**:
   - If switch 3 is set to `1` (as it is), the script sets memory locations with specific data:
     - `LOCAL OFFSET-101,DATA-'10LALL '`: Sets a parameter for report filtering or processing mode.
     - `LOCAL OFFSET-122,DATA-'ALL'`: Sets a filter for all locations.
     - `LOCAL OFFSET-155,DATA-'ALL'`: Sets a filter for all products or classes.
     - `LOCAL OFFSET-173,DATA-'ALL'`: Sets a filter for all containers or another category.
     - `LOCAL OFFSET-216,DATA-'SEL010123123179N015020240221'`: Sets a complex parameter, likely including selection criteria, date ranges (e.g., `20240221` for February 21, 2024), and other flags.
     - `LOCAL OFFSET-509,DATA-'1980'`: Sets a year or code, possibly for Y2K or reference purposes.

3. **Run GSY2K Procedure**:
   - `// GSY2K`: Calls the System/36 procedure `GSY2K`, which likely initializes the environment, sets system parameters, or handles Y2K date configurations (also used in `PRICEGEN.clp`, `BI944B`, and `BI942E`).

4. **Load and Run BB953B Program**:
   - `// LOAD BB953B`: Loads the `BB953B` program.
   - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Specifies the contract file (`?9?BICONT`, e.g., `ABICONT`), opened in shared read mode with record locking.
   - `// RUN`: Executes `BB953B`, which likely performs initial processing or validation of contract data for rack pricing.

5. **Clear RKPRCE File**:
   - `CLRPFM FILE(?9?RKPRCE)`: Clears the rack price output file (`?9?RKPRCE`, e.g., `ARKPRCE`), preparing it for new data.

6. **Delete Temporary Files**:
   - `// GSDELETE BB953X,BB953P,BB9531,BB953S,BB9538,BB953U,BB953T,,?9?`: Deletes temporary files (e.g., `ABB953X`, `ABB953P`, etc.) to ensure a clean state.
   - `// GSDELETE BB9532,,,,,,,,?9?`: Deletes the temporary file `?9?BB9532`.

7. **Create Temporary File BB9532**:
   - Conditional logic based on the `?9?` parameter:
     - `IF ?9?/G CRTPF FILE(QS36F/?9?BB9532) RCDLEN(169) SIZE(*NOMAX)`: If `?9? ≠ 'G'`, creates `?9?BB9532` (e.g., `ABB9532`) in library `QS36F` with a record length of 169 bytes and unlimited size.
     - `IFF ?9?/G CRTPF FILE(QS36FTEST/?9?BB9532) RCDLEN(169) SIZE(*NOMAX)`: If `?9? = 'G'`, creates `?9?BB9532` (e.g., `GBB9532`) in library `QS36FTEST`.

8. **Build Temporary File (BB9531)**:
   - `// LOAD BB9531`: Loads the `BB9531` program.
   - Files specified:
     - `BBPRCE,LABEL-?9?BBPRCE,DISP-SHRRM`: Pricing history file, shared read with record locking.
     - `GSTABL,LABEL-?9?GSTABL,DISP-SHRRM`: Table file for product class/division, shared read with record locking.
     - `GSPROD,LABEL-?9?GSPROD,DISP-SHRRM`: Product file, shared read with record locking.
     - `BB9531,LABEL-?9?BB9531,RECORDS-999000,EXTEND-999000`: Temporary output file with capacity for 999,000 records.
   - `// RUN`: Executes `BB9531`, which likely builds `?9?BB9531` with pricing data from `?9?BBPRCE`, enriched with product and table data.

9. **Process Temporary File (BB9534)**:
   - `// LOAD BB9534`: Loads the `BB9534` program.
   - Files specified:
     - `BB9531,LABEL-?9?BB9531,DISP-SHR`: Input temporary file, shared mode.
     - `BB9532,LABEL-?9?BB9532,DISP-SHR`: Output temporary file, shared mode.
   - `// RUN`: Executes `BB9534`, which processes `?9?BB9531` to produce `?9?BB9532`, likely performing additional calculations or filtering.

10. **Sort Temporary File**:
    - `// LOAD #GSORT`: Loads the System/36 sort utility.
    - Files specified:
      - `INPUT,LABEL-?9?BB9532,DISP-SHRRM`: Input file (`?9?BB9532`), shared read with record locking.
      - `OUTPUT,LABEL-?9?BB953S,RECORDS-999000,EXTEND-999000,RETAIN-T`: Output file (`?9?BB953S`), temporary with 999,000 record capacity, retained.
    - Sort specification (`HSORTR`):
      - `20A`: Sorts in ascending order on 20 fields.
      - `3X 169 N`: Processes records of 169 bytes, no additional options.
      - `I C 1 1NECD`: Excludes records where position 1 ≠ `'D'` (non-deleted records).
      - Sort fields:
        - `FNC 1 2`: Company number.
        - `FNC 3 5`: Location.
        - `FNC 163 164`: Likely division or another field.
        - `FNC 165 167`: Likely container code.
        - `FNC 8 11`: Likely product code.
        - `FNC 12 14`: Likely additional key field.
        - `FNC 15 17`: Likely another key field.
      - `FDC 1 169`: Includes all 169 bytes in the output.
    - `// RUN`: Executes the sort, producing `?9?BB953S`.

11. **Print Rack Price Report (BB953)**:
    - `// LOAD BB953`: Loads the `BB953` program.
    - Files specified:
      - `BB9531,LABEL-?9?BB953S`: Sorted temporary file as input.
      - `BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Contract file, shared read with record locking.
      - `INLOC,LABEL-?9?INLOC,DISP-SHRRM`: Location file, shared read with record locking.
      - `GSTABL,LABEL-?9?GSTABL,DISP-SHRRM`: Table file, shared read with record locking.
      - `OUTFILE,LABEL-?9?RKPRCE,DISP-SHR`: Output rack price file, shared mode.
      - `GSCNTR1,LABEL-?9?GSCNTR1,DISP-SHR`: Container file, shared mode.
    - `// RUN`: Executes `BB953`, which generates the rack price report, writing to `?9?RKPRCE` using data from `?9?BB953S` and other files.

12. **Cleanup Temporary Files**:
    - `// GSDELETE BB953X,BB953P,BB9531,BB953S,BB9538,BB953U,BB953T,,?9?`: Deletes temporary files.
    - `// GSDELETE BB9532,,,,,,,,?9?`: Deletes `?9?BB9532`.

### Business Rules (Inferred)

1. **Purpose**: Generates a rack price report for the latest product prices for blended lubes within a selected date range, processing pricing data and producing `?9?RKPRCE`.
2. **Date Range Filtering**: The `LOCAL OFFSET-216` data (`SEL010123123179N015020240221`) likely specifies a date range (e.g., up to February 21, 2024) for selecting relevant pricing records.
3. **File Preparation**:
   - Clears `?9?RKPRCE` to ensure a fresh output.
   - Deletes temporary files to prevent data conflicts.
   - Creates `?9?BB9532` dynamically based on the `?9?` parameter.
4. **Processing Stages**:
   - `BB953B`: Initializes or validates contract data.
   - `BB9531`: Builds initial pricing data in `?9?BB9531`.
   - `BB9534`: Processes `?9?BB9531` into `?9?BB9532`.
   - `#GSORT`: Sorts `?9?BB9532` into `?9?BB953S` by company, location, division, container, and product.
   - `BB953`: Generates the final report in `?9?RKPRCE`.
5. **Context**: Invoked by `PRICES.ocl36.txt`, processes `?9?PRSABLW` (from `BI942E`) to produce rack price output, complementing the pricing workflow.

### External Programs Called

1. **GSY2K**: System/36 procedure for environment initialization or Y2K handling.
2. **BB953B**: Initial processing or validation of contract data.
3. **BB9531**: Builds temporary pricing file `?9?BB9531`.
4. **BB9534**: Processes `?9?BB9531` into `?9?BB9532`.
5. **#GSORT**: System/36 sort utility for sorting `?9?BB9532` into `?9?BB953S`.
6. **BB953**: Generates the final rack price report in `?9?RKPRCE`.

### Tables (Files) Used

All files use dynamic labels with `?9?` replaced by a parameter (e.g., `'A'` for `ABICONT`):
1. **BICONT** (`?9?BICONT`): Contract file, shared read with record locking.
2. **BBPRCE** (`?9?BBPRCE`): Pricing history file, shared read with record locking.
3. **GSTABL** (`?9?GSTABL`): Table file for product class/division, shared read with record locking.
4. **GSPROD** (`?9?GSPROD`): Product file, shared read with record locking.
5. **BB9531** (`?9?BB9531`): Temporary file for initial pricing data, 999,000 records.
6. **BB9532** (`?9?BB9532`): Temporary file for processed pricing data, 169 bytes.
7. **BB953S** (`?9?BB953S`): Sorted temporary file, 999,000 records.
8. **INLOC** (`?9?INLOC`): Location file, shared read with record locking.
9. **RKPRCE** (`?9?RKPRCE`): Output rack price file, shared mode.
10. **GSCNTR1** (`?9?GSCNTR1`): Container file, shared mode.

### Additional Notes

- **Context**: Invoked by `PRICES.ocl36.txt` after `BI942E`, processes pricing data from `?9?PRSABLW` to generate a rack price report, complementing the final pricing output.
- **System/36 Environment**: Uses OCL and System/36 utilities, likely on AS/400.
- **Limitations**: Without RPG source code for `BB953B`, `BB9531`, `BB9534`, or `BB953`, exact logic is inferred. They likely handle pricing extraction, transformation, and reporting.
- **Error Handling**: Relies on System/36 and called programs for error handling.

If you have the RPG source code for `BB953B`, `BB9531`, `BB9534`, `BB953`, or the `SA505*` programs, I can provide further details. Let me know if you have additional questions or files!