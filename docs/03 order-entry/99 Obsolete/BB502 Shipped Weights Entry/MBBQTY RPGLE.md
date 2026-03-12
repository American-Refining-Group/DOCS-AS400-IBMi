The `MBBQTY.rpgle.txt` is an RPG program (converted to RPGLE, as noted in JB15) called from the `BB502.ocl36.txt` OCL program and referenced in `BB502.rpg36.txt` as a replacement for `BB5009` (JB12). It is a utility module designed to convert order quantities to pounds, gallons, and shipping weight for use in shipment processing. Below, I’ll explain the process steps, business rules, tables used, and external programs called based on the provided RPG source code.

---

### Process Steps of the RPG Program

The `MBBQTY` program converts an order’s container quantity (`CTQT`) into pounds (`NETLBS`), gross gallons (`GGAL`), net gallons (`NGAL`), and shipping weight (`SHPLBS`) based on the unit of measure (`UM`), product (`PROD`), container (`CNTR`), and other parameters. It uses conversion factors from files and formulas to ensure accurate calculations. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - The program defines input files (`BICONT`, `GSPROD`, `GSCTUM`, `GSUMCV`, `GSCTWT`) for retrieving product, container, and conversion data.
   - A parameter data structure (`QTPARM`) is defined to handle input and output:
     - **Input Fields**: Company (`CO`), product (`PROD`), container (`CNTR`, alphanumeric per JB09), unit of measure (`UM`), container quantity (`CTQT`, sometimes output), tare weight (`TARE`), gross weight (`GVWT`), car capacity (`CACA`), outage (`OUTA`), carrier code (`CACD`), fluid code (`FLCD`), temperature (`TEMP`), gravity (`GRAV`), volume correction factor (`VCF`).
     - **Output Fields**: Shipped quantity (`SQTY`), gross gallons (`GGAL`), net gallons (`NGAL`), temperature correction factor (`T6DFAC`), net pounds (`NETLBS`), shipping pounds (`SHPLBS`), error code (`ERR`).
   - Additional data structures (`KYGRAV`, `KYFACT`, `P@LBS`, `P@GAL`) are used for conversion calculations, and `T6DTEM`, `T6DGRA`, `T6DVCF` store temperature and gravity data.

2. **Retrieve Company Data (`BICONT`)**:
   - The program chains to the `BICONT` file using the company number (`CO`) to verify the company record.
   - If the record is not found or marked as deleted (`BCDEL = 'D'`), it sets the outage percentage (`BCIOPE`) to zero and sets indicator `49`.

3. **Retrieve Product Data (`GSPROD`)**:
   - It constructs a key (`HLD6`) by combining `CO` and `PROD` and chains to the `GSPROD` file to retrieve product details (e.g., standard API gravity `TPGRAV`, fluid code `TPFLCD`, VCF code `TPVCF`).
   - If the record is not found or deleted (`TPDEL = 'D'`), it sets `TPFLCD`, `TPGRAV`, and `TPVCF` to blank/zero and sets an error code (`ERR = 'E'`, JB13).

4. **Populate Missing Input Fields**:
   - If input fields `FLCD`, `GRAV`, or `VCF` are blank/zero, they are populated from the `GSPROD` record (`TPFLCD`, `TPGRAV`, `TPVCF`).

