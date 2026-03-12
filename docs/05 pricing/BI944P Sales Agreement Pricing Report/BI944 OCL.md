The provided document, `BI944.ocl36.txt`, is an Operation Control Language (OCL) program used on IBM midrange systems (e.g., AS/400 or IBM i) to orchestrate the printing of a Customer Sales Agreement Master File Listing. This OCL program is called by the main OCL program (`BI944P.ocl36.txt`) under certain conditions (e.g., when `KYJOBQ = 'Y'`) and works in conjunction with the RPG program `BI944P.rpg36.txt` to process and sort data for the final report. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used.

### Process Steps of the OCL Program

The OCL program `BI944.ocl36.txt` is responsible for preparing, sorting, and printing the Customer Sales Agreement Master File Listing. It involves file manipulation, data sorting, and execution of multiple programs to generate the final output. Here’s a step-by-step breakdown of the process:

1. **Initial File Deletion**:
   - `// GSDELETE BI944S,BI944T,BB203?WS?,,,,,,?9?`: Deletes temporary files `BI944S`, `BI944T`, and `BB203?WS?` from the library specified by `?9?` (a placeholder for the library name, dynamically substituted at runtime). This ensures a clean slate for new data processing.

2. **System Configuration**:
   - `// GSY2K`: Likely a directive or comment indicating Year 2000 compliance or a specific system module. It does not directly impact execution but may set context for date handling.

3. **Local Data Area (LDA) Modifications**:
   - The program modifies the Local Data Area (LDA) based on conditions checked at specific offsets:
     - `// IF ?L'155,3'?/SEL LOCAL OFFSET-4,DATA-'O COAC'`: If the value at LDA offset 155 (3 bytes) is `'SEL'`, sets offset 4 to `'O COAC'`. Otherwise, sets it to `'O*CO*C'`. This likely controls output formatting or selection criteria.
     - `// IF ?L'103,1'?/B LOCAL OFFSET-10,DATA-'I*C'`: If the value at LDA offset 103 (1 byte) is `'B'`, sets offset 10 to `'I*C'`. Otherwise, sets it to `'IAC'`. This may control division-specific processing.
     - `// IF ?L'321,3'?/SEL LOCAL OFFSET-13,DATA-'IAC'`: If the value at LDA offset 321 (3 bytes) is `'SEL'`, sets offset 13 to `'IAC'`. Otherwise, sets it to `'I*C'`. This likely affects salesman selection.
     - `// IFF ?L'358,3'?/ LOCAL OFFSET-16,DATA-'IAC'`: If the value at LDA offset 358 (3 bytes) is blank, sets offset 16 to `'IAC'`. Otherwise, sets it to `'I*C'`. This may relate to unit of measure processing.
   - These LDA modifications configure parameters for subsequent programs, likely influencing sort or output behavior.

4. **Clear Physical Files**:
   - `CLRPFM FILE(?9?BICUAGX)`: Clears the physical file `BICUAGX` in library `?9?`, which is used as a temporary file for intermediate data.
   - `CLRPFM FILE(?9?BICUAG2)`: Clears the physical file `BICUAG2` in library `?9?`, another temporary file for sorted data.

5. **Create Duplicate Object for `BB203W`**:
   - `// IF ?9?/G CRTDUPOBJ OBJ(BB203W) FROMLIB(DATA) OBJTYPE(*FILE) TOLIB(QS36F) NEWOBJ(?9?BB203?WS?)`: If the library `?9?` is `'G'`, duplicates the file `BB203W` from library `DATA` to `QS36F` as `?9?BB203?WS?`.
   - `// ELSE CRTDUPOBJ OBJ(BB203W) FROMLIB(DATADEV) OBJTYPE(*FILE) TOLIB(QS36FTEST) NEWOBJ(?9?BB203?WS?)`: Otherwise, duplicates `BB203W` from `DATADEV` to `QS36FTEST`. This creates a working copy of the file for processing, with the target library depending on the environment.

6. **Run Program `BI9443`**:
   - `// LOAD BI9443`: Loads the program `BI9443`.
   - `// FILE` statements specify input files:
     - `BICUAG` (label `?9?BICUAG`, shared access)
     - `GSTABL` (label `?9?GSTABL`, shared read mode)
     - `ARCUST` (label `?9?ARCUST`, shared read mode)
     - `GSPROD` (label `?9?GSPROD`, shared read mode)
     - `BICUAGNW` (label `?9?BICUAGX`, shared access, output file)
   - `// RUN`: Executes `BI9443`, which likely processes the input files to populate `BICUAGX` with filtered or transformed data based on the parameters set by `BI944P.rpg36.txt`.

