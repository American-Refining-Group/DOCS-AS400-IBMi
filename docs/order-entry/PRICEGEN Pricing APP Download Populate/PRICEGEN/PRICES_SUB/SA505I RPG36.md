The provided document, `SA505I.rpg36.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `SA505I`. It is invoked by the OCL script `SA505I.ocl36.txt` within the `PRICES.ocl36.txt` workflow, part of the `PRICEGEN.clp` pricing generation process for blended lubes. The program processes the detailed sales and pricing data from `SA505H` (`TMPFILX`), enriches it with customer (`ARCUST`), container (`GSCNTR`), product (`GSPROD`), and unit of measure conversion data, and writes results to the pricing file (`PRSABLW`) for the Customer Shipping Analysis Report. It also calls an external program (`MBBQTY`) to calculate quantities. Below, I will explain the process steps, business rules, tables used, and external programs called, despite the truncation in the document .

### Process Steps of the RPG Program

1. **File Definitions**:
   - **Input Files**:
     - `TMPFILX` (Input Primary, `IP`, 110 bytes, disk): Detailed sales and pricing data from `SA505H`, mapped to `?9?TMPFILX`, shared mode.
     - `GSCNTR` (Input, `IF`, 512 bytes, indexed with 3-byte key, disk): Container master file, mapped to `?9?GSCNTR`, shared mode.
     - `ARCUST` (Input, `IF`, 384 bytes, indexed with 8-byte key, disk): Customer master file, mapped to `?9?ARCUST`, shared mode.
     - `GSPROD` (Input, `IF`, 512 bytes, indexed with 6-byte key, disk): Product master file, mapped to `?9?GSPROD`, shared mode.
   - **Output File**:
     - `PRSABLW` (Output, `O`, 271 bytes, disk): Pricing file for blended lubes, mapped to `?9?PRSABLW`, shared mode.

2. **Input Specifications**:
   - **TMPFILX** (record type `NS 01`):
     - `BACONO` (1–2): Company number.
     - `BACUST` (3–8): Customer number.
     - `BAKEY` (1–8): Key field (company + customer).
     - `BAPR01` (9–12): Product code.
     - `BACNTR` (13–15): Container code.
     - `BAUNMS` (16–18): Unit of measure.
     - `BASTD8` (19–26): Start date (CYMD, CCYYMMDD).
     - `BASTTM` (29–32): Start time (HHMM).
     - `BAPRCE` (33–37, packed, 4 decimals): Price.
     - `QTSOLD` (38–42, packed, 4 decimals): Quantity sold (billing gallons).
     - `AMTSLD` (43–48, packed, 4 decimals): Sales amount.
     - `REC1` (1–110): Entire record for processing.
   - **ARCUST** (record type `NS`):
     - `ARDEL` (1): Delete code.
     - `ARCO` (2–3): Company number.
     - `ARCUST` (4–9): Customer number.
     - `ARNAME` (10–39): Customer name.
     - `ARADR1` (40–69): Address line 1.
     - `ARADR2` (70–99): Address line 2.
     - `ARADR3` (100–129): Address line 3.
     - `ARADR4` (130–159): Address line 4.
     - `ARZIP5` (160–164): ZIP code (5 digits).
     - `ARZIP9` (165–168): ZIP + 4.
     - `ARZP14` (169–173): ZIP + 4 + 5.
   - **GSCNTR** (record type `NS`):
     - `TCDEL` (1): Delete code.
     - `TCCNTR` (2–4): Container code.
     - `TCDESC` (14–43): Container description.
   - **GSPROD** (record type `NS`):
     - `TPDEL` (1): Delete code.
     - `TPPROD` (2–5): Product code.
     - `TPDES1` (6–25): Product description.
   - **User Data Structure (UDS)** (inferred from `SA505C.ocl36.txt` and prior programs):
     - `KYFRDT`, `KYTODT`: Date range filters (CYMD).
     - `KYDIV`, `KYLOC1`–`KYLOC5`, `KYPC01`–`KYPC10`, `KYCT01`–`KYCT05`, `KYPD01`–`KYPD10`: Filters for division, location, product class, container, and product.

3. **Calculation Specifications** (partially provided due to truncation):
   - **Initialization** (implied, likely in `ONETIM` subroutine, similar to `SA505H`):
     - Sets system date and time (`SYDATE`, `TIMEOF`).
     - Converts `FRDT`, `TODT` to `KYFRDT`, `KYTODT` (CYMD) for date filtering.
   - **Main Loop**:
     - Processes each `TMPFILX` record (`01 DO`, implied).
     - Applies filters (e.g., `KYFRDT` to `KYTODT`, `BACONO`, `BACUST`, `BAPR01`, `BACNTR`, `BAUNMS`).
   - **Chaining**:
     - Chains `ARCUST` using `BAKEY` (company + customer) to retrieve `ARNAME`, address, and ZIP code.
     - Chains `GSCNTR` using `BACNTR` to retrieve `TCDESC` (container description).
     - Chains `GSPROD` using `BAPR01` to retrieve `TPDES1` (product description).
   - **Quantity Conversion**:
     - Initializes fields for quantity calculation:
       - `MOVEL*BLANK Q@FLCD`: Clears fluid code.
       - `Z-ADD60 Q@TEMP`: Sets temperature to 60°F.
       - `Z-ADD*ZERO Q@GRAV`: Clears gravity.
       - `MOVEL*BLANK Q@VCF`: Clears volume correction factor.
       - `Z-ADD*ZERO Q@SQTY`, `Q@GGAL`, `Q@NGAL`, `Q@6DFA`, `Q@APFA`, `Q@NLBS`, `Q@SLBS`: Clears quantity, gross gallons, net gallons, and weight fields.
     - `CALL 'MBBQTY'`: Calls external program `MBBQTY` with parameter `QTPARM` to calculate quantities (e.g., converting `QTSOLD` to gallons or weights using `GSCTUM`, `GSUMCV`, `GSCTWT`).
   - **Output**:
     - Writes to `PRSABLW` via `EADD` exception, populating fields with data from `TMPFILX`, `ARCUST`, `GSPROD`, and calculated values.

4. **Output Specifications**:
   - **PRSABLW** (`EADD`, 271 bytes):
     - `BACONO` (1–2): Company number.
     - Position 3–5: Hardcoded `'001'` (possibly location or status code).
     - `BACUST` (6–11): Customer number.
     - `ARNAME` (12–41): Customer name.
     - `ARSTAT` (42–43): Customer status (from `ARCUST` or derived).
     - Position 44–62: Hardcoded `'ALL'` (possibly for division or group).
     - `BAPR01` (63–66): Product code.
     - `TPDES1` (67–76): Product description.
     - `BAUNMS` (77–79): Unit of measure.
     - `BACNTR` (80–82): Container code.
     - `BAPRCE` (83–87, packed): Price.
     - `ZERO94` (88–92, packed): Zeroed field (previous price or placeholder).
     - `BASTD8` (93–110): Start date (CYMD).
     - Position 111–118: Hardcoded `'20791231'` (end date, far future).
     - `ZERO7` (119–122, packed): Zeroed minimum quantity.
     - `ZERO7` (123–126, packed): Zeroed maximum quantity.
     - `ZERO94` (127–131, packed): Zeroed offset price.
     - `BASTTM` (132–135): Start time (HHMM).
     - Position 136–139: Hardcoded `'2359'` (end time, 23:59).
     - Position 140: Hardcoded `'Y'` (PPD flag).
     - Position 141–172: Hardcoded `'0000000000000000000000000000'` (filler or flags).
     - `CAGAPR` (188–192, packed): Calculated price or margin (from `MBBQTY` or prior logic).
     - `ZERO5` (193–195, packed): Zeroed route type.
     - `ZERO5` (196–198, packed): Zeroed ship type.
     - `QTSOLD` (199–203, packed): Quantity sold.
     - `AMTSLD` (204–209, packed): Sales amount.
     - Position 210: Hardcoded `'R'` (record type or status).

### Business Rules

1. **Purpose**: Processes `TMPFILX` data from `SA505H`, enriches it with customer, container, and product details, performs quantity conversions using `MBBQTY`, and writes results to `PRSABLW` for the Customer Shipping Analysis Report.
2. **Data Enrichment**:
   - Retrieves customer details (`ARNAME`, address, ZIP) from `ARCUST`.
   - Retrieves container description (`TCDESC`) from `GSCNTR`.
   - Retrieves product description (`TPDES1`) from `GSPROD`.
   - Uses `GSCTUM`, `GSUMCV`, and `GSCTWT` (via `MBBQTY`) to convert quantities (`QTSOLD`) to gallons or weights.
3. **Filtering**:
   - Applies date range filters (`KYFRDT` to `KYTODT`, compared to `BASTD8`).
   - Uses UDS filters (from `SA505C.ocl36.txt`): division (`KYDIV`), locations (`KYLOC1`–`KYLOC5`), product classes (`KYPC01`–`KYPC10`), containers (`KYCT01`–`KYCT05`), products (`KYPD01`–`KYPD10`).
4. **Output**:
   - Writes to `PRSABLW` with enriched data, hardcoded defaults (e.g., end date `'20791231'`, `'Y'` for PPD), and calculated quantities/sales.
   - Hardcodes `'R'` as record type, indicating a specific report or pricing status.
5. **Quantity Conversion**:
   - Calls `MBBQTY` to convert `QTSOLD` (billing gallons) to other units (e.g., gross/net gallons, pounds) using `Q@TEMP` (60°F), clearing gravity (`Q@GRAV`) and volume correction factor (`Q@VCF`).
6. **Context**: Follows `SA505H` in the `PRICES.ocl36.txt` workflow, finalizing data for the Customer Shipping Analysis Report by updating `PRSABLW`, complementing rack pricing (`BB953B`) and blended lubes pricing (`BI942E`).

### Tables (Files) Used

1. **TMPFILX** (`?9?TMPFILX`):
   - **Access**: Input Primary (`IP`), shared mode.
   - **Purpose**: Detailed sales and pricing data from `SA505H`.
2. **GSCNTR** (`?9?GSCNTR`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Container descriptions (`TCDESC`).
3. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Customer details (`ARNAME`, address, ZIP).
4. **GSPROD** (`?9?GSPROD`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Product descriptions (`TPDES1`).
5. **PRSABLW** (`?9?PRSABLW`):
   - **Access**: Output (`O`), shared mode.
   - **Purpose**: Pricing file updated with enriched data.
6. **GSCTUM** (`?9?GSCTUM`):
   - **Access**: Input, shared mode (from OCL, not in RPG).
   - **Purpose**: Unit of measure conversion data (used by `MBBQTY`).
7. **GSCTWT** (`?9?GSCTWT`):
   - **Access**: Input, shared mode (from OCL, not in RPG).
   - **Purpose**: Container weight data (used by `MBBQTY`).
8. **GSUMCV** (`?9?GSUMCV`):
   - **Access**: Input, shared mode (from OCL, not in RPG).
   - **Purpose**: Unit of measure conversion table (used by `MBBQTY`).

### External Programs Called

1. **MBBQTY**: External program called to calculate quantities (e.g., converting `QTSOLD` to gallons or weights) using `QTPARM` parameter, leveraging `GSCTUM`, `GSUMCV`, and `GSCTWT`.

### Additional Notes

- **Context**: Invoked by `SA505I.ocl36.txt` after `SA505H`, processing `?9?TMPFILX` to produce enriched pricing data in `?9?PRSABLW`, part of the Customer Shipping Analysis Report workflow.
- **System/36 Environment**: Uses RPG II/III syntax, likely on AS/400.
- **Truncation**: The document is truncated, but key sections (`F`, `I`, `C`, `O`) provide sufficient detail. Missing calculations likely involve chaining and filtering logic.
- **Error Handling**: Uses chaining indicators (not shown) and relies on System/36 for file errors.
- **Output**: Updates `PRSABLW` with comprehensive data, suitable for reporting or further processing (e.g., by `SA505J`, `SA505L`).

If you have the RPG source code for `SA505E`, `SA505J`, or `SA505L`, or need further analysis, please provide those details! Let me know if you have additional questions or files to share.