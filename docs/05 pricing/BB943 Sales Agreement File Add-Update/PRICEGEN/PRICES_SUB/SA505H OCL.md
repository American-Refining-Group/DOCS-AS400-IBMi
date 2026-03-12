The provided document, `SA505H.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is part of the pricing generation process for blended lubes, invoked by `PRICES.ocl36.txt` within the `PRICEGEN.clp` workflow, following `SA505G`. The script generates or refines a Customer Shipping Analysis Report by executing the `SA505H` program, which processes data from previous steps (`SA505G`, `SA505C`) and produces output files for further analysis or reporting. Below, I will explain the process steps, business rules, external programs called, and tables (files) used.

### Process Steps of the OCL Script

1. **Clear Output Files (Conditional)**:
   - `// IF DATAF1-?9?TMPFILX CLRPFM ?9?TMPFILX`: If the output file `?9?TMPFILX` (e.g., `ATMPFILX`) exists, clears it to ensure a fresh dataset.
   - `// IF DATAF1-?9?TMPFILY CLRPFM ?9?TMPFILY`: If the secondary output file `?9?TMPFILY` (e.g., `ATMPFILY`) exists, clears it.

2. **Load and Run SA505H Program**:
   - `// LOAD SA505H`: Loads the `SA505H` RPG program.
   - Files specified:
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file, shared mode.
     - `SA505GX,LABEL-?9?SA505G,DISP-SHR`: Primary output file from `SA505G`, shared mode.
     - `SA505G2,LABEL-?9?SA505G2,DISP-SHR`: Secondary output file from `SA505G`, shared mode.
     - `BICONT,LABEL-?9?BICONT,DISP-SHR`: Contract file, shared mode.
     - `SA505EX,LABEL-?9?SA505EX,DISP-SHR`: Additional shipping analysis data file, shared mode.
     - `RKPRCEX,LABEL-?9?RKPRCEX,DISP-SHR`: Rack price extension file, shared mode.
     - `OUTFILE,LABEL-?9?TMPFILX,DISP-SHR`: Primary output file, shared mode.
     - `OUTFILE2,LABEL-?9?TMPFILY,DISP-SHR`: Secondary output file, shared mode.
   - `// RUN`: Executes `SA505H`, which processes input files to refine or summarize the Customer Shipping Analysis Report, writing results to `?9?TMPFILX` and `?9?TMPFILY`.

3. **Clear Local Data Area**:
   - `// LOCAL BLANK-*ALL`: Clears the Local Data Area (LDA) to reset parameters for subsequent steps or programs.

### Business Rules (Inferred)

1. **Purpose**: Refines or summarizes the Customer Shipping Analysis Report by processing data from `SA505G` (`?9?SA505G`, `?9?SA505G2`), integrating customer (`?9?ARCUST`), contract (`?9?BICONT`), rack pricing (`?9?RKPRCEX`), and additional sales data (`?9?SA505EX`), producing output files `?9?TMPFILX` and `?9?TMPFILY` for further analysis or reporting.
2. **Data Processing**:
   - Likely aggregates or formats data from `SA505G` outputs, combining customer, ship-to, and pricing details.
   - Produces two output files, possibly for different levels of detail (e.g., detailed transactions in `?9?TMPFILX` vs. summarized data in `?9?TMPFILY`) or different formats (e.g., for reporting or export).
3. **Context**: Part of the `PRICES.ocl36.txt` workflow, following `SA505G`, which integrates sales and pricing data. `SA505H` likely performs additional processing, such as formatting, filtering, or generating specific report outputs.
4. **File Management**:
   - Clears output files (`?9?TMPFILX`, `?9?TMPFILY`) to prevent data conflicts.
   - Uses shared mode (`DISP-SHR`) for all files to allow concurrent access.
5. **Cleanup**: Resets the LDA to ensure no residual parameters affect subsequent programs.

### External Programs Called

1. **SA505H**: RPG program that processes input files to generate or refine the Customer Shipping Analysis Report, writing to `?9?TMPFILX` and `?9?TMPFILY`.

### Tables (Files) Used

1. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input, shared mode (`DISP-SHR`).
   - **Purpose**: Customer master data for customer details (e.g., name, number).
2. **SA505GX** (`?9?SA505G`):
   - **Access**: Input, shared mode.
   - **Purpose**: Primary output from `SA505G`, containing summarized sales and pricing data.
3. **SA505G2** (`?9?SA505G2`):
   - **Access**: Input, shared mode.
   - **Purpose**: Secondary output from `SA505G`, likely a higher-level summary.
4. **BICONT** (`?9?BICONT`):
   - **Access**: Input, shared mode.
   - **Purpose**: Contract data for company or agreement details.
5. **SA505EX** (`?9?SA505EX`):
   - **Access**: Input, shared mode.
   - **Purpose**: Additional shipping analysis data, possibly an extension or summary.
6. **RKPRCEX** (`?9?RKPRCEX`):
   - **Access**: Input, shared mode.
   - **Purpose**: Rack price extension data, likely related to `?9?RKPRCE` from `BB953`.
7. **OUTFILE** (`?9?TMPFILX`):
   - **Access**: Output, shared mode.
   - **Purpose**: Primary output file for the refined report data.
8. **OUTFILE2** (`?9?TMPFILY`):
   - **Access**: Output, shared mode.
   - **Purpose**: Secondary output file, possibly for a different format or summary.

### Additional Notes

- **Context**: Invoked by `PRICES.ocl36.txt` after `SA505G`, building on its outputs (`?9?SA505G`, `?9?SA505G2`) and integrating rack pricing (`?9?RKPRCEX`) and customer data to produce refined outputs, complementing the rack pricing process (`BB953B`).
- **System/36 Environment**: Uses OCL and System/36 conventions, likely on AS/400.
- **Limitations**: Without the `SA505H` RPG source code, exact processing logic is inferred. It likely performs further summarization, formatting, or filtering of `SA505G` data, possibly for a specific report format or export (e.g., Excel).
- **Error Handling**: Relies on System/36 and `SA505H` for error handling, with file clearing to prevent data conflicts.
- **Relation to Other Programs**: Depends on `SA505G` outputs and complements `SA505C`, indicating a multi-step process for generating comprehensive shipping analysis reports.

If you have the RPG source code for `SA505H`, `SA505E`, `SA505I`, `SA505J`, or `SA505L`, or need further analysis of the pricing workflow, please provide those details! Let me know if you have additional questions or files to share.