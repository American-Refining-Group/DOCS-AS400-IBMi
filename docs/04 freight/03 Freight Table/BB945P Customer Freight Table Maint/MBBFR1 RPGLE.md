The RPG program `MBBFR1.rpgle` is a module within the Bradford Order Entry/Invoices system designed to calculate freight charges for orders or invoices. It is called by programs such as `BB101` (Order Entry), `BB495` (Bill of Lading Entry), and `BB500` (Invoice Entry) to compute freight-related costs, including line haul amounts, fuel surcharges, insurance, and other accessorial charges. The program supports two modes: Order Entry (`FROEI = 'O'`) and Invoice Entry (`FROEI = 'I'`), with the latter allowing manual override of freight rates. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of `MBBFR1.rpgle`

The program operates through a series of subroutines to initialize parameters, retrieve data, and calculate freight charges. The main steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters** (via `*entry` plist, stored in `FRGHT` data structure):
     - `FCO`: Company number.
     - `FCUST`: Customer number.
     - `FORDR`: Order number.
     - `FSHIP`: Ship-to number.
     - `FCAID`: Carrier ID.
     - `FAMT`: Total freight amount (output).
     - `FFPU`: Freight per unit (input for override in Invoice mode).
     - `FRQDT`: Pickup date (YYMMDD).
     - `FQTY`: Quantity.
     - `FMSG`: Message field for errors or status.
     - `FCACD`: Carrier code.
     - `FCAFR`: Calculate freight flag.
     - `FRQCN`: Pickup date century.
     - `FFRCD`: Freight code.
     - `FRCAR`: Railcar number (for Invoice mode).
     - `FROEI`: Mode ('O' for Order Entry, 'I' for Invoice Entry).
     - Additional fields: `FSFRT` (separate freight flag), `SVPROD` (product code), `SVCNTR` (container code), `SVUM` (unit of measure), `CSMILS` (miles), `MLCNT` (multi-load count), `ROUTE`, `FGALT`, `FGGALT`, `FQTYT` (total quantity), `FWT` (weight).
   - Defines `FSPC` (fuel surcharge percentage) as a field like `bkfrpc` from `BBCFSD1`.
   - Initializes output fields (e.g., `FAMT`, `CFAMT`, `SCAM`, `CSCAM`, `INAM`, `CINAM`, `RCRAT`, `MSFR`, `OTFR`, `LNHA`, `CLNHA`, `SCPC`, `CSCPC`) to zero.

2. **Retrieve Customer Freight Data**:
   - Uses the `FRTTBL` data structure (mapped to `BICUFR` file fields) to read customer freight details.
   - Chains to `BICUFR` using key fields (`BFCONO`, `BFCUST`, `BFSHIP`, `BFLOC`, `BFCAID`) to retrieve freight parameters such as:
     - `CFRPUM`: Freight rate per unit of measure.
     - `CFSCHG`: Fuel surcharge percentage.
     - `CFINSP`: Insurance percentage.
     - `CFCLEN`, `CFPUMP`, `CFHOSE`, `CFPUUP`, `CFTOLL`, `CFDENT`, `CFINTA`, `CFSTOP`, `CFSCAL`: Accessorial charges.
     - `BFSCCA`: Surcharge calculation code ('N' for no carrier fuel surcharge, 'M' for mileage-based surcharge).

3. **Calculate Freight**:
   - **Determine Quantity**:
     - If `FROEI = 'I'` (Invoice Entry) and `FFPU ≠ 0`, uses the override freight rate (`FFPU`) instead of `CFRPUM`.
     - Calculates the effective quantity (`RESULT`) based on `FQTY`, `SVUM`, and conversion factors from `GSUMCV` (unit of measure conversion).
     - Applies minimum freight quantity (`FMIN`, `FMINNG`) if defined in `BICUFR`.
   - **Line Haul Amount**:
     - Multiplies `RESULT` by `CFRPUM` (or `FFPU`) to compute the carrier line haul amount (`CLNHA`) and initial freight amount (`CFAMT`).
     - Adjusts for negative quantities if `RESULT < 0` using `FMINNG`.

4. **Calculate Fuel Surcharge**:
   - Checks `BFSCCA` (surcharge calculation code):
     - If `BFSCCA ≠ 'N'` (JB02, 05/04/2020):
       - If `CFSCHG ≠ 0`, uses the customer-specific fuel surcharge percentage (`CFSCHG`):
         - `CSCHG = CFAMT * CFSCHG`.
         - Adds `CSCHG` to `CFAMT`.
         - Sets `CSCPC = CFSCHG * 100`, `CSCAM = CSCHG`.
       - Otherwise, retrieves the carrier fuel surcharge percentage (`FSPC`) from `BB080` (called with `FCO`, `FCAID`, `FRQCN + FRQDT`, `0`):
         - `CSCHG = CFAMT * FSPCT` (where `FSPCT = FSPC / 100`).
         - Adds `CSCHG` to `CFAMT`.
         - Sets `CSCPC = FSPCT * 100`, `CSCAM = CSCHG`.
     - If `BFSCCA = 'M'` (MG01, 07/18/2018):
       - Retrieves railcar fuel surcharge rate (`RCRAT`) from `BBRCSC2` by chaining with `FCO`, `FRCAR`, and `FRQCN + FRQDT`.
       - Calculates railcar surcharge amount (`RCSAM`) and adds it to `CFAMT` and `CSCAM`.

