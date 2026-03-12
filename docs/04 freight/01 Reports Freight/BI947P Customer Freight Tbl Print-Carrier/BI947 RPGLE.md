The provided document, `BI947.rpgle.txt`, is an RPG (Report Program Generator) program for an IBM System/36 or AS/400 environment, named `BI947`. It is called by the OCL program `BI947.ocl36.txt` and works in conjunction with `BI947P.rpgle.txt` to produce a "Customer Freight Table List by Carrier ID." Below, I’ll explain the process steps, business rules, tables/files used, and any external programs called. Note that the document is truncated, so some logic may be incomplete, but I’ll analyze based on the available content and context from related files.

---

### Process Steps of the RPG Program

The `BI947` RPG program processes sorted freight table data to generate a printed report (`list164`) and an optional disk file (`FR947XL`) for logistics analysis. It includes detailed formatting for freight rates, carrier information, and customer data, with enhancements for current records, freight calculations, and additional fields (e.g., zip codes, surcharges). Here’s a step-by-step breakdown of the process:

1. **File and Data Structure Definitions**:
   - **Files**:
     - `BICUFR` (Input Primary, Line 0001): A 640-byte disk file containing sorted freight table data (from `BI947.ocl36.txt`’s `#GSORT` output, `?9?BI947S`).
     - `BICONT` (Input File, Line  --

System: I'm sorry, but the document `BI947.rpgle.txt` you provided is incomplete, as it was truncated at 28,083 characters, and the full program logic is not available. This limits my ability to fully analyze the process steps, business rules, and program behavior beyond what’s visible. However, based on the provided content, the context from the related OCL (`BI947.ocl36.txt`) and RPG (`BI947P.rpgle.txt`) programs, and the file definitions, I can provide a detailed analysis of the available portion and infer the likely functionality.

Below, I’ll explain the process steps and business rules based on the visible code, list the tables/files used, and identify any external programs called. I следую

System: ### Process Steps of the RPG Program

The `BI947` RPG program generates a printed report (`list164`) and an optional disk file (`FR947XL`) listing customer freight table data sorted by carrier ID, incorporating data from multiple files and applying various formatting and calculation rules. The program includes enhancements for current records, freight calculations, and additional fields like zip codes and surcharges. Here’s a breakdown of the process steps based on the available code:

1. **File and Data Structure Definitions**:
   - **Files**:
     - `BICUFR` (Line 0001): A 640-byte primary input disk file containing sorted freight table data (output from `#GSORT` in `BI947.ocl36.txt`, labeled `?9?BI947S`).
     - `BICONT` (Line 0002): A 256-byte indexed disk file (key at position 2) for freight table master data.
     - `ARCUST` (Line 0003): A 384-byte indexed disk file (key at position 2) for customer master data.
     - `BBCAID` (Line jk01): A 200-byte indexed disk file (key at position 2) for carrier ID data, replacing `GSTABL` (per change `jk01`).
     - `SHIPTO` (Line FL01): A 2048-byte indexed disk file (key at position 2) for ship-to data.
     - `list164` (Line 0004): A 164-byte printer output file for the report, with overflow indicator `*INOF`.
     - `FR947XL` (Line mg01): A 630-byte disk output file for logistics analysis data.
   - **Data Structures**:
     - `SEP` (Line 0005): An array of 82 two-byte elements for separator lines in the report.
     - `PRD` (Line FL01): An array of 10 four-byte elements for product codes.
     - `FRGHT` (Line jb07): A data structure for freight calculation parameters passed to `MBBFRT`, including fields like company number, customer number, carrier ID, freight amount, and various rates.
     - `FRTTBL` (Line jb07): A data structure mapping `BICUFR` fields, including company number, customer number, ship-to, location, carrier ID, rates, and surcharges.
     - `UDS` (Line JB02): User Data Structure containing:
       - `CS` (positions 126–185): Carrier IDs.
       - `KYCUR` (positions 186–188): Current records flag (`CUR` or blank).
       - `Y2KCEN` (positions 509–510): Century for Y2K compliance.
       - `Y2KCMP` (positions 511–512): Company number for Y2K compliance.

2. **Input Processing**:
   - **Input Specifications**:
     - `BICUFR` (Lines 0006–0014): Reads fields such as:
       - `BFDEL` (position 1): Delete flag (`D` for deleted).
       - `BFCONO` (positions 2–3): Company number.
       - `BFCUST` (positions 4–9): Customer number.
       - `BFSHIP` (positions 10–12): Ship-to number.
       - `BFLOC` (positions 13–15): Location.
       - `BFCAID` (positions 16–21): Carrier ID.
       - `BFCNCD` (positions 22–24): Container code.
       - `BFPR01–10` (positions 25–64): Product codes.
       - `BFSTD8`, `BFEXD8` (positions 65–80): Start and end dates (CYMD format).
       - Various rate fields (e.g., `BFRPM`, `BFCWT`, `BFRPG`) and surcharge fields.
     - `ARCUST` (Line 0015): Reads `ARKEY` (positions 2–9) and `ARNAME` (customer name).
     - `SHIPTO` (Line FL01): Reads `SSKEY` (positions 2–12) and `SCOZIP` (zip code).
     - Other files (`BICONT`, `BBCAID`) provide additional data for cross-referencing.

3. **Output Processing**:
   - **Printer Output (`list164`)**:
     - **Header Section (Lines 0008–0034)**:
       - Prints a header for each carrier ID (`BFCAID`) and page, including:
         - Report title (`BI947`, `* FREIGHT TABLE AGREEMENTS LIST *`).
         - Company name (`BCNAME`).
         - Page number (`PAGE`), system date (`SYSDAT`), and time (`SYSTIM`).
         - Carrier name (`CICANM` from `BBCAID`, replacing `TBDESC` per `jk01`).
         - Column headers for customer number, ship-to, location, container code, product codes, rates, and surcharges.
       - Uses separator lines (`SEP`) for formatting.
     - **Detail Section (Lines 0035–0066)**:
       - Prints detail lines for each `BICUFR` record, including:
         - `BFCUST`, `BFSHIP`, `BFLOC`, `BFCNCD` (customer, ship-to, location, container).
         - `BFPR01–10` (product codes).
         - `BFSTD8`, `BFEXD8` (start/end dates).
         - Rates (`BFRPM`, `BFCWT`, `BFRPG`, `BFRPUM`, `BFUM`, `BFFLAT`, `BFMIN`, etc.).
         - Surcharges and fees (`BFCLEN`, `BFPUMP`, `BFHOSE`, etc.).
         - Carrier-specific rates (`CFRPM`, `CFCWT`, `CFRPG`, etc.).
         - Freight total (`f$btot`, `f$ctot` from `MBBFRT`).
         - Customer name (`ARNAME`), zip code (`SCOZIP`), and carrier codes (`BFCACD`, `BFCAC2`, `BFCAC3`).
       - Handles overflow (`OFNL1`) to start new pages.
   - **Disk Output (`FR947XL`, Lines mg01)**:
     - Writes a subset of fields to a 630-byte disk file for logistics analysis, including:
       - Rates, surcharges, carrier codes, customer name, and freight totals.
       - Fields like `csctst`, `csstat`, `csmils`, `bftseq`.

4. **Freight Calculation (Implied, Line jb07)**:
   - The `FRGHT` data structure suggests a call to `MBBFRT` (not shown in the truncated code) to calculate freight amounts (`f$btot`, `f$ctot`) based on parameters like company number, customer number, carrier ID, and rates.

---

### Business Rules

Based on the code and context from related programs (`BI947P.rpgle.txt`, `BI947.ocl36.txt`), the following business rules are enforced:

1. **Data Selection**:
   - Excludes deleted records (`BFDEL ≠ 'D'`).
   - Filters by company number (`BFCONO`) and carrier ID (`BFCAID`) based on input from `BI947P` (`KYALCO`, `KYCO1–3`, `KYALCS`, `CS`).
   - Optionally filters for current records only (`KYCUR = 'CUR'`, per `JB02`), ensuring records are within valid start/end dates (`BFSTD8`, `BFEXD8`).

2. **Report Formatting**:
   - Organizes data by carrier ID (`BFCAID`, level break `L1`) with headers for each carrier.
   - Includes customer, ship-to, location, and product code details.
   - Displays freight rates (e.g., rate per mile, per hundredweight, per gallon, per unit) and surcharges (e.g., cleaning, pump, hose).
   - Shows carrier-specific rates and differences (`BFACDF`).
   - Aligns fields for readability (e.g., moved left 9 bytes per `JB07`).

3. **Freight Calculation**:
   - Uses `MBBFRT` to calculate freight amounts (`f$btot`, `f$ctot`) and surcharges (e.g., `f$SCAM`, `f$CSCAM`).
   - Supports separate surcharge display on invoices (`BFSCSP`, per `DC02`).
   - Includes insurance percentages (`BFINSP`, `CFINSP`).

4. **Output Options**:
   - Prints to `list164` for physical reports.
   - Writes to `FR947XL` for logistics analysis (per `MG01`).
   - Supports multiple copies (`KYCOPY` from `BI947P`).

5. **Data Validation**:
   - Inherits validated input from `BI947P` (e.g., valid company numbers, carrier IDs, job queue settings).
   - Cross-references data with `BICONT`, `ARCUST`, `BBCAID`, and `SHIPTO` for accuracy.

---

### Tables/Files Used

1. **Input Files**:
   - `BICUFR`: Sorted freight table data (640 bytes).
   - `BICONT`: Freight table master file (256 bytes, indexed).
   - `ARCUST`: Customer master file (384 bytes, indexed).
   - `BBCAID`: Carrier ID file (200 bytes, indexed, replaced `GSTABL` per `jk01`).
   - `SHIPTO`: Ship-to file (2048 bytes, indexed).

2. **Output Files**:
   - `list164`: Printer file for the report (164 bytes).
   - `FR947XL`: Disk file for logistics analysis (630 bytes).

---

### External Programs Called

1. **MBBFRT** (Implied, Line jb07):
   - Likely called to perform freight calculations using the `FRGHT` data structure, returning values like `f$btot`, `f$ctot`, and surcharge amounts.
   - Not explicitly shown in the truncated code but inferred from the `FRGHT` structure and context.

---

### Summary

- **Process Steps** (based on available code):
  1. Define input files (`BICUFR`, `BICONT`, `ARCUST`, `BBCAID`, `SHIPTO`) and output files (`list164`, `FR947XL`).
  2. Read sorted freight table data (`BICUFR`) and related data from other files.
  3. Generate report headers for each carrier ID, including company name, date, time, and carrier name.
  4. Output detail lines with customer, ship-to, location, product codes, rates, surcharges, and freight totals.
  5. Write logistics analysis data to `FR947XL`.
  6. Perform freight calculations (via `MBBFRT`, implied).
- **Business Rules**:
  - Exclude deleted records and filter by company/carrier based on `BI947P` input.
  - Support current records only (`KYCUR`).
  - Format rates, surcharges, and carrier data for clarity.
  - Calculate freight amounts and surcharges.
  - Produce printed and disk outputs.
- **Tables/Files**:
  - Input: `BICUFR`, `BICONT`, `ARCUST`, `BBCAID`, `SHIPTO`.
  - Output: `list164`, `FR947XL`.
- **External Programs**:
  - `MBBFRT` (implied for freight calculations).

### Notes on Truncation

The truncation of the code prevents analysis of the main processing logic (e.g., the `C` specs for calculations, loops, and calls to `MBBFRT`). However, the file definitions, data structures, and output specs provide sufficient context to infer the program’s purpose and behavior, especially when combined with `BI947.ocl36.txt` and `BI947P.rpgle.txt`. If you have the full `BI947.rpgle.txt` or additional details, I can provide a more complete analysis!