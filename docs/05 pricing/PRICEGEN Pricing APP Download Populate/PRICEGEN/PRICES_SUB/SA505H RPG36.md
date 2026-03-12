The provided document, `SA505H.rpg36.txt`, is an RPG (Report Program Generator) program for the IBM System/36 environment (or AS/400 in compatibility mode), named `SA505H`. It is invoked by the OCL script `SA505H.ocl36.txt` within the `PRICES.ocl36.txt` workflow, part of the `PRICEGEN.clp` pricing generation process for blended lubes. The program processes data from `SA505G` (`SA505GX`, `SA505G2`), integrates it with rack pricing (`RKPRCEX`), customer (`ARCUST`), and sales data (`SA505EX`), and produces output files (`OUTFILE`, `OUTFILE2`) for the Customer Shipping Analysis Report. Below, I will explain the process steps, business rules, tables used, and external programs called, despite the truncation in the document.

### Process Steps of the RPG Program

1. **File Definitions**:
   - **Input Files**:
     - `SA505G2` (Input Primary, `IP`, 8 bytes, indexed with 8-byte key, disk): Secondary output from `SA505G`, mapped to `?9?SA505G2`, shared mode .
     - `RKPRCEX` (Input, `IF`, 121 bytes, indexed with 15-byte key, disk): Rack price extension file, mapped to `?9?RKPRCEX`, shared mode.
     - `BICONT` (Input, `IF`, 256 bytes, indexed with 2-byte key, disk): Contract file, mapped to `?9?BICONT`, shared mode.
     - `ARCUST` (Input, `IF`, 384 bytes, indexed with 8-byte key, disk): Customer master file, mapped to `?9?ARCUST`, shared mode.
     - `SA505GX` (Input, `IF`, 18 bytes, indexed with 18-byte key, disk): Primary output from `SA505G`, mapped to `?9?SA505G`, shared mode.
     - `SA505EX` (Input, `IF`, 40 bytes, indexed with 18-byte key, disk): Additional shipping analysis data, mapped to `?9?SA505EX`, shared mode.
   - **Output Files**:
     - `OUTFILE` (Output, `O`, 110 bytes, disk): Primary output file, mapped to `?9?TMPFILX`, shared mode.
     - `OUTFILE2` (Output, `O`, 128 bytes, disk): Secondary output file, mapped to `?9?TMPFILY`, shared mode.

2. **Input Specifications**:
   - **SA505G2** (record type `NS 01`):
     - `G2CONO` (1–2): Company number.
     - `G2CUST` (3–8): Customer number.
     - `G2KEY8` (1–8): Key field (company + customer).
   - **SA505GX** (record type `NS`):
     - `SGCO#` (1–2): Company number.
     - `SGCUST` (3–8): Customer number.
     - `SGPROD` (9–12): Product code.
     - `SGCNTR` (13–15): Container code.
     - `SGUM` (16–18): Unit of measure.
   - **RKPRCEX** (record type `NS`):
     - `XXCONO` (1–2): Company number.
     - `XXLOC` (3–5): Location.
     - `XXPRCL` (6–8): Product class code.
     - `XXPROD` (9–12): Product code.
     - `XXDES1` (13–32): Product description.
     - `XXCNTR` (33–35): Container code.
     - `XXUNMS` (36–38): Unit of measure.
     - `XXDATE` (39–46): Effective date (CYMD, CCYYMMDD).
     - `XXTIME` (47–52): Effective time (HHMMSS).
     - `XXQT01` (53–62): Quantity level.
     - `XXPRCE` (63–67, packed, 4 decimals): Price.
     - `XXINAC` (88): Inactive flag (`b`, `I`, or `B`).
   - **SA505EX** (record type `NS`):
     - `KEY18` (1–18): Key field (likely company + customer + product + container + unit of measure).
     - `SBQTY3` (not shown): Quantity (billing gallons).
     - `SBSALE` (not shown): Sales amount.
   - **User Data Structure (UDS)**:
     - `FRDT`, `TODT`: Input dates, converted to `KYFRDT`, `KYTODT` (CYMD).
     - Other fields (from `SA505C.ocl36.txt` or inherited): `KYDIV`, `KYLOC1`–`KYLOC5`, `KYPC01`–`KYPC10`, `KYCT01`–`KYCT05`, `KYPD01`–`KYPD10` for filtering.