5. **Calculate Insurance**:
   - If `CFINSP ≠ 0`:
     - Computes insurance percentage (`CINSP = CFINSP * 0.01`).
     - Calculates insurance amount (`CINSA = CLNHA * CINSP`).
     - Adds `CINSA` to `CFAMT` and sets `CINAM = CINSA`.

6. **Add Accessorial Charges**:
   - If `CLNHA < 0`, negates accessorial charges (`CFCLEN`, `CFPUMP`, `CFHOSE`, `CFPUUP`, `CFTOLL`, `CFDENT`, `CFINTA`, `CFSTOP`, `CFSCAL`) to maintain consistency.
   - Sums accessorial charges into `CFOTH`:
     - `CFOTH = CFCLEN + CFPUMP + CFHOSE + CFPUUP + CFTOLL + CFDENT + CFINTA + CFSTOP + CFSCAL`.
   - For Invoice Entry (`FROEI = 'I'`) with multi-loads (`MLCNT ≠ 0`):
     - Multiplies `CFOTH` by `MLCNT` to account for multiple loads.
   - Adds `CFOTH` to `CFAMT` (JB03, 01/31/2023, ensures correct summation).

7. **Finalize Freight Amount**:
   - Sets the total carrier freight amount (`CTFAM = CFAMT`).
   - Updates output parameters in `FRGHT`:
     - `FAMT = CFAMT`: Total freight amount.
     - `LNHA = CLNHA`: Line haul amount.
     - `SCPC = CSCPC`: Fuel surcharge percentage.
     - `SCAM = CSCAM`: Fuel surcharge amount.
     - `INAM = CINAM`: Insurance amount.
     - `RCRAT`: Railcar fuel surcharge rate (if applicable).

8. **Program End**:
   - Returns control to the calling program with updated `FRGHT` data structure values.

---

### Business Rules

The program enforces the following business rules to ensure accurate freight charge calculations:
1. **Mode-Specific Behavior**:
   - **Order Entry (`FROEI = 'O'`)**: Reads order transaction data and uses `BICUFR` freight rates without override.
   - **Invoice Entry (`FROEI = 'I'`)**: Reads invoice transaction data (keyed by railcar number `FRCAR`) and allows override of freight rate per unit (`FFPU`).

2. **Quantity Calculation**:
   - Converts order/invoice quantity (`FQTY`) to the appropriate unit of measure using `GSUMCV` conversion factors.
   - Applies minimum freight quantities (`FMIN` for positive, `FMINNG` for negative) if specified in `BICUFR`.

3. **Fuel Surcharge Calculation**:
   - If `BFSCCA = 'N'` (JB02, 05/04/2020), skips carrier fuel surcharge calculation.
   - If `CFSCHG ≠ 0`, uses customer-specific fuel surcharge percentage from `BICUFR`.
   - Otherwise, calls `BB080` to retrieve carrier fuel surcharge percentage (`FSPC`) based on the National Diesel Fuel Index (`BBNDFI2`).
   - For `BFSCCA = 'M'` (MG01, 07/18/2018), adds railcar fuel surcharge (`RCSAM`) from `BBRCSC2`.

4. **Insurance Calculation**:
   - Applies insurance percentage (`CFINSP`) to the line haul amount (`CLNHA`) if non-zero.

5. **Accessorial Charges**:
   - Sums charges for cleaning (`CFCLEN`), pumping (`CFPUMP`), hose (`CFHOSE`), pump-up (`CFPUUP`), tolls (`CFTOLL`), detention (`CFDENT`), interchange (`CFINTA`), stops (`CFSTOP`), and scales (`CFSCAL`).
   - Adjusts for negative line haul amounts by negating accessorial charges.
   - Multiplies accessorial charges by multi-load count (`MLCNT`) in Invoice Entry mode.

6. **Data Precision**:
   - Per revision JB04 (06/17/2024), the fuel surcharge percentage (`FSPC`) is stored as a 5.2 packed decimal for increased precision (previously 3.1).

7. **Error Handling**:
   - Returns error or status messages in `FMSG` if issues occur (e.g., invalid data or missing records).

---

### Tables (Files) Used

The program accesses the following files:
1. **GSUMCV**: Product unit of measure conversion file (input, 64 bytes, 12-byte key).
2. **BBCFSH**: Carrier fuel surcharge header file (input, 84 bytes, 16-byte key).
3. **BBCFSD1**: Carrier fuel surcharge detail file (input, 96 bytes, 22-byte key).
4. **BBNDFI2**: National Diesel Fuel Index file (input, 88 bytes, 17-byte key).
5. **BBRCSC2**: Railcar fuel surcharge file (input, 88 bytes, 14-byte key).
6. **BICUFR** (not explicitly declared but referenced via `FRTTBL`): Customer freight file for freight rates and charges.

---

### External Programs Called

The program calls the following external program:
1. **BB080**: Retrieves the carrier fuel surcharge percentage (`FSPC`) based on company, carrier ID, effective date, and time.

---

### Summary

The `MBBFR1.rpgle` program, called by `BB101`, `BB495`, and `BB500`, calculates freight charges for orders or invoices in the Bradford Order Entry/Invoices system. It supports Order Entry and Invoice Entry modes, handling line haul amounts, fuel surcharges (customer-specific, carrier-based, or railcar-based), insurance, and accessorial charges. The program uses `BICUFR` for customer freight data, `GSUMCV` for unit conversions, and `BB080` for carrier fuel surcharges, with railcar surcharges from `BBRCSC2` (MG01). Revisions (e.g., JB02, JB04) enhance surcharge calculations and field precision. The program returns a comprehensive set of freight-related values in the `FRGHT` data structure for use by the calling program.