7. **Sort Data with `#GSORT`** (Conditional on `KYSTYP`):
   - `// IF ?L'96,7'?/EXCLUDE GOTO EXCL`: If the value at LDA offset 96 (7 bytes) is `'EXCLUDE'`, branches to the `EXCL` tag for exclusion-based sorting. Otherwise, proceeds with inclusion-based sorting.
   - **Inclusion Sort**:
     - `// LOAD #GSORT`: Loads the system sort utility (`#GSORT`).
     - `// FILE NAME-INPUT,LABEL-?9?BICUAGX,DISP-SHR`: Uses `BICUAGX` as input.
     - `// FILE NAME-OUTPUT,LABEL-?9?BI944S,RECORDS-999000,EXTEND-999000,RETAIN-T`: Outputs to `BI944S`, with a capacity of 999,000 records, extendable, and retained temporarily.
     - `// RUN` with sort specifications:
       - `HSORTR 25A 3X 263 N`: Sorts in ascending order, processing 25 records at a time, with a 3-byte exclusion field at position 263, no sequence checking.
       - Inclusion conditions (`I C`): Excludes deleted records (`1NECD`) and applies filters based on LDA values (e.g., company, salesman range, unit of measure).
       - Sort fields (`FNC`): Company # (2–3), Customer # (4–9), Container Code (164–166), Location # (10–12), Contract (170–180).
       - Data fields (`FDC`): Positions 1–256 and 257–263.
     - `// END`: Completes the sort, producing `BI944S`.
   - **Exclusion Sort (EXCL)**:
     - Similar to inclusion sort but uses exclusion conditions (`O C`, `OOC`) to exclude specific customer numbers (positions 4–9) based on LDA values at offsets 158–272 (20 possible customer numbers).
     - Outputs to `BI944S` with the same sort fields and data fields.

8. **Run Program `BI9444`**:
   - `// TAG NEXT`: Marks the next processing step.
   - `// LOAD BI9444`: Loads the program `BI9444`.
   - `// FILE NAME-BICUAGNW,LABEL-?9?BI944S,DISP-SHR`: Uses sorted file `BI944S` as input.
   - `// FILE NAME-BICUAGN2,LABEL-?9?BICUAG2,DISP-SHR`: Outputs to `BICUAG2`.
   - `// RUN`: Executes `BI9444`, which likely performs additional processing or transformation on the sorted data to prepare it for the final sort.

9. **Second Sort with `#GSORT`**:
   - `// LOAD #GSORT`: Loads the sort utility again.
   - `// FILE NAME-INPUT,LABEL-?9?BICUAG2`: Uses `BICUAG2` as input.
   - `// FILE NAME-OUTPUT,LABEL-?9?BI944T,RECORDS-999000,EXTEND-999000,RETAIN-T`: Outputs to `BI944T`.
   - `// RUN` with sort specifications:
     - `HSORTR 20A 3X 263 N`: Sorts in ascending order, 20 records at a time, with a 3-byte exclusion field at position 263.
     - Inclusion condition: Excludes records where positions 193–207 are blank (`I C 193 207EQC`).
     - Sort fields (`FNC`): Company # (2–3), Division (257–258), Product Class (259–261), Product Code (13–16), Customer # (4–9), Ship-to Code (167–169).
     - Data fields (`FDC`): Positions 1–128 and 129–263.
   - `// END`: Produces the final sorted file `BI944T`.

10. **Run Program `BI944`**:
    - `// LOAD BI944`: Loads the main report program `BI944`.
    - `// FILE` statements specify input files:
      - `BICUAGXX` (label `?9?BI944T`, sorted input)
      - `ARCUST` (customer master, shared read)
      - `ARCUSP` (customer pricing, shared read)
      - `BICONT` (control file, shared access)
      - `GSTABL` (general system table, shared read)
      - `SHIPTO` (ship-to data, shared read)
      - `ARCUPR` (customer price records, shared read)
      - `INLOC` (location file, shared read)
      - `ZIPCODE` (zip code file, shared read)
      - `BICUA7` (unknown, possibly a custom file, shared read)
      - `BBPRCY` (pricing file, shared read)
      - `GSCTWT` (weight table, shared read)
      - `GSUMCV` (unit of measure conversion, shared read)
      - `GSCTUM2` (unit of measure table, shared read)
      - `BB203W` (label `?9?BB203?WS?`, shared access)
      - `GSCNTR1` (container table, shared read)
      - `BICUAGC` (unknown, possibly a custom file, shared read)
      - `GSPROD` (product master, shared read)
    - `// RUN`: Executes `BI944`, which generates the final Customer Sales Agreement Master File Listing using the sorted data in `BI944T` and additional reference files.

