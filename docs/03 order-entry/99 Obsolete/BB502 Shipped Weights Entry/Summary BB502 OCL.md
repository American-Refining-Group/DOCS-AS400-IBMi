Based on the provided RPG program files (`BB502.rpg36.txt`, `BB1014.rpgle.txt`, `BB503N.rpg.txt`, `MBBQTY.rpgle.txt`, `MINT6D.rpgle.txt`, `MINLBGL1.rpgle.txt`, `BB5003.rpgle.txt`, `VSCDS1R.rpg.txt`), the call stack represents a system for processing shipment weight entry as part of an order entry and shipment processing workflow. Below, I’ll identify the use cases implemented by this call stack and then provide a concise function requirement document for a non-interactive version of the primary use case, focusing on business requirements and necessary calculations.

---

### Use Cases Implemented by the Call Stack

The call stack, driven by `BB502` and its subroutines, supports multiple use cases related to shipment weight entry and order processing. The following use cases are derived from the programs’ functionality and their interactions:

1. **Retrieve Location Information (BB1014)**:
   - **Purpose**: Retrieve location details (name, phone number, inventory type) for a given company and location code.
   - **Description**: The `BB1014` program is called to fetch location information from the `INLOC` file based on a company number and location code. It formats the phone number and returns the data for use in shipment processing or order validation.
   - **Inputs**: Company number (`@4CONO`), location code (`@4LOC`).
   - **Outputs**: Location name (`@4LOCN`), formatted phone number (`@4LOCP`), inventory type (`@4LINT`).

2. **Create SCO Files for Shipment Weight Entry (BB503N)**:
   - **Purpose**: Generate shipment data files (`BBFPORH`, `BBFPORD`, `BBFPORA`, `BBFPORF`) for Supply Chain Operations (SCO) processing, including freight processor handling.
   - **Description**: The `BB503N` program processes order and shipment data, converts quantities, calculates weights and gallons, and populates SCO files. It handles internal and external freight processors, calling appropriate PC programs (`SCOXML.EXE` or `InsertARGLMSOrder.EXE`) based on the processor type.
   - **Inputs**: Order header and detail data, shipment details, freight processor code.
   - **Outputs**: Populated `BBFPORx` files, freight processor addresses, and execution of external PC programs.

3. **Convert Order Quantity to Pounds and Gallons (MBBQTY)**:
   - **Purpose**: Convert order quantities to pounds, gross gallons, net gallons, and shipping weight for shipment processing.
   - **Description**: The `MBBQTY` program converts container quantities (`CTQT`) to pounds (`NETLBS`), gross gallons (`GGAL`), net gallons (`NGAL`), and shipping weight (`SHPLBS`) using conversion factors from files (`GSCTUM`, `GSUMCV`, `GSCTWT`) or formulas for specific units (e.g., `KG`, `LI`, `ML`, `OZ`).
   - **Inputs**: Company (`CO`), product (`PROD`), container (`CNTR`), unit of measure (`UM`), container quantity (`CTQT`), tare weight (`TARE`), gross weight (`GVWT`), car capacity (`CACA`), outage (`OUTA`), carrier code (`CACD`), fluid code (`FLCD`), temperature (`TEMP`), gravity (`GRAV`), VCF code (`VCF`).
   - **Outputs**: Shipped quantity (`SQTY`), gross gallons (`GGAL`), net gallons (`NGAL`), net pounds (`NETLBS`), shipping pounds (`SHPLBS`), error code (`ERR`).

4. **Calculate Table 6D Factor (MINT6D)**:
   - **Purpose**: Calculate the Table 6D factor for converting between gross and net quantities based on temperature and gravity.
   - **Description**: The `MINT6D` program computes a volume correction factor (`T6DFAC`) using temperature (`T6DTEM`), gravity (`T6DGRA`), and VCF code (`T6DVCF`) with predefined constants for product types (e.g., crude oil, gasoline).
   - **Inputs**: Temperature (`T6DTEM`), gravity (`T6DGRA`), VCF code (`T6DVCF`).
   - **Outputs**: Table 6D factor (`T6DFAC`).

