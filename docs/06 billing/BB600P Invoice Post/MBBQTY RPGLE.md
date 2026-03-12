The `MBBQTY.rpgle.txt` RPGLE program is a module, not a standalone program, called by other programs (e.g., `BB113`) within the invoice posting workflow on an IBM System/36 or AS/400 environment. Its primary function is to convert order quantities to pounds (LBS) and gallons (GAL), calculate net and gross gallons, and determine shipping weight for invoice processing. The program includes revisions to improve conversion accuracy, handle specific units of measure (e.g., kilograms, liters, milliliters, ounces), and support viscosity barcode calculations. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the MBBQTY RPGLE Program

The `MBBQTY` program takes input parameters (e.g., company, product, container, order quantity, unit of measure) and converts the order quantity to pounds (`NETLBS`), net gallons (`NGAL`), gross gallons (`GGAL`), and shipping weight (`SHPLBS`). It uses data from the `GSCTUM` and `GSCTWT` files for conversion factors and weights, and calls the `MINLBGL1` program for specific conversions.

#### Process Steps:
1. **Program Initialization**:
   - The program receives input parameters:
     - `CO`: Company number.
     - `PROD`: Product code.
     - `CNTR`: Container code (alpha/numeric, per `JB09`).
     - `CTQT`: Order quantity.
     - `UM`: Unit of measure (e.g., `LBS`, `GAL`, `KG`, `LI`, `ML`, `OZ`).
     - `GRAV`: Specific gravity for conversions.
   - Output parameters:
     - `NETLBS`: Net weight in pounds (7 digits, 0 decimals).
     - `NGAL`: Net gallons (7 digits, 2 decimals).
     - `GGAL`: Gross gallons (7 digits, 2 decimals).
     - `SHPLBS`: Shipping weight in pounds (7 digits, 0 decimals).
   - The program opens two input files:
     - `GSCTUM`: Unit of measure conversion file (256 bytes per record, input file `IF`).
     - `GSCTWT`: Container weight file (256 bytes per record, input file `IF`).

2. **Unit of Measure Conversion (GSCTUM)**:
   - The program constructs a key (`UMKEY`) using `CO`, `PROD`, and `CNTR` to chain to `GSCTUM`.
   - If a matching record is found and not deleted (`UMDEL ≠ 'D'`), it retrieves:
     - `UMFACT`: Conversion factor to convert from `UM` to gallons.
     - `T6DFAC`: Conversion factor for gross gallons (if applicable).
   - If no record is found or the record is deleted, `UMFACT` is set to 1 (per `JB01`), assuming no conversion is needed.
   - The program handles specific units of measure:
     - **LBS**: Sets `NETLBS = CTQT`, no further conversion needed.
     - **GAL**: Sets `NGAL = CTQT`, calculates `NETLBS` using `MINLBGL1`.
     - **KG** (per `JB10`): Converts kilograms to pounds (`KG * 2.20462 = NETLBS`), then to gallons using `MINLBGL1`.
     - **LI, ML, OZ** (per `JB11`): Does not require `GSCTUM` record; uses predefined formulas (not detailed in the snippet) to convert to gallons and pounds.
     - Other units: Uses `UMFACT` to convert `CTQT` to `NGAL` (`NGAL = CTQT * UMFACT`).

3. **Gross Gallons Calculation (per JB03)**:
   - For gross vehicle weight minus tare (GVW-TARE) calculations:
     - Converts `CTQT` to `NETLBS` first, then to `GGAL` (gross gallons) using `T6DFAC` or `MINLBGL1`.
     - Calculates `NGAL` (net gallons) from `NETLBS` to ensure accuracy (avoids direct conversion to net gallons).
   - For non-`LBS`, `GAL`, `ECH` units, converts `NGAL` to `CTQT` using the gallon-to-unit factor (per `JB07`).

4. **Net Gallons to Pounds Conversion**:
   - For units requiring conversion to pounds (e.g., `GAL`, `KG`, `LI`, `ML`, `OZ`):
     - Initializes `KYGRAV` (specific gravity), `KYFACT` (factor, set to 0), `P@LBS` (pounds, set to 0), and `P@GAL` (gallons, set to `NGAL`).
     - Calls `MINLBGL1` with parameters `KYGRAV`, `P@LBS`, and `P@GAL`.
     - `MINLBGL1` calculates `P@LBS` based on the formula (likely `P@LBS = P@GAL * KYGRAV * conversion factor`).
     - Sets `NETLBS = P@LBS`.

5. **Shipping Weight Calculation (per JB02, JB05)**:
   - The program calculates the shipping weight (`SHPLBS`) including packaging:
     - Constructs a key (`WTKEY`) using `CO`, `PROD`, and `CNTR` to chain to `GSCTWT`.
     - If a matching record is found, not deleted (`WTDEL ≠ 'D'` or `'I'`, per `JB14`), multiplies container quantity (`CTQT`) by gross weight (`WTGROS`) to set `SHPLBS`.
     - If no `GSCTWT` record exists or is deleted/invalid, uses `NETLBS` as `SHPLBS` for:
       - `UM = 'LBS'`: `SHPLBS = NETLBS`.
       - `UM = 'GAL'`: `SHPLBS = NETLBS` (per `JB05`).
       - Other units: `SHPLBS = NETLBS` (default).

6. **Program Termination**:
   - The program jumps to the `ENDQTY` tag after completing conversions.
   - Returns `NETLBS`, `NGAL`, `GGAL`, and `SHPLBS` to the calling program.
   - Closes all files and ends the subroutine (`ENDSR`).

