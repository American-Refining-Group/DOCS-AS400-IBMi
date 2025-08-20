The `AR822R.rpgle.txt` RPG program, written in RPG/36 (or early RPG IV) for the IBM System/36 or AS/400, is designed for **Price Lookup for Customer Service Representatives (CSR)**. It is called by the `AR820.ocl36.txt` OCL program and retrieves pricing information based on customer, product, container, and company inputs, displaying results in subfiles on a workstation screen. Below is a detailed explanation of the process steps, business rules, tables/files used, and external programs called.

---

### Process Steps of the RPG Program

The program operates in a loop, processing user input and displaying pricing data in multiple subfiles (`SFLC1`, `SFLC2`, `SFLC3`, `SFLC4`) for different pricing scenarios (e.g., sales agreements, rack pricing, or no pricing). It handles input validation, retrieves data from various files, and supports navigation via function keys. The program uses subroutines to modularize logic. Here’s a step-by-step breakdown:

1. **File and Data Definitions**:
   - **Files**:
     - `AR822RD`: Workstation file with four subfiles (`SFLC1`, `SFLC2`, `SFLC3`, `SFLC4`) for displaying pricing data.
     - `BBPRCY`: Rack pricing file (128 bytes, keyed by company, location, product, container).
     - `BICUA6`: Sales agreement file (256 bytes, keyed by company, customer, location).
     - `GSTABL`: General system table (256 bytes, commented out in parts, replaced by other files).
     - `GSPROD`: Product master file (512 bytes, keyed by product code).
     - `ARCUPR`: Customer pricing file (80 bytes, keyed by company, customer, ship-to, product, and container type after JB07).
     - `ARCUSP`: Customer special pricing file (1344 bytes, keyed by company, customer).
     - `ARCUST`: Customer master file (384 bytes, keyed by company, customer).
     - `SHIPTO`: Ship-to address file (2048 bytes, keyed by company, customer, ship-to).
     - `GSCNTR1`: Container master file (512 bytes, keyed by container code).
   - **Data Structures**:
     - `UDS`: Local data area (LDA) fields: `CO` (company, 2 digits), `PROD` (product, 4 characters), `CNTR` (container, 3 characters), `CUST` (customer, 6 characters).
     - `PSDS##`: Program status data structure for error handling (`PS#ERR`) and parameter count (`PS#PRM`).
     - Input specifications define fields for each file (e.g., `RKDEL`, `RKCONO`, `BADEL`, `CPDEL`, etc.).
   - **Entry Parameters** (DC02):
     - Parameters (`P$CO`, `P$PROD`, `P$CNTR`, `P$CUST`) can be passed from a calling program. If none are provided, LDA values are used.

2. **Initialization** (DC02, `*INZSR` Subroutine):
   - Checks for passed parameters (company, product, container, customer) via `*ENTRY PLIST`.
   - If parameters are passed (`PS#PRM >= 1`), moves them to `CO`, `PROD`, `CNTR`, `CUST`. Otherwise, uses LDA values.
   - Error handling via `*PSSR` checks for invalid parameter counts (`PS#ERR = 221`) and sets a return code.

3. **Main Processing Loop** (Lines B001–E001):
   - The program runs in a `DO` loop until `NEXT` equals `'$EOJ'`.
   - Uses a `CASE` structure to route control to subroutines based on the `NEXT` variable:
     - `$XCUTC`: Displays customer subfile (`SFLC1`).
     - `$XCUTC2`: Displays sales agreement subfile (`SFLC2`).
     - `$XCUTC3`: Displays no sales agreement screen (`SFLC3`).
     - `$XCUTC4`: Displays no rack pricing screen (`SFLC4`).
     - `$SERCH`: Performs rack pricing search.
     - `$SERCH2`: Performs sales agreement search.
     - Default: Calls `$ENTRY` for initial setup.