3. **Calculation Specifications**:
   - **Initialization** (`ONETIM` subroutine):
     - `TIME TIME12`: Gets system time (`TIME12`, 12 digits).
     - `MOVE TIME12 SYDATE`: Moves time to `SYDATE` (6 digits, YYMMDD).
     - `MOVELTIME12 TIMEOF`: Moves time to `TIMEOF` (6 digits, HHMMSS).
     - `FRDT MULT 100.0001 KYFRDT`: Converts `FRDT` (YYMMDD) to `KYFRDT` (CYMD, CCYYMMDD).
     - `TODT MULT 100.0001 KYTODT`: Converts `TODT` to `KYTODT` (CYMD).
     - `SETON 05`: Sets indicator `05` (likely for initialization).
   - **Main Loop**:
     - Processes each `SA505G2` record (`01 DO`, implied).
     - Applies filters (e.g., `KYFRDT` to `KYTODT`, company, location, product, container).
   - **Chaining**:
     - `KEY18 CHAINSA505GX 90`: Chains to `SA505GX` using `KEY18` (company + customer + product + container + unit of measure) to retrieve detailed sales data.
     - `KEY18 CHAINRKPRCEX 91`: Chains to `RKPRCEX` to retrieve rack price data.
     - `KEY18 CHAINSA505EX 95`: Chains to `SA505EX` to retrieve additional sales data (quantity, sales amount).
   - **Data Accumulation**:
     - If `SA505EX` hit (`N95`):
       - `Z-ADDSBQTY3 QTY3`: Adds `SBQTY3` (quantity) to `QTY3`.
       - `Z-ADDSBSALE SALE`: Adds `SBSALE` (sales amount) to `SALE`.
     - If no `SA505EX` hit (`95`):
       - `Z-ADD*ZEROS QTY3`: Sets `QTY3` to zero.
       - `Z-ADD*ZEROS SALE`: Sets `SALE` to zero.
   - **Output**:
     - `Z-ADDG2CUST XXCUST`: Moves `G2CUST` to `XXCUST` for output.
     - `EXCPT`: Writes to `OUTFILE` and `OUTFILE2` via exception output.
   - **Loop Control**:
     - `GOTO AGAIN1`: Loops to process the next record.
     - `END` tag: Marks end of processing for each record.

4. **Output Specifications**:
   - **OUTFILE** (`EADD`, `?9?TMPFILX`, 110 bytes):
     - `EKEY` (1–18): Key field (company + customer + product + container + unit of measure).
     - `XXDATE` (19–26): Effective date (CYMD).
     - `XXTIME` (27–32): Effective time (HHMMSS).
     - `XXPRCE` (33–37, packed): Price.
     - `QTY3` (38–42, packed): Total quantity.
     - `SALE` (43–48, packed): Total sales amount.
   - **OUTFILE2** (`EADD`, `AGREE`, `?9?TMPFILY`, 128 bytes):
     - `SGCO#` (1–2): Company number.
     - `SGCUST` (3–8): Customer number.
     - `SGPROD` (9–12): Product code.
     - `SGCNTR` (13–15): Container code.
     - `SGUM` (16–18): Unit of measure.

### Business Rules

