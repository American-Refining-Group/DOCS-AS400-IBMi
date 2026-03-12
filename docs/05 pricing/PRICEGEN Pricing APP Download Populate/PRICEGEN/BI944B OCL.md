The provided document, `BI944B.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode) to manage the execution of programs and file operations for populating a file (`?9?BICUAGP`) for blended lubes as part of the pricing generation process initiated by `PRICEGEN.clp`. Below, I will explain the process steps of the OCL script, list the external programs called, and identify the tables (files) used .

### Process Steps of the OCL Script

1. **Set System Switch**:
   - `// SWITCH 00100000`: Sets the System/36 switch settings, specifically setting switch 3 to `1` (bit 3 in an 8-bit switch register). This controls conditional logic later in the script (e.g., `IF SWITCH3-1`).

2. **Conditional Local Data Setup (if SWITCH3-1)**:
   - If switch 3 is set to `1` (as it is from the `SWITCH` command), the script executes several `LOCAL OFFSET` commands to populate specific memory locations with data:
     - `OFFSET-1, DATA-'   O*CO*CI*CI*CI*C'`: Sets a string for pricing application parameters.
     - `OFFSET-101, DATA-'  LALL '`: Sets a value indicating "ALL" for a specific parameter.
     - `OFFSET-122, DATA-'ALL'`: Sets another "ALL" indicator.
     - `OFFSET-155, DATA-'ALL'`: Sets another "ALL" indicator.
     - `OFFSET-278, DATA-'ALL'`: Sets another "ALL" indicator.
     - `OFFSET-321, DATA-'ALL    N0150'`: Sets a combined value with an "N0150" suffix.
     - `OFFSET-341, DATA-'Y'`: Sets a flag to `Y`.
     - `OFFSET-350, DATA-'0000000N   00000020000000ALL'`: Sets a complex string with numeric and flag values.
     - `OFFSET-509, DATA-'1980'`: Sets a year or code value.
   - These settings likely configure parameters for the programs executed later.

3. **Run GSY2K Procedure**:
   - `GSY2K`: Calls the System/36 procedure `GSY2K`, likely for environment initialization or setup (also called in `PRICEGEN.clp`).

4. **Clear File ?9?BICUAGP**:
   - `CLRPFM FILE(?9?BICUAGP)`: Clears the physical file `?9?BICUAGP` (e.g., `ABICUAGP` if `&P$GRP` = `'A'` from `PRICEGEN.clp`), preparing it for new data.

5. **Run BI944A Program**:
   - `// LOAD BI944A`: Loads the `BI944A` program.
   - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHRRM`: Specifies the `BICONT` file (e.g., `ABICONT`) in shared read mode with record locking (`SHRRM`).
   - `// RUN`: Executes `BI944A`, which likely processes contract data from `?9?BICONT` to support pricing generation.

6. **Delete Temporary Files**:
   - `// GSDELETE BI944S,BI944T,BB203?WS?,,,,,,?9?`: Deletes temporary files `?9?BI944S`, `?9?BI944T`, and `?9?BB203?WS?` (e.g., `ABI944S`, `ABI944T`, `ABB203?WS?`) to ensure clean data for subsequent steps.

7. **Second GSY2K Call**:
   - `// GSY2K`: Calls `GSY2K` again, possibly to reset or reinitialize the environment.

8. **Conditional Parameter Updates**:
   - Several `IF`/`ELSE` blocks check local memory locations (`?L'offset,length'?`) and set data at specific offsets:
     - `OFFSET-4`: Sets `'O COAC'` if `?L'155,3'?` = `'SEL'`, else `'O*CO*C'`.
     - `OFFSET-10`: Sets `'I*C'` if `?L'103,1'?` = `'B'`, else `'IAC'`.
     - `OFFSET-13`: Sets `'IAC'` if `?L'321,3'?` = `'SEL'`, else `'I*C'`.
     - `OFFSET-16`: Sets `'IAC'` if `?L'358,3'?` is blank (`/`), else `'I*C'`.
   - These settings configure parameters for subsequent programs, likely related to pricing or customer options.

9. **Clear Additional Files**:
   - `CLRPFM FILE(?9?BICUAGX)`: Clears the file `?9?BICUAGX` (e.g., `ABICUAGX`).
   - `CLRPFM FILE(?9?BICUAG2)`: Clears the file `?9?BICUAG2` (e.g., `ABICUAG2`).

10. **Create Duplicate Object for BB203**:
    - Conditional logic (`IF ?9?/G`) checks if the parameter `?9?` equals `'G'`:
      - If true: `CRTDUPOBJ OBJ(BB203W) FROMLIB(DATA) OBJTYPE(*FILE) TOLIB(QS36F) NEWOBJ(?9?BB203?WS?)` creates a copy of `BB203W` from library `DATA` to `QS36F` as `?9?BB203?WS?` (e.g., `ABB203?WS?`).
      - If false: Copies from `DATADEV` to `QS36FTEST`.
    - This step prepares a temporary work file for the `BB203` process.

