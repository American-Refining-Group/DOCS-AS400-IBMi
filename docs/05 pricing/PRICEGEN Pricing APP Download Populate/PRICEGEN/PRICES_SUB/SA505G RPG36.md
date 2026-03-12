The provided document, `SA505G.rpg36.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `SA505G`. It is invoked by the OCL script `SA505G.ocl36.txt` within the `PRICES.ocl36.txt` workflow, part of the `PRICEGEN.clp` pricing generation process for blended lubes. The program processes customer shipping and pricing data, cross-referencing sales analysis data (`SA505CX`, `SA505EX`) with rack prices (`RKPRCEX`) and blended lubes pricing (`PRSABL`), producing output files (`OUTFILE`, `OUTFILE2`) for the Customer Shipping Analysis Report. Below, I will explain the process steps, business rules, tables used, and external programs called, despite the truncation in the document .

### Process Steps of the RPG Program

1. **File Definitions**:
   - **Input Files**:
     - `PRSABL` (Update Primary, `UP`, 271 bytes, disk): Pricing file for blended lubes, mapped to `?9?PRSABLW`, shared mode.
     - `RKPRCEX` (Input, `IF`, 121 bytes, indexed with 15-byte key, disk): Rack price extension file, mapped to `?9?RKPRCEX`, shared mode.
     - `BICONT` (Input, `IF`, 256 bytes, indexed with 2-byte key, disk): Contract file, mapped to `?9?BICONT`, shared mode.
     - `ARCUST` (Input, `IF`, 384 bytes, indexed with 8-byte key, disk): Customer master file, mapped to `?9?ARCUST`, shared mode.
     - `SA505CX` (Input, `IF`, 40 bytes, indexed with 21-byte key, disk): Customer shipping analysis data, mapped to `?9?SA505CX`, shared mode.
     - `SA505EX` (Input, `IF`, 40 bytes, indexed with 18-byte key, disk): Additional shipping analysis data, mapped to `?9?SA505EX`, shared mode.
   - **Output/Update Files**:
     - `OUTFILE` (Update, `UF`, 18 bytes, indexed with 18-byte key, disk): Primary output file, mapped to `?9?SA505G`, shared mode.
     - `OUTFILE2` (Update, `UF`, 8 bytes, indexed with 8-byte key, disk): Secondary output file, mapped to `?9?SA505G2`, shared mode.

2. **Input Specifications**:
   - **PRSABL** (record type `NS 01`):
     - `BACONO` (1–2): Company number.
     - `BALOC` (3–5): Location.
     - `BACUST` (6–11): Customer number.
     - `BACUNM` (12–41): Customer name.
     - `BACTST` (42–59): Customer status or additional data.
     - `BASHIP` (60–62): Ship-to number.
     - `BAPR01` (63–66): Product code.
     - `BAPRDS` (67–76): Product description.
     - `BAUNMS` (77–79): Unit of measure.
     - `BACNTR` (80–82): Container code.
     - `BAPRCE` (83–87, packed, 4 decimals): Price.
     - `BAPRVP` (88–92, packed, 4 decimals): Previous price.
     - `BAFRDS` (93–102): Freight description.
     - `BASTD8` (103–110): Start date (CYMD, CCYYMMDD).
     - `BAEND8` (111–118): End date (CYMD, CCYYMMDD).
     - `BAMNQY` (119–122, packed, 4 decimals): Minimum quantity.
     - `BAMXQY` (123–126, packed, 4 decimals): Maximum quantity.
     - `BAOFFP` (127–131, packed, 4 decimals): Offset price.
     - `BASTTM` (132–135): Start time (HHMM).
     - `BAENTM` (136–139): End time (HHMM).
     - `BAPPD` (140): PPD flag.
     - `BAALSH` (141): Allow ship flag.
   - **RKPRCEX** (record type `NS`):
     - `RKKEY` (1–15): Key field (likely company + location + product + container).
     - `SBQTY3` (not shown): Quantity (billing gallons).
     - `SBSALE` (not shown): Sales amount.
     - `SBRTYP` (not shown): Route type.
     - `SBSTYP` (not shown): Ship type.
   - **BICONT** (record type `NS`):
     - `BCDEL` (1): Delete code.
     - `BCCO` (2–3): Company number.
     - `BCNAME` (4–33): Company name.
   - **ARCUST** (record type `NS`):
     - `ARDEL` (1): Delete code.
     - `ARCUST` (2–7): Customer number.
     - `ARNAME` (8–37): Customer name.
   - **SA505CX**, **SA505EX** (record type `NS`):
     - Fields not shown due to truncation, but likely contain sales quantities (`SAQTY3`), sales amounts (`SASALE`), route types (`SARTYP`), and ship types (`SASTYP`), based on `SA505C` and calculation logic.
   - **User Data Structure (UDS)**:
     - `KYFRDT`, `KYTODT` (from `SA505C.ocl36.txt`): Date range filters.
     - `FRDT`, `TODT` (not shown): Input dates, converted to `KYFRDT`, `KYTODT` (CYMD).
     - Other fields (e.g., `KYDIV`, `KYLOC1`–`KYLOC5`, `KYPC01`–`KYPC10`, `KYCT01`–`KYCT05`, `KYPD01`–`KYPD10`) inherited from prior steps for filtering.

3. **Calculation Specifications**:
   - **Initialization** (`ONETIM` subroutine):
     - `TIME TIME12`: Gets system time (`TIME12`, 12 digits).
     - `MOVE TIME12 SYDATE`: Moves time to `SYDATE` (6 digits, likely YYMMDD).
     - `MOVELTIME12 TIMEOF`: Moves time to `TIMEOF` (6 digits, likely HHMMSS).
     - `FRDT MULT 100.0001 KYFRDT`: Converts `FRDT` (YYMMDD) to `KYFRDT` (CYMD, CCYYMMDD) for date filtering.
     - `TODT MULT 100.0001 KYTODT`: Converts `TODT` to `KYTODT` (CYMD).
     - `SETON 05`: Sets indicator `05` (likely for initialization).
   - **Main Loop**:
     - Processes each `PRSABL` record (`01 DO`, implied).
     - Applies filters (e.g., `KYFRDT` to `KYTODT`, `BASTD8` to `BAEND8`, company, location, product, container).
   - **Chaining**:
     - `CKEY CHAINSA505CX 50`: Chains to `SA505CX` using `CKEY` (likely company + customer + ship-to + product + container) to retrieve sales data (quantity, sales, route, ship type).
     - `EKEY CHAINSA505EX 51`: Chains to `SA505EX` using `EKEY` (similar key) for additional sales data.
     - `RKKEY CHAINRKPRCEX 95`: Chains to `RKPRCEX` using `RKKEY` (company + location + product + container) for rack price data.
   - **Data Accumulation**:
     - If `SA505CX` hit (`50` on): Adds `SBQTY3`, `SBSALE`, `SBRTYP`, `SBSTYP` to `QTY3`, `SALE`, `RTYP`, `STYP`.
     - If `SA505EX` hit (`51` on): Adds `SAQTY3`, `SASALE`, `SARTYP`, `SASTYP` to `QTY3`, `SALE`, `RTYP`, `STYP`.
   - **Output**:
     - Writes to `OUTFILE` (`?9?SA505G`) via `AGREE` exception:
       - `CKEY2` (18 bytes): Key for summary data.
     - Writes to `OUTFILE2` (`?9?SA505G2`) via `AGREE` exception:
       - `BAKEY` (8 bytes): Key (likely company + customer).
     - Writes regardless of `SA505CX` or `SA505EX` hits (commented-out indicators `50` or `51` suggest all records are processed).
   - **Chaining to Output Files**:
     - `BAKEY CHAINOUTFILE2 61`: Checks `OUTFILE2` for existing records.
     - `CKEY2 CHAINOUTFILE 60`: Checks `OUTFILE` for existing records, updates if found.

4. **Output Specifications**:
   - **PRSABL** (`E`):
     - `RTYP` (195, packed): Route type.
     - `STYP` (198, packed): Ship type.
     - `QTY3` (203, packed): Total quantity.
     - `SALE` (209, packed): Total sales amount.
     - Position 210: Writes `'A'` (likely an active flag).
   - **OUTFILE** (`EADD`, `AGREE`, condition `60`):
     - `CKEY2` (1–18): Key field for summary data.
   - **OUTFILE2** (`EADD`, `AGREE`, condition `61`):
     - `BAKEY` (1–8): Key field (company + customer).

### Business Rules

1. **Purpose**: Generates a Customer Shipping Analysis Report by cross-referencing pricing data (`PRSABL`, `RKPRCEX`) with sales analysis data (`SA505CX`, `SA505EX`), producing summary output files (`?9?SA505G`, `?9?SA505G2`) for customer, ship-to, product, and pricing details.
2. **Data Integration**:
   - Matches `PRSABL` records (pricing for blended lubes) with sales data (`SA505CX`, `SA505EX`) and rack prices (`RKPRCEX`).
   - Accumulates quantities (`QTY3`), sales amounts (`SALE`), route types (`RTYP`), and ship types (`STYP`) from `SA505CX` or `SA505EX`.
3. **Filtering**:
   - Applies date range filters (`KYFRDT` to `KYTODT`, compared to `BASTD8`, `BAEND8`).
   - Uses UDS filters (from `SA505C.ocl36.txt` or inherited):
     - Division (`KYDIV`), locations (`KYLOC1`–`KYLOC5`), product classes (`KYPC01`–`KYPC10`), containers (`KYCT01`–`KYCT05`), products (`KYPD01`–`KYPD10`).
   - Processes all `PRSABL` records, regardless of sales data matches (commented-out indicators `50` or `51`).
4. **Output**:
   - `OUTFILE` (`?9?SA505G`): Summary data (18 bytes), likely including totals by company, customer, ship-to, product, and container.
   - `OUTFILE2` (`?9?SA505G2`): Higher-level summary (8 bytes), likely by company and customer.
   - Updates `PRSABL` with aggregated data (`RTYP`, `STYP`, `QTY3`, `SALE`, `'A'`).
5. **Context**: Follows `SA505C` in the `PRICES.ocl36.txt` workflow, integrating rack pricing (`RKPRCEX`, from `BB953`) and blended lubes pricing (`PRSABL`, from `BI942E`) with sales analysis data to produce a comprehensive report.

### Tables (Files) Used

1. **PRSABL** (`?9?PRSABLW`):
   - **Access**: Update Primary (`UP`), shared mode.
   - **Purpose**: Pricing data for blended lubes, updated with aggregated sales data.
2. **RKPRCEX** (`?9?RKPRCEX`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Rack price data, likely derived from `?9?RKPRCE` (`BB953`).
3. **BICONT** (`?9?BICONT`):
   - **Access**: Input (`IF`), shared mode.
   - **Purpose**: Company name (`BCNAME`).
4. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input (`IF`), shared mode.
   - **Purpose**: Customer name (`ARNAME`).
5. **SA505CX** (`?9?SA505CX`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Customer shipping analysis data (from `SA505C`).
6. **SA505EX** (`?9?SA505EX`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Additional shipping analysis data.
7. **OUTFILE** (`?9?SA505G`):
   - **Access**: Update (`UF`), shared mode.
   - **Purpose**: Primary output file for summary data.
8. **OUTFILE2** (`?9?SA505G2`):
   - **Access**: Update (`UF`), shared mode.
   - **Purpose**: Secondary output file for higher-level summary.

### External Programs Called

- **None**: The `SA505G` RPG program does not call external programs or subroutines. It performs internal processing using the `ONETIM` subroutine for initialization.

### Additional Notes

- **Context**: Invoked by `SA505G.ocl36.txt` after `SA505C`, integrating pricing (`PRSABL`, `RKPRCEX`) and sales data (`SA505CX`, `SA505EX`) to produce summary files, complementing the rack pricing process (`BB953B`).
- **System/36 Environment**: Uses RPG II/III syntax, likely on AS/400.
- **Truncation**: The document is truncated, but key sections (`F`, `I`, `C`, `O`) provide sufficient detail. Missing calculations likely involve additional filtering and key construction (`CKEY`, `EKEY`, `BAKEY`).
- **Error Handling**: Uses indicators (`50`, `51`, `60`, `61`, `95`) for chain failures and relies on System/36 for file errors.
- **Output Strategy**: Produces two output files, suggesting a detailed summary (`?9?SA505G`) and a higher-level summary (`?9?SA505G2`), possibly for different reporting needs (e.g., Excel export vs. internal use).

If you have the RPG source code for `SA505E`, `SA505H`, `SA505I`, `SA505J`, or `SA505L`, or need further analysis, please provide those details! Let me know if you have additional questions or files to share.