---

### Business Rules

1. **Unit of Measure Conversion**:
   - Converts order quantity (`CTQT`) to pounds (`NETLBS`) and gallons (`NGAL`, `GGAL`) based on the unit of measure (`UM`).
   - Uses `GSCTUM` for conversion factors (`UMFACT`, `T6DFAC`); defaults to 1 if no record is found (per `JB01`).
   - Supports specific units:
     - `LBS`: `NETLBS = CTQT`, no further conversion.
     - `GAL`: `NGAL = CTQT`, converts to `NETLBS` via `MINLBGL1`.
     - `KG`: Converts to pounds (`NETLBS = CTQT * 2.20462`), then to gallons via `MINLBGL1` (per `JB10`).
     - `LI`, `ML`, `OZ`: Uses predefined formulas, does not require `GSCTUM` record (per `JB11`).

2. **Gross vs. Net Gallons (per JB03, JB06)**:
   - For GVW-TARE calculations:
     - Converts to `NETLBS` first, then to `GGAL` (gross gallons).
     - Derives `NGAL` (net gallons) from `NETLBS` to ensure accuracy.
     - If possible, converts from pounds to unit of measure (`UM`); if not, converts from gallons to `UM` (per `JB06`).
   - For non-`LBS`, `GAL`, `ECH` units, converts `NGAL` to `CTQT` using gallon-to-unit factor (per `JB07`).

3. **Shipping Weight Calculation (per JB02, JB05)**:
   - Uses `GSCTWT` for gross weight (`WTGROS`) including packaging if available and valid (`WTDEL ≠ 'D'`, `'I'`).
   - Defaults to `NETLBS` for `SHPLBS` if no valid `GSCTWT` record or for `LBS`, `GAL`, or other units.

4. **Kilogram Conversion (per JB10)**:
   - Converts kilograms (`KG`) to pounds using a fixed factor (`KG * 2.20462 = NETLBS`).
   - Uses `MINLBGL1` to convert pounds to gallons, reducing user errors from manual conversion factors.

5. **No GSCTUM Requirement for Certain Units (per JB11)**:
   - Units `LI` (liters), `ML` (milliliters), and `OZ` (ounces) do not require a `GSCTUM` record; uses predefined formulas for conversion.

6. **Error Handling**:
   - If no `GSCTUM` record is found, defaults to a conversion factor of 1 (per `JB01`).
   - If no valid `GSCTWT` record is found, uses `NETLBS` as `SHPLBS`.

7. **Integration with Calling Programs**:
   - Designed to be called by programs like `BB113` for order quantity conversions during invoice processing.
   - Returns converted values (`NETLBS`, `NGAL`, `GGAL`, `SHPLBS`) to the caller.

---

### Tables (Files) Used

1. **GSCTUM**:
   - **Description**: Unit of measure conversion file.
   - **Attributes**: 256 bytes per record, input file (`IF`), disk-based.
   - **Fields Used**:
     - `UMFACT`: Conversion factor to gallons.
     - `T6DFAC`: Conversion factor for gross gallons.
     - `UMDEL`: Delete flag (`'D'` for deleted records).
   - **Purpose**: Provides conversion factors for units of measure.
   - **Usage**: Chained using `UMKEY` (company, product, container) to retrieve conversion factors.

2. **GSCTWT**:
   - **Description**: Container weight file.
   - **Attributes**: 256 bytes per record, input file (`IF`), disk-based.
   - **Fields Used**:
     - `WTGROS`: Gross weight including packaging.
     - `WTDEL`: Delete flag (`'D'` or `'I'` for invalid/deleted records).
   - **Purpose**: Provides gross weight for shipping calculations.
   - **Usage**: Chained using `WTKEY` (company, product, container) to calculate `SHPLBS`.

---

### External Programs Called

1. **MINLBGL1**:
   - **Description**: Program to convert between pounds and gallons.
   - **Parameters**:
     - `KYGRAV`: Specific gravity.
     - `P@LBS`: Pounds (output).
     - `P@GAL`: Gallons (input/output).
   - **Purpose**: Calculates pounds from gallons (or vice versa) using specific gravity and conversion factors.
   - **Usage**: Called to convert `NGAL` to `NETLBS` for units like `GAL`, `KG`, `LI`, `ML`, `OZ`.

---

### Summary

The `MBBQTY` RPGLE program, called by programs like `BB113`, converts order quantities to pounds (`NETLBS`), net gallons (`NGAL`), gross gallons (`GGAL`), and shipping weight (`SHPLBS`) by:
- Reading conversion factors from `GSCTUM` and weights from `GSCTWT`.
- Converting quantities based on unit of measure (`UM`), with special handling for `LBS`, `GAL`, `KG`, `LI`, `ML`, `OZ`.
- Calculating gross gallons from pounds (per `JB03`, `JB06`) and shipping weight from `GSCTWT` or `NETLBS` (per `JB02`, `JB05`).
- Using `MINLBGL1` for pounds-to-gallons conversions (per `JB04`).
- Supporting kilograms with a fixed conversion factor (per `JB10`) and predefined formulas for `LI`, `ML`, `OZ` (per `JB11`).
- Returning converted values to the calling program.

**Tables Used**: `GSCTUM` (unit of measure conversions), `GSCTWT` (container weights).
**External Programs Called**: `MINLBGL1` (pounds-to-gallons conversion).

This program ensures accurate quantity conversions for invoice processing, supporting the broader invoice posting workflow with robust handling of various units of measure and weight calculations.