5. **Convert Between Pounds and Net Gallons (MINLBGL1)**:
   - **Purpose**: Convert quantities between pounds and net gallons using API gravity.
   - **Description**: The `MINLBGL1` program converts pounds (`KYLBS`) to net gallons (`KYGAL`) or vice versa using the API standard formula and gravity (`KYGRAV`).
   - **Inputs**: API gravity (`KYGRAV`), pounds (`KYLBS`) or net gallons (`KYGAL`).
   - **Outputs**: Converted pounds (`KYLBS`) or net gallons (`KYGAL`).

6. **Retrieve Selling Tank Number (BB5003)**:
   - **Purpose**: Retrieve the selling tank and rail car tank numbers for an order.
   - **Description**: The `BB5003` program retrieves tank numbers (`STTANK`, `STRCTK`) from the `INSLTZ` file using a provided key (`SELKEY`), falling back to the previous non-deleted record if necessary.
   - **Inputs**: 17-character key (`SELKEY`).
   - **Outputs**: Selling tank number (`STTANK`), rail car tank number (`STRCTK`).

7. **Select Carrier Information (VSCDS1R)**:
   - **Purpose**: Retrieve or interactively select carrier information (code, description, ID) for shipment processing.
   - **Description**: The `VSCDS1R` program validates a provided carrier code or allows interactive selection via a subfile, searching carrier descriptions in the `VSCARR` file.
   - **Inputs**: Carrier code (`@RTCOD`).
   - **Outputs**: Carrier code (`@RTCOD`), description (`@RTDSC`), carrier ID (`@RTCAI`).

---

### Function Requirement Document: Shipment Weight Entry and SCO File Generation

#### Function Name
`ProcessShipmentWeight`

#### Purpose
To process shipment weight entry for an order, converting order quantities to pounds, gallons, and shipping weight, retrieving location, tank, and carrier information, and generating SCO files (`BBFPORH`, `BBFPORD`, `BBFPORA`, `BBFPORF`) for internal or external freight processing.

#### Inputs
- **Order Data**:
  - Company number (`CO`, 2 digits).
  - Order number (`BORDNO`, 6 characters).
  - Customer number (`BOCUST`, 6 characters).
  - Ship-to number (`BOSHIP`, 6 characters).
  - Freight processor code (`BOFPCD`, 3 characters).
  - Location code (`BOLOC`, 3 characters).
  - Product code (`BDPROD`, 4 characters).
  - Container code (`CNTR`, 3 alphanumeric characters).
  - Unit of measure (`UM`, 3 characters, e.g., `LBS`, `GAL`, `KG`, `LI`, `ML`, `OZ`).
  - Container quantity (`CTQT`, 7 digits).
  - Tare weight (`TARE`, 7 digits).
  - Gross weight (`GVWT`, 7 digits).
  - Car capacity (`CACA`, 7 digits).
  - Outage (`OUTA`, 7 digits).
  - Carrier code (`CACD`, 2 characters).
  - Fluid code (`FLCD`, 1 character).
  - Temperature (`TEMP`, 3 digits).
  - API gravity (`GRAV`, 3 digits, 1 decimal).
  - VCF code (`VCF`, 2 characters).
- **Database Access**:
  - Access to files: `INLOC`, `BBORDR`, `SHIPTO`, `ARCUST`, `BBORH1`, `BBORD1`, `BBORA1`, `BBSHPH`, `BBSHPD`, `BBFRPR`, `SHPADR`, `BBORF`, `BICONT`, `GSPROD`, `GSCTUM`, `GSUMCV`, `GSCTWT`, `INSLTZ`, `VSCARR`.

#### Outputs
- **Shipment Data**:
  - Location name (`@4LOCN`, 20 characters), phone number (`@4LOCP`, 14 characters), inventory type (`@4LINT`, 1 character).
  - Selling tank number (`STTANK`, 4 characters), rail car tank number (`STRCTK`, 4 characters).
  - Carrier code (`@RTCOD`, 10 characters), description (`@RTDSC`, 33 characters), carrier ID (`@RTCAI`, 6 characters).
  - Shipped quantity (`SQTY`, 7 digits), gross gallons (`GGAL`, 7 digits), net gallons (`NGAL`, 7 digits), net pounds (`NETLBS`, 7 digits), shipping pounds (`SHPLBS`, 7 digits).
  - Populated SCO files (`BBFPORH`, `BBFPORD`, `BBFPORA`, `BBFPORF`) with header, detail, accessorial, and freight data.
