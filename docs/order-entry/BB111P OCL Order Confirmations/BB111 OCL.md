### Process Steps of the OCL Program BB111

This OCL (Operation Control Language) script for IBM i/AS/400 handles the printing of order confirmations and related documents (e.g., pick lists, packaging lists) from pre-selected order data. It prepares data through sorting and preprocessing, sets up printer overrides, runs the main print program, and optionally processes spool files for server copying (via Spoolflex). The script uses conditional logic based on parameters like `?9?` (library prefix), `?13?` (mode, e.g., 'PP' for pick print), and switches. It assumes input files like BBOCFR (from prior selection) contain sorted order records. Below is a step-by-step breakdown:

1. **Initial Setup and File Check**:
   - Sets local offsets: 470 to '?13?' (mode or parameter), 480 to '?9?' (library prefix).
   - Checks if file `?9?BBOCFH` exists and has records (`?F'A,?9?BBOCFH'?/0`); if empty (0 records), jumps to "END" (skips processing, as no orders to print).
   - Calls `GSY2K` (likely a year-2000 date compliance routine).
   - Calls `STRPCOCLP` (starts PC organizer or client access process, possibly for integration).

2. **Conditional Local Data Setup**:
   - If location `?L'103,3'?` is 'SEL', sets offset 1 to 'O COAC' (possibly a sort or output option for selected orders); otherwise, sets to 'O*CO*C' (default company/output code).

3. **Initial Sorting of Order Records**:
   - Loads sort utility `#GSORT`.
   - Defines input file `?9?BBOCFR` (shared), output `?9?BB111S` (extendable, job-retained).
   - Runs sort with specifications:
     - Header: SORTR 15A, 3X 512 N (ascending sort on 512-byte records).
     - Input conditions: Exclude if position 1='D' (deleted?), include if positions 2-3='C10' (company-specific?).
     - Fields: Company (2-3), Order (4-9), Seq# (10-12), with special handling for positions 121-123 (record types: header first, then marks, details last).
     - Forces header records first by sorting on record type markers.
   - Ends sort.

4. **Preprocess for Pick List Sorting (JB01 Section)**:
   - Calls `GSDELETE` to delete `?9?BB1103` if exists.
   - Creates temporary physical file `QTEMP/?9?BB1113` (record length 554).
   - Loads program `BB1103` (preprocess for sorting detail records based on group-by in header).
   - Files: Input `?9?BB111S` (from prior sort), output `?9?BB1113` (extendable), `?9?GSCNTR` (shared).
   - Runs `BB1103`.
   - Loads `#GSORT` again.
   - Input `?9?BB1113`, output `?9?BB1113S` (extendable).
   - Sort specs: SORTR 42A, 3X 553 N.
     - Conditions: Exclude pos 1='D', include pos 554='C'.
     - Fields: Company (2-3), Order (4-9), Header first (513), SORT1 asc (524-533), SORT2 desc (534-543), SORT3 asc (544-553), Seq# (10-12).
   - Ends sort.
   - Deletes `?9?BB1113` (cleanup temporary).

5. **Main Print Program Load and File Setup**:
   - Loads program `BB110` (core printing logic for confirmations/picks).
   - Defines numerous input files (all shared DISP-SHR): `?9?BB1113S` (sorted input), `?9?BBOCFR` (input/output), `?9?BBORCL`, `?9?ARCUST`, `?9?ARCUPR`, `?9?SHIPTO`, `?9?GSCONT`, `?9?GSTABL`, `?9?GSUMCV`, `?9?GSCTWT`, `?9?GSCTUM`, `?9?BICONT`, `?9?GSHAZM`, `?9?BBFRPR`, `?9?BBSRNH`, `?9?CUADR`, `?9?SHPADR`, `?9?EDICUS`, `?9?BBASND`, `?9?BBSHPH`, `?9?BBSHPD`, `?9?BBORHS1`, `?9?BBORDS1`, `?9?BBORA`, `?9?BBORHS1` (duplicate), `?9?BBORDS1` (duplicate), `?9?BBFPORH`, `?9?BBFPORH` (duplicate), `?9?BBFPORD`, `?9?BBFPORA`, `?9?BBORDB`, `?9?BBORA` (duplicate), `?9?BBORF`, `?9?BBFPORF`, `?9?ARCUFMX`, `?9?GSCNTR`, `?9?GSCNTR1`, `?9?BBOCFR` (duplicate as BBORTR3), `?9?GSPROD`, `?9?GSPRCT`, `?9?GSMLCD`, `?9?BBCAID`.

6. **Printer Overrides and Setup**:
   - Conditional on `?9?/G` (likely production library) and `?13?/PP` (pick print mode):
     - Overrides printer file `LIST` (form type PICK, CPI 10, output queue varies, drawer 1, align yes).
     - Similar for `LIST2` (added 10/24/2013 for packaging, form PKGP).
     - For `LIST3` (added for order confirmations, form ORCF, CPI 15, output queues like ORDCONOUTQ or TESTOUTQ).
   - These ensure correct form alignment, compression, and routing for different document types (picks, packaging, confirmations).

7. **Run Main Print Program**:
   - Runs `BB110` (prints confirmations/picks using prepared files).

