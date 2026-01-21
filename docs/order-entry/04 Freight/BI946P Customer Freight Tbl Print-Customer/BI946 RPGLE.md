The provided document is an **RPG III program** named `BI946.rpgle.txt`, called from the OCL program `BI946.ocl36.txt`. This program is responsible for printing customer freight tables, performing freight calculations, and generating output for both printed reports and disk files for further analysis. Below, I’ll explain the process steps, business rules, tables/files used, and any external programs called, based on the provided code and its context within the OCL program.

---

### Process Steps of the RPG Program

The RPG program `BI946` processes freight table data, calculates freight costs, and produces a printed report (`list164`) and two disk files (`FR946XL`, `BICUFRC`) for logistics and pricing analysis. The program incorporates updates for calculating average gallons sold, handling current records, and including carrier-specific rates. Here’s a step-by-step breakdown of the process:

1. **File and Data Structure Definitions**:
   - **Files** (File Specifications, Lines `F`):
     - `BICUFR` (Input, Primary, 640 bytes, disk): Input file containing sorted freight data (from `BI946S` in the OCL).
     - `BICONT` (Input, Full Procedural, 256 bytes, keyed, disk): Freight table master file, used for company data (key starts at byte 2).
     - `ARCUST` (Input, Full Procedural, 384 bytes, keyed, disk): Customer master file (key starts at byte 2).
     - `SHIPTO` (Input, Full Procedural, 2048 bytes, keyed, disk): Ship-to address file (key starts at byte 2, added in 2017 update).
     - `GSCNTR1` (Input, Full Procedural, 512 bytes, keyed, disk): Control or configuration file (key starts at byte 5, added in 2020 update).
     - `PRICWK` (Input, Full Procedural, keyed, disk): Pricing work file, used for average gallons calculation (added in 2020 update).
     - `SA5FJZD` (Commented out, likely deprecated): Previously used for sales agreement data.
     - `list164` (Output, 164 bytes, printer): Printed report output with overflow indicator `*INOF`.
     - `FR946XL` (Output, 630 bytes, disk): Disk file for logistics analysis (added in 2018 update).
     - `BICUFRC` (Output, 258 bytes, disk): Disk file for consolidated pricing report for salesmen (added in 2019 update).
   - **Data Structures**:
     - `t#` (Time conversion, `jb04`): Breaks down a 14-digit timestamp (`t#time`) into components (hour, minute, second, month, day, century, year) for date handling.
     - `FRGHT` (Parameters for `MBBFRT` call, `jb02`): Defines fields for freight calculations (e.g., company, customer, order, carrier ID, freight amount, rates, surcharges).
     - `FRTTBL` (Freight table, `jb02`): Maps fields from `BICUFR` (e.g., `BFDEL`, `BFCONO`, `BFCUST`, `BFSHIP`, rates, surcharges, carrier codes).
     - `UDS` (User Data Structure, `jb02`): Includes `KYCUR` (current records flag), `Y2KCEN` (century), `Y2KCMP` (company) for input parameters and Y2K compliance.

2. **Input Processing** (Input Specifications, Lines `I`):
   - Reads `BICUFR` as the primary file (record format `NS`, indicator `01`).
   - Key fields:
     - `BFSTD8` (start date, bytes 65–72), `BFEXD8` (end date, bytes 73–80): Dates in CYMD format.
     - `ARKEY` (bytes 2–9): Customer key for `ARCUST`.
     - `SSKEY` (bytes 2–12): Ship-to key for `SHIPTO`.
     - `BFDEL` (byte 1): Delete flag (‘D’ for deleted).
     - `BFCONO` (bytes 2–3): Company number.
     - `BFCUST` (bytes 4–9): Customer number.
   - The program processes records sequentially from `BICUFR`, filtering out deleted records (`BFDEL = 'D'`) and applying date-based filtering for current records (`KYCUR = 'CUR'`).

3. **Main Processing Logic** (Calculation Specifications, Truncated):
   - Although the calculation section is truncated, the program likely:
     - Reads `BICUFR` records and matches them with `BICONT`, `ARCUST`, `SHIPTO`, `GSCNTR1`, and `PRICWK` using `CHAIN` operations to retrieve company, customer, ship-to, and pricing data.
     - Filters records based on `KYCUR` (current records only, if `'CUR'`, comparing `BFSTD8` and `BFEXD8` with the current date).
     - Calls `MBBFRT` (implied by the `FRGHT` data structure) to calculate freight costs, including:
       - Rates per mile (`BFRPM`, `CFRPM`), hundredweight (`BFCWT`, `CFCWT`), gallon (`BFRPG`, `CFRPG`), unit of measure (`BFRPUM`, `CFRPUM`).
       - Flat rates (`BFFLAT`, `CFFLAT`), minimums (`BFMIN`, `CFMIN`), and additional fees (cleaning, pump, hose, pickup, tolls, detention, etc.).
       - Surcharges (`BFSCHG`, `CFSCHG`), insurance percentages (`BFINSP`, `CFINSP`), and carrier-specific costs.
     - Calculates average gallons sold (`avggal`) using `PRICWK` data (from `jb05` update, 2020).
     - Uses constants 6500 (bulk) or 25000 (rail) for freight calculations (`jb07`, 2020).
     - Accumulates totals (`f$btot` for billed freight, `f$ctot` for carrier freight).
   - Writes output to:
     - `list164` (printed report with headers and details).
     - `FR946XL` (logistics analysis file).
     - `BICUFRC` (pricing report file).