- **External Program Execution**:
  - For internal freight processors: Execute `InsertARGLMSOrder.EXE`.
  - For external freight processors: Execute `SCOXML.EXE`.

#### Process Steps
1. **Retrieve Location Information**:
   - Use `CO` and `BOLOC` to chain to `INLOC`.
   - If found and not deleted, retrieve location name (`@4LOCN`), format phone number (`@4LOCP` as `(XXX) XXX-XXXX`), and inventory type (`@4LINT`).
   - If not found, return blanks for `@4LOCN`, `@4LOCP`, `@4LINT`.

2. **Retrieve Selling Tank Information**:
   - Use a 17-character key (`SELKEY`) to chain to `INSLTZ`.
   - If found and not deleted, retrieve `STTANK` and `STRCTK`.
   - If not found or deleted, read the previous non-deleted record with a matching key prefix; return blanks if none found.

3. **Validate Carrier Information**:
   - Use `CACD` to chain to `VSCARR`.
   - If found, retrieve carrier code (`@RTCOD`), description (`@RTDSC`), and carrier ID (`@RTCAI`).
   - If not found, return blanks.

4. **Convert Order Quantities**:
   - For each order line item:
     - Retrieve product details from `GSPROD` (gravity `TPGRAV`, fluid code `TPFLCD`, VCF code `TPVCF`).
     - If `FLCD`, `GRAV`, or `VCF` are blank, use `TPFLCD`, `TPGRAV`, `TPVCF`.
     - Calculate Table 6D factor (`T6DFAC`) using `TEMP`, `GRAV`, `VCF`:
       - If `TEMP = 60` or `VCF = blank`, set `T6DFAC = 1.0`.
       - Otherwise, use constants (`K0`, `K1`, `K2`) based on `VCF` (`CR`, `FO`, `JF`, `TZ`, `GA`, `LU`) and compute `T6DFAC` using the formula:
         - `P3 = (141.5 * 999.016) / (GRAV + 131.5)`.
         - `KX = ((K0 / P3) + K1) / P3 + K2`.
         - `DIF = TEMP - 60.0068749`, `DIFX = DIF + 0.013749795`.
         - `X1 = KX * DIF`, `X3 = X1 * (1 + (0.8 * KX * DIFX))`, `E = -X3`.
         - `T6DFAC = 1 + E + (E²/2) + (E³/6) + ... + (E¹⁰/3628800)`.
     - Convert quantities based on `UM`:
       - For `KG`: `NETLBS = SQTY * 2.20462`, then `NGAL = NETLBS / ((141.5 / (131.5 + GRAV)) * 999.01599999999996) * 119.8`, `GGAL = NGAL / T6DFAC`.
       - For `LI`: `NGAL = SQTY / 3.78541`, `GGAL = NGAL / T6DFAC`, `NETLBS = NGAL * ((141.5 / (131.5 + GRAV)) * 999.01599999999996) / 119.8`.
       - For `ML`: `NGAL = SQTY / 3785.41`, `GGAL = NGAL / T6DFAC`, `NETLBS` as above.
       - For `OZ`: `NGAL = SQTY / 128`, `GGAL = NGAL / T6DFAC`, `NETLBS` as above.
       - For `LBS`: `NETLBS = SQTY`, `NGAL = NETLBS / ((141.5 / (131.5 + GRAV)) * 999.01599999999996) * 119.8`, `GGAL = NGAL / T6DFAC`.
       - For `GAL`: `GGAL = SQTY`, `NGAL = GGAL * T6DFAC`, `NETLBS` as above.
       - For other units: Use `GSCTWT` for `WTGAL` (`GGAL = CTQT * WTGAL`, `NGAL = GGAL * T6DFAC`) or `GSUMCV` for conversion factor (`UCCVFA`, `NGAL = SQTY * UCCVFA` or `SQTY / UCCVFA`, `GGAL = NGAL / T6DFAC`), default to 1 if not found.
     - Calculate shipping weight (`SHPLBS`):
       - Use `GSCTWT` for `WTGROS` (`SHPLBS = CTQT * WTGROS`) if available.
       - Otherwise, set `SHPLBS = NETLBS`.