4. **$ENTRY Subroutine**:
   - Initializes screen fields (`S1CUST`, `S1PROD`, `S1CNTR`) from `CUST`, `PROD`, `CNTR`.
   - Clears pricing and unit of measure fields (`RKUNMS1`, `RKUNMS2`, `RKUNMS3`, `RKPRC1`, `RKPRC2`, `RKPRC3`).
   - Retrieves customer name from `ARCUST` using `CUSKEY` (company + customer); if found, sets `S1NAME`.
   - Retrieves product description from `GSPROD` using `KLPROD` (company + product); if found, sets `S1DESC`.
   - Retrieves container description and type from `GSCNTR1` using `CNTR`; if found, sets `S1CTDS` and `WKCNTY` (container type, e.g., `'B'`, `'P'`).
   - Writes initial screen (`SCRE1`) and sets indicators for subfile control.
   - Handles function keys:
     - F3: Sets `NEXT` to `'$EOJ'` to exit.
     - F5: Sets `NEXT` to `'$SERCH2'` for sales agreement search.
     - F7: Sets `NEXT` to `'$EOJ'` and `U7` indicator.
   - If `PO#` is blank, initializes subfile control (`CTLC2`) and clears subfile record numbers (`RRNC1`, `RRNC2`, `RRNC3`, `RRNC4`).
   - Sets `NEXT` to `'$SERCH'` for rack pricing search.

5. **$XCUTC, $XCUTC2, $XCUTC3, $XCUTC4 Subroutines**:
   - Each subroutine displays a specific subfile (`SFLC1`, `SFLC2`, `SFLC3`, `SFLC4`) by writing control records (`SCRC1`, `SCRC2`, `SCRC3`, `SCRC4`) and formatting the subfile (`EXFMT`).
   - Handles function keys:
     - F3: Sets `NEXT` to `'$EOJ'` to exit.
     - F5 (for `$XCUTC`, `$XCUTC4`): Sets `NEXT` to `'$SERCH2'` for sales agreement search.
     - F7: Sets `NEXT` to `'$EOJ'` and `U7` indicator.
   - These subroutines manage the display of customer, sales agreement, no sales agreement, and no rack pricing screens, respectively.

6. **$SERCH Subroutine** (Truncated, but key logic described):
   - Searches for rack pricing in `BBPRCY` using keys (`RKCONO`, `RKLOC`, `RKPROD`, `RKCNTR`).
   - Skips deleted records (`RKDEL = 'D'`) and records where `RKDATE` exceeds current date (`DATE`).
   - Populates subfile fields (`S2PROD`, `S2DESC`, `S2QTY1`, `S2QTY2`, `S2QTY3`, `S2QTY4`, `S2PRC1`, `S2PRC2`, `S2PRC3`, `S2PRC4`, `S2MINQ`) with pricing data.
   - Calls `$WRITE` to write subfile records.
   - If records are found (`RRNC1 > 0`), sets `NEXT` to `'$XCUTC'`. Otherwise, calls `$WRITE4` and sets `NEXT` to `'$XCUTC4'` (no rack pricing).

7. **$SERCH2 Subroutine**:
   - Searches for sales agreements in `BICUA6` using `FLD30` (company, customer, product, container, date, and time).
   - Skips deleted records (`BADEL = 'D'`), non-matching customer (`BACUST`), product (`BAPR01`), container (`BACNTR`), company (`BACONO`), or expired records (`BASTD8 > DATE` or `BAEND8 < DATE` unless `BAEND8 = 0`).
   - Retrieves ship-to description from `SHIPTO` using `BBKEY` (company, customer, ship-to).
   - Determines freight terms (`S4TERM`):
     - If `BAFRCD` is blank, chains to `ARCUPR` using `FRTKEY` (company, customer, ship-to, product, container type). If not found, tries with blank container type (JB07).
     - Sets `S4TERM` based on freight codes (`CPFRCD`, `CSFRCD`, `BAFRCD`):
       - `'C'`: "COLLECT" (or "COLLECT." if `CPCAFR = 'Y'`, JB09).
       - `'A'`: "PPD & ADD".
       - `'P'`: "3RD PARTY" (if `CPSFRT = 'N'` and `CPCAFR = 'N'`) or "PREPAID".
   - Populates subfile fields (`S4PRCE`, `S4UOFM`, `S4SHIP`, `S4LOC`, `S4DESC`, `S4MNQY`, `S4MXQY`, `S4POY`, `S4PORD`) with sales agreement data.
   - If `BAPRCE` is non-zero, uses it for `S4PRCE` and sets `S4TYPE` to `'P'`. Otherwise, uses `BAOFFP` (percent off price) and sets `S4TYPE` to `'O'` with yellow highlighting (JK01).
   - Calls `$WRITE2` to write subfile records.
   - If records are found (`RRNC2 > 0`), sets `NEXT` to `'$XCUTC2'`. Otherwise, calls `$WRITE3` and sets `NEXT` to `'$XCUTC3'` (no sales agreement).

