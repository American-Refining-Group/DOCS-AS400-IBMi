The provided document, `SA505G.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is part of the pricing generation process for blended lubes, invoked by `PRICES.ocl36.txt` within the `PRICEGEN.clp` workflow, following `SA505C`. The script generates a Customer Shipping Analysis Report by executing the `SA505G` program, which processes sales and pricing data to produce output files for analysis. Below, I will explain the process steps, business rules, external programs called, and tables (files) used.

### Process Steps of the OCL Script

1. **Clear Output Files (Conditional)**:
   - `// IF DATAF1-?9?SA505G CLRPFM ?9?SA505G`: If the output file `?9?SA505G` (e.g., `ASA505G`) exists, clears it to ensure a fresh dataset.
   - `// IF DATAF1-?9?SA505G2 CLRPFM ?9?SA505G2`: If the secondary output file `?9?SA505G2` (e.g., `ASA505G2`) exists, clears it.

2. **Load and Run SA505G Program**:
   - `// LOAD SA505G`: Loads the `SA505G` RPG program.
   - Files specified:
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file, shared mode.
     - `PRSABL,LABEL-?9?PRSABLW,DISP-SHR`: Pricing file for blended lubes, shared mode (likely from `BI942E` in the `PRICEGEN.clp` workflow).
     - `SHIPTO,LABEL-?9?SHIPTO,DISP-SHR`: Ship-to file, shared mode.
     - `BICONT,LABEL-?9?BICONT,DISP-SHR`: Contract file, shared mode.
     - `SA505CX,LABEL-?9?SA505CX,DISP-SHR`: Customer shipping analysis data file, shared mode (likely from `SA505C`).
     - `SA505EX,LABEL-?9?SA505EX,DISP-SHR`: Additional shipping analysis data file, shared mode.
     - `RKPRCEX,LABEL-?9?RKPRCEX,DISP-SHR`: Rack price extension file, shared mode.
     - `OUTFILE,LABEL-?9?SA505G,DISP-SHR`: Primary output file, shared mode.
     - `OUTFILE2,LABEL-?9?SA505G2,DISP-SHR`: Secondary output file, shared mode.
   - `// RUN`: Executes `SA505G`, which processes input files to generate the Customer Shipping Analysis Report, writing results to `?9?SA505G` and `?9?SA505G2`.

3. **Clear Local Data Area**:
   - `// LOCAL BLANK-*ALL`: Clears the Local Data Area (LDA) to reset parameters for subsequent steps or programs.

### Business Rules (Inferred)

1. **Purpose**: Generates a Customer Shipping Analysis Report by processing sales and pricing data (`?9?SA505CX`, `?9?SA505EX`, `?9?PRSABLW`, `?9?RKPRCEX`) and customer/ship-to details (`?9?ARCUST`, `?9?SHIPTO`, `?9?BICONT`), producing output files `?9?SA505G` and `?9?SA505G2` for analysis or reporting.
2. **Data Processing**:
   - Combines customer, ship-to, contract, and pricing data to analyze shipping patterns, likely cross-referencing sales data with rack prices.
   - Produces two output files, possibly for different report formats (e.g., detailed vs. summary) or data export (e.g., Excel).
3. **Context**: Part of the `PRICES.ocl36.txt` workflow, following `SA505C`, which generates initial shipping analysis data. `SA505G` likely refines or summarizes this data, integrating rack prices (`?9?RKPRCEX`) and blended lubes pricing (`?9?PRSABLW`).
4. **File Management**:
   - Clears output files (`?9?SA505G`, `?9?SA505G2`) to prevent data conflicts.
   - Uses shared mode (`DISP-SHR`) for all files to allow concurrent access.
5. **Cleanup**: Resets the LDA to ensure no residual parameters affect subsequent programs.

### External Programs Called

1. **SA505G**: RPG program that processes input files to generate the Customer Shipping Analysis Report, writing to `?9?SA505G` and `?9?SA505G2`.

### Tables (Files) Used

1. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input, shared mode (`DISP-SHR`).
   - **Purpose**: Customer master data for customer details (e.g., name, number).
2. **PRSABL** (`?9?PRSABLW`):
   - **Access**: Input, shared mode.
   - **Purpose**: Pricing data for blended lubes, likely from `BI942E`.
3. **SHIPTO** (`?9?SHIPTO`):
   - **Access**: Input, shared mode.
   - **Purpose**: Ship-to data for shipping location details.
4. **BICONT** (`?9?BICONT`):
   - **Access**: Input, shared mode.
   - **Purpose**: Contract data for company or agreement details.
5. **SA505CX** (`?9?SA505CX`):
   - **Access**: Input, shared mode.
   - **Purpose**: Customer shipping analysis data, likely from `SA505C`.
6. **SA505EX** (`?9?SA505EX`):
   - **Access**: Input, shared mode.
   - **Purpose**: Additional shipping analysis data, possibly an extension or summary.
7. **RKPRCEX** (`?9?RKPRCEX`):
   - **Access**: Input, shared mode.
   - **Purpose**: Rack price extension data, likely related to `?9?RKPRCE` from `BB953`.
8. **OUTFILE** (`?9?SA505G`):
   - **Access**: Output, shared mode.
   - **Purpose**: Primary output file for the report data.
9. **OUTFILE2** (`?9?SA505G2`):
   - **Access**: Output, shared mode.
   - **Purpose**: Secondary output file, possibly for a different format or summary.

### Additional Notes

- **Context**: Invoked by `PRICES.ocl36.txt` after `SA505C`, integrating pricing (`?9?PRSABLW`, `?9?RKPRCEX`) and shipping analysis data (`?9?SA505CX`, `?9?SA505EX`) to produce a comprehensive report, complementing the rack pricing process (`BB953B`).
- **System/36 Environment**: Uses OCL and System/36 conventions, likely on AS/400.
- **Limitations**: Without the `SA505G` RPG source code, exact processing logic is inferred. It likely aggregates sales, pricing, and customer data, producing detailed and summary outputs.
- **Error Handling**: Relies on System/36 and `SA505G` for error handling, with file clearing to prevent data conflicts.
- **Relation to Other Programs**: Builds on `SA505C` output (`?9?SA505CX`) and rack pricing data (`?9?RKPRCEX`, possibly derived from `?9?RKPRCE`), indicating a dependency on prior steps in the workflow.

If you have the RPG source code for `SA505G`, `SA505E`, `SA505H`, `SA505I`, `SA505J`, or `SA505L`, or need further analysis of the pricing workflow, please provide those details! Let me know if you have additional questions or files to share.