The provided document, `SA505X.rpg36.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `SA505X`. It is invoked by the OCL script `SA505C.ocl36.txt` as part of the Customer Shipping Analysis Report process within the `PRICES.ocl36.txt` workflow, which is itself called by `PRICEGEN.clp`. The program processes sorted sales data from `SA5FILD` (`?9?S5505S`), filtering records based on freight code (`KYFRCD`) and carrier code (`KYCACD`) to exclude non-matching records, updating the file in place. Below, I will explain the process steps, business rules, tables used, and external programs called, despite the truncation in the document.

### Process Steps of the RPG Program

1. **File Definitions**:
   - **Input/Output File**:
     - `SA5FILD` (Update Primary, `UP`, 1029 bytes, disk): Sorted sales data file, mapped to `?9?S5505S` (e.g., `AS5505S`), used for both reading and updating records.
   - **Input File**:
     - `SA5SHX` (Input, `IF`, 1500 bytes, indexed with 15-byte key, disk): Sales history index file, mapped to `?9?SA5SHX` (e.g., `ASA5SHX`), used for chaining to retrieve additional data.

2. **Input Specifications**:
   - **SA5FILD** (record type `NS 01`):
     - `ARCKEY` (2–9): Key field (likely company + customer).
     - `SHKEY` (2–16): Key field for chaining to `SA5SHX` (likely company + customer + invoice).
     - `SACO#` (2–3): Company number.
     - `SACUST` (4–9): Customer number.
     - `SAINV#` (10–16): Invoice number.
     - `SASHIP` (17–19): Ship-to number.
     - `SASLMN` (20–21): Salesman code.
     - `SAINDT` (22–27): Invoice date (YMD).
     - `SAQTY3` (28–34): Ship quantity (billing gallons).
     - `SAPRCE` (35–39, packed, 4 decimals): Price.
     - `SACOST` (40–44, packed, 4 decimals): Cost.
     - `SALOC` (45–47): Location.
     - `SAPROD` (48–51): Product code.
     - `SATANK` (52–55): Tank code.
     - `SAITEM` (45–59): Item number (location/product/tank).
     - `SATYPE` (60): Type (`' '`, `M`, `C`, `R`).
     - `SAORD#` (61–66): Order number.
     - `SASEQ` (67–69): Order sequence number.
     - `SASHDT` (70–75): Ship date (YMD).
     - `SAUM` (76–78): Unit of measure.
     - `SATIME` (79–82): Time of sale (HHMM).
     - `SAPRGP` (83–84): Product group.
     - `SADESC` (87–101): Description.
     - `SAGGAL` (102–105, packed, 4 decimals): Gross gallons.
     - `SANGAL` (106–109, packed, 4 decimals): Net gallons.
     - `SATEMP` (110–112): Temperature.
     - `SAGRAV` (113–115): Gravity.
     - `SAGLCD` (116): Gallons code (`G` or `N`).
     - `SARKPR` (118–122, packed, 4 decimals): Rack price.
     - `SAPRCD` (123): Price code (`R`, `A`, `T`, `S`, `O`, `P`).
     - `SHFRCD` (193–194): Freight code.
     - `SHCACD` (195–197): Carrier code.
     - `SHDLTM` (1053–1067): Delivery time description.
     - `SHTKBY` (1068–1070): Order taken by (initials).
     - `SHCNCT` (1071–1100): Customer contact.
     - `SHSFRT` (1101): Separate freight flag.
     - `SHRRCO` (1102–1103): Auto receipts company number.
     - `SHEDIS` (1104): Send invoice EDI flag.
     - `SHCAFR` (1105): Calculate freight flag (`Y` or `N`).
     - `SHOONO` (1106–1111): Original order number.
     - `SHCAID` (1112–1117): Carrier ID.
     - `SHRTCD` (1118–1121): Route code.
     - `SHTRK#` (1122–1146): Tracking number.
     - `SHFOBC` (1147): FOB code.
   - **User Data Structure (UDS)**:
     - `KYFRCD` (232): Freight code filter (blank or specific code).
     - `KYCACD` (233–234): Carrier code filter (blank or specific code).