5. **Populate SCO Files**:
   - Read order header (`BBORDR`), detail (`BBORD1`), shipment header (`BBSHPH`), and detail (`BBSHPD`) data.
   - Determine freight processor type (`BOFPCD`) from `BBFRPR`:
     - Internal: Use order addresses, call `InsertARGLMSOrder.EXE`.
     - External: Use freight processor address from `SHPADR`, call `SCOXML.EXE`.
   - Write to `BBFPORH` (header: `BOCO`, `BORDNO`, `BOCUST`, `BOSHIP`, `BORUSH`, `BOORPR`, addresses).
   - Write to `BBFPORD` (detail: `BDPROD`, `BDQTY`, `BDCTQT`, `BDSQTY`, `SDTARE`, `SDGRVW`, `SDACTW`, `BDPRCE`).
   - Write to `BBFPORA` (accessorial: `BAAAMT`, `BAAACS`, `BATCST`).
   - Write to `BBFPORF` (freight: `BFGGAL`, `BFTWGT`, `BFTQTY`).

6. **Execute External Programs**:
   - For internal freight processors: Execute `\\B-P-APP1\APPLICATIONS\ARGLMS\PROD\InsertARGLMSOrder.EXE`.
   - For external freight processors: Execute `\\BRADFORD15\SCOPRODBIN$\SCOXML.EXE`.

#### Business Rules
1. **Location Retrieval**:
   - Must return valid, non-deleted location data or blanks if not found.
   - Phone number formatted as `(XXX) XXX-XXXX` or blank if invalid.

2. **Tank Retrieval**:
   - Return non-deleted tank numbers; fall back to previous record if exact match is deleted or not found.
   - Return blanks if no valid record exists.

3. **Carrier Validation**:
   - Return validated carrier details; return blanks if invalid or not found.

4. **Quantity Conversion**:
   - Convert quantities using `GSCTWT` or `GSUMCV` when available; default to factor of 1 if not found.
   - Use fixed formulas for `KG` (`2.20462`), `LI` (`3.78541`), `ML` (`3785.41`), `OZ` (`128`).
   - Gross gallons (`GGAL`) derived from net gallons (`NGAL`) using `T6DFAC`.
   - Shipping weight (`SHPLBS`) includes packaging from `GSCTWT` or defaults to `NETLBS`.

5. **Freight Processor Handling**:
   - Internal processors: Use order addresses, call `InsertARGLMSOrder.EXE`.
   - External processors: Use freight processor address, call `SCOXML.EXE`.

6. **SCO File Population**:
   - Ensure all required fields (e.g., `BDSQTY`, `BFGGAL`) are populated to avoid null data.
   - Include accessorial charges (`BATCST`) in `BBFPORA`.
   - Copy rush (`BORUSH`) and order process code (`BOORPR`) to `BBFPORH`.

7. **Error Handling**:
   - Return blanks for missing or invalid data (location, tank, carrier).
   - Set error code (`ERR = 'E'`) for missing conversion factors or product data.

#### Calculations
- **Table 6D Factor (`T6DFAC`)**:
  - If `TEMP = 60` or `VCF = blank`, `T6DFAC = 1.0`.
  - Otherwise: `P3 = (141.5 * 999.016) / (GRAV + 131.5)`, `KX = ((K0 / P3) + K1) / P3 + K2`, `T6DFAC = exp(-KX * (TEMP - 60.0068749) * (1 + 0.8 * KX * (TEMP - 60.0068749 + 0.013749795)))` (10-term Taylor series).
- **Pounds to Gallons**: `Gallons = Pounds / ((141.5 / (131.5 + GRAV)) * 999.01599999999996) * 119.8`.
- **Gallons to Pounds**: `Pounds = Gallons * ((141.5 / (131.5 + GRAV)) * 999.01599999999996) / 119.8`.
- **Shipping Weight**: `SHPLBS = CTQT * WTGROS` (from `GSCTWT`) or `NETLBS`.

