The provided document, `BI942E.rpg36.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `BI942E`. It is invoked by the OCL script `BI942E.ocl36.txt` as part of the pricing generation process for blended lubes, initiated by `PRICEGEN.clp`. The program processes pricing agreement records from the `BICUAGP` file, calculates prices per gallon (and other units) using container and unit of measure data, and writes the results to the `PRSABL` file. Below, I will explain the process steps, business rules, tables used, and external programs called, despite the truncation in the input specifications .

### Process Steps of the RPG Program

1. **File Definitions**:
   - **Input Files**:
     - `BICUAGP` (Input Primary, `IP`, 271 bytes, disk): Primary input file, mapped to `?9?BICUAGP` (e.g., `ABICUAGP`), containing pricing agreement records from `BI944`.
     - `GSCNTR` (Input, `IF`, 512 bytes, indexed with 3-byte key, disk): Container file for container details.
   - **Output File**:
     - `PRSABL` (Output, `O`, 271 bytes, disk): Output file, mapped to `?9?PRSABL` (e.g., `APRSABL`), for processed pricing data.

2. **Input Specifications**:
   - **BICUAGP** (record type `NS 01`):
     - `BACONO` (1–2): Company number.
     - `BALOC` (3–5): Location.
     - `BACUST` (6–11): Customer number.
     - `BASHIP` (60–62): Ship-to number.
     - `BAPR01` (63–66): Product code 1.
     - `BAUNMS` (77–79): Unit of measure.
     - `BACNTR` (80–82): Container code.
     - `BAPRCE` (83–87.4): Price (4 decimal places).
     - `BASTD8` (103–110): Start date (CYMD).
     - `BAEND8` (111–118): End date (CYMD).
     - `BAMNQY` (119–122.4): Minimum quantity (4 decimal places).
     - `BAMXQY` (123–126.4): Maximum quantity (4 decimal places).
     - `BAOFFP` (127–131.4): Off-price (4 decimal places).
     - `BASTTM` (132–135): Start time (HM).
     - `BAENTM` (136–139): End time (HM).
     - `BAPORD` (173–187): Bill-to purchase order.
     - `BACAPR` (188–192.4): Calculated gallon price (4 decimal places).
     - `REC1` (1–256): First 256 bytes of the record.
   - **GSCNTR** (record type `NS`):
     - `TCDEL` (1): Delete code.
     - `TCCNTR` (2–4): Container code.
     - `TCCNTA` (5–7): Alpha container code.
     - `TCCNSM` (8–13): Container summary code.
     - `TCDESC` (14–43): Table description.
     - `TCDSCS` (44–51): Short description.
     - `TCDSCL` (52–73): Long description.
     - `TCFIL2` (74–179): Filler.
     - `TCCNTY` (180): Container type (B for bulk, P for package).
     - `TCFRTB` (181–182): Freight table code.
     - `TCFIL3` (183–210): Filler.
     - `TCDES2` (211–240): Second description.
     - `TCFIL4` (241–256): Filler.
     - `TCCTRS` (257): Container source.
     - `TCIUM` (258–260): IMS unit of measure.
     - `TCFIL5` (261–512): Filler.
   - **Data Structure (QTPARM)**: Used for parameters to the `MBBQTYPR` program:
     - `Q@CO` (1–2): Company.
     - `Q@PROD` (3–6): Product code.
     - `Q@CNTR` (7–9): Container code.
     - `Q@UM` (10–12): Unit of measure.
     - `Q@GLCD` (13): Gallon code.
     - `Q@CTQT` (14–20): Container quantity.
     - `Q@TARE` (21–27): Tare weight.
     - `Q@GVWT` (28–34): Gross weight.
     - `Q@CACA` (35–41): Calculated capacity.
     - `Q@OUTA` (42–48): Output amount.
     - `Q@CACD` (49–50): Capacity code.
     - `Q@FLCD` (51): Fluid code.
     - `Q@TEMP` (52–54): Temperature.
     - Fields like `Q@SQTY`, `Q@NGAL`, `Q@NLBS` (net gallons, pounds) for quantity conversions.

3. **Calculation Specifications**:
   - **Main Loop**: Processes each `BICUAGP` record (`01 DO`, implied).
   - **Container Lookup**:
     - Chains `GSCNTR` using `BACNTR` to retrieve container details (e.g., `TCCNTY`, `TCIUM`).
   - **Price Conversion** (`$PRCE` subroutine, partially shown):
     - Converts `BAPRCE` (price in `BAUNMS` unit) to gallon price (`GALPRC`), pound price (`LBPRCE`), kilogram price (`KGPRCE`), milliliter price (`MLPRCE`), and liter price (`LIPRCE`).
     - **For `BAUNMS = 'GAL'` (gallons)**:
       - `GALPRC = BAPRCE`.
       - `LBPRCE = GALPRC * FACTOR` (where `FACTOR = Q@NLBS / Q@NGAL` from `MBBQTYPR`).
       - `KGPRCE = LBPRCE / 0.453592`.
       - `MLPRCE = GALPRC / 3785.41`.
       - `LIPRCE = GALPRC / 3.78541`.
       - `CAGAPR = GALPRC` (calculated gallon price).
     - **For `BAUNMS = 'LBS'` (pounds)**:
       - Calls `$MQTY` to get `Q@NGAL`, `Q@NLBS`.
       - `FACTOR = Q@NLBS / Q@NGAL` (if `Q@NGAL ≠ 0`, else `Q@NGAL = 1`).
       - `LBPRCE = BAPRCE`.
       - `GALPRC = LBPRCE * FACTOR`.
       - `KGPRCE = LBPRCE / 0.453592`.
       - `MLPRCE = GALPRC / 3785.41`.
       - `LIPRCE = GALPRC / 3.78541`.
       - `CAGAPR = GALPRC`.
     - **For `BAUNMS = 'KG '` (kilograms)**:
       - Calls `$MQTY`.
       - `FACTOR = Q@NLBS / Q@NGAL`.
       - `KGPRCE = BAPRCE`.
       - `LBPRCE = KGPRCE * 0.453592`.
       - `GALPRC = LBPRCE * FACTOR`.
       - `MLPRCE`, `LIPRCE`, `CAGAPR` calculated as above.
     - **For other units (`BAUNMS ≠ 'GAL', 'LBS', 'KG ', or blank`)**:
       - Calls `$MQTY`.
       - `WRKPRC = Q@SQTY * BAPRCE` (source quantity * price).
       - `GALPRC = WRKPRC / Q@NGAL` (if `Q@NGAL ≠ 0`).
       - `LBPRCE`, `KGPRCE`, `MLPRCE`, `LIPRCE`, `CAGAPR` calculated as above.
   - **Quantity Conversion** (`$MQTY` subroutine):
     - Sets up `QTPARM` for `MBBQTYPR`:
       - `Q@CO = BACONO`, `Q@PROD = BAPR01`, `Q@CNTR = BACNTR`, `Q@UM = BAUNMS`.
       - `Q@GLCD = blank`, `Q@TEMP = 60`, `Q@TARE`, `Q@GVWT`, `Q@CACA`, `Q@OUTA`, `Q@GRAV`, `Q@VCF = 0/blank`.
       - If `TCCNTY = 'B'` (bulk): `Q@CACD = 'TT'`, `Q@CTQT = 25000`.
       - Else (package): `Q@CACD = 'PT'`, `Q@CTQT = 10000`.
     - Calls `MBBQTYPR` to calculate quantities (`Q@SQTY`, `Q@NGAL`, `Q@NLBS`).
   - **Output**: Writes to `PRSABL` for valid records.

4. **Output Specifications**:
   - `PRSABL` (`EADD`):
     - `REC1` (1–256): Copies first 256 bytes from `BICUAGP`.
     - `CAGAPR` (188–192): Calculated gallon price (packed, 4 decimals).
     - `ZERO5` (193–195, 196–198): Zero fields (5 digits, packed).
     - `ZERO9` (199–203): Zero field (9 digits, packed).
     - `ZERO11` (204–209): Zero field (11 digits, packed).
     - Position 210: Writes `'R'` (constant, likely a record type indicator).

### Business Rules

1. **Purpose**: Processes pricing agreements from `BICUAGP` to calculate prices per gallon (and other units) and writes results to `PRSABL` for the final pricing step (`PRICES` in `PRICEGEN.clp`).
2. **Price Conversion**:
   - Converts `BAPRCE` (in `BAUNMS` units) to gallon price (`CAGAPR`) and other units (pounds, kilograms, milliliters, liters).
   - Uses `MBBQTYPR` to get conversion factors (`Q@NGAL`, `Q@NLBS`, `Q@SQTY`) based on company, product, container, and unit of measure.
   - Handles units: `'GAL'`, `'LBS'`, `'KG '`, or others, with specific calculations for each.
3. **Container Logic**:
   - Uses `GSCNTR` to validate container (`BACNTR`) and determine type (`TCCNTY` = 'B' for bulk, else package).
   - Sets default container quantities: 25,000 for bulk, 10,000 for package.
4. **Output**: Copies `BICUAGP` record (256 bytes), adds calculated gallon price (`CAGAPR`), zero fields, and `'R'` indicator to `PRSABL`.
5. **Context**: Follows `BI944B`, processing `?9?BICUAGP` to produce `?9?PRSABL` for `PRICES`, ensuring standardized pricing data.

### Tables (Files) Used

1. **BICUAGP** (`?9?BICUAGP`): Input pricing agreements (271 bytes, shared).
2. **GSCNTR** (`?9?GSCNTR`): Container details (shared).
3. **PRSABL** (`?9?PRSABL`): Output pricing table (271 bytes, shared).
4. **Unused Files** (defined in OCL but not in RPG):
   - `PRICINY` (`?9?PRICINY`): Pricing inventory.
   - `BICONT` (`?9?BICONT`): Contracts.
   - `GSPROD` (`?9?GSPROD`): Products.
   - `GSCTUM` (`?9?GSCTUM`): Contract unit measures.
   - `GSUMCV` (`?9?GSUMCV`): Summary conversions.
   - `GSCTWT` (`?9?GSCTWT`): Contract weights.

### External Programs Called

1. **MBBQTYPR**: Called in `$MQTY` to calculate quantity conversions (`Q@SQTY`, `Q@NGAL`, `Q@NLBS`) based on `QTPARM`.

### Additional Notes

- **Context**: Part of `PRICEGEN.clp`, invoked after `BI944B`, preparing `?9?PRSABL` for `PRICES`.
- **Truncation**: The code is truncated in the input specifications, but the provided logic is sufficient to understand the core functionality.
- **System/36**: Runs in a System/36 environment, likely on AS/400.
- **Error Handling**: Relies on `MBBQTYPR` and System/36 for errors; no explicit error logic in the RPG.

If you have the `PRICES` source code or the full `BI942E.rpg36.txt`, I can provide further details. Let me know if you have additional questions or files!