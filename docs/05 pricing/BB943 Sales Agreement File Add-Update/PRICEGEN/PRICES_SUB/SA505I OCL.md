The provided document, `SA505I.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is part of the pricing generation process for blended lubes, invoked by `PRICES.ocl36.txt` within the `PRICEGEN.clp` workflow, following `SA505H`. The script executes the `SA505I` program to process data from previous steps, particularly the output from `SA505H` (`?9?TMPFILX`), and produces a Customer Shipping Analysis Report or intermediate data, leveraging various master files for additional details. Below, I will explain the process steps, business rules, external programs called, and tables (files) used.

### Process Steps of the OCL Script

1. **Run GSY2K Procedure**:
   - `// GSY2K`: Calls the System/36 procedure `GSY2K`, which likely initializes the environment or handles Y2K date configurations (also used in `PRICEGEN.clp`, `BI944B`, `BI942E`, `BB953B`, and `SA505C`).

2. **Load and Run SA505I Program**:
   - `// LOAD SA505I`: Loads the `SA505I` RPG program.
   - Files specified:
     - `TMPFILX,LABEL-?9?TMPFILX,DISP-SHR`: Primary output file from `SA505H`, shared mode.
     - `GSCNTR,LABEL-?9?GSCNTR,DISP-SHR`: Container master file, shared mode.
     - `BICONT,LABEL-?9?BICONT,DISP-SHR`: Contract file, shared mode.
     - `GSCTUM,LABEL-?9?GSCTUM,DISP-SHR`: Unit of measure conversion file, shared mode.
     - `GSCTWT,LABEL-?9?GSCTWT,DISP-SHR`: Container weight file, shared mode.
     - `GSUMCV,LABEL-?9?GSUMCV,DISP-SHR`: Unit of measure conversion table, shared mode.
     - `ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Customer master file, shared mode.
     - `PRSABLW,LABEL-?9?PRSABLW,DISP-SHR`: Pricing file for blended lubes, shared mode (likely from `BI942E`).
     - `GSPROD,LABEL-?9?GSPROD,DISP-SHR`: Product master file, shared mode.
   - `// RUN`: Executes `SA505I`, which processes `?9?TMPFILX` and integrates data from master files to produce or refine the Customer Shipping Analysis Report. No explicit output file is specified, suggesting `SA505I` may update existing files or produce a printed report.

3. **No Cleanup Specified**:
   - Unlike previous scripts (e.g., `SA505C`, `SA505G`, `SA505H`), no `LOCAL BLANK-*ALL` or file deletion statements are included, implying the LDA and temporary files persist for subsequent steps or are managed by `SA505I`.

### Business Rules (Inferred)

1. **Purpose**: Processes the Customer Shipping Analysis Report data from `SA505H` (`?9?TMPFILX`), enriching it with container (`?9?GSCNTR`), unit of measure (`?9?GSCTUM`, `?9?GSUMCV`), weight (`?9?GSCTWT`), customer (`?9?ARCUST`), pricing (`?9?PRSABLW`), and product (`?9?GSPROD`) details, likely producing a report or updating files for further analysis.
2. **Data Processing**:
   - Integrates `?9?TMPFILX` (detailed sales and pricing data) with master files to add descriptions, conversions (e.g., unit of measure to gallons), or weight calculations.
   - Likely performs calculations for pricing or quantities, using `?9?GSCTUM` and `?9?GSUMCV` for unit conversions and `?9?GSCTWT` for weights.
3. **Context**: Part of the `PRICES.ocl36.txt` workflow, following `SA505H`, which refined data from `SA505G`. `SA505I` likely prepares final report data or intermediate files, complementing rack pricing (`BB953B`) and blended lubes pricing (`BI942E`).
4. **File Management**:
   - Uses shared mode (`DISP-SHR`) for all files to allow concurrent access.
   - Does not clear input or output files, suggesting reliance on `SA505H` for cleanup or that `SA505I` manages its own output.
5. **Output**: No explicit output file is defined, indicating `SA505I` may update `?9?TMPFILX`, produce a printed report (e.g., via a `LIST` printer file), or write to another file not specified in the OCL.

### External Programs Called

1. **GSY2K**: System/36 procedure for environment initialization or Y2K date handling.
2. **SA505I**: RPG program that processes input files to refine or generate the Customer Shipping Analysis Report.

### Tables (Files) Used

1. **TMPFILX** (`?9?TMPFILX`):
   - **Access**: Input, shared mode (`DISP-SHR`).
   - **Purpose**: Detailed sales and pricing data from `SA505H`.
2. **GSCNTR** (`?9?GSCNTR`):
   - **Access**: Input, shared mode.
   - **Purpose**: Container master data for container descriptions.
3. **BICONT** (`?9?BICONT`):
   - **Access**: Input, shared mode.
   - **Purpose**: Contract data for company details (e.g., name).
4. **GSCTUM** (`?9?GSCTUM`):
   - **Access**: Input, shared mode.
   - **Purpose**: Unit of measure conversion data.
5. **GSCTWT** (`?9?GSCTWT`):
   - **Access**: Input, shared mode.
   - **Purpose**: Container weight data for weight calculations.
6. **GSUMCV** (`?9?GSUMCV`):
   - **Access**: Input, shared mode.
   - **Purpose**: Additional unit of measure conversion table.
7. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input, shared mode.
   - **Purpose**: Customer master data for customer details (e.g., name).
8. **PRSABLW** (`?9?PRSABLW`):
   - **Access**: Input, shared mode.
   - **Purpose**: Pricing data for blended lubes, likely from `BI942E`.
9. **GSPROD** (`?9?GSPROD`):
   - **Access**: Input, shared mode.
   - **Purpose**: Product master data for product descriptions.

### Additional Notes

- **Context**: Invoked by `PRICES.ocl36.txt` after `SA505H`, building on its output (`?9?TMPFILX`) to produce a refined Customer Shipping Analysis Report, integrating additional master data for containers, units, weights, and products.
- **System/36 Environment**: Uses OCL and System/36 conventions, likely on AS/400.
- **Limitations**: Without the `SA505I` RPG source code, exact processing logic is inferred. It likely enriches `?9?TMPFILX` with descriptions, performs unit conversions, or calculates weights, possibly producing a final report or updating files.
- **Error Handling**: Relies on System/36 and `SA505I` for error handling, with no explicit file clearing in the OCL.
- **Relation to Other Programs**: Depends on `SA505H` output (`?9?TMPFILX`) and integrates with pricing (`?9?PRSABLW`) and rack pricing data (via prior steps), indicating a multi-step workflow.

If you have the RPG source code for `SA505I`, `SA505E`, `SA505J`, or `SA505L`, or need further analysis of the pricing workflow, please provide those details! Let me know if you have additional questions or files to share.