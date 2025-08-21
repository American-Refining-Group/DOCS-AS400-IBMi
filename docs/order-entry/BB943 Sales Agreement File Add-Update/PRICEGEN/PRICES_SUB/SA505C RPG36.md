The provided document, `SA505C.rpg36.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `SA505C`. It is invoked by the OCL script `SA505C.ocl36.txt` as the final step in generating the Customer Shipping Analysis Report within the `PRICES.ocl36.txt` workflow, part of the `PRICEGEN.clp` pricing generation process for blended lubes. The program processes filtered sales data from `SA5FILD` (`?9?S5505S`), aggregates it by company, customer, ship-to, container, and product, and produces a formatted report (`LIST`) and/or an output file (`EXCELOUT`, `?9?SA505C`). Below, I will explain the process steps, business rules, tables used, and external programs called, despite the truncation in the document.

### Process Steps of the RPG Program

1. **File Definitions**:
   - **Input Files**:
     - `SA5FILD` (Input Primary, `IP`, 1029 bytes, disk): Sorted and filtered sales data, mapped to `?9?S5505S` (e.g., `AS5505S`), shared mode.
     - `SA5SHX` (Input, `IF`, 1500 bytes, indexed with 15-byte key, disk): Sales history index, mapped to `?9?SA5SHX`, shared mode.
     - `BICONT` (Input, `IF`, 256 bytes, indexed with 2-byte key, disk): Contract file, mapped to `?9?BICONT`, shared mode.
     - `ARCUST` (Input, `IF`, 384 bytes, indexed with 8-byte key, disk): Customer master file, mapped to `?9?ARCUST`, shared mode.
     - `GSPRCL` (Input, `IF`, 38 bytes, indexed with 5-byte key, disk): Product class file, mapped to `?9?GSPRCL`, shared mode.
     - `SHIPTO` (Input, `IC`, 2048 bytes, indexed with 11-byte key, disk): Ship-to file, mapped to `?9?SHIPTO`, shared mode.
   - **Update Files**:
     - `SAINVC` (Update, `UF`, 32 bytes, indexed with 15-byte key, disk): Sales invoice file, mapped to `?9?SAINVC`, job-level retention.
     - `SACSTC` (Update, `UF`, 32 bytes, indexed with 8-byte key, disk): Customer summary file, mapped to `?9?SACSTC`, job-level retention.
   - **Output Files**:
     - `EXCELOUT` (Output, `O`, 40 bytes, disk): Output file for report data, mapped to `?9?SA505C`, temporary retention.
     - `LIST` (Output, `O`, 164 bytes, printer): Printer file for the formatted Customer Shipping Analysis Report.

2. **Extension Specifications**:
   - `SEP` (80 elements, 2 bytes): Separator lines (`--`) for formatting the report between groups.

3. **Input Specifications**:
   - **SA5FILD** (record type `NS 01`, condition `1NCD`, excludes deleted records where position 1 = `'D'`):
     - `ARCKEY` (2–9): Key field (company + customer).
     - `SHKEY` (2–16): Key field for chaining to `SA5SHX` (company + customer + invoice).
     - `SACO#` (2–3): Company number.
     - `SACUST` (4–9): Customer number.
     - `SAINV#` (10–16): Invoice number.
     - `SASHIP` (17–19): Ship-to number.
     - `SASLMN` (20–21): Salesman code.
     - `SAINDT` (22–27): Invoice date (YMD).
     - `SAQTY3` (28–34): Ship quantity (billing gallons).
     - `SAPRCE` (35–39, packed, 4 decimals): Price.
     - `SACOST` (40–44, packed, 4 decimals): Cost.
     - `SAITEM` (45–59): Item number (location/product/tank).
     - `SALOC` (45–47): Location.
     - `SAPROD` (48–51): Product code.
     - `SATANK` (52–55): Tank code.
     - `SATYPE` (60): Type (`' '`, `M`, `C`, `R`).
     - `SAORD#` (61–66): Order number.
     - `SASEQ` (67–69): Order sequence number.
     - `SASHDT` (70–75): Ship date (YMD).
     - `SAUM` (76–78): Unit of measure.
     - `SATIME` (79–82): Time of sale (HHMM).
     - `SAPRGP` (83–84): Product group.
     - `SADESC` (87–101): Description.
     - `SACNTR` (193–195): Container code.
   - **BICONT** (record type `NS`):
     - `BCDEL` (1): Delete code.
     - `BCCO` (2–3): Company number.
     - `BCNAME` (4–33): Company name.
   - **ARCUST** (record type `NS`):
     - `ARDEL` (1): Delete code.
     - `ARCUST` (2–7): Customer number.
     - `ARNAME` (8–37): Customer name.
   - **GSPRCL** (record type `NS`):
     - `PRDEL` (1): Delete code.
     - `PRPRCL` (2–4): Product class code.
     - `PRDESC` (5–34): Product class description.
   - **SHIPTO** (record type `NS`):
     - `SHDEL` (1): Delete code.
     - `SHCO#` (2–3): Company number.
     - `SHCUST` (4–9): Customer number.
     - `SHSHIP` (10–12): Ship-to number.
     - `SHNAME` (13–42): Ship-to name.
   - **User Data Structure (UDS)** (partial, based on `SA505X` and OCL):
     - `KYFRCD` (232): Freight code filter.
     - `KYCACD` (233–234): Carrier code filter.
     - `KYDIV`, `KYLOSL`, `KYLOC1`–`KYLOC5`, `KYPCSL`, `KYPC01`–`KYPC10`, `KYCTSL`, `KYCT01`–`KYCT05`, `KYPDSL`, `KYPD01`–`KYPD10`, `KYFRDT`, `KYTODT` (from `SA505C.ocl36.txt`): Filters for division, location, product class, container, product, and date range.

