The provided document is an **RPG III program** named `BI945.rpgle.txt`, called from the OCL program `BI945.ocl36.txt`. This program creates a pricing work file (`PRICWK`) by summarizing sales gallons and the total number of orders to calculate the average gallons sold per order. It processes data from the sorted input file `SA5FILWK` and uses `SA5SHY` for additional data, producing output for subsequent freight table processing (e.g., by `BI946`). Below, I’ll explain the process steps, business rules, tables/files used, and any external programs called.

---

### Process Steps of the RPG Program

The RPG program `BI945` processes sales data to summarize gallons sold and order counts by company, customer, ship-to, location, product, container, and unit of measure, and calculates average gallons per order. Here’s a step-by-step breakdown of the process:

1. **File and Data Structure Definitions** (File and Input Specifications):
   - **Files**:
     - `SA5FILWK` (Input, Primary, 1024 bytes, disk): Contains sorted detail records from `SA5FILD` and `SA5MOVD` (as prepared by `#GSORT` in the OCL).
     - `SA5SHY` (Input, Full Procedural, keyed, disk): Sales history or summary file, used to retrieve freight code (`SHFRCD`).
     - `PRICWK` (Update/Add, keyed, disk): Output file for summarized pricing data.
   - **Input Specifications** (Lines 0016–0027):
     - Record format `NS`, indicators `01`, control level `127` (`C` or `CC`).
     - Fields from `SA5FILWK`:
       - `SALOC` (bytes 45–47): Location code (control level L4).
       - `SAPROD` (bytes 48–51): Product code (control level L3).
       - `SACO` (bytes 2–3): Company number (control level L6).
       - `SACUST` (bytes 4–9): Customer number (control level L5).
       - `SAINVN` (bytes 10–16): Invoice number.
       - `SASHIP` (bytes 17–19): Ship-to number (control level L5).
       - `SAORD` (bytes 61–66): Order number.
       - `SAUM` (bytes 76–78): Unit of measure (control level L1).
       - `SANGAL` (bytes 106–109, packed): Net gallons.
       - `SACNTR` (bytes 193–195): Container code (control level L2).
       - `SASRN#` (bytes 299–301): Shipping reference number.

2. **Initialization Subroutine (`*INZSR`)** (Lines starting at `csr *inzsr`):
   - Defines a key list (`klsa5shy`) for `SA5SHY` using fields `SACO`, `SAORD`, `SASRN#`, `SACUST`, and `SAINVN` to retrieve freight code (`SHFRCD`).
   - Initializes the program environment.

3. **Main Processing Logic** (Calculation Specifications):
   - **Reset Totals** (Lines starting at `c *inL1`):
     - At control level `L1` (unit of measure change), resets accumulators:
       - `l1cnoor` (collect orders), `l1ctoga` (collect gallons).
       - `l1pnoor` (prepaid orders), `l1ptoga` (prepaid gallons).
       - `l1anoor` (prepaid & add orders), `l1atoga` (prepaid & add gallons).
   - **Retrieve Freight Code** (Line `c klsa5shy chain sa5shy 90`):
     - Chains to `SA5SHY` using the key list to retrieve `SHFRCD` (freight code: ‘C’ for collect, ‘P’ for prepaid, ‘A’ for prepaid & add).
     - Sets indicator `90` if the record is not found.
   - **Accumulate Totals** (Lines starting at `c select`):
     - Uses a `SELECT` block to process records based on `SHFRCD`:
       - If `SHFRCD = 'C'` (collect):
         - Adds `SANGAL` to `l1ctoga` (total collect gallons).
         - If `SANGAL > 0`, increments `l1cnoor` (collect orders); otherwise, decrements it.
       - If `SHFRCD = 'P'` (prepaid):
         - Adds `SANGAL` to `l1ptoga` (total prepaid gallons).
         - If `SANGAL > 0`, increments `l1pnoor` (prepaid orders); otherwise, decrements it.
       - If `SHFRCD = 'A'` (prepaid & add):
         - Adds `SANGAL` to `l1atoga` (total prepaid & add gallons).
         - If `SANGAL > 0`, increments `l1anoor` (prepaid & add orders); otherwise, decrements it.
   - **Level 1 Totaling (`L1tot` Subroutine)** (Lines starting at `csr l1tot`):
     - Triggered at control level `L1` (unit of measure change).
     - Writes records to `PRICWK` for each freight code with non-zero gallons:
       - Sets `pwdel` (delete flag) to blank.
       - Copies `SACO` to `pwcono`, `SACUST` to `pwcust`, `SASHIP` to `pwship`, `SALOC` to `pwloc`, `SAPROD` to `pwprod`, `SACNTR` to `pwcntr`, `SAUM` to `pwunms`.
       - For collect (`l1ctoga ≠ 0`):
         - Sets `pwfrcd = 'C'`, `pwnoor = l1cnoor`, `pwtoga = l1ctoga`, `pwavgl = 0`.
         - Writes to `PRICWKpf`.
       - For prepaid (`l1ptoga ≠ 0`):
         - Sets `pwfrcd = 'P'`, `pwnoor = l1pnoor`, `pwtoga = l1ptoga`, `pwavgl = 0`.
         - Writes to `PRICWKpf`.
       - For prepaid & add (`l1atoga ≠ 0`):
         - Sets `pwfrcd = 'A'`, `pwnoor = l1anoor`, `pwtoga = l1atoga`, `pwavgl = 0`.
         - Writes to `PRICWKpf`.
   - **Level 6 Totaling (`L6tot` Subroutine)** (Lines starting at `csr l6tot`):
     - Triggered at control level `L6` (company change).
     - Calculates average gallons per order for each `PRICWK` record with matching `SACO`:
       - Positions to the first `PRICWK` record for `SACO` using `SETLL`.
       - Reads each matching record (`READE`) until end of file or company changes (indicator `96`).
       - If `pwnoor ≠ 0`, calculates `pwavgl = pwtoga / pwnoor` (average gallons per order, rounded half-up).
       - Updates the `PRICWKpf` record with `pwavgl`.