---

<xaiArtifact artifact_id="9e4525fc-1d06-448c-bc11-f50b107b25c2" artifact_version_id="02f2c879-2321-4094-80fc-932f0a51c033" title="ShipmentWeightEntryRequirements.md" contentType="text/markdown">

# Function Requirement Document: ProcessShipmentWeight

## Purpose
To process shipment weight entry for an order, converting quantities to pounds, gallons, and shipping weight, retrieving location, tank, and carrier information, and generating SCO files for freight processing.

## Inputs
- **Order Data**:
  - `CO` (2 digits): Company number.
  - `BORDNO` (6 chars): Order number.
  - `BOCUST` (6 chars): Customer number.
  - `BOSHIP` (6 chars): Ship-to number.
  - `BOFPCD` (3 chars): Freight processor code.
  - `BOLOC` (3 chars): Location code.
  - `BDPROD` (4 chars): Product code.
  - `CNTR` (3 alphanumeric): Container code.
  - `UM` (3 chars): Unit of measure (e.g., `LBS`, `GAL`, `KG`, `LI`, `ML`, `OZ`).
  - `CTQT` (7 digits): Container quantity.
  - `TARE` (7 digits): Tare weight.
  - `GVWT` (7 digits): Gross weight.
  - `CACA` (7 digits): Car capacity.
  - `OUTA` (7 digits): Outage.
  - `CACD` (2 chars): Carrier code.
  - `FLCD` (1 char): Fluid code.
  - `TEMP` (3 digits): Temperature.
  - `GRAV` (3.1 digits): API gravity.
  - `VCF` (2 chars): Volume correction factor code.
- **Database Access**: Files `INLOC`, `BBORDR`, `SHIPTO`, `ARCUST`, `BBORH1`, `BBORD1`, `BBORA1`, `BBSHPH`, `BBSHPD`, `BBFRPR`, `SHPADR`, `BBORF`, `BICONT`, `GSPROD`, `GSCTUM`, `GSUMCV`, `GSCTWT`, `INSLTZ`, `VSCARR`.

## Outputs
- **Shipment Data**:
  - `@4LOCN` (20 chars): Location name.
  - `@4LOCP` (14 chars): Formatted phone number.
  - `@4LINT` (1 char): Inventory type.
  - `STTANK` (4 chars): Selling tank number.
  - `STRCTK` (4 chars): Rail car tank number.
  - `@RTCOD` (10 chars): Carrier code.
  - `@RTDSC` (33 chars): Carrier description.
  - `@RTCAI` (6 chars): Carrier ID.
  - `SQTY` (7 digits): Shipped quantity.
  - `GGAL` (7 digits): Gross gallons.
  - `NGAL` (7 digits): Net gallons.
  - `NETLBS` (7 digits): Net pounds.
  - `SHPLBS` (7 digits): Shipping pounds.
  - Populated `BBFPORH`, `BBFPORD`, `BBFPORA`, `BBFPORF` files.
- **External Program Execution**:
  - Internal freight: `InsertARGLMSOrder.EXE`.
  - External freight: `SCOXML.EXE`.

## Process Steps
1. **Retrieve Location**:
   - Chain to `INLOC` with `CO`, `BOLOC`.
   - If found and not deleted, return `@4LOCN`, `@4LOCP` (formatted `(XXX) XXX-XXXX`), `@4LINT`; else return blanks.
2. **Retrieve Tank**:
   - Chain to `INSLTZ` with `SELKEY` (17 chars).
   - If found and not deleted, return `STTANK`, `STRCTK`; else read previous non-deleted record or return blanks.
3. **Validate Carrier**:
   - Chain to `VSCARR` with `CACD`.
   - If found, return `@RTCOD`, `@RTDSC`, `@RTCAI`; else return blanks.