5. **Unit of Measure Conversion**:
   - The program handles conversions based on the unit of measure (`UM`):
     - **Kilograms (KG, JB10)**:
       - Converts `SQTY` (in kilograms) to pounds (`NETLBS`) using the formula: `SQTY * 2.20462`.
       - Calls `MINLBGL1` to convert pounds to net gallons (`NGAL`) using `GRAV`.
       - Calculates gross gallons (`GGAL`) by dividing `NGAL` by the temperature correction factor (`T6DFAC`).
     - **Liters (LI, JB11)**:
       - Converts `SQTY` to net gallons: `SQTY / 3.78541`.
       - Calls `MINLBGL1` to convert net gallons to pounds (`NETLBS`).
       - Calculates gross gallons: `NGAL / T6DFAC`.
     - **Milliliters (ML, JB11)**:
       - Converts `SQTY` to net gallons: `SQTY / 3785.41`.
       - Calls `MINLBGL1` to convert net gallons to pounds.
       - Calculates gross gallons: `NGAL / T6DFAC`.
     - **Ounces (OZ, JB11)**:
       - Converts `SQTY` to net gallons: `SQTY / 128`.
       - Calls `MINLBGL1` to convert net gallons to pounds.
       - Calculates gross gallons: `NGAL / T6DFAC`.
     - **Pounds (LBS)**:
       - Sets `NETLBS` to `SQTY`.
       - Calls `MINLBGL1` to convert pounds to net gallons (`NGAL`).
       - Calculates gross gallons: `NGAL / T6DFAC`.
     - **Gallons (GAL)**:
       - Sets `GGAL` to `SQTY`.
       - Calculates net gallons: `GGAL * T6DFAC`.
       - If the GL code (`GLCD`) is `'G'`, sets `SQTY` to `GGAL`; otherwise, sets `SQTY` to `NGAL`.
       - Calls `MINLBGL1` to convert net gallons to pounds (`NETLBS`).
     - **Other Units (e.g., non-LBS, GAL, ECH, JB07, JB14)**:
       - Chains to `GSCTWT` using a key (`WTKEY`) combining `CO`, `PROD`, and `CNTR` to get gallons per container (`WTGAL`, expanded to 12.7 digits in JB16).
       - If found and not deleted (`WTDEL ≠ 'D' or 'I'`), calculates gross gallons: `CTQT * WTGAL` and net gallons: `GGAL * T6DFAC`.
       - If no `GSCTWT` record, chains to `GSUMCV` using a key (`UCKEY`) combining `CO`, `PROD`, `UM`, and `'GAL'` to get a conversion factor (`UCCVFA`).
       - If no `GSUMCV` record, sets `UCOPER` to `'M'` and `UCCVFA` to 1 (JB01), and sets `ERR = 'E'` (JB13).
       - Applies conversion: `SQTY * UCCVFA` (if `UCOPER = 'M'`) or `SQTY / UCCVFA` (if `UCOPER = 'D'`) to get `NGAL`.
       - Calculates gross gallons: `NGAL / T6DFAC`.
       - Calls `MINLBGL1` to convert net gallons to pounds (`NETLBS`).

6. **Shipping Weight Calculation (JB02, JB05)**:
   - Chains to `GSCTWT` to retrieve gross weight (`WTGROS`, including packaging).
   - If found and not deleted, calculates shipping weight: `CTQT * WTGROS`.
   - If no `GSCTWT` record:
     - For `UM = 'LBS'`, sets `SHPLBS` to `NETLBS`.
     - For `UM = 'GAL'` or other units, sets `SHPLBS` to `NETLBS`.

7. **Program Termination**:
   - The program jumps to the `ENDQTY` tag and ends, returning the updated `QTPARM` data structure with `SQTY`, `GGAL`, `NGAL`, `NETLBS`, `SHPLBS`, and `ERR` to the calling program.

---

### Business Rules

The program enforces the following business rules, based on the code and change history:

1. **Unit of Measure Conversions**:
   - Supports conversions for `KG`, `LI`, `ML`, `OZ`, `LBS`, `GAL`, and other units (`ECH`, etc.) using specific formulas or conversion factors from `GSUMCV` and `GSCTWT` (JB10, JB11).
   - For `KG`, converts to pounds using a fixed factor (`2.20462`) before converting to gallons (JB10).
   - For `LI`, `ML`, `OZ`, uses fixed factors (`3.78541`, `3785.41`, `128`) to convert to gallons, avoiding user-entered factors (JB11).
   - For other units, prefers `GSCTWT` for gallons per container (`WTGAL`) if available; otherwise, uses `GSUMCV` or defaults to a factor of 1 (JB01, JB14).