4. **Calculation Specifications** (inferred due to truncation):
   - **Main Loop**:
     - Processes each `SA5FILD` record, excluding deleted records (`1NCD`, position 1 ≠ `'D'`).
   - **Lookups**:
     - Chains `SA5SHX` using `SHKEY` to retrieve additional sales data.
     - Chains `BICONT` using `SACO#` for company name (`BCNAME`).
     - Chains `ARCUST` using `ARCKEY` for customer name (`ARNAME`).
     - Chains `SHIPTO` using `SHCO#`, `SHCUST`, `SHSHIP` for ship-to name (`SHNAME`).
     - Chains `GSPRCL` using `SAPRCL` (implied, from `SACNTR` or product data) for product class description (`PRDESC`).
   - **Aggregation**:
     - Accumulates totals for:
       - `L5QTY3MB`, `L5SALEMB`, `L5INVCMB`: Company-level totals (quantity, sales, invoice count).
       - `L4QTY3MB`, `L4SALEMB`, `L4INVCMB`: Customer-level totals.
       - `L3QTY3MB`, `L3SALEMB`: Ship-to-level totals.
       - `L1QTY3MB`, `L1SALEMB`: Container-level totals.
       - `L2QTY3MB`, `L2SALEMB`: Product-level totals.
     - Calculates sales amounts (`SAPRCE * SAQTY3` for `L*SALEMB`) and tracks quantities (`SAQTY3`).
   - **Filtering**:
     - Applies filters from UDS (e.g., `KYFRDT`, `KYTODT` for date range; `KYLOC1`–`KYLOC5` for locations; `KYPC01`–`KYPC10` for product classes; `KYCT01`–`KYCT05` for containers; `KYPD01`–`KYPD10` for products).
   - **Output**:
     - Writes detail lines and totals to `LIST` (printer) and summary data to `EXCELOUT` (`?9?SA505C`).
     - Updates `SAINVC` and `SACSTC` with summary data.

5. **Output Specifications**:
   - **LIST** (printer, 164 bytes):
     - **Detail Lines** (`EF10` or similar):
       - `SAPROD` (7): Product code.
       - `SACNTR` (7): Container code.
       - `SASHIP` (7): Ship-to number.
       - `SACUST` (7): Customer number.
       - `SACO#` (24): Company number.
       - `TTLPRCM` (84): Total price margin.
       - `SALESMB` (97): Sales amount.
       - `SADELV` (100): Delivery code.
       - `SACACD` (105): Carrier code.
       - `SHRTG1` (132), `SHRTG2` (164): Route or tracking data.
     - **Totals**:
       - Product totals (`EF12`): `L2QTY3MB`, `L2SALEMB`, with label `* PRODUCT TOTALS`.
       - Container totals (`EF02`): `L1QTY3MB`, `L1SALEMB`, with label `*** CONTAINER TOTALS`.
       - Ship-to totals (`EF01`): `L3QTY3MB`, `L3SALEMB`, `EDIID` (if `N95`), with separators.
       - Customer totals (`EF02`): `L4QTY3MB`, `L4SALEMB`, `L4INVCMB`, with separators.
       - Company totals (`EF2`): `L5QTY3MB`, `L5SALEMB`, `L5INVCMB`, with label `*** COMPANY TOTALS`.
   - **EXCELOUT** (`?9?SA505C`, 40 bytes):
     - Summary data (fields not specified due to truncation, likely includes totals or key fields).
   - **SAINVC**, **SACSTC**:
     - Updated with summary data (e.g., invoice and customer totals).

