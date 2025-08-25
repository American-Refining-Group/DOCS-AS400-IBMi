The provided document, `BI9445.rpgle.txt`, is an RPGLE (RPG IV) program used within the context of the `BI9445.ocl36.txt` OCL program on an IBM midrange system (e.g., AS/400 or iSeries). The program generates a customer sales agreement price change report, processing data from the `BICUAGXX` file (aliased as `BI944T` in the OCL) and referencing multiple files to produce printed and Excel-compatible output. It also handles updates to the `BICUAGC` file for CRM uploads when run in a specific mode. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BI9445` RPGLE program processes customer sales agreement data, validates it, and generates a detailed report (`prtdown` and `prtexcel`) while optionally updating the `B204W` (aliased as `BB204T`) and `BICUAGC` files. The program has been modified over time to address issues like pricing, container codes, and freight terms. Here’s a detailed breakdown of the process steps:

1. **Program Initialization**:
   - **Header Specifications**:
     - `h fixnbr(*zoned:*inputpacked)`: Ensures proper handling of zoned and packed decimal numeric fields.
     - `h dftname(bi9445)`: Sets the default program name to `BI9445`.
   - **File Definitions**:
     - Input Files (all with indexed access, `aidisk`):
       - `bicuagxx ip f 263 disk`: Primary input file (263 bytes, aliased as `BI944T` in OCL).
       - `bicont if f 256 2aidisk keyloc(2)`: Contract file.
       - `prsablx if f 88 24aidisk keyloc(2)`: Sales agreement file (from `BI945P`).
       - `bbprcy if f 128 27aidisk keyloc(2)`: Pricing file.
       - `arcust if f 384 8aidisk keyloc(2)`: Customer master file.
       - `gstabl if f 256 12aidisk keyloc(2)`: General table file.
       - `shipto if f 2048 11aidisk keyloc(2)`: Ship-to address file.
       - `arcupr if f 80 16aidisk keyloc(2)`: Unit pricing file.
       - `arcusp if f 1344 8aidisk keyloc(2)`: Customer pricing file.
       - `inloc if f 512 5aidisk keyloc(2)`: Location file.
       - `bicua7 if f 261 37aidisk keyloc(1) extk`: Sales agreement file (replaces `BICUA5`).
       - `gsumcv if f 64 12aidisk keyloc(2)`: Unit of measure conversion file.
       - `gscntr1 if f 512 3aidisk keyloc(5)`: Container code file (replaces `GSTABL` for container codes).
     - Output Files:
       - `bb204t o a f 327 11aidisk keyloc(1) extk`: Work file for storing new start dates.
       - `prtdown o f 164 printer oflind(*inog)`: Printed report output.
       - `prtexcel o f 224 printer`: Excel-compatible report output.
       - `bicuagc o a f 271 disk extind(*inu3)`: Output file for CRM uploads, updated only when indicator `U3` is on (hourly procedure).
   - **Data Definitions**:
     - Arrays: `sep` (separators), `bap` (product codes), `bad` (product descriptions), `fld`, `cty`, `st` (state codes), `desc` (freight terms), `com` (division descriptions), `err` (error messages).
     - Data Structures:
       - `shpkey`: Combines company (`bacono`), customer (`bacust`), and ship-to (`baship`).
       - `prckey`: Combines company (`co`), customer (`cust`), and product code (`bapr01`).
       - `pskey`: Combines company (`cono`), location (`loc`), customer (`cust#`), product (`baprod`), unit of measure (`baum`), contract (`bacntr`), and ship-to (`ship`).
     - User Data Structure (UDS): Includes LDA fields like `kysprd`, `kyindc`, `kydlch`, `kyadda`, `kystdt`, `kystd8` for controlling processing.
   - **Input Record Definition (`bicuagxx`)**:
     - Key fields: `arkey` (2–9, company/customer), `bacono`/`co`/`cono` (2–3), `bacust`/`cust`/`cust#` (4–9), `baloc`/`loc` (10–12), `baship`/`ship` (167–169).
     - Product fields: `baprod`/`bapr01` (13–16), `bapr02`–`bapr10` (17–52).
     - Dates: `bastdt` (53–59), `basttm` (60–63), `baendt` (64–70), `bastd8` (118–125), `baend8` (126–133).
     - Pricing: `baprce` (76–80, packed), `baoffp` (82–86, packed).
     - Quantities: `bamnqy` (87–93), `bamxqy` (94–100).
     - Other: `bappd` (101, prepaid), `baalsh` (102, apply to all ship-to), `baprim` (106, pricing type), `bacntr` (164–166, container code), `badelv` (134, delivery), `bafrcd` (135, freight code), `bacrdt` (136–143, creation date), `baludt` (150–157, last update date), `baseqn` (184–192, sequence number), `baprcl` (product class, derived from `BI944X`).
   - Purpose: Sets up files, fields, and data structures for processing sales agreements.