2. **Gross vs. Net Gallons (JB03, JB06)**:
   - Gross gallons (`GGAL`) are calculated from net gallons (`NGAL`) by dividing by the temperature correction factor (`T6DFAC`).
   - For `LBS`, converts to net gallons first, then to gross gallons.
   - For non-`LBS`/`GAL` units, prefers converting to gallons before pounds if possible; otherwise, converts to pounds first (JB06).

3. **Shipping Weight Calculation (JB02, JB05)**:
   - Uses `GSCTWT` for shipping weight (`SHPLBS`) if available, as it includes packaging weight.
   - Falls back to `NETLBS` for `LBS`, `GAL`, or other units if no `GSCTWT` record exists.

4. **Error Handling (JB13)**:
   - Sets an error code (`ERR = 'E'`) if no valid record is found in `GSPROD` or `GSUMCV`, indicating a conversion failure.

5. **Rounding (JB12)**:
   - Applies rounding to certain unit of measure conversions to ensure accuracy.

6. **Container Code (JB09)**:
   - The container code (`CNTR`) is alphanumeric, allowing flexibility in container identification.

7. **Gallons Precision (JB16)**:
   - The `WTGAL` field in `GSCTWT` was expanded from 8.2 to 12.7 digits for higher precision in gallon calculations.

8. **Fallback for Missing Data**:
   - If `FLCD`, `GRAV`, or `VCF` are missing, they are populated from `GSPROD` to ensure valid conversions.
   - If no conversion factor is found in `GSUMCV`, a default factor of 1 is used (JB01).

---

### Tables (Files) Used

The program uses the following input files, as defined in the `F` (File) specifications:

1. **BICONT**: Container master file (256 bytes, indexed, read-only, key length 2). Contains outage percentage (`BCIOPE`).
2. **GSPROD**: Product master file (512 bytes, indexed, read-only, key length 8). Contains product details (`TPGRAV`, `TPFLCD`, `TPVCF`).
3. **GSCTUM**: Unit of measure file (64 bytes, indexed, read-only, key length 12). Contains issue unit of measure (`CUIUM`, `CUHAZM`).
4. **GSUMCV**: Unit conversion file (64 bytes, indexed, read-only, key length 12). Contains conversion factors (`UCCVFA`, `UCOPER`).
5. **GSCTWT**: Container weight file (64 bytes, indexed, read-only, key length 9). Contains gross weight (`WTGROS`) and gallons per container (`WTGAL`).

---

### External Programs Called

The program calls one external program:

1. **MINLBGL1** (JB04, JB05, JB11):
   - A utility program that converts between pounds and gallons based on gravity (`KYGRAV`) and other parameters.
   - Input parameters: `KYGRAV` (gravity), `P@LBS` (pounds), `P@GAL` (gallons).
   - Output: Updated `P@LBS` or `P@GAL` based on the conversion direction.
   - Replaces `MINLBGL` to directly calculate gallons or pounds instead of returning a factor (JB04).

---

### Summary

The `MBBQTY` RPGLE program is a utility called from `BB502` (and potentially other programs like `BB113`) to convert order quantities (`CTQT`) to pounds (`NETLBS`), gross gallons (`GGAL`), net gallons (`NGAL`), and shipping weight (`SHPLBS`). It uses product, container, and conversion data from five files (`BICONT`, `GSPROD`, `GSCTUM`, `GSUMCV`, `GSCTWT`) and applies specific formulas for units like `KG`, `LI`, `ML`, `OZ`, `LBS`, and `GAL`. It enforces business rules for accurate conversions, error handling, and fallback logic, calling `MINLBGL1` for pound-to-gallon conversions. The program ensures precision (e.g., JB16) and avoids user errors in conversion factors (JB10, JB11).

If you need further details on specific calculations, validations, or related content (e.g., analysis of X posts or web searches), let me know!