11. **Run BI9443 Program**:
    - `// LOAD BI9443`: Loads the `BI9443` program.
    - Files specified:
      - `BICUAG` (`?9?BICUAG`, e.g., `ABICUAG`, shared mode).
      - `GSTABL` (`?9?GSTABL`, shared read mode with record locking).
      - `ARCUST` (`?9?ARCUST`, shared read mode with record locking).
      - `GSPROD` (`?9?GSPROD`, shared read mode with record locking).
      - `BICUAGNW` (`?9?BICUAGX`, shared mode).
    - `// RUN`: Executes `BI9443`, which likely processes pricing data using these files.

12. **Sort BICUAGX File (Conditional)**:
    - Conditional check: `IF ?L'96,7'?/EXCLUDE GOTO EXCL`:
      - If the condition at memory location 96 (7 bytes) is `'EXCLUDE'`, skip to the `EXCL` tag.
      - Otherwise, proceed with the first sort operation.
    - **First Sort**:
      - `// LOAD #GSORT`: Loads the System/36 sort utility.
      - Input file: `?9?BICUAGX` (e.g., `ABICUAGX`, shared mode).
      - Output file: `?9?BI944S` (e.g., `ABI944S`, temporary file with 999,000 records capacity, retained).
      - Sort specification (`HSORTR`):
        - Sorts on multiple fields (positions 2–3, 4–9, 164–166, 10–12, 170–180 for company, customer, container, location, and contract).
        - Includes/excludes records based on conditions (e.g., exclude if position 1 = `'D'` for deleted records).
      - `// RUN`: Executes the sort, creating `?9?BI944S`.
    - **EXCL Tag (Alternative Sort)**:
      - If `EXCLUDE` condition is met, a similar sort is performed with slightly different output conditions (`OOC` instead of `O*`).
      - The sort criteria and files remain the same, producing `?9?BI944S`.

13. **Run BI9444 Program**:
    - `// LOAD BI9444`: Loads the `BI9444` program.
    - Files specified:
      - `BICUAGNW` (`?9?BI944S`, shared mode).
      - `BICUAGN2` (`?9?BICUAG2`, shared mode).
    - `// RUN`: Executes `BI9444`, which likely processes the sorted `?9?BI944S` to produce or update `?9?BICUAG2`.

14. **Sort BICUAG2 File**:
    - `// LOAD #GSORT`: Loads the sort utility again.
    - Input file: `?9?BICUAG2` (e.g., `ABICUAG2`).
    - Output file: `?9?BI944T` (e.g., `ABI944T`, temporary file with 999,000 records capacity, retained).
    - Sort specification:
      - Sorts on fields (positions 2–3, 257–258, 259–261, 13–16, 4–9, 167–169 for company, division, product class, product code, customer, and ship-to).
      - Includes records where positions 193–207 are equal to a specific value.
    - `// RUN`: Executes the sort, creating `?9?BI944T`.

15. **Run BI944 Program**:
    - `// LOAD BI944`: Loads the `BI944` program.
    - Files specified (all with dynamic labels, e.g., `A<FILE>` if `?9?` = `'A'`):
      - `BICUAGXX` (`?9?BI944T`, input).
      - `ARCUST` (`?9?ARCUST`, shared read mode with record locking).
      - `ARCUSP` (`?9?ARCUSP`, shared read mode with record locking).
      - `BICONT` (`?9?BICONT`, shared mode).
      - `GSTABL` (`?9?GSTABL`, shared read mode with record locking).
      - `SHIPTO` (`?9?SHIPTO`, shared read mode with record locking).
      - `ARCUPR` (`?9?ARCUPR`, shared read mode with record locking).
      - `INLOC` (`?9?INLOC`, shared read mode with record locking).
      - `ZIPCODE` (`?9?ZIPCODE`, shared read mode with record locking).
      - `BICUA7` (`?9?BICUA7`, shared read mode with record locking).
      - `BBPRCY` (`?9?BBPRCY`, shared read mode with record locking).
      - `GSCTWT` (`?9?GSCTWT`, shared read mode with record locking).
      - `GSUMCV` (`?9?GSUMCV`, shared read mode with record locking).
      - `GSCTUM2` (`?9?GSCTUM2`, shared read mode with record locking).
      - `BB203W` (`?9?BB203?WS?`, shared mode).
      - `GSCNTR1` (`?9?GSCNTR1`, shared mode).
      - `BICUAGC` (`?9?BICUAGP`, shared mode, target output file).
      - `GSPROD` (`?9?GSPROD`, shared mode).
    - Printer overrides: `OVRPRTF FILE(PRTDOWN) OUTQ(QUSRSYS/JUNKOUTQ)` and `OVRPRTF FILE(PRTEXCEL) OUTQ(QUSRSYS/JUNKOUTQ)` redirect printed output to a junk output queue.
    - `// RUN`: Executes `BI944`, which populates `?9?BICUAGP` with blended lubes pricing data.