### Business Rules

1. **Purpose**: Generates a Customer Shipping Analysis Report, summarizing sales by company, customer, ship-to, container, and product, with quantities, sales amounts, and invoice counts, output to a printer (`LIST`) or file (`?9?SA505C`).
2. **Aggregation**:
   - Totals are calculated at multiple levels:
     - Company (`L5*`): Across all records for a company.
     - Customer (`L4*`): Per customer within a company.
     - Ship-to (`L3*`): Per ship-to within a customer.
     - Container (`L1*`): Per container within a ship-to.
     - Product (`L2*`): Per product within a container.
   - Tracks quantities (`SAQTY3`), sales amounts (`SAPRCE * SAQTY3`), and invoice counts.
3. **Filtering**:
   - Excludes deleted records (`SA5FILD` position 1 ≠ `'D'`, set by `SA505X`).
   - Applies UDS filters (from `SA505C.ocl36.txt`):
     - Date range (`KYFRDT` to `KYTODT`).
     - Locations (`KYLOSL = 'SEL'`, `KYLOC1`–`KYLOC5`).
     - Product classes (`KYPCSL = 'SEL'`, `KYPC01`–`KYPC10`).
     - Containers (`KYCTSL = 'SEL'`, `KYCT01`–`KYCT05`).
     - Products (`KYPDSL = 'SEL'`, `KYPD01`–`KYPD10`).
     - Freight and carrier codes (`KYFRCD`, `KYCACD`, applied in `SA505X`).
4. **Output Formatting**:
   - Groups report by company, customer, ship-to, container, and product.
   - Includes descriptions from `BICONT` (`BCNAME`), `ARCUST` (`ARNAME`), `SHIPTO` (`SHNAME`), and `GSPRCL` (`PRDESC`).
   - Uses separator lines (`SEP`) for readability.
   - Supports printer (`LIST`) or file output (`EXCELOUT`), controlled by LDA position 174 (`'P'` or `'D'`).
5. **Context**: Complements rack pricing (`BB953B`) by analyzing shipping data, likely for pricing validation or sales performance reporting.

### Tables (Files) Used

1. **SA5FILD** (`?9?S5505S`):
   - **Access**: Input Primary (`IP`), shared mode.
   - **Purpose**: Sorted and filtered sales data from `SA505X`.
2. **SA5SHX** (`?9?SA5SHX`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Sales history index for additional data.
3. **BICONT** (`?9?BICONT`):
   - **Access**: Input (`IF`), shared mode.
   - **Purpose**: Company name (`BCNAME`).
4. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input (`IF`), shared mode.
   - **Purpose**: Customer name (`ARNAME`).
5. **GSPRCL** (`?9?GSPRCL`):
   - **Access**: Input (`IF`), shared mode.
   - **Purpose**: Product class description (`PRDESC`).
6. **SHIPTO** (`?9?SHIPTO`):
   - **Access**: Input (`IC`), shared mode.
   - **Purpose**: Ship-to name (`SHNAME`).
7. **SAINVC** (`?9?SAINVC`):
   - **Access**: Update (`UF`), job-level retention.
   - **Purpose**: Sales invoice data, updated with summary data.
8. **SACSTC** (`?9?SACSTC`):
   - **Access**: Update (`UF`), job-level retention.
   - **Purpose**: Customer summary data, updated with totals.
9. **EXCELOUT** (`?9?SA505C`):
   - **Access**: Output (`O`), temporary retention.
   - **Purpose**: Report data (e.g., for Excel).
10. **LIST**:
    - **Access**: Output (`O`), printer.
    - **Purpose**: Formatted Customer Shipping Analysis Report.

### External Programs Called

- **None**: The `SA505C` RPG program does not call external programs or subroutines. It performs internal processing and output formatting.

### Additional Notes

- **Context**: Invoked by `SA505C.ocl36.txt` after `SA505X`, producing the final report and updating summary files, part of the `PRICES.ocl36.txt` workflow.
- **System/36 Environment**: Uses RPG II/III syntax, likely on AS/400.
- **Truncation**: The document is truncated, but output specifications provide sufficient detail. Missing calculations likely involve chaining, totaling, and filtering logic.
- **Error Handling**: Uses `1NCD` to skip deleted records and relies on System/36 for file errors.
- **Output Flexibility**: Supports printer (`LIST`) or file output (`?9?SA505C`), controlled by LDA position 174.

If you have the RPG source code for `SA505E`, `SA505G`, `SA505H`, `SA505I`, `SA505J`, or `SA505L`, or need further analysis, please provide those details! Let me know if you have additional questions or files to share.