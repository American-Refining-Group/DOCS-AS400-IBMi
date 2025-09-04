The provided document, `GSDT12.ocl36.txt`, is a System/36 OCL (Operation Control Language) script used in an IBM System/36 environment (or AS/400 in compatibility mode). It is referenced in the `SA505C.ocl36.txt` script as part of the pricing generation process for blended lubes within the `PRICES.ocl36.txt` workflow, invoked by `PRICEGEN.clp`. The script executes the `GSDT12` program to prompt for or process date-related data, likely setting up date parameters for the Customer Shipping Analysis Report. Below, I will explain the process steps, business rules, external programs called, and tables (files) used.

### Process Steps of the OCL Script

1. **Load and Run GSDT12 Program**:
   - `// LOAD GSDT12`: Loads the `GSDT12` RPG or System/36 program .
   - File specified:
     - `ARCONT,LABEL-?9?ARCONT,DISP-SHR`: Contract master file, mapped to `?9?ARCONT` (e.g., `AARCONT`), accessed in shared mode.
   - `// RUN`: Executes `GSDT12`, which processes the `?9?ARCONT` file to prompt for or set date parameters, likely updating the Local Data Area (LDA) with date values for subsequent programs (e.g., `SA505C`).

### Business Rules (Inferred)

1. **Purpose**: Facilitates date prompting or validation for the Customer Shipping Analysis Report, likely setting date range parameters (e.g., `KYFRDT`, `KYTODT`) in the LDA for use by programs like `SA505C`, `SA505X`, and others in the `PRICES.ocl36.txt` workflow.
2. **Data Processing**:
   - Uses `?9?ARCONT` to retrieve contract-related data, possibly for company-specific date ranges or validation.
   - Likely prompts the user for start and end dates or retrieves default dates, storing them in the LDA (e.g., at offsets specified in `SA505C.ocl36.txt`).
3. **Context**: Called early in `SA505C.ocl36.txt` with parameter `'G'`, before sorting and processing sales data, indicating its role in initializing date parameters for the workflow.
4. **File Management**:
   - Accesses `?9?ARCONT` in shared mode (`DISP-SHR`) to allow concurrent access.
   - Does not specify output files, suggesting output is written to the LDA or a display/printer.
5. **Output**: Updates the LDA with date parameters or displays prompts, with no direct file output specified.

### External Programs Called

1. **GSDT12**: RPG or System/36 program that handles date prompting or processing, likely updating the LDA with date parameters.

### Tables (Files) Used

1. **ARCONT** (`?9?ARCONT`):
   - **Access**: Input, shared mode (`DISP-SHR`).
   - **Purpose**: Contract master file, likely containing company-specific data used for date validation or default settings.

### Additional Notes

- **Context**: Invoked by `SA505C.ocl36.txt` with parameter `'G'`, part of the `PRICES.ocl36.txt` workflow, setting up date parameters for the Customer Shipping Analysis Report, complementing rack pricing (`BB953B`) and blended lubes pricing (`BI942E`).
- **System/36 Environment**: Uses OCL and System/36 conventions, likely on AS/400.
- **Limitations**: Without the `GSDT12` RPG source code, exact logic is inferred. It likely interacts with the user or `?9?ARCONT` to set date parameters (e.g., `KYFRDT`, `KYTODT`) used in filtering sales data.
- **Error Handling**: Relies on System/36 and `GSDT12` for error handling, with no explicit file cleanup or LDA reset.
- **Relation to Other Programs**: Provides date parameters for `SA505C`, `SA505X`, and subsequent programs, ensuring consistent date filtering across the workflow.

If you have the RPG source code for `GSDT12`, `SA505E`, `SA505J`, or `SA505L`, or need further analysis of the pricing workflow, please provide those details! Let me know if you have additional questions or files to share.