8. **$CLEAR Subroutine**:
   - Clears subfile fields (`S2PROD`, `S2DESC`, `S2QTY1`, `S2QTY2`, `S2QTY3`, `S2QTY4`, `S2PRC1`, `S2PRC2`, `S2PRC3`, `S2PRC4`, `S2MINQ`, `S4PRCE`, `S4UOFM`, `S4SHIP`, `S4LOC`, `S4TERM`, `S4TYPE`, `S4MNQY`, `S4MXQY`, `SVPROD`, `SVUNMS`, `SVSHIP`, `SVLOC`, `SVCUST`) and resets indicators.

9. **$WRITE, $WRITE2, $WRITE3, $WRITE4 Subroutines** (Truncated):
   - Write records to subfiles `SFLC1`, `SFLC2`, `SFLC3`, and `SFLC4`, respectively, based on pricing data retrieved.

10. **Termination**:
    - Exits when `NEXT = '$EOJ'` or function keys (F3, F7) are pressed, setting `LR` (last record) indicator.

---

### Business Rules

1. **Input Handling**:
   - Accepts company (`CO`), product (`PROD`), container (`CNTR`), and customer (`CUST`) from parameters or LDA (DC02).
   - Validates inputs by retrieving customer name (`ARCUST`), product description (`GSPROD`), and container description/type (`GSCNTR1`).

2. **Pricing Lookup**:
   - **Rack Pricing (`BBPRCY`)**:
     - Retrieves pricing based on company, location, product, and container.
     - Skips deleted records (`RKDEL = 'D'`) and future-dated records (`RKDATE > DATE`).
     - Displays quantities and prices (`S2QTY1–4`, `S2PRC1–4`, `S2MINQ`).
   - **Sales Agreements (`BICUA6`)**:
     - Matches records by company, customer, product, container, and ship-to location.
     - Skips deleted records (`BADEL = 'D'`), non-matching records, or expired records (`BASTD8 > DATE` or `BAEND8 < DATE` unless `BAEND8 = 0`, JB03).
     - Uses `BAPRCE` (price) or `BAOFFP` (percent off price) for `S4PRCE`, with `S4TYPE` indicating price type (`'P'` or `'O'`, JK01).
     - Displays purchase order fields (`S4POY`, `S4PORD`) if `BAPORD` is non-blank (JB03).
     - Retrieves ship-to city (`S4DESC`) from `SHIPTO` (DC01).

3. **Freight Terms**:
   - Determines freight terms (`S4TERM`) based on `BAFRCD` (from `BICUA6`), `CPFRCD` (from `ARCUPR`), or `CSFRCD` (from `ARCUSP`):
     - `'C'`: "COLLECT" (or "COLLECT." for freight collect with calculation, JB09).
     - `'A'`: "PPD & ADD".
     - `'P'`: "PREPAID" or "3RD PARTY" (based on `CPSFRT` and `CPCAFR`).
   - For non-fluid products, tries container type `'P'` if initial lookup fails (JB07).
   - Special case: Freight collect with service fee (`CYY`, JB10) adds $100 to customer billing when arranged by ARG but billed by the carrier.

