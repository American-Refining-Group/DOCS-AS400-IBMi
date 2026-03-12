### Process Steps of the OCL Program BB114O

This OCL (Operation Control Language) program is designed for printing proforma invoices on an IBM midrange system (e.g., AS/400 or iSeries), including data preparation via sorting, temporary file creation, and printer overrides. It generates printed output (or spool files) for proforma invoices without emailing, and includes post-processing for spool file splitting, PDF conversion via Spoolflex-like tools, and archiving. The program is conditional on file existence and switches, and is called from the main OCL (e.g., during proforma invoice selection in BB114). It uses variables like ?9? (library prefix), ?13? (possibly a print parameter), and ?10? (possibly a termination flag). Below is a step-by-step breakdown of the process:

1. **Initial Setup and Validation**:
   - Sets local offsets: Offset 470 to '?13?' (print-related parameter) and 480 to '?9?' (library prefix).
   - Checks if file ?9?BBPROH exists and has records (?F'A,?9?BBPROH'?/0); if empty (0 records), jumps to END (terminates early, as no data to process).

2. **System Routines and Conditional Local Setup**:
   - Calls GSY2K (likely a Y2K date compliance routine).
   - Calls STRPCOCLP (possibly starts a PC-related OCL processor or printer control).
   - Checks ?L'103,3'?/SEL: If 'SEL', sets local offset 1 to 'O COAC' (possibly an output or company access code); otherwise, sets to 'O*CO*C' (default company code).

3. **File Cleanup and Initial Sorting**:
   - Deletes files BB1143, BB114S, BB1143S in library ?9? (prepares for fresh data).
   - Loads #GSORT (a sorting utility program).
   - Defines input file as ?9?BBPROR (shared) and output as ?9?BB114S (extendable, retained as job).
   - Runs #GSORT: Sorts records where position 1 ≠ 'D' (not deleted) and positions 2-3 = 'C10' (specific code), by company (pos 2-3), order (pos 4-9), seq# (pos 10-12). Outputs full records (pos 1-512).

4. **Temporary File Creation and Preprocessing for Detail Sorting (JB01 Section)**:
   - Deletes BB1143 in ?9?.
   - Creates physical file QTEMP/?9?BB1143 with record length 554.
   - Loads BB1143 (a program for preprocessing pick list details).
   - Defines files: Input BBORTR as ?9?BB114S (shared), output BB1143 as ?9?BB1143 (extendable), GSCNTR as ?9?GSCNTR (shared).
   - Runs BB1143: Processes sorted data, likely adding or transforming fields (e.g., grouping by header fields, using GSCNTR for country data).

5. **Secondary Sorting on Preprocessed Data**:
   - Loads #GSORT again.
   - Defines input as ?9?BB1143 (shared) and output as ?9?BB1143S (extendable, retained).
   - Runs #GSORT: Sorts records where pos 1 ≠ 'D' (not deleted), by company (pos 2-3), order (pos 4-9), header priority (pos 513 to ensure header first), custom sort fields (pos 524-533 asc, 534-543 desc, 544-553 asc from BB1101 logic), seq# (pos 10-12). Outputs extended records (pos 1-553).
   - Deletes BB1143 in ?9? (cleanup temp file).

6. **Load Main Printing Program (BB110)**:
   - Loads BB110 (core program for generating proforma invoice print output).
   - Defines numerous files (all shared/DISP-SHR): BBORTR as ?9?BB1143S (sorted details), BBORTU/BBORTO as ?9?BBPROR (unsorted/original), BBORCL (order control), ARCUST (customers), ARCUPR (customer pricing), SHIPTO (ship-to), GSCONT (contacts?), GSTABL (general tables), GSUMCV (unit conversions?), GSCTWT (carton weights?), GSCTUM (carton UOM?), BICONT (bill-to contacts?), GSHAZM (hazmat), BBFRPR (freight pricing?), BBSRNH (serial numbers?), CUADR (customer addresses), SHPADR (ship addresses), EDICUS (EDI customers?), BBASND (ASN data?), BBSHPH (ship headers?), BBSHPD (ship details?), BBOTHS1/BBORH1 as ?9?BBORHS1 (order headers), BBOTDS1/BBORD1 as ?9?BBORDS1 (order details), BBOTA1/BBORA1 as ?9?BBORA (order attachments?), BBFPORH1/BBFPORH as ?9?BBFPORH (proforma headers), BBFPORD (proforma details), BBFPORA (proforma attachments), BBORDRB as ?9?BBORDB (order rebates?), BBORF/BBFPORF (order/proforma footers?), ARCUFMX (customer formats?), GSCNTR/GSCNTR1 (countries), BBORTR3 as ?9?BBPROR (another ref), GSPROD (products), GSPRCT (product categories?), GSMLCD (multi-codes?), BBCAID (carrier IDs).

7. **Printer Overrides for Output Files**:
   - Conditional on ?9?/G (library group 'G') and ?13?/PP (print parameter 'PP'):
     - Overrides LIST (pick list) to formtype PICK, CPI 10, outqueue QUSRSYS/PICKNWOUTQ or PRTPJ, drawer 1.
     - Overrides LIST2 (packaging) to formtype PKGP, CPI 10, outqueue QUSRSYS/PICKOUTQ or PRTPJ, drawer 1.
     - Overrides LIST3 (order confirmations) to formtype ORCF, CPI 15, outqueue QUSRSYS/ORDCONOUTQ or TESTOUTQ.
     - Overrides LIST4 (proforma invoices) to formtype PROF, CPI 15, outqueue QUSRSYS/PROFOROUTQ or TESTOUTQ.
   - These ensure correct formatting, alignment, and routing for printed/spooled output.