4. **Program Flow**:
   - Reads `SA5FILWK` records sequentially, accumulating gallons and order counts by freight code.
   - At `L1` breaks (unit of measure), writes summarized records to `PRICWK`.
   - At `L6` breaks (company), calculates and updates average gallons in `PRICWK`.
   - Terminates when all records are processed (`LR` indicator).

---

### Business Rules

The RPG program enforces the following business rules:
1. **Data Summarization**:
   - Summarizes sales gallons (`SANGAL`) and order counts by company (`SACO`), customer (`SACUST`), ship-to (`SASHIP`), location (`SALOC`), product (`SAPROD`), container (`SACNTR`), and unit of measure (`SAUM`).
2. **Freight Code Processing**:
   - Accumulates gallons and orders separately for freight codes:
     - `C` (collect): Tracks `l1ctoga` (gallons), `l1cnoor` (orders).
     - `P` (prepaid): Tracks `l1ptoga` (gallons), `l1pnoor` (orders).
     - `A` (prepaid & add): Tracks `l1atoga` (gallons), `l1anoor` (orders).
   - Increments order counts for positive gallons, decrements for zero or negative gallons (likely to handle returns or corrections).
3. **Output Records**:
   - Writes a `PRICWK` record for each freight code with non-zero gallons, including company, customer, ship-to, location, product, container, unit of measure, freight code, order count, and total gallons.
4. **Average Gallons Calculation**:
   - Calculates average gallons per order (`pwavgl = pwtoga / pwnoor`) for each `PRICWK` record at the company level (`L6`), updating the file only if the order count is non-zero.
5. **Data Integrity**:
   - Ensures no deleted records are processed (handled by `#GSORT` in the OCL).
   - Resets accumulators at each unit of measure change (`L1`).

---

### Tables/Files Used

1. **Input Files**:
   - `SA5FILWK` (1024 bytes, primary input): Sorted sales data from `SA5FILD` and `SA5MOVD`, containing detail records.
   - `SA5SHY` (keyed, input): Sales history or summary file, used to retrieve freight code (`SHFRCD`).

2. **Output File**:
   - `PRICWK` (keyed, update/add): Pricing work file, containing summarized gallons, order counts, and average gallons per order.

---

### External Programs Called

- **None**: The RPG program does not explicitly call any external programs (no `CALL` operations). It relies on the sorted input from `#GSORT` (in the OCL) and produces output for subsequent programs like `BI946`.

---

### Summary

- **Process Steps**: Reads sorted sales data from `SA5FILWK`, retrieves freight codes from `SA5SHY`, accumulates gallons and order counts by freight code (`C`, `P`, `A`) at the unit of measure level (`L1`), writes summarized records to `PRICWK`, and calculates average gallons per order at the company level (`L6`).
- **Business Rules**: Summarizes sales by key fields, tracks gallons and orders by freight code, adjusts order counts for positive/negative gallons, and calculates average gallons per order for non-zero order counts.
- **Tables/Files**: Input: `SA5FILWK`, `SA5SHY`. Output: `PRICWK`.
- **External Programs**: None.

If you have additional details about the `PRICWK` file structure or need further analysis of how `BI945` integrates with `BI946`, let me know!