11. **Cleanup**:
    - `// GSDELETE BI944S,BI944T,,,,,,,?9?`: Deletes temporary files `BI944S` and `BI944T` to clean up after processing.
    - `// SWITCH 00000000`: Resets all switches to zero, ensuring a clean state.
    - `// LOCAL BLANK-*ALL`: Clears all local variables to prevent residual data from affecting future runs.

### External Programs Called

The OCL program explicitly calls the following programs:
1. **BI9443**: Processes input files (`BICUAG`, `GSTABL`, `ARCUST`, `GSPROD`) to produce `BICUAGX`.
2. **#GSORT**: System sort utility, called twice:
   - First to sort `BICUAGX` into `BI944S` (inclusion or exclusion logic).
   - Second to sort `BICUAG2` into `BI944T`.
3. **BI9444**: Processes `BI944S` to produce `BICUAG2`.
4. **BI944**: The main report program, generating the final listing using `BI944T` and multiple reference files.

### Tables (Files) Used

The OCL program uses the following files across its steps:
1. **BICUAG** (input for `BI9443`, shared access)
2. **GSTABL** (general system table, shared read, used by `BI9443` and `BI944`)
3. **ARCUST** (customer master, shared read, used by `BI9443` and `BI944`)
4. **GSPROD** (product master, shared read, used by `BI9443` and `BI944`)
5. **BICUAGX** (temporary output from `BI9443`, input for `#GSORT`, shared access)
6. **BI944S** (temporary sorted output from first `#GSORT`, input for `BI9444`, shared access)
7. **BICUAG2** (temporary output from `BI9444`, input for second `#GSORT`, shared access)
8. **BI944T** (final sorted output from second `#GSORT`, input for `BI944`)
9. **ARCUSP** (customer pricing, shared read, used by `BI944`)
10. **BICONT** (control file, shared access, used by `BI944`)
11. **SHIPTO** (ship-to data, shared read, used by `BI944`)
12. **ARCUPR** (customer price records, shared read, used by `BI944`)
13. **INLOC** (location file, shared read, used by `BI944`)
14. **ZIPCODE** (zip code file, shared read, used by `BI944`)
15. **BICUA7** (unknown, possibly custom file, shared read, used by `BI944`)
16. **BBPRCY** (pricing file, shared read, used by `BI944`)
17. **GSCTWT** (weight table, shared read, used by `BI944`)
18. **GSUMCV** (unit of measure conversion, shared read, used by `BI944`)
19. **GSCTUM2** (unit of measure table, shared read, used by `BI944`)
20. **BB203W** (label `?9?BB203?WS?`, duplicated file, shared access, used by `BI944`)
21. **GSCNTR1** (container table, shared read, used by `BI944`)
22. **BICUAGC** (unknown, possibly custom file, shared read, used by `BI944`)

### Summary

- **Process Steps**:
  1. Delete temporary files (`BI944S`, `BI944T`, `BB203?WS?`).
  2. Set system configuration (`GSY2K`).
  3. Modify LDA based on selection criteria (`KYLOSL`, `KYDIV`, `KYSMSL`, `KYUOFM`).
  4. Clear temporary files (`BICUAGX`, `BICUAG2`).
  5. Create a duplicate of `BB203W` based on the library environment.
  6. Run `BI9443` to process input files into `BICUAGX`.
  7. Sort `BICUAGX` into `BI944S` using `#GSORT`, with inclusion or exclusion logic based on `KYSTYP`.
  8. Run `BI9444` to process `BI944S` into `BICUAG2`.
  9. Sort `BICUAG2` into `BI944T` using `#GSORT`.
  10. Run `BI944` to generate the final report using `BI944T` and reference files.
  11. Clean up temporary files and reset switches/variables.

- **External Programs Called**:
  - `BI9443`: Initial data processing.
  - `#GSORT`: System sort utility (called twice).
  - `BI9444`: Intermediate data processing.
  - `BI944`: Final report generation.

- **Tables Used**:
  - Input: `BICUAG`, `GSTABL`, `ARCUST`, `GSPROD`, `ARCUSP`, `BICONT`, `SHIPTO`, `ARCUPR`, `INLOC`, `ZIPCODE`, `BICUA7`, `BBPRCY`, `GSCTWT`, `GSUMCV`, `GSCTUM2`, `BB203W`, `GSCNTR1`, `BICUAGC`.
  - Temporary: `BICUAGX`, `BI944S`, `BICUAG2`, `BI944T`.

This OCL program orchestrates a multi-step process to prepare and sort data for the Customer Sales Agreement Master File Listing, leveraging multiple programs and files to produce the final report. If you need further details on `BI9443`, `BI9444`, or `BI944` (e.g., RPG source code), please provide those files or additional context.