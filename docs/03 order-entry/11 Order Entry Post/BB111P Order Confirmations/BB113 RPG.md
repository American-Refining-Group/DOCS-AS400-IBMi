### Process Steps of the RPG Program BB113

This RPG III program (BB113) generates freight purchase order (PO) files (BBFPORX series) during the picklist printing process (called from BB110 in the overall workflow). It processes order data to create freight-related records, performs calculations (e.g., quantity conversions to units, lbs, gallons), and invokes external PC-based programs for freight processing (internal or external). The program supports distinctions between internal (ARG LMS) and external (SCOXML) freight processors, with revisions adding features like skipping deleted lines, handling rush orders, and address management. It runs in a standard RPG cycle: read input → chain lookups → calculate → output → call external. Based on the file specs, input mappings, and visible logic (code truncated), below is a step-by-step breakdown:

1. **Program Initialization**:
   - No explicit *INZSR; uses default RPG init.
   - Sets keys for reading (e.g., company + order# from input parms or prior context).
   - Opens files (inputs for order data, lookups; outputs additive for freight POs).

2. **Read Input Records**:
   - Reads primary input file `BBORTR3` (sorted order records from prior steps like BB111).
   - Identifies record types via indicators (01 for headers: 10 C0, 11 C0, 12 C0).
   - Processes headers first (seq# 000), then details/misc (other seq#).

3. **Header Processing**:
   - Maps fields from input (e.g., BODEL for delete flag, BOCO company, BORDNO order#, BOCUST customer, BOSHIP ship-to, freight codes like BOCACD carrier, BOFPER period).
   - Chains to lookup files:
     - `SHIPTO` for ship-to details (name, addr, city/state/zip/country).
     - `ARCUST` for customer details (name, addr).
     - `BBOTHS1`/`BBOTDS1`/`BBOTA1` for order headers/details/attachments.
     - `BBSHPH`/`BBSHPD` for shipment headers/details.
     - `BBFRPR` for freight pricing.
     - `BBORF` for order freight (copied to BBFPORF per MG10/JB11).
     - `BBFPORH1` for existing freight PO headers.
   - Handles freight bill address: From BB110 parms; for internal, don't use processor addr; for external, use it (JB08).
   - Copies/adds fields to output: e.g., rush flag (BORUSH), order process code (BOORPR) from DC02; dispatch marks print (JB05).
   - Writes header to `BBFPORH` (EXCEPT HEADH or similar).

4. **Detail Processing and Calculations**:
   - For detail lines (non-header seq#):
     - Skips if deleted (BODEL='D'; JB04, but sends deleted orders to internal per JB09).
     - Maps fields like BDLOC location, BDPROD product, BDQTY qty, BDCNTR container, BDUM unit measure.
     - Calculates conversions (JB01): Order container qty to UM qty, lbs, net gallons (using lookups like GSCTWT/GSCTUM implied from system).
     - Accumulates totals: Accessorial costs to BBFPORA (JB02); total gross gallons to BBFPORF (JB12).
     - Handles product cross-ref expansion to 20 chars (MG15); desc to 30 chars.
     - Writes details to `BBFPORD` (line items), `BBFPORA` (accessorials), `BBFPORF` (freight specifics; copy from BBORF per MG10/JB11/JB12).

5. **Output Writing**:
   - Additive outputs (EADD) to freight PO files:
     - `BBFPORH`: Header data (e.g., order#, customer, ship-to addr, totals like BOPRTO product total, BOMITO misc, BOFRTO freight).
     - `BBFPORD`: Detail lines (e.g., product, qty, weights).
     - `BBFPORA`: Accessorial/misc charges.
     - `BBFPORF`: Freight details (e.g., BFFTCN ftcn, BFMILE miles, BFTWGT total weight, BFTGAL gallons, etc.; expanded per JB11/JB12).
   - Includes fields like marks (BXOMK1-4 order, BXBMK1-4 bill, BXIMK1-2 invoice, BXDSP1-4 dispatch), addresses (ARNAME/ADR, SNAM/SAD, freight processor via @FB* from JB08).

6. **External Freight Processing Call**:
   - Builds command strings from ** constants (COM array):
     - For external: Calls SCOXML.EXE on server (e.g., \\BRADFORD15\SCOPRODBIN$\SCOXML.EXE /EMAIL /FTP=/EMAIL).
     - For internal: Calls InsertARGLMSOrder.EXE (e.g., \\B-P-APP1\APPLICATIONS\ARGLMS\PROD\InsertARGLMSOrder.EXE).
   - Executes via STRPCCMD (implied from constants; JB08/JB14 for server changes).
   - Handles test/prod environments.

7. **Program Termination**:
   - Ends on EOF of input (LR on).
   - No explicit cleanup; files close automatically.

The program is called per order/batch from BB110, focuses on freight calc/setup for shipping.

### Business Rules

- **Freight Processor Distinction (JB08)**: 
  - Internal (ARG LMS): Populate BBFPOR tables, call InsertARGLMSOrder.EXE, don't use freight processor addr as bill-to (use customer/ship-to instead). Send deleted orders (JB09).
  - External (SCOXML): Populate tables, call SCOXML.EXE, use freight processor addr as bill-to.
- **Deleted Handling**: Skip detail lines if deleted (JB04), but send full deleted orders to internal processor (JB09).
- **Calculations**: Convert container qty to UM/lbs/net gallons (JB01); total gross gallons (JB12). Accumulate accessorial totals correctly (JB02).
- **Addresses**: Compress/use correct bill-to (from BB110; DC01 for BBSHSP->SHIPTO replacement). Include country/state/zip.
- **Special Fields**: Add dispatch marks print (JB05), rush/order process code to header (DC02). Expanded cross-ref/desc (MG15). Add BDSQTY to avoid nulls (MG13).
- **Copy Rules**: Direct copy BBORF to BBFPORF (MG10, revised layout JB11/JB12).
- **Environment**: Test/prod server paths (JB14); no internet/external installs.

### Tables Used

- `BBORTR3`: Input (IF) disk file (512 bytes, key loc 2: company + order# + seq#). Primary order records.
- `SHIPTO`: Input (IF) disk file (2048 bytes, key loc 2). Ship-to addresses (replaced BBSHSP per DC01).
- `ARCUST`: Input (IF) disk file (384 bytes, key loc 2). Customers.
- `BBOTHS1`: Input (IF) disk file (512 bytes, key loc 2). Order headers.
- `BBOTDS1`: Input (IF) disk file (512 bytes, key loc 2). Order details.
- `BBOTA1`: Input (IF) disk file (512 bytes, key loc 2). Order attachments.
- `BBSHPH`: Input (IF) disk file (128 bytes, key loc 2). Shipment headers.
- `BBSHPD`: Input (IF) disk file (256 bytes, key loc 2). Shipment details.
- `BBFRPR`: Input (IF) disk file (256 bytes, key loc 2). Freight pricing.
- `BBFPORH1`: Input (IF) disk file (2176 bytes, key loc 2). Existing freight PO headers.
- `BBORF`: Input (IF) disk file (640 bytes, key loc 2). Order freight (copied to output).
- `BBFPORH`: Output additive (O A F) disk file (2176 bytes, key loc 2). Freight PO headers.
- `BBFPORD`: Output additive (O A F) disk file (768 bytes, key loc 2). Freight PO details.
- `BBFPORA`: Output additive (O A F) disk file (256 bytes, key loc 2). Freight PO accessorials.
- `BBFPORF`: Output additive (O A F) disk file (640 bytes, key loc 2). Freight PO freight (expanded per JB11).

Inputs are keyed for chains; outputs additive for building POs.

### External Programs Called

- `SCOXML.EXE`: PC executable for external freight (called via STRPCCMD, e.g., on \\BRADFORD15\SCOPRODBIN$ or test; params /EMAIL /FTP=/EMAIL).
- `InsertARGLMSOrder.EXE`: PC executable for internal freight (called via STRPCCMD, e.g., on \\B-P-APP1\APPLICATIONS\ARGLMS\PROD or test).

No other calls; uses system STRPCCMD for PC integration.