2. **Main Processing Loop**:
   - The program processes each record from `bicuagxx` (indicator `01` set for valid records). Key steps include:
     - **Retrieve Customer Data**:
       - Chains `arcust` using `arkey` (company/customer) to get customer name (`arname`) and other details.
     - **Retrieve Ship-to Data**:
       - Chains `shipto` using `shpkey` (company, customer, ship-to) to get city/state (`ctyst`) and freight details.
       - If no ship-to record exists, sets an error message (`err(3)` or `err(4)`).
     - **Retrieve Product Class Description**:
       - Chains `gstabl` using product class (`baprcl`) to get description (`prclds`).
       - If no product class is found, sets error message `err(1)` ("NO PRODUCT CLASS LISTED").
     - **Retrieve Container Code**:
       - Chains `gscntr1` using `bacntr` to validate container code (per revision `JK01`).
     - **Retrieve Pricing Data**:
       - Chains `arcupr` using `prckey` (company, customer, product) with container code (`bacntr`), trying:
         - Exact container code first (per `JB05`).
         - Blank container code if no match.
         - 'P' container code for non-fluid products.
       - Retrieves previous price (`prvprc`) and compares it with current price (`baprce`).
     - **Freight Terms**:
       - Uses `bafrcd` (from `bicuagxx`, per `MG09`) for freight terms, mapping to descriptions in `desc` array (e.g., COLLECT, PPD & ADD).
       - Computes freight difference (`frtdif`) for specific cases (e.g., `CNY` for freight collect from non-Bradford locations, per `JB10`).
     - **Validate Dates**:
       - Compares start date (`bastd8`) and end date (`baend8`) with system date (`kystd8`) to ensure validity.
       - For perpetual agreements (`baend8 = 0`), displays '00/00/00' in the report.
     - **Calculate New Price**:
       - Applies price change logic based on `kyindc` (price index) and `kydlch` (delta change) if `kysprd` = 'Y'.
       - Computes new price (`price`) for the report.

3. **Output Generation**:
   - **Printed Report (`prtdown`)**:
     - Header (level `L4`):
       - Prints customer name (`bcname`), report title, page number (`page`), date (`sysdat`), time (`systim`), and division name (`divnam`).
       - Indicates report mode: "REPORT ONLY" if `kyadda = 'N'`, or "ADD/UPDATE OF AGREEMENT TABLE" if `kyadda = 'Y'`.
       - Prints column headers: Product Code, Description, Customer No., Name, Ship-to, Location, Container, Prices, Freight Terms, Start/End Dates.
     - Detail Lines (level `L3`, `L4`):
       - Prints product code (`bap(1)`), description (`bad(1)`), customer number (`bacust`), name (`arname`), ship-to (`baship` or 'ALL'), city/state (`ctyst`), container code (`bacntr`), current price (`baprce`), previous price (`prvprc`), location (`baloc`), unit of measure (`baunms`), freight terms (`frtdsc`), start date (`stdt`), end date (`endt` or '00/00/00'), new price (`price`).
       - Includes minimum/maximum quantities (`bamnqy`, `bamxqy`) and freight difference (`frtdif`).
     - Total (level `LR`):
       - Prints total record count (`count`).
   - **Excel Report (`prtexcel`)**:
     - Similar to `prtdown` but with wider format (224 bytes) and additional fields like off-price (`baoffp`).
     - Includes headers and detail lines with similar data, formatted for Excel compatibility.
   - **Work File (`bb204t`)**:
     - Writes new start dates and other agreement data for use in subsequent processes.
   - **CRM Upload File (`bicuagc`)**:
     - Writes to `bicuagc` (271 bytes) only if indicator `U3` is on (hourly CRM upload, per `MG05`).
     - Includes fields like customer, product, pricing, dates, and freight terms.

4. **Error Handling**:
   - Sets error messages for missing product class (`err(1)`), invalid product (`err(2)`), or missing ship-to record (`err(3)`, `err(4)`).
   - Prints errors in the report to highlight issues.

5. **Program Termination**:
   - At last record (`LR`), outputs the total count and closes files.
   - Purpose: Produces a comprehensive report and updates work/CRM files as needed.

---

### Business Rules

1. **Agreement Validation**:
   - Validates product class, container code, customer, and ship-to data using reference files (`gstabl`, `gscntr1`, `arcust`, `shipto`).
   - Flags errors for missing or invalid data (e.g., no product class, no ship-to record).
