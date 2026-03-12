The provided document, `PRICES.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is the final step in the pricing generation process for blended lubes, as initiated by `PRICEGEN.clp`, following the execution of `BIFX43`, `BIFX44`, `BI944B`, and `BI942E`. The script performs file copying operations and invokes a series of programs to finalize pricing data. Below, I will explain the process steps, business rules, external programs called, and tables (files) used .

### Process Steps of the OCL Script

1. **Copy PRSABL to PRSABLW**:
   - `CPYF FROMFILE(*LIBL/?9?PRSABL) TOFILE(*LIBL/?9?PRSABLW) MBROPT(*REPLACE) FMTOPT(*MAP)`:
     - Copies the file `?9?PRSABL` (e.g., `APRSABL` if `&P$GRP` = `'A'` from `PRICEGEN.clp`) to `?9?PRSABLW` (e.g., `APRSABLW`).
     - `MBROPT(*REPLACE)`: Replaces the contents of the target file (`?9?PRSABLW`) if it exists.
     - `FMTOPT(*MAP)`: Maps fields between files based on matching field names, accommodating any structural differences.
   - **Note**: The commented-out line `CPYF FROMFILE(QS36F/GPRSABL) TOFILE(QS36F/GPRSABLW)` suggests a legacy or alternative copy operation for a specific library (`QS36F`), possibly overridden by the dynamic `?9?` version.

2. **Invoke Pricing Programs**:
   - The script calls a series of programs, passing the `?9?` parameter (e.g., `'A'`) to each:
     - `BB953B ,,,,,,,,?9?`: Executes the `BB953B` program.
     - `SA505C ,,,,,,,,?9?`: Executes the `SA505C` program.
     - `SA505E ,,,,,,,,?9?`: Executes the `SA505E` program.
     - `SA505G ,,,,,,,,?9?`: Executes the `SA505G` program.
     - `SA505H ,,,,,,,,?9?`: Executes the `SA505H` program.
     - `SA505I ,,,,,,,,?9?`: Executes the `SA505I` program.
     - `SA505J ,,,,,,,,?9?`: Executes the `SA505J` program.
     - `**SA505L ,,,,,,,,?9?`: Commented out, indicating `SA505L` is not currently executed but may be part of the workflow in specific scenarios.
   - Each program likely performs specific pricing calculations or updates on `?9?PRSABLW` or related files, using the `?9?` parameter to reference the appropriate file set (e.g., `A` for `APRSABLW`).

3. **Copy PRSABLW Back to PRSABL (Filtered)**:
   - `CPYF FROMFILE(*LIBL/?9?PRSABLW) TOFILE(*LIBL/?9?PRSABL) MBROPT(*REPLACE) INCREL((*IF BATYPE *EQ 'A'))`:
     - Copies records from `?9?PRSABLW` back to `?9?PRSABL`, replacing the contents of `?9?PRSABL`.
     - `INCREL((*IF BATYPE *EQ 'A'))`: Filters records where the field `BATYPE` equals `'A'`, ensuring only specific agreement types are copied back.
     - `FMTOPT(*MAP)`: Ensures field mapping between source and target files.
   - **Note**: The commented-out line `CPYF FROMFILE(QS36F/GPRSABLW) TOFILE(QS36F/GPRSABL)` indicates a similar legacy operation, possibly replaced by the dynamic `?9?` version.

### Business Rules (Inferred)

1. **Purpose**: The `PRICES` script finalizes the pricing generation process by copying `?9?PRSABL` to a working file (`?9?PRSABLW`), running a series of pricing programs to update or validate data, and copying back filtered records (where `BATYPE = 'A'`) to `?9?PRSABL` for use in downstream processes.
2. **File Backup and Restoration**:
   - The initial copy to `?9?PRSABLW` creates a working copy of the pricing data, allowing programs to modify it without affecting the original `?9?PRSABL` until the final step.
   - The final copy back to `?9?PRSABL` ensures only records with `BATYPE = 'A'` (likely indicating active or specific agreement types) are retained.
3. **Program Sequence**: The sequential execution of `BB953B`, `SA505C`, `SA505E`, `SA505G`, `SA505H`, `SA505I`, and `SA505J` suggests a modular approach to pricing calculations, where each program handles a specific aspect (e.g., price adjustments, validations, or updates).
4. **Context**: As the final step in `PRICEGEN.clp`, this script processes the output of `BI942E` (`?9?PRSABL`) to produce the final pricing data, likely for use in invoicing or customer sales agreements.

### External Programs Called

1. **BB953B**: Likely performs pricing calculations or updates on `?9?PRSABLW`.
2. **SA505C**: Part of the pricing workflow, possibly handling specific agreement types or calculations.
3. **SA505E**: Continues pricing processing, potentially validating data.
4. **SA505G**: Further processes pricing data.
5. **SA505H**: Likely applies additional pricing rules or updates.
6. **SA505I**: Continues the pricing workflow.
7. **SA505J**: Finalizes pricing calculations or updates.
8. **SA505L** (commented out): Not currently executed but part of the workflow in specific cases.

### Tables (Files) Used

1. **?9?PRSABL** (e.g., `APRSABL` if `&P$GRP` = `'A'`):
   - **Role**: Source and target file for pricing data.
   - **Access**: Input for the first `CPYF`, output for the final `CPYF`.
   - **Purpose**: Contains pricing data populated by `BI942E`, filtered to `BATYPE = 'A'` in the final step.
2. **?9?PRSABLW** (e.g., `APRSABLW`):
   - **Role**: Working file for pricing programs.
   - **Access**: Output for the first `CPYF`, input/output for programs, input for the final `CPYF`.
   - **Purpose**: Temporary storage for pricing data during processing by `BB953B`, `SA505C`, etc.

### Additional Notes

- **Context with PRICEGEN.clp**: The script is invoked by `PRICEGEN.clp` via `STRS36PRC PRC(PRICES) PARM(&PARM9)` after `BI942E`. It finalizes the pricing data in `?9?PRSABL` for use in downstream processes (e.g., invoicing or reporting).
- **System/36 Environment**: The use of OCL and `CPYF` commands indicates a legacy System/36 environment, likely running on an AS/400 in compatibility mode.
- **Commented-Out Commands**: The commented `CPYF` commands referencing `QS36F/GPRSABL` and `QS36F/GPRSABLW` suggest a legacy setup or alternative library, possibly for testing or specific scenarios.
- **Limitations**: Without the RPG source code for `BB953B`, `SA505C`, `SA505E`, `SA505G`, `SA505H`, `SA505I`, `SA505J`, or `SA505L`, the exact logic of each program is unknown. They likely perform specific pricing calculations, validations, or updates on `?9?PRSABLW`.
- **Error Handling**: The script relies on the System/36 environment and the called programs for error handling, as no explicit error logic is present in the OCL.

If you have the RPG source code for `BB953B`, `SA505C`, `SA505E`, `SA505G`, `SA505H`, `SA505I`, `SA505J`, or `SA505L`, or need further analysis of the pricing generation workflow, please provide those details! Let me know if you have additional questions or files to share.