16. **Cleanup and Reset**:
    - `// GSDELETE BI944S,BI944T,,,,,,,?9?`: Deletes temporary files `?9?BI944S` and `?9?BI944T`.
    - `// SWITCH 00000000`: Resets all system switches to `0`.
    - `// LOCAL BLANK-*ALL`: Clears all local memory data, resetting the environment.

### External Programs Called

The OCL script invokes the following programs:
1. **GSY2K**: System/36 procedure for environment initialization (called twice).
2. **BI944A**: Program processing contract data from `?9?BICONT`.
3. **BI9443**: Program processing pricing data from multiple files.
4. **#GSORT**: System/36 sort utility (called twice for sorting `?9?BICUAGX` and `?9?BICUAG2`).
5. **BI9444**: Program processing sorted data from `?9?BI944S` to produce `?9?BICUAG2`.
6. **BI944**: Main program populating `?9?BICUAGP` with blended lubes pricing data.

### Tables (Files) Used

The script uses the following files (all with dynamic labels based on `?9?`, e.g., `A<FILE>` if `?9?` = `'A'`):
1. **?9?BICUAGP**: Target output file, cleared and populated with blended lubes data.
2. **?9?BICONT**: Contract file, used by `BI944A` (shared read mode with record locking).
3. **?9?BICUAGX**: Temporary file, cleared and used as input/output for `BI9443` and first sort.
4. **?9?BICUAG2**: Temporary file, cleared and used as output by `BI9444` and input for second sort.
5. **?9?BICUAG**: Pricing file, used by `BI9443` (shared mode).
6. **?9?GSTABL**: Table file, used by `BI9443` and `BI944` (shared read mode with record locking).
7. **?9?ARCUST**: Customer file, used by `BI9443` and `BI944` (shared read mode with record locking).
8. **?9?GSPROD**: Product file, used by `BI9443` and `BI944` (shared mode).
9. **?9?BI944S**: Temporary sorted file, output of first sort, input to `BI9444`.
10. **?9?BI944T**: Temporary sorted file, output of second sort, input to `BI944`.
11. **?9?BB203?WS?**: Temporary work file, created via `CRTDUPOBJ` for `BB203` process.
12. **?9?ARCUSP**: Customer pricing file, used by `BI944` (shared read mode with record locking).
13. **?9?SHIPTO**: Ship-to file, used by `BI944` (shared read mode with record locking).
14. **?9?ARCUPR**: Customer price file, used by `BI944` (shared read mode with record locking).
15. **?9?INLOC**: Location file, used by `BI944` (shared read mode with record locking).
16. **?9?ZIPCODE**: Zip code file, used by `BI944` (shared read mode with record locking).
17. **?9?BICUA7**: Additional pricing file, used by `BI944` (shared read mode with record locking).
18. **?9?BBPRCY**: Pricing history file, used by `BI944` (shared read mode with record locking).
19. **?9?GSCTWT**: Contract weight file, used by `BI944` (shared read mode with record locking).
20. **?9?GSUMCV**: Summary file, used by `BI944` (shared read mode with record locking).
21. **?9?GSCTUM2**: Contract unit measure file, used by `BI944` (shared read mode with record locking).
22. **?9?GSCNTR1**: Container file, used by `BI944` (shared mode).

### Additional Notes

- **Context with PRICEGEN.clp**: The `BI944B.ocl36` script is invoked by `PRICEGEN.clp` via `STRS36PRC PRC(BI944B) PARM(&PARM9)`, with `?9?` corresponding to `&P$GRP` (e.g., `'A'` for `ABICUAGP`). It follows `BIFX43` and `BIFX44`, which ensure timestamp consistency in `?9?BICUAG`.
- **Purpose**: The script orchestrates a complex process to populate `?9?BICUAGP` with blended lubes pricing data, involving multiple programs, file sorts, and data validations.
- **Temporary Files**: Files like `?9?BICUAGX`, `?9?BICUAG2`, `?9?BI944S`, and `?9?BI944T` are used as intermediates, cleared and deleted to manage workflow.
- **System/36 Environment**: The use of OCL, `GSY2K`, and `#GSORT` indicates a legacy System/36 environment, possibly on an AS/400 in compatibility mode.
- **Limitations**: Without the RPG source code for `BI944A`, `BI9443`, `BI9444`, and `BI944`, the exact logic of each step is unclear. The script suggests a multi-stage process involving contract, customer, product, and pricing data integration.

If you have the RPG source code for `BI944A`, `BI9443`, `BI9444`, `BI944`, or the remaining procedures (`BI942E`, `PRICES`), or need further analysis of the pricing generation workflow, please provide those details! Let me know if you have additional questions or files to share.