8. **Execute Main Printing and Check Switch**:
   - Runs BB110: Generates the proforma invoice print/spool files (LIST4 primary for proforma).
   - If switch 7 is off (SWITCH7-0), jumps to END (skips post-processing).

9. **Spool File Post-Processing**:
   - Calls SFASPLIT: Splits spool file named 'PROFORMA INV LIST4' from outqueue PROFOROUTQ, renames to 'PROFORMA INVOICE', no combine, moves to QUSRSYS/PROFOROQSV.
   - Calls FFAEDOC: Converts to e-doc named 'ARG PROFORMA INVOICE', to printer PROFPRTR, selects from PROFOROUTQ, renames, moves to PROFOROQSV.
   - Calls SFACOPY: Copies spool 'PROFORMA INV LIST4' from PROFOROUTQ, no combine, moves to PROFOROQSV, renames to 'PROFORMA NAMING EMAL' (possibly for naming/email prep, though no emailing).

10. **Termination**:
    - Jumps to END tag.
    - If ?10?/ is true, blanks all local variables and resets switches to 00000000 (cleanup).

The program flows sequentially with conditionals for early exit or skips, focusing on data sorting/grouping before printing.

### Business Rules

- **Data Preparation and Filtering**: Assumes input from BBPROH/BBPROR (proforma headers/records); skips if no data. Sorts to group headers first, then details/marks, using custom fields (e.g., from BB1101 for sorting logic). Excludes deleted records (pos 1 ≠ 'D').
- **Grouping and Sorting**: Initial sort by company/order/seq; secondary by group fields in header (ensures header precedes details). Uses extended records for additional sort keys.
- **Printing and Output Routing**: Overrides based on library group (?9?/G) and print type (?13?/PP) to route to specific queues (e.g., PROFOROUTQ for proforma, TESTOUTQ for testing). Uses specific form types (PROF for proforma), CPI (15 for proforma), and drawers.
- **Post-Print Handling**: Only if switch 7 on; splits/converts spool to PDF-like e-docs, renames, and archives to save queues (PROFOROQSV). No emailing, but naming suggests prep for it.
- **Temporary Files**: Creates/deletes QTEMP files for intermediate processing to avoid permanent data changes.
- **Error/Skip Conditions**: Early exit if no input data; switch 7 controls post-processing (off skips PDF/spool handling).
- **Multi-Environment Support**: File labels prefixed with ?9? allow dynamic library usage (e.g., G vs. Z groups implied from prior programs).

### Tables/Files Used

These are physical/logical files explicitly defined or referenced (all with DISP-SHR unless noted; prefixed with ?9? unless QTEMP):

- BBPROH (checked for existence/records).
- BBPROR (input for sorting; also as BBORTU, BBORTO, BBORTR3).
- BB114S (sorted output).
- BB1143 (temp created in QTEMP; extended output).
- BB1143S (sorted output).
- GSCNTR (countries; shared).
- BBORCL (order control).
- ARCUST (customers).
- ARCUPR (customer pricing).
- SHIPTO (ship-to).
- GSCONT (contacts?).
- GSTABL (general tables).
- GSUMCV (unit conversions?).
- GSCTWT (carton weights?).
- GSCTUM (carton UOM?).
- BICONT (bill-to contacts?).
- GSHAZM (hazmat).
- BBFRPR (freight pricing?).
- BBSRNH (serial numbers?).
- CUADR (customer addresses).
- SHPADR (ship addresses).
- EDICUS (EDI customers?).
- BBASND (ASN data?).
- BBSHPH (ship headers?).
- BBSHPD (ship details?).
- BBORHS1 (order headers; as BBOTHS1, BBORH1).
- BBORDS1 (order details; as BBOTDS1, BBORD1).
- BBORA (order attachments?; as BBOTA1, BBORA1).
- BBFPORH (proforma headers; as BBFPORH1).
- BBFPORD (proforma details).
- BBFPORA (proforma attachments).
- BBORDB (order rebates?; as BBORDRB).
- BBORF (order footers).
- BBFPORF (proforma footers).
- ARCUFMX (customer formats?).
- GSCNTR1 (countries alt).
- GSPROD (products).
- GSPRCT (product categories?).
- GSMLCD (multi-codes?).
- BBCAID (carrier IDs).

Printer files: LIST (pick list), LIST2 (packaging), LIST3 (order confirmations), LIST4 (proforma invoices).

### External Programs Called

- GSY2K: Called for date/Y2K handling.
- STRPCOCLP: Called to start PC/OCL processing.
- #GSORT: Loaded and run twice (sorting utility).
- BB1143: Loaded and run for preprocessing details.
- BB110: Loaded and run for main printing.
- SFASPLIT: Called for spool splitting.
- FFAEDOC: Called for e-doc/PDF conversion.
- SFACOPY: Called for spool copying/renaming.