3. **Calculation Specifications**:
   - **Main Loop**:
     - Processes each `SA5FILD` record (`01 DO`, implied).
   - **Chaining to SA5SHX**:
     - `SHKEY CHAINSA5SHX 98`: Chains to `SA5SHX` using `SHKEY` (company + customer + invoice) to retrieve additional data. If no record is found, sets indicator `98` and proceeds to filtering logic.
   - **Filtering Logic** (within `N98 DO` block):
     - **Case 1: Both KYFRCD and KYCACD blank**:
       - If `KYFRCD = *BLANKS` and `KYCACD = *BLANKS`, skips to `END` tag (no filtering, record retained).
     - **Case 2: Both KYFRCD and KYCACD specified**:
       - If `KYFRCD ≠ *BLANKS` and `KYCACD ≠ *BLANKS`:
         - If `SHFRCD ≠ KYFRCD` or `SHCACD ≠ KYCACD`, executes `EXCPTDEL` to mark the record for deletion and skips to `END`.
     - **Case 3: KYFRCD blank, KYCACD specified**:
       - If `KYFRCD = *BLANKS` and `KYCACD ≠ *BLANKS`:
         - If `SHCACD = KYCACD`, skips to `END` (retains record).
         - Else, executes `EXCPTDEL` and skips to `END`.
     - **Case 4: KYFRCD specified, KYCACD blank**:
       - If `KYFRCD ≠ *BLANKS` and `KYCACD = *BLANKS`:
         - If `SHFRCD = KYFRCD`, skips to `END` (retains record).
         - Else, executes `EXCPTDEL` and skips to `END`.
   - **End of Processing**:
     - `END` tag: Moves to the next record.
   - **Output**:
     - `EXCPTDEL`: Marks records for deletion by writing `'D'` to position 1 of `SA5FILD`.

4. **Output Specifications**:
   - `SA5FILD` (`E`, `DEL`):
     - Position 1: Writes `'D'` to mark the record for deletion (effectively excluding it from further processing).

### Business Rules

1. **Purpose**: Filters sales records in `SA5FILD` (`?9?S5505S`) based on freight code (`KYFRCD`) and carrier code (`KYCACD`), marking non-matching records as deleted (`'D'`) to refine the dataset for the Customer Shipping Analysis Report.
2. **Filtering Logic**:
   - Retains records if:
     - Both `KYFRCD` and `KYCACD` are blank (no filtering).
     - Both are specified, and `SHFRCD = KYFRCD` and `SHCACD = KYCACD`.
     - `KYFRCD` is blank, and `SHCACD = KYCACD`.
     - `KYCACD` is blank, and `SHFRCD = KYFRCD`.
   - Marks records for deletion (`'D'`) otherwise.
3. **Data Preservation**: Updates `SA5FILD` in place, marking excluded records rather than creating a new file.
4. **Context**: Prepares sorted sales data (`?9?S5505S`) from `#GSORT` for `SA505C`, which generates the final report. Part of the `SA505C.ocl36.txt` workflow within `PRICES.ocl36.txt`.
5. **Chaining**: Uses `SA5SHX` to retrieve additional data via `SHKEY`, ensuring accurate filtering (skips processing if no match, `98` on).

### Tables (Files) Used

1. **SA5FILD** (`?9?S5505S`):
   - **Access**: Update Primary (`UP`), shared mode (`DISP-SHR`).
   - **Purpose**: Sorted sales data, updated to mark excluded records.
2. **SA5SHX** (`?9?SA5SHX`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Sales history index for retrieving additional data via `SHKEY`.

### External Programs Called

- **None**: The `SA505X` RPG program does not call external programs or subroutines. It performs internal filtering logic.

### Additional Notes

- **Context**: Invoked by `SA505C.ocl36.txt` after `#GSORT`, preparing `?9?S5505S` for `SA505C` by filtering out records that do not match specified freight or carrier codes.
- **System/36 Environment**: Uses RPG II/III syntax, likely on AS/400.
- **Truncation**: The document is truncated, but the provided code includes critical sections (`F`, `I`, `C`, `O`), sufficient to understand the filtering logic.
- **Error Handling**: Uses indicator `98` for `SA5SHX` chain failures and relies on System/36 for file errors.
- **Relation to SA505C**: Directly supports `SA505C` by refining the sales dataset, ensuring only relevant records are included in the report.

If you have the RPG source code for `SA505C` or other `SA505*` programs, or need further analysis of the pricing workflow, please provide those details! Let me know if you have additional questions or files to share.