4. **Subfile Display**:
   - `SFLC1`: Customer pricing details (rack pricing).
   - `SFLC2`: Sales agreement details (price, PO, freight terms).
   - `SFLC3`: Displayed when no sales agreement is found.
   - `SFLC4`: Displayed when no rack pricing is found.
   - Supports up to 100 records in `SFLC2` before setting indicator `80`.

5. **Function Key Behavior**:
   - **F3**: Exits the program.
   - **F5**: Triggers sales agreement search (`$SERCH2`) from `SFLC1` or `SFLC4`.
   - **F7**: Exits with `U7` indicator set.

6. **Historical Modifications**:
   - **DC01 (05/12/2010)**: Replaced `BBSHSP` with `SHIPTO` for ship-to data.
   - **DC02 (11/27/2011)**: Added parameter passing, falling back to LDA if none provided.
   - **JB03 (01/09/2013)**: Added purchase order fields (`BAPORD`, `S4POY`, `S4PORD`) and allowed zero end date for sales agreements.
   - **JB04 (01/09/2013)**: Added fields for no rack required and inactive status in display.
   - **MG05 (04/25/2013)**: Ignores deleted rack pricing records.
   - **JB06 (06/05/2013)**: Ignores deleted sales agreement records.
   - **JK01 (01/23/2015)**: Displays `BAOFFP` in yellow if non-zero, removes calculated off-price logic.
   - **JB07 (05/18/2015)**: Added container type to `ARCUPR` key, tries blank or `'P'` container type if lookup fails.
   - **JK02 (04/13/2016)**: Replaced `GSTABL` container lookup with `GSCNTR1`.
   - **JB09 (05/19/2020)**: Added freight collect logic (`CNY`) for non-Bradford shipments.
   - **JB10 (01/17/2022)**: Added freight collect with service fee (`CYY`, $100 charge).
   - **JK03 (01/02/2023)**: Replaced `GSTABL` product lookup with `GSPROD`.

---

### Tables/Files Used

1. **AR822RD**: Workstation file with subfiles `SFLC1`, `SFLC2`, `SFLC3`, `SFLC4` for displaying pricing data.
2. **BBPRCY**: Rack pricing file (key: company, location, product, container).
3. **BICUA6**: Sales agreement file (key: company, customer, location).
4. **GSTABL**: General system table (partially commented out, replaced by `GSPROD` and `GSCNTR1`).
5. **GSPROD**: Product master file (key: product code).
6. **ARCUPR**: Customer pricing file (key: company, customer, ship-to, product, container type).
7. **ARCUSP**: Customer special pricing file (key: company, customer).
8. **ARCUST**: Customer master file (key: company, customer).
9. **SHIPTO**: Ship-to address file (key: company, customer, ship-to).
10. **GSCNTR1**: Container master file (key: container code).

---

### External Programs Called

- **None**: The `AR822R` program does not call any external RPG programs. It relies on internal subroutines (`$ENTRY`, `$XCUTC`, `$XCUTC2`, `$XCUTC3`, `$XCUTC4`, `$SERCH`, `$SERCH2`, `$CLEAR`, `$WRITE`, `$WRITE2`, `$WRITE3`, `$WRITE4`, `*PSSR`, `@PARMS`, `*INZSR`) for its logic.

---

### Summary

The `AR822R.rpgle.txt` program is a pricing lookup tool for CSRs, retrieving rack pricing (`BBPRCY`) and sales agreement data (`BICUA6`) based on company, customer, product, and container inputs. It validates inputs using `ARCUST`, `GSPROD`, and `GSCNTR1`, determines freight terms from `ARCUPR` and `ARCUSP`, and displays results in subfiles. It supports purchase order fields, freight collect scenarios (including a $100 service fee for `CYY`), and flexible container type lookups. No external programs are called, and the listed files support the pricing and customer data required.