1. **Purpose**: Generates a refined Customer Shipping Analysis Report by processing `SA505G2` (company/customer summaries) and cross-referencing with `SA505GX`, `RKPRCEX`, and `SA505EX` to produce detailed output files (`?9?TMPFILX`, `?9?TMPFILY`) with sales quantities, amounts, and pricing data by ship-to, product, container, and unit of measure.
2. **Data Integration**:
   - Matches `SA505G2` (company/customer) with `SA505GX` (detailed sales) and `RKPRCEX` (rack prices) using `KEY18`.
   - Retrieves sales quantities (`SBQTY3`) and amounts (`SBSALE`) from `SA505EX`, defaulting to zero if no match.
3. **Filtering**:
   - Applies date range filters (`KYFRDT` to `KYTODT`, compared to `XXDATE`).
   - Uses UDS filters (inherited from `SA505C.ocl36.txt`):
     - Division (`KYDIV`), locations (`KYLOC1`–`KYLOC5`), product classes (`KYPC01`–`KYPC10`), containers (`KYCT01`–`KYCT05`), products (`KYPD01`–`KYPD10`).
4. **Output**:
   - `OUTFILE` (`?9?TMPFILX`): Detailed records (110 bytes) with key, date, time, price, quantity, and sales amount.
   - `OUTFILE2` (`?9?TMPFILY`): Summary records (128 bytes) with company, customer, product, container, and unit of measure.
5. **Unit of Measure Handling**:
   - Notes the need to calculate gallon prices if `SGUM` is not gallons (not implemented in provided code).
6. **Context**: Follows `SA505G` in the `PRICES.ocl36.txt` workflow, refining `SA505G` outputs by integrating rack prices and sales data, complementing rack pricing (`BB953B`).

### Tables (Files) Used

1. **SA505G2** (`?9?SA505G2`):
   - **Access**: Input Primary (`IP`), shared mode.
   - **Purpose**: Company/customer summary from `SA505G`.
2. **RKPRCEX** (`?9?RKPRCEX`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Rack price data, likely derived from `?9?RKPRCE` (`BB953`).
3. **BICONT** (`?9?BICONT`):
   - **Access**: Input (`IF`), shared mode.
   - **Purpose**: Company name (`BCNAME`).
4. **ARCUST** (`?9?ARCUST`):
   - **Access**: Input (`IF`), shared mode.
   - **Purpose**: Customer name (`ARNAME`).
5. **SA505GX** (`?9?SA505G`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Detailed sales data from `SA505G`.
6. **SA505EX** (`?9?SA505EX`):
   - **Access**: Input (`IF`), shared mode, indexed.
   - **Purpose**: Additional shipping analysis data.
7. **OUTFILE** (`?9?TMPFILX`):
   - **Access**: Output (`O`), shared mode.
   - **Purpose**: Detailed output file for report data.
8. **OUTFILE2** (`?9?TMPFILY`):
   - **Access**: Output (`O`), shared mode.
   - **Purpose**: Summary output file.

### External Programs Called

- **None**: The `SA505H` RPG program does not call external programs or subroutines. It uses the `ONETIM` subroutine for initialization.

### Additional Notes

- **Context**: Invoked by `SA505H.ocl36.txt` after `SA505G`, refining its outputs (`?9?SA505G`, `?9?SA505G2`) by integrating rack pricing (`?9?RKPRCEX`) and sales data (`?9?SA505EX`), producing detailed and summary files for the Customer Shipping Analysis Report.
- **System/36 Environment**: Uses RPG II/III syntax, likely on AS/400.
- **Truncation**: The document is truncated, but key sections (`F`, `I`, `C`, `O`) provide sufficient detail. Missing calculations likely involve additional filtering or key construction (`KEY18`).
- **Error Handling**: Uses indicators (`90`, `91`, `95`) for chain failures and relies on System/36 for file errors.
- **Unit of Measure**: Acknowledges potential need for price conversion (not implemented), suggesting flexibility for future enhancements.

If you have the RPG source code for `SA505E`, `SA505I`, `SA505J`, or `SA505L`, or need further analysis, please provide those details! Let me know if you have additional questions or files to share.