2. **Price Change Logic**:
   - Retrieves previous price (`prvprc`) from `arcupr` using container code, falling back to blank or 'P' if needed (per `JB05`).
   - Applies price index (`kyindc`) and delta change (`kydlch`) to compute new price (`price`) if special pricing is enabled (`kysprd = 'Y'`).
3. **Freight Terms**:
   - Uses freight code (`bafrcd`) from `bicuagxx` (per `MG09`), ignoring `shipto` freight data (reversing `JB08`).
   - Computes freight difference (`frtdif`) for specific cases (e.g., `CNY` for freight collect from non-Bradford locations, per `JB10`).
4. **Date Handling**:
   - Displays '00/00/00' for perpetual agreements (`baend8 = 0`).
   - Validates start/end dates against system date (`kystd8`).
5. **Output Modes**:
   - If `kyadda = 'Y'`, updates agreement tables (`bb204t`, `bicuagc` if `U3` is on).
   - If `kyadda = 'N'`, produces report only without updates.
6. **CRM Integration**:
   - Updates `bicuagc` only during hourly runs (`U3` on) for CRM uploads (per `MG05`).

---

### Tables Used

1. **Input Files**:
   - `bicuagxx` (aliased as `BI944T`): Primary input, containing sorted sales agreement data (263 bytes).
   - `bicont`: Contract file.
   - `prsablx`: Sales agreement file (from `BI945P`).
   - `bbprcy`: Pricing file.
   - `arcust`: Customer master file.
   - `gstabl`: General table file (product class descriptions).
   - `shipto`: Ship-to address file.
   - `arcupr`: Unit pricing file (previous prices).
   - `arcusp`: Customer pricing file.
   - `inloc`: Location file.
   - `bicua7`: Sales agreement file (replaces `BICUA5`, indexed from `BICUAGX`).
   - `gsumcv`: Unit of measure conversion file.
   - `gscntr1`: Container code file (replaces `GSTABL` for container codes).
2. **Output Files**:
   - `bb204t`: Work file for new start dates (327 bytes).
   - `prtdown`: Printed report (164 bytes).
   - `prtexcel`: Excel-compatible report (224 bytes).
   - `bicuagc`: CRM upload file (271 bytes, updated if `U3` is on).
3. **Related Files (from OCL Context)**:
   - `BICUAG`, `BICUAGX`, `BICUAG2`, `BI944S`: Used in `BI9445.ocl36.txt` for data preparation.
   - `GPRSABLO`, `GPRSABLN` (from `BI945P.ocl36.txt`): Source data for `prsablx`.

---

### External Programs Called

The `BI9445` RPGLE program does not explicitly call external programs (no `CALL` or `CALLP` operations). It relies on file operations (`CHAIN`, `READ`, `WRITE`) and internal logic to process data. The OCL program (`BI9445.ocl36.txt`) orchestrates the overall process, calling `BI944X` and `#GSORT` before `BI9445`.

---

### Summary

The `BI9445` RPGLE program generates a customer sales agreement price change report, validating data from `bicuagxx` against multiple reference files, computing new prices, and handling freight terms. It produces printed (`prtdown`) and Excel (`prtexcel`) reports, updates `bb204t` with new start dates, and optionally updates `bicuagc` for CRM uploads. Errors are flagged for invalid data.

**Process Steps**:
1. Initialize files, arrays, and data structures.
2. For each `bicuagxx` record:
   - Retrieve customer, ship-to, product class, container, and pricing data.
   - Validate data and flag errors.
   - Compute new price and freight terms.
   - Output to `prtdown`, `prtexcel`, `bb204t`, and `bicuagc` (if `U3` is on).
3. Print total record count at the end.

**Business Rules**:
1. Validate product, customer, and ship-to data.
2. Retrieve previous prices with container code priority.
3. Use `bicuagxx` freight code for terms.
4. Apply price changes if special pricing is enabled.
5. Handle perpetual agreements with default end date display.
6. Update CRM file only during hourly runs.

**Tables Used**:
- Input: `bicuagxx`, `bicont`, `prsablx`, `bbprcy`, `arcust`, `gstabl`, `shipto`, `arcupr`, `arcusp`, `inloc`, `bicua7`, `gsumcv`, `gscntr1`.
- Output: `bb204t`, `prtdown`, `prtexcel`, `bicuagc`.
- Related: `BICUAG`, `BICUAGX`, `BICUAG2`, `BI944S`, `GPRSABLO`, `GPRSABLN`.

**External Programs Called**:
- None (handled by OCL: `BI944X`, `#GSORT`).

If you need further details on specific subroutines or file structures, let me know!