4. **Convert Quantities**:
   - Retrieve `TPGRAV`, `TPFLCD`, `TPVCF` from `GSPROD`.
   - Use `TPFLCD`, `TPGRAV`, `TPVCF` if `FLCD`, `GRAV`, `VCF` are blank.
   - Calculate `T6DFAC`:
     - If `TEMP = 60` or `VCF = blank`, `T6DFAC = 1.0`.
     - Else: `P3 = (141.5 * 999.016) / (GRAV + 131.5)`, `KX = ((K0 / P3) + K1) / P3 + K2`, `T6DFAC = exp(-KX * (TEMP - 60.0068749) * (1 + 0.8 * KX * (TEMP - 60.0068749 + 0.013749795)))`.
   - Convert based on `UM`:
     - `KG`: `NETLBS = SQTY * 2.20462`, `NGAL = NETLBS / ((141.5 / (131.5 + GRAV)) * 999.01599999999996) * 119.8`, `GGAL = NGAL / T6DFAC`.
     - `LI`: `NGAL = SQTY / 3.78541`, `GGAL = NGAL / T6DFAC`, `NETLBS = NGAL * ((141.5 / (131.5 + GRAV)) * 999.01599999999996) / 119.8`.
     - `ML`: `NGAL = SQTY / 3785.41`, `GGAL = NGAL / T6DFAC`, `NETLBS` as above.
     - `OZ`: `NGAL = SQTY / 128`, `GGAL = NGAL / T6DFAC`, `NETLBS` as above.
     - `LBS`: `NETLBS = SQTY`, `NGAL = NETLBS / ((141.5 / (131.5 + GRAV)) * 999.01599999999996) * 119.8`, `GGAL = NGAL / T6DFAC`.
     - `GAL`: `GGAL = SQTY`, `NGAL = GGAL * T6DFAC`, `NETLBS` as above.
     - Other: Use `GSCTWT` (`GGAL = CTQT * WTGAL`, `NGAL = GGAL * T6DFAC`) or `GSUMCV` (`NGAL = SQTY * UCCVFA` or `SQTY / UCCVFA`, `GGAL = NGAL / T6DFAC`), default factor = 1.
   - Calculate `SHPLBS`:
     - If `GSCTWT` exists, `SHPLBS = CTQT * WTGROS`; else `SHPLBS = NETLBS`.
5. **Populate SCO Files**:
   - Read `BBORDR`, `BBORH1`, `BBORD1`, `BBORA1`, `BBSHPH`, `BBSHPD`, `BBORF`.
   - For internal `BOFPCD` (from `BBFRPR`): Use order addresses, call `InsertARGLMSOrder.EXE`.
   - For external `BOFPCD`: Use `SHPADR` address, call `SCOXML.EXE`.
   - Write `BBFPORH` (header data), `BBFPORD` (detail data), `BBFPORA` (accessorial charges), `BBFPORF` (freight data).
6. **Execute External Programs**:
   - Internal: `\\B-P-APP1\APPLICATIONS\ARGLMS\PROD\InsertARGLMSOrder.EXE`.
   - External: `\\BRADFORD15\SCOPRODBIN$\SCOXML.EXE`.

## Business Rules
1. Return valid, non-deleted location data or blanks.
2. Return non-deleted tank numbers or previous record; else blanks.
3. Return validated carrier details or blanks.
4. Convert quantities using `GSCTWT`, `GSUMCV`, or fixed formulas; default factor = 1.
5. Internal processors use order addresses; external use freight processor addresses.
6. Ensure non-null data in SCO files (`BDSQTY`, `BFGGAL`).
7. Include accessorial charges (`BATCST`) and order metadata (`BORUSH`, `BOORPR`).
8. Set `ERR = 'E'` for missing conversion factors or product data.

## Calculations
- **Table 6D Factor**: `T6DFAC = exp(-KX * (TEMP - 60.0068749) * (1 + 0.8 * KX * (TEMP - 60.0068749 + 0.013749795)))`.
- **Pounds to Gallons**: `Gallons = Pounds / ((141.5 / (131.5 + GRAV)) * 999.01599999999996) * 119.8`.
- **Gallons to Pounds**: `Pounds = Gallons * ((141.5 / (131.5 + GRAV)) * 999.01599999999996) / 119.8`.
- **Shipping Weight**: `SHPLBS = CTQT * WTGROS` or `NETLBS`.

</xaiArtifact>