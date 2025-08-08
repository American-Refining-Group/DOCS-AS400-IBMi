The `MBBQTY.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment, though it is primarily designed as a module called by `BB113` or other programs needing to convert order quantities to pounds (LBS), gallons (GAL), and shipping weight. Written prior to 06/07/10 (based on revision JB01), it includes multiple revisions to enhance conversion logic, handle shipping weight calculations, and support additional units of measure like kilograms (KG). Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `MBBQTY` program converts an order quantity from a specified unit of measure (UM) to pounds (LBS), gallons (GAL), and calculates shipping weight, using data from the `gsctum`, `gsumcv`, and `gsctwt` files. The steps are as follows:

1. **Program Initialization**:
   - **File Definitions** (assumed, as not explicitly shown in the provided snippet):
     - `gsctum`: Input-only file for customer unit of measure data (keyed).
     - `gsumcv`: Input-only file for unit measure conversion data (keyed).
     - `gsctwt`: Input-only file for container weight data (keyed).
   - **Parameters** (received via `*ENTRY PLIST`, assumed):
     - `CO` (2 characters): Company number.
     - `PROD` (4 characters): Product code.
     - `CNTR` (3 alphanumeric characters, JB09): Container code.
     - `UM` (unit of measure, e.g., 'LBS', 'GAL', 'KG', 'ECH').
     - `SQTY` (quantity in input unit of measure).
     - `GRAV` (specific gravity, used for conversions).
     - Output parameters: `NETLBS` (net pounds, 7 digits), `NGAL` (net gallons, 7 digits), `SHPLBS` (shipping weight in pounds, 7 digits).
   - **Work Fields**:
     - `HLD6` (6 characters): Temporary field for key construction.
     - `WTKEY` (9 characters): Key for `gsctwt` (company, product, container).
     - `KYGRAV` (specific gravity for `MINLBGL1` call).
     - `KYFACT` (conversion factor, initialized to zero).
     - `P@LBS`, `P@GAL` (parameters for `MINLBGL1` call).
   - **Indicators**:
     - 88: Used for `gsctwt` chain and delete/inactive status checks (`WTDEL='D'` or `'I'`).
     - Other indicators (e.g., 77 for `gsctum` chain) assumed from standard RPGLE usage.

2. **Quantity Conversion to Pounds and Gallons**:
   - **Step 1: Check Unit of Measure (`UM`)**:
     - **If `UM='LBS'`**:
       - Sets `NETLBS = SQTY` (input quantity is already in pounds).
       - Calls `MINLBGL1` to convert pounds to gallons:
         - Parameters: `KYGRAV` (specific gravity), `P@LBS` (input pounds), `P@GAL` (output gallons).
         - Sets `NGAL = P@GAL`.
       - Proceeds to shipping weight calculation (`ENDQTY` tag).
     - **If `UM='GAL'`**:
       - Sets `NGAL = SQTY` (input quantity is already in gallons).
       - Calls `MINLBGL1` to convert gallons to pounds:
         - Parameters: `KYGRAV`, `P@LBS` (output pounds), `P@GAL` (input gallons).
         - Sets `NETLBS = P@LBS`.
       - Proceeds to shipping weight calculation.
     - **If `UM='KG'` (JB10)**:
       - Converts kilograms to pounds: `NETLBS = SQTY * 2.20462`.
       - Calls `MINLBGL1` to convert pounds to gallons:
         - Parameters: `KYGRAV`, `P@LBS` (input pounds), `P@GAL` (output gallons).
         - Sets `NGAL = P@GAL`.
       - Proceeds to shipping weight calculation.
     - **If `UM='ECH'` (each, assumed for containers)**:
       - Chains to `gsctum` using key (company `CO`, product `PROD`, container `CNTR`, unit of measure `UM`).
       - If record found and not deleted (`N77`, `CTDEL<>'D'`):
         - Multiplies `SQTY` by `CTGAL` (gallons per unit) to get `NGAL`.
         - Calls `MINLBGL1` to convert gallons to pounds:
           - Parameters: `KYGRAV`, `P@LBS` (output pounds), `P@GAL` (input gallons).
           - Sets `NETLBS = P@LBS`.
       - If no record or deleted (`77` or `CTDEL='D'`):
         - Chains to `gsumcv` using key (company `CO`, unit of measure `UM`).
         - If record found and not deleted (`N77`, `CVDEL<>'D'`):
           - Multiplies `SQTY` by `CVFACT` (conversion factor) to get `NGAL`.
           - Calls `MINLBGL1` to convert gallons to pounds.
         - If no record or deleted, uses `SQTY * 1` for `NGAL` (JB01).
           - Calls `MINLBGL1` to convert gallons to pounds.
       - Proceeds to shipping weight calculation.
     - **Other Units of Measure**:
       - Chains to `gsumcv` to get conversion factor (`CVFACT`).
       - If found and not deleted, multiplies `SQTY` by `CVFACT` to get `NGAL` (JB07 for non-LBS/GAL/ECH units).
       - If not found, uses `SQTY * 1` for `NGAL` (JB11).
       - Calls `MINLBGL1` to convert gallons to pounds.
       - Proceeds to shipping weight calculation.

3. **Shipping Weight Calculation (JB02, JB05)**:
   - Constructs key `WTKEY` (company `CO`, product `PROD`, container `CNTR`) for `gsctwt`.
   - Chains to `gsctwt` using `WTKEY` (indicator 88).
   - If record found and not deleted or inactive (`N88`, `WTDEL<>'D'` or `'I'`):
     - Multiplies container quantity (`CTQT`) by gross weight (`WTGROS`) to get `SHPLBS` (shipping weight, including packaging).
     - Proceeds to program end (`ENDGWT` tag).
   - If no record or deleted/inactive:
     - If `UM='LBS'`, sets `SHPLBS = NETLBS`.
     - If `UM='GAL'`, sets `SHPLBS = NETLBS` (calculated via `MINLBGL1`).
     - For other units, sets `SHPLBS = NETLBS` (JB05).
     - Proceeds to program end.

4. **Program Termination**:
   - Jumps to `ENDQTY` tag, then `ENDGWT` tag for shipping weight.
   - Returns output parameters: `NETLBS` (net pounds), `NGAL` (net gallons), `SHPLBS` (shipping weight).
   - Sets last record indicator (`LR`, assumed) to exit.

---

### Business Rules

The program enforces the following business rules:

1. **Quantity Conversion**:
   - Converts order quantity (`SQTY`) from input unit of measure (`UM`) to net pounds (`NETLBS`) and net gallons (`NGAL`).
   - For `UM='LBS'`, `NETLBS = SQTY`, then converts to gallons using `MINLBGL1`.
   - For `UM='GAL'`, `NGAL = SQTY`, then converts to pounds using `MINLBGL1`.
   - For `UM='KG'`, converts kilograms to pounds (`SQTY * 2.20462`) and then to gallons (JB10).
   - For `UM='ECH'`, uses `gsctum` (gallons per unit) or `gsumcv` (conversion factor) to get `NGAL`, then converts to pounds.
   - For other units, uses `gsumcv` conversion factor or defaults to `SQTY * 1` (JB01, JB11).
   - For non-LBS/GAL/ECH units, converts `NGAL` to `SQTY` using gallon-based factor (`CVFACT`) instead of gross gallons (JB07).

2. **Shipping Weight Calculation**:
   - Uses `gsctwt` to calculate gross shipping weight (`SHPLBS`) including packaging if record exists and is not deleted/inactive (JB02).
   - If no valid `gsctwt` record, uses `NETLBS` as `SHPLBS` for all units of measure (JB05).
   - Gross weight calculations prioritize pounds-to-unit conversion over gallons-to-unit if possible (JB06).

3. **GVW-Tare Calculations**:
   - Calculates gross gallons from pounds, then derives net gallons (JB03).
   - For GVW-Tare, prioritizes pounds-to-unit conversion; if not possible, uses gallons-to-unit (JB06).

4. **Data Validation**:
   - Validates container (`CNTR`) as alphanumeric (JB09).
   - Checks for deleted records in `gsctum` (`CTDEL='D'`) and `gsumcv` (`CVDEL='D'`).
   - Checks for deleted/inactive records in `gsctwt` (`WTDEL='D'` or `'I'`).

5. **Error Handling**:
   - If no conversion factor is found in `gsumcv`, uses a factor of 1 (JB01, JB11) to avoid errors.
   - Relies on `MINLBGL1` for accurate pounds-to-gallons conversion, reducing user errors (JB04, JB10).

---

### Tables (Files) Used

The program interacts with the following files:

1. **gsctum**: Input-only customer unit of measure file (keyed).
   - Fields: `CO` (company), `PROD` (product), `CNTR` (container), `UM` (unit of measure), `CTGAL` (gallons per unit), `CTDEL` (delete flag).
2. **gsumcv**: Input-only unit measure conversion file (keyed).
   - Fields: `CO` (company), `UM` (unit of measure), `CVFACT` (conversion factor), `CVDEL` (delete flag).
3. **gsctwt**: Input-only container weight file (keyed).
   - Fields: `WTKEY` (company, product, container), `CTQT` (container quantity), `WTGROS` (gross weight), `WTDEL` (delete/inactive flag).

---

### External Programs Called

The program calls the following external program:

1. **MINLBGL1**:
   - Called to convert between pounds and gallons (JB04).
   - Parameters:
     - `KYGRAV` (specific gravity): Input for conversion formula.
     - `P@LBS` (pounds): Input or output, depending on conversion direction.
     - `P@GAL` (gallons): Input or output, depending on conversion direction.

---

### Summary

The `MBBQTY` RPGLE program, called as a module by `BB113` or other programs (including potentially `BB101.ocl36.txt`), converts order quantities from various units of measure (`LBS`, `GAL`, `KG`, `ECH`, etc.) to net pounds (`NETLBS`), net gallons (`NGAL`), and shipping weight (`SHPLBS`). It uses `gsctum`, `gsumcv`, and `gsctwt` for conversion factors and weights, and calls `MINLBGL1` for pounds-to-gallons conversions. Business rules ensure accurate conversions, prioritize pounds-based calculations, and handle missing data with default factors. The program interacts with three files and relies on `MINLBGL1` for precise calculations, supporting order processing in the Customer Orders system.