8. **Spoolflex Processing**:
   - If `?9?/G`, `?13?/PP`, and switch bit 6 is 1: Calls `BB111AC` (Spoolflex to copy picks to server; errors pop up on user workstation if PDF is open).

9. **Termination (TAG END)**:
   - If `?10?/` is true (possibly a cancel or end flag), blanks all local variables and resets switches to 00000000 (cleanup).

The script is batch-oriented, assuming prior selection populated BBOCFH/BBOCFR. It emphasizes sorting for proper order (headers first, details last) and conditional printing based on environment/mode.

### Business Rules

- **Data Preparation**: Orders must be sorted by company, order#, sequence, with headers prioritized (via record type markers in pos 121-123/513). Deleted records ('D' in pos 1) are excluded. Custom sorts (SORT1 asc, SORT2 desc, SORT3 asc) apply to details, likely for bin/location grouping in pick lists.
- **File Existence Check**: No processing if BBOCFH empty (business: Avoid unnecessary runs/printer waste).
- **Environment-Specific Handling**: In production (`?9?/G`), uses specific output queues (e.g., PICKNWOUTQ, ORDCONOUTQ); test uses TESTOUTQ. Pick mode (`?13?/PP`) triggers pick/pack forms and Spoolflex.
- **Printer Configuration**: Forms must align (ALIGN-YES), CPI set for readability (10 for picks/pack, 15 for confirmations). Drawer 1 default.
- **Spoolflex Rule**: Only in live environment, for server PDF copying; user must close conflicting files (error popup).
- **Temporary Files**: Created in QTEMP, deleted post-use (cleanup to avoid clutter).
- **Record Types**: Forces headers > marks > details in output for coherent printing.
- **Assumptions**: Relies on prior programs (e.g., BB112) to populate selection files. No direct updates; read-only shared access.

### Tables Used

The script interacts with the following physical files (tables) via FILE NAME- specifications (all shared DISP-SHR unless noted). Many are inputs for lookup/printing; some are outputs or temporaries:

- `?9?BBOCFH`: Checked for records (input).
- `?9?BBOCFR`: Input for sorting, also as BBORTR/BBORTO/BBORTR3 (input/output).
- `?9?BB111S`: Sorted output from first sort (input to preprocess).
- `QTEMP/?9?BB1113`: Temporary created (output from BB1103, input to second sort).
- `?9?BB1113S`: Sorted output from second sort (input to BB110).
- `?9?GSCNTR`: Shared input (country data, multiple uses).
- `?9?BBORCL`: Shared input (order control).
- `?9?ARCUST`: Shared input (customers).
- `?9?ARCUPR`: Shared input (customer pricing?).
- `?9?SHIPTO`: Shared input (ship-to addresses).
- `?9?GSCONT`: Shared input (contacts?).
- `?9?GSTABL`: Shared input (tables/lookups).
- `?9?GSUMCV`: Shared input (unit conversions?).
- `?9?GSCTWT`: Shared input (carton weights?).
- `?9?GSCTUM`: Shared input (carton UOM?).
- `?9?BICONT`: Shared input (bill-to contacts?).
- `?9?GSHAZM`: Shared input (hazmat data).
- `?9?BBFRPR`: Shared input (freight pricing?).
- `?9?BBSRNH`: Shared input (serial numbers?).
- `?9?CUADR`: Shared input (customer addresses).
- `?9?SHPADR`: Shared input (ship addresses).
- `?9?EDICUS`: Shared input (EDI customers).
- `?9?BBASND`: Shared input (ASN data?).
- `?9?BBSHPH`: Shared input (ship headers?).
- `?9?BBSHPD`: Shared input (ship details?).
- `?9?BBORHS1`: Shared input (order headers, duplicate).
- `?9?BBORDS1`: Shared input (order details, duplicate).
- `?9?BBORA`: Shared input (order attachments?, duplicate).
- `?9?BBFPORH`: Shared input (freight PO headers, duplicate).
- `?9?BBFPORD`: Shared input (freight PO details).
- `?9?BBFPORA`: Shared input (freight PO attachments).
- `?9?BBORDB`: Shared input (order rebates? as BBORDRB).
- `?9?BBORF`: Shared input (order freight?).
- `?9?BBFPORF`: Shared input (freight PO freight).
- `?9?ARCUFMX`: Shared input (customer format matrix?).
- `?9?GSCNTR1`: Shared input (country alternate?).
- `?9?GSPROD`: Shared input (products).
- `?9?GSPRCT`: Shared input (product categories?).
- `?9?GSMLCD`: Shared input (multi-location codes?).
- `?9?BBCAID`: Shared input (carrier IDs).

Printer files: LIST (picks), LIST2 (packaging), LIST3 (confirmations) – overridden but not traditional tables.

### External Programs Called

The script calls or loads the following external programs/utilities/commands:

- `GSY2K`: Called for date handling.
- `STRPCOCLP`: Called to start PC-related process.
- `#GSORT`: Loaded and run twice (system sort utility).
- `GSDELETE`: Called to delete files (custom command?).
- `BB1103`: Loaded and run for preprocess sorting.
- `BB110`: Loaded and run for main printing.
- `BB111AC`: Called conditionally for Spoolflex.

System commands: CRTPF (create physical file), OVRPRTF (override printer file), IFF/IF (conditionals), PRINTER (printer setup, possibly custom).