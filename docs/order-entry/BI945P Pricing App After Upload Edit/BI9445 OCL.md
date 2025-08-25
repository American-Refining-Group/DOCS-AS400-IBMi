The document provided, `BI9445.ocl36.txt`, is an Operation Control Language (OCL) program used on an IBM midrange system (e.g., AS/400 or iSeries) to print a customer sales agreement master file. This OCL script orchestrates a series of data processing steps, including file clearing, data sorting, copying, and program execution. It is called conditionally by the `BI945P.ocl36.txt` program, as noted in the prior analysis. Below, I’ll explain the process steps, business rules, external programs called, and tables used.

---

### Process Steps of the OCL Program

The `BI9445` OCL program manages the preparation and printing of customer sales agreement data. The steps are executed sequentially, with conditional logic and sorting operations to prepare data for the final report. Here’s a detailed breakdown:

1. **File Deletion (GSDELETE)**:
   - `// GSDELETE BI944S,BI944T,BB204?WS?,,,,,,?9?`:
     - Deletes files `BI944S`, `BI944T`, and a file with a placeholder suffix `BB204?WS?` (likely `BB204WS`), where `?9?` is a library placeholder and `?WS?` is a workspace identifier.
     - This ensures that temporary or work files from previous runs are removed to avoid data conflicts.
   - Purpose: Clears old data to prepare for fresh processing.

2. **Year 2000 Compliance (GSY2K)**:
   - `// GSY2K`: Invokes a Year 2000 compliance routine, likely to ensure date fields are processed correctly (e.g., handling century indicators for dates).

3. **Conditional Local Data Area (LDA) Updates**:
   - The program updates specific offsets in the LDA based on conditions:
     - `// IF ?L'155,3'?/SEL LOCAL OFFSET-4,DATA-'O COAC' ELSE LOCAL OFFSET-4,DATA-'O*CO*C'`:
       - Checks if the LDA field at position 155 (3 bytes) equals 'SEL'.
       - If true, sets offset 4 to 'O COAC'; otherwise, sets it to 'O*CO*C'.
       - Likely controls company or contract selection logic.
     - `// IF ?L'103,1'?/B LOCAL OFFSET-10,DATA-'I*C' ELSE LOCAL OFFSET-10,DATA-'IAC'`:
       - Checks if the LDA field at position 103 (1 byte) equals 'B'.
       - If true, sets offset 10 to 'I*C'; otherwise, sets it to 'IAC'.
       - Likely controls division or business unit filtering.
     - `// IF ?L'421,3'?/SEL LOCAL OFFSET-13,DATA-'IAC' ELSE LOCAL OFFSET-13,DATA-'I*C'`:
       - Checks if the LDA field at position 421 (3 bytes) equals 'SEL'.
       - If true, sets offset 13 to 'IAC'; otherwise, sets it to 'I*C'.
       - Likely controls sales method or location selection.
     - `// IFF ?L'458,3'?/ LOCAL OFFSET-16,DATA-'IAC' ELSE LOCAL OFFSET-16,DATA-'I*C'`:
       - Checks if the LDA field at position 458 (3 bytes) is blank (note: `IFF` might be a typo for `IF`).
       - If blank, sets offset 16 to 'IAC'; otherwise, sets it to 'I*C'.
       - Likely controls unit of measure or additional filtering.
   - Purpose: Configures the LDA with parameters to control data selection and filtering for the subsequent steps.

4. **Clear Physical Files**:
   - `CLRPFM FILE(?9?BICUAGX)`: Clears the `BICUAGX` file in the library specified by `?9?`.
   - `CLRPFM FILE(?9?BICUAG2)`: Clears the `BICUAG2` file.
   - `CLRPFM FILE(?9?BB204T)`: Clears the `BB204T` file.
   - Purpose: Ensures that the output and temporary files are empty before new data is written.

5. **Load and Run BI944X**:
   - `// LOAD BI944X`:
     - Loads the `BI944X` program.
   - `// FILE NAME-BICUAG,LABEL-?9?BICUAG,DISP-SHR`:
     - Specifies the input file `BICUAG` in shared mode.
   - `// FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHRRM`:
     - Specifies the table `GSTABL` in shared read mode.
   - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHRRM`:
     - Specifies the customer file `ARCUST` in shared read mode.
   - `// FILE NAME-BICUAGNW,LABEL-?9?BICUAGX,DISP-SHR`:
     - Specifies the output file `BICUAGX` (aliased as `BICUAGNW`).
   - `// RUN`: Executes the `BI944X` program.
   - Purpose: Processes data from `BICUAG`, `GSTABL`, and `ARCUST` to populate `BICUAGX` with sales agreement data.

6. **Conditional Sorting (GSORT)**:
   - The program checks if the LDA field at position 96 (7 bytes) equals 'EXCLUDE':
     - `// IF ?L'96,7'?/EXCLUDE GOTO EXCL`:
       - If true, jumps to the `EXCL` tag for exclusion-based sorting.
       - If false, proceeds with inclusion-based sorting.
   - **Inclusion-Based Sorting** (`// LOAD #GSORT`):
     - Input file: `?9?BICUAGX` (shared mode).
     - Output file: `?9?BI944S` (temporary file, 999,000 records, job retention).
     - Sorting rules (`HSORTR`):
       - Sorts by a 25-byte ascending key, with additional checks on fields at positions 263 and customer fields (`kycs01`–`kycs20` at positions 158–277).
       - Includes records based on conditions:
         - `I C 1NECD`: Excludes records where position 1 is not blank.
         - Matches division (`?L'10,3'?` vs. positions 441–443), from/to sales methods (`?L'13,3'?` vs. 424–425, 426–427), and unit of measure (`?L'16,3'?` vs. 458–460).
       - Output fields:
         - `FNC 2 3`: Company number.
         - `FNC 4 9`: Customer number.
         - `FNC 164 166`: Container code.
         - `FNC 10 12`: Location number.
         - `FNC 170 180`: Contract.
         - `FDC 1 256`, `FDC 257 263`: Full record and additional fields.
     - Purpose: Sorts `BICUAGX` into `BI944S` based on inclusion criteria (e.g., matching customer numbers in the LDA).
   - **Exclusion-Based Sorting** (`// TAG EXCL`):
     - Similar to inclusion-based sorting but uses exclusion logic:
       - Excludes records where customer numbers match `kycs01`–`kycs20` (`OOC 4 9EQC?L'158,6'?`, etc.).
       - Same output fields and conditions as above.
     - Purpose: Sorts `BICUAGX` into `BI944S` based on exclusion criteria.

7. **Copy Sorted Data**:
   - `CPYF FROMFILE(?9?BI944S) TOFILE(?9?BICUAG2) MBROPT(*REPLACE) FMTOPT(*NOCHK)`:
     - Copies data from `BI944S` to `BICUAG2`, replacing existing data without format checking.
     - Purpose: Transfers sorted data to `BICUAG2` for further processing.

8. **Final Sorting (GSORT)**:
   - `// LOAD #GSORT`:
     - Input file: `?9?BICUAG2`.
     - Output file: `?9?BI944T` (temporary file, 999,000 records, job retention).
     - Sorting rules (`HSORTR`):
       - Sorts by a 20-byte ascending key, checking fields at positions 193–207.
       - Output fields:
         - `FNC 2 3`: Company number.
         - `FNC 257 258`: Division.
         - `FNC 259 261`: Product class.
         - `FNC 13 16`: Product code.
         - `FNC 4 9`: Customer number.
         - `FNC 167 169`: Ship-to code.
         - `FDC 1 128`, `FDC 129 263`: Full record and additional fields.
     - Purpose: Sorts `BICUAG2` into `BI944T` for the final report, organized by company, division, product class, product code, customer, and ship-to.

9. **Load and Run BI9445**:
   - `// LOAD BI9445`:
     - Loads the `BI9445` program.
   - Files used:
     - `BICUAGXX` (aliased as `?9?BI944T`): Primary input file.
     - `ARCUST`, `PRSABLX`, `ARCUSP`, `BICONT`, `GSTABL`, `SHIPTO`, `ARCUPR`, `INLOC`, `ZIPCODE`, `BICUA7`, `BBPRCY`, `GSUMCV`, `GSCTUM2`, `BB204T`, `GSCNTR1`, `BICUAGC`: All in shared or shared read mode.
   - `// RUN`: Executes the `BI9445` program.
   - Purpose: Processes the sorted data in `BI944T` along with reference files to generate the final customer sales agreement report.

10. **Cleanup**:
    - `// GSDELETE BI944S,BI944T,,,,,,,?9?`: Deletes temporary files `BI944S` and `BI944T`.
    - `// SWITCH 00000000`: Resets all switches to off.
    - `// LOCAL BLANK-*ALL`: Clears all local variables in the LDA.
    - Purpose: Ensures a clean environment after processing.

---

### Business Rules

1. **File Cleanup**:
   - Temporary files (`BI944S`, `BI944T`, `BB204T`, etc.) are cleared or deleted before and after processing to prevent data overlap.
2. **LDA Configuration**:
   - Conditional updates to the LDA control data selection (e.g., company, division, sales method, unit of measure) based on specific field values (`SEL`, `B`, blank).
3. **Data Filtering**:
   - Sorting can be inclusion-based (include records matching LDA customer numbers) or exclusion-based (exclude matching customer numbers), controlled by the LDA field at position 96 (`EXCLUDE`).
4. **Sorting Hierarchy**:
   - First sort (`BI944S`): Organizes data by company, customer, container code, location, and contract.
   - Second sort (`BI944T`): Reorganizes by company, division, product class, product code, customer, and ship-to for the final report.
5. **Data Integrity**:
   - Uses shared read modes for reference files to ensure data consistency without locking.
   - Copies data without format checking (`FMTOPT(*NOCHK)`) to ensure compatibility between files.

---

### Tables Used

The program uses the following files/tables:
1. **Input and Output Files**:
   - `BICUAGX`: Cleared and populated by `BI944X`, used as input for sorting.
   - `BICUAG2`: Receives sorted data from `BI944S` via `CPYF`.
   - `BI944S`: Temporary file for first sort output.
   - `BI944T`: Temporary file for final sort output, used as input for `BI9445`.
   - `BB204T`: Cleared and used as an output or work file.
2. **Reference Files**:
   - `BICUAG`: Input file for `BI944X`, likely contains raw sales agreement data.
   - `GSTABL`: Reference table, possibly for general setup or translation data.
   - `ARCUST`: Customer master file.
   - `PRSABLX`: Sales agreement file (also used in `BI945P`).
   - `ARCUSP`: Customer pricing file.
   - `BICONT`: Contract file.
   - `SHIPTO`: Ship-to address file.
   - `ARCUPR`: Unit pricing file.
   - `INLOC`: Location file.
   - `ZIPCODE`: Zip code file.
   - `BICUA7`: Additional sales agreement file.
   - `BBPRCY`: Pricing file.
   - `GSUMCV`: Unit of measure conversion file.
   - `GSCTUM2`: Contract unit of measure file.
   - `GSCNTR1`: Contract file.
   - `BICUAGC`: Additional sales agreement file.
3. **Related Files (from OCL Context)**:
   - `GPRSABLO`: Source table for `PRSABLX` (from `BI945P.ocl36.txt`).
   - `GPRSABLN`: Obsolete spreadsheet (from `BI945P.ocl36.txt`).

---

### External Programs Called

1. **BI944X**:
   - Loaded and executed to process data from `BICUAG`, `GSTABL`, and `ARCUST` into `BICUAGX`.
2. **#GSORT**:
   - System sort utility, called twice:
     - First to sort `BICUAGX` into `BI944S` (inclusion or exclusion logic).
     - Second to sort `BICUAG2` into `BI944T`.
3. **BI9445**:
   - The main program, loaded to process the sorted `BI944T` file with multiple reference files to generate the final report.
4. **GSDELETE**:
   - Utility to delete temporary files (`BI944S`, `BI944T`, `BB204WS`).

---

### Summary

The `BI9445` OCL program orchestrates the preparation and printing of a customer sales agreement master file. It clears temporary files, configures the LDA, processes input data through `BI944X`, sorts data twice (with inclusion or exclusion logic), copies sorted data, and runs `BI9445` to produce the final report. The process ensures data is filtered and organized by company, customer, product, and other criteria.

**Process Steps**:
1. Delete temporary files.
2. Apply Y2K compliance.
3. Update LDA fields based on conditions.
4. Clear output files (`BICUAGX`, `BICUAG2`, `BB204T`).
5. Run `BI944X` to populate `BICUAGX`.
6. Sort `BICUAGX` into `BI944S` (inclusion or exclusion).
7. Copy `BI944S` to `BICUAG2`.
8. Sort `BICUAG2` into `BI944T`.
9. Run `BI9445` with multiple reference files.
10. Clean up temporary files and reset the environment.

**Business Rules**:
1. Clear files to prevent data overlap.
2. Configure LDA for filtering (company, division, sales method, unit of measure).
3. Sort data with inclusion or exclusion logic based on LDA settings.
4. Organize final output by company, division, product class, product code, customer, and ship-to.
5. Use shared read modes for data integrity.

**Tables Used**:
- Input/Output: `BICUAGX`, `BICUAG2`, `BI944S`, `BI944T`, `BB204T`.
- Reference: `BICUAG`, `GSTABL`, `ARCUST`, `PRSABLX`, `ARCUSP`, `BICONT`, `SHIPTO`, `ARCUPR`, `INLOC`, `ZIPCODE`, `BICUA7`, `BBPRCY`, `GSUMCV`, `GSCTUM2`, `GSCNTR1`, `BICUAGC`.
- Related: `GPRSABLO`, `GPRSABLN` (from `BI945P`).

**External Programs Called**:
- `BI944X`, `#GSORT`, `BI9445`, `GSDELETE`.

If you provide the RPGLE source for `BI9445` or `BI944X`, I can further detail their logic and integration with this OCL program.