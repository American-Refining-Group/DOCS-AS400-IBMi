Below is a table that lists the programs from the provided documents (`AR745P.ocl36.txt`, `AR745P.rpgle.txt`, `AR745.ocl36.txt`, `AR745.rpg36.txt`, and `AR745A.ocl36.txt`) as rows and the tables (files) used as columns. At each intersection, a brief description indicates how the program modifies or creates/writes to the table. If a program does not interact with a specific table, the cell is marked with "-". The tables include all explicitly mentioned files and implied files (e.g., `ARDETL` in `AR745A` via queries).

| **Program** | **ARCONT** | **ARCUST** | **GSCONT** | **AR745PD** | **ARDETL** | **AR745S** | **AREFTD** | **LIST** | **LIST1** | **LIST2** | **LIST3** | **?9?ARINVCR** | **QPQUPRFIL** |
|-------------|------------|------------|------------|-------------|------------|------------|------------|----------|----------|----------|----------|----------------|---------------|
| **GSGENIEC** | - | - | - | - | - | - | - | - | - | - | - | - | - |
| **AR745P.ocl36.txt** | Reads for validation | Reads for validation | - | - | - | - | - | - | - | - | - | - | - |
| **AR745P.rpgle.txt** | Chains to validate company number | Chains to validate customer and EFT flag | Chains to retrieve default company number | Reads/writes for user input and error display | - | - | - | - | - | - | - | - | - |
| **#GSORT** | - | - | - | - | Reads for sorting | Creates and writes sorted records | - | - | - | - | - | - | - |
| **AR745.ocl36.txt** | Reads for company data | Reads for customer data | - | - | Reads as input for sorting | Creates and reads as sorted input | Creates (batch mode) and reads | Writes report headers | - | - | - | - | - |
| **AR745.rpg36.txt** | Chains for company data and G/L codes | Chains for customer data and EFT validation | - | - | Reads sorted data (via `AR745S`) | Reads as primary input | Writes EFT transaction data | Writes minimal headers | Writes detailed report headers/lines | Writes summary report | Writes detailed report | - | - |
| **AR745A.ocl36.txt** | - | - | - | - | Implied read via queries for credit invoices | - | - | - | - | - | - | Reads condition for credit invoices | Writes credit invoice report |

### Notes on Table Interactions
- **GSGENIEC**: No explicit file interactions; likely updates system control areas (not listed as tables).
- **AR745P.ocl36.txt**: Reads `ARCONT` and `ARCUST` for validation during job setup.
- **AR745P.rpgle.txt**: Interacts with `AR745PD` (workstation display) for input/output, chains to `ARCONT`, `ARCUST`, and `GSCONT` for validation and default values.
- **#GSORT**: Reads `ARDETL`, creates and writes to `AR745S` with sorted records.
- **AR745.ocl36.txt**: Manages file creation (`AREFTD` in batch mode), reads `ARDETL`, `AR745S`, `ARCUST`, `ARCONT`, and writes to `LIST` for report headers.
- **AR745.rpg36.txt**: Reads `AR745S` (sorted `ARDETL`), chains to `ARCUST` and `ARCONT`, writes to `LIST`, `LIST1`, `LIST2`, `LIST3` for reports, and `AREFTD` for EFT data.
- **AR745A.ocl36.txt**: Reads `?9?ARINVCR` for credit invoice condition, writes to `QPQUPRFIL` for report output, and implies access to `ARDETL` via queries.

### Table Definitions
- **ARCONT**: Accounts receivable control file (company data, G/L codes).
- **ARCUST**: Customer master file (customer details, EFT flag).
- **GSCONT**: Control file for default company number.
- **AR745PD**: Workstation display file for user interaction.
- **ARDETL**: Accounts receivable detail file (invoices).
- **AR745S**: Sorted work file for EFT report data.
- **AREFTD**: EFT output file (bank upload) and temporary batch files.
- **LIST, LIST1, LIST2, LIST3**: Printer output files for reports.
- **?9?ARINVCR**: Data area for credit invoice condition.
- **QPQUPRFIL**: Printer file for credit invoice report.

If you need further clarification, additional details on specific interactions, or inclusion of other implied files, let me know!