4. **Output Processing** (Output Specifications):
   - **Printed Report (`list164`)**:
     - **Header** (Indicator `E`, Lines `jb02`):
       - Prints program name (`BI946`), company name (`BCNAME`), page number (`PAGE`), system date (`SYSDAT`), and time (`SYSTIM`).
       - Includes customer number (`BFCUST`), customer name (`ARNAME`), and static text (“FREIGHT TABLE AGREEMENTS LIST”).
       - Column headers for fields like delete flag, ship-to, container, carrier, product, rates, and surcharges.
     - **Detail** (Indicator `01`, `10`, Lines `jb02`):
       - Prints fields from `BICUFR` and calculated fields (e.g., `BFDEL`, `BFSHIP`, `BFCNTR`, `BFCAID`, `BFPR01–10`, `BFRPM`, `BFCWT`, `BFRPG`, `BFRPUM`, `BFUM`, `f$btot`).
       - Carrier-specific fields (e.g., `CFRPM`, `CFCWT`, `f$ctot`) printed if `BFACDF = 'Y'` (different actual freight cost).
   - **Disk File `FR946XL`** (Indicator `01`, `10`, Lines `mg01`):
     - Outputs detailed freight data for logistics analysis, including keys (`BFDEL`, `BFCONO`, `BFCUST`, `BFSHIP`), rates, fees, and calculated totals (`f$btot`, `f$ctot`).
   - **Disk File `BICUFRC`** (Indicator `01`, `10`, Lines `mg02`):
     - Outputs condensed freight data for salesmen’s pricing report, including keys, product code (`PRODCD`), unit of measure (`BFUM`), and calculated fields (`avggal`).

5. **Time Conversion Subroutine** (Lines `jb04`, Truncated):
   - Converts dates for freight calculations (e.g., `t#mmdd` to `t#cymdlyr`) to ensure proper date comparisons for current records.

---

### Business Rules

Based on the file definitions, output specifications, and comments, the program enforces the following business rules:
1. **Record Filtering**:
   - Excludes records with `BFDEL = 'D'` (deleted records).
   - If `KYCUR = 'CUR'`, only processes records where the current date falls between `BFSTD8` (start date) and `BFEXD8` (end date).
2. **Freight Calculations**:
   - Uses `MBBFRT` to calculate freight costs based on:
     - Rates per mile, hundredweight, gallon, or unit of measure.
     - Flat rates, minimums, and additional fees (cleaning, pump, hose, pickup, tolls, detention, etc.).
     - Surcharges and insurance percentages (customer and carrier).
   - Applies constants 6500 (bulk) or 25000 (rail) for freight calculations (`jb07`).
3. **Average Gallons Sold**:
   - Calculates average gallons sold (`avggal`) using `PRICWK` data, based on sales agreements (`jb04`, `jb05`).
4. **Output Formatting**:
   - Printed report (`list164`) includes customer, company, and freight details with headers.
   - `FR946XL` includes detailed freight data for logistics analysis.
   - `BICUFRC` provides condensed data for salesmen’s pricing reports.
5. **Carrier Cost Differentiation**:
   - If `BFACDF = 'Y'`, includes carrier-specific rates and amounts in the report.
6. **Date Handling**:
   - Uses Y2K-compliant date fields (`Y2KCEN`, `Y2KCMP`) and converts dates for accurate comparisons.

---

### Tables/Files Used

1. **Input Files**:
   - `BICUFR` (640 bytes, primary input): Sorted freight data (from `BI946S` in OCL).
   - `BICONT` (256 bytes, keyed): Freight table master file (company data).
   - `ARCUST` (384 bytes, keyed): Customer master file.
   - `SHIPTO` (2048 bytes, keyed): Ship-to address file.
   - `GSCNTR1` (512 bytes, keyed): Control or configuration file.
   - `PRICWK` (keyed): Pricing work file for average gallons calculation.
   - `SA5FJZD` (commented out, likely deprecated): Sales agreement file.

2. **Output Files**:
   - `list164` (164 bytes, printer): Printed freight table report.
   - `FR946XL` (630 bytes, disk): Logistics analysis file.
   - `BICUFRC` (258 bytes, disk): Consolidated pricing report file.

---

### External Programs Called

1. **MBBFRT**:
   - Implied by the `FRGHT` data structure, which defines parameters for a freight calculation program.
   - Calculates freight amounts, surcharges, and carrier-specific costs based on input fields (e.g., company, customer, order, carrier ID, quantity).

---

### Summary

- **Process Steps**: Reads sorted freight data from `BICUFR`, matches with `BICONT`, `ARCUST`, `SHIPTO`, `GSCNTR1`, and `PRICWK`, filters records (e.g., `KYCUR`, `BFDEL`), calculates freight costs via `MBBFRT`, and outputs to a printed report (`list164`) and disk files (`FR946XL`, `BICUFRC`).
- **Business Rules**: Filters out deleted records, processes current records if specified, calculates freight costs with carrier-specific rates, computes average gallons sold, and formats output for printing and analysis.
- **Tables/Files**: Input: `BICUFR`, `BICONT`, `ARCUST`, `SHIPTO`, `GSCNTR1`, `PRICWK`. Output: `list164`, `FR946XL`, `BICUFRC`.
- **External Programs**: `MBBFRT` (freight calculation).

If you have the calculation section of the RPG code or the source for `MBBFRT`, I can provide a more detailed analysis of the logic. Let me know if you need further clarification or additional context!