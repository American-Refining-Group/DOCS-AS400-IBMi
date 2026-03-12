The provided document, `BI942E.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is part of the pricing generation process for blended lubes, initiated by `PRICEGEN.clp`, and is invoked after `BI944B.ocl36.txt`. The script prepares the environment and executes the `BI942E` program to process pricing data. Below, I will explain the process steps of the OCL script, list the external programs called, and identify the tables (files) used.

### Process Steps of the OCL Script

1. **Run GSY2K Procedure**:
   - `// GSY2K`: Calls the System/36 procedure `GSY2K`, which likely initializes the environment, sets system parameters, or handles Y2K-related date configurations. This procedure is also called in `PRICEGEN.clp` and `BI944B.ocl36.txt`, indicating it is a standard setup step.

2. **Clear PRSABL File**:
   - `CLRPFM ?9?PRSABL`: Clears the physical file `?9?PRSABL` (e.g., `APRSABL` if `&P$GRP` = `'A'` from `PRICEGEN.clp`). This ensures the file is empty before `BI942E` populates it with new pricing data.

3. **Load and Run BI942E Program**:
   - `// LOAD BI942E`: Loads the `BI942E` program into memory for execution.
   - `// FILE` statements define the files used by `BI942E`:
     - `BICUAGP,LABEL-?9?BICUAGP,DISP-SHR`: Pricing file (e.g., `ABICUAGP`), shared mode.
     - `GSCNTR,LABEL-?9?GSCNTR,DISP-SHR`: Container file, shared mode.
     - `PRICINY,LABEL-?9?PRICINY,DISP-SHR`: Pricing inventory file, shared mode.
     - `PRSABL,LABEL-?9?PRSABL,DISP-SHR`: Output pricing table, shared mode.
     - `BICONT,LABEL-?9?BICONT,DISP-SHR`: Contract file, shared mode.
     - `GSPROD,LABEL-?9?GSPROD,DISP-SHR`: Product file, shared mode.
     - `GSCTUM,LABEL-?9?GSCTUM,DISP-SHR`: Contract unit measure file, shared mode.
     - `GSUMCV,LABEL-?9?GSUMCV,DISP-SHR`: Summary conversion file, shared mode.
     - `GSCTWT,LABEL-?9?GSCTWT,DISP-SHR`: Contract weight file, shared mode.
   - `// RUN`: Executes the `BI942E` program, which processes data from the input files and writes to `?9?PRSABL`.

### Business Rules (Inferred)

1. **Purpose**: The OCL script sets up and runs `BI942E` to process pricing agreements from `?9?BICUAGP` (populated by `BI944`), likely generating or updating pricing data in `?9?PRSABL` for use in the final pricing step (`PRICES` in `PRICEGEN.clp`).
2. **File Preparation**: Clearing `?9?PRSABL` ensures a clean slate for new pricing data, preventing residual records from affecting the output.
3. **Shared Access**: All files are opened in shared mode (`DISP-SHR`), allowing concurrent access by other processes, which is typical in a multi-job System/36 environment.
4. **Context**: As the penultimate step in the `PRICEGEN.clp` workflow (before `PRICES`), `BI942E` likely validates or transforms pricing data from `?9?BICUAGP`, incorporating contract, product, and unit of measure details to prepare `?9?PRSABL` for final processing.

### External Programs Called

1. **GSY2K**: System/36 procedure for environment initialization or Y2K date handling.
2. **BI942E**: The main program executed by the script, responsible for processing pricing data.

### Tables (Files) Used

All files use dynamic labels with `?9?` replaced by a parameter (e.g., `'A'` for `ABICUAGP` if `&P$GRP` = `'A'` from `PRICEGEN.clp`):
1. **BICUAGP** (`?9?BICUAGP`): Input pricing file, populated by `BI944`, containing blended lubes pricing data (shared mode).
2. **GSCNTR** (`?9?GSCNTR`): Container file, providing container details (shared mode).
3. **PRICINY** (`?9?PRICINY`): Pricing inventory file, likely for inventory-related pricing data (shared mode).
4. **PRSABL** (`?9?PRSABL`): Output pricing table, cleared and populated by `BI942E` (shared mode).
5. **BICONT** (`?9?BICONT`): Contract file, providing contract details (shared mode).
6. **GSPROD** (`?9?GSPROD`): Product file, providing product details (shared mode).
7. **GSCTUM** (`?9?GSCTUM`): Contract unit measure file, for unit of measure conversions (shared mode).
8. **GSUMCV** (`?9?GSUMCV`): Summary conversion file, for volume conversions (shared mode).
9. **GSCTWT** (`?9?GSCTWT`): Contract weight file, for gallons/container data (shared mode).

### Additional Notes

- **Context with PRICEGEN.clp**: The script is invoked by `PRICEGEN.clp` via `STRS36PRC PRC(BI942E) PARM(&PARM9)`, following `BIFX43`, `BIFX44`, and `BI944B`. It processes `?9?BICUAGP` (output of `BI944`) to produce `?9?PRSABL`, which is likely used by the final `PRICES` program.
- **System/36 Environment**: The use of OCL and `GSY2K` indicates a legacy System/36 environment, possibly running on an AS/400 in compatibility mode.
- **Limitations**: Without the `BI942E` RPG source code, the exact logic (e.g., filtering, transformations) is inferred. It likely validates pricing agreements, incorporates container and product data, and prepares `?9?PRSABL` for final pricing calculations.
- **Error Handling**: The script relies on the System/36 environment or `BI942E` for error handling, as no explicit error logic is present in the OCL.

If you have the RPG source code for `BI942E` or `PRICES`, or need further analysis of the pricing generation workflow, please provide those details! Let me know if you have additional questions or files to share.