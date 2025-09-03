The call stack consists of three components: the OCL procedure (`IN873.ocl36.txt`), the main RPG program (`IN873.rpg36.txt`), and the called RPGLE module (`MINLBGL1.rpgle.txt`). These components work together in an IBM System/36 or AS/400 environment to convert between pounds and net gallons for inventory or production purposes. Below, I will first identify the use cases implemented by this call stack and then provide a function requirement document for a non-interactive version of the primary use case, focusing on business requirements and calculations.

### List of Use Cases

The call stack implements the following use cases, derived from the functionality of the `IN873` program and its interaction with `MINLBGL1`:

1. **Convert Pounds to Net Gallons Interactively**
   - **Description**: Allows a user to input a company number, product code, specific gravity (optional), and a quantity in pounds via a workstation interface. The program validates the inputs, retrieves product data if needed, and converts the pounds to net gallons using an API-standard formula.
   - **Components Involved**:
     - `IN873.ocl36.txt`: Loads the `IN873` program and allocates `INCONT` and `GSPROD` files.
     - `IN873.rpg36.txt`: Provides the interactive interface (screens `IN873S1` and `IN873S2`), validates inputs, retrieves specific gravity from `GSPROD` if not provided, and calls `MINLBGL1` for the conversion.
     - `MINLBGL1.rpgle.txt`: Performs the pounds-to-gallons conversion using the formula: `(Pounds / ((141.5 / (131.5 + API Gravity)) * 999.01599999999996)) * 119.8`.
   - **Key Features**:
     - Validates company number against `INCONT`.
     - Validates product code and retrieves specific gravity from `GSPROD` if not provided.
     - Ensures only pounds are entered (not both pounds and gallons).
     - Displays the product description and conversion result (gallons and pounds) on the screen.

2. **Convert Net Gallons to Pounds Interactively**
   - **Description**: Allows a user to input a company number, product code, specific gravity (optional), and a quantity in gallons via a workstation interface. The program validates the inputs, retrieves product data if needed, and converts the gallons to pounds using an API-standard formula.
   - **Components Involved**:
     - Same as above (`IN873.ocl36.txt`, `IN873.rpg36.txt`, `MINLBGL1.rpgle.txt`).
     - `MINLBGL1.rpgle.txt`: Performs the gallons-to-pounds conversion using the formula: `(Gallons * ((141.5 / (131.5 + API Gravity)) * 999.01599999999996)) / 119.8`.
   - **Key Features**:
     - Same validation and data retrieval as the pounds-to-gallons use case.
     - Ensures only gallons are entered (not both pounds and gallons).
     - Displays the product description and conversion result (pounds and gallons) on the screen.

3. **Validate Company and Product Data Interactively**
   - **Description**: Validates the company number and product code entered by the user against the `INCONT` and `GSPROD` files, respectively, and displays error messages if invalid. This is a supporting use case for the conversion process, ensuring data integrity before performing calculations.
   - **Components Involved**:
     - `IN873.ocl36.txt`: Allocates `INCONT` and `GSPROD` files.
     - `IN873.rpg36.txt`: Performs validation using `CHAIN` operations on `INCONT` (for company number) and `GSPROD` (for product code).
   - **Key Features**:
     - Displays "INVALID COMPANY NO" if the company number is not found in `INCONT`.
     - Displays "INVALID PRODUCT CODE" if the product code is not found in `GSPROD`.
     - Retrieves product description and specific gravity from `GSPROD` for valid product codes.

### Function Requirement Document

The function requirement document assumes a non-interactive function that encapsulates the primary use case (bidirectional conversion between pounds and net gallons) without screen interaction. It takes all necessary inputs as parameters, performs validations and calculations, and returns the results. The document focuses on business requirements and concisely explains the process steps and calculations.

<xaiArtifact artifact_id="12ec42c1-c2d0-4611-a1ec-b7eaf8773907" artifact_version_id="3d26fb30-8392-4b7c-9ec0-4649edb74137" title="ConvertPoundsGallonsFunction.md" contentType="text/markdown">

# Function Requirement: Convert Pounds to Gallons or Gallons to Pounds

## Purpose
Provide a non-interactive function to convert between pounds and net gallons for a specified company and product, using API gravity, with validation against inventory and product data.

## Inputs
- **Company Number** (`KCO`, 2-digit numeric): Identifies the company, validated against the `INCONT` file.
- **Product Code** (`KPROD`, 4-character): Identifies the product, validated against the `GSPROD` file.
- **Specific Gravity** (`KGRAV`, 3.1 numeric): API gravity for the product; if zero, retrieved from `GSPROD`.
- **Pounds** (`KLBS`, 7.0 numeric): Input quantity in pounds (for pounds-to-gallons conversion); must be zero if gallons provided.
- **Gallons** (`KGAL`, 7.0 numeric): Input quantity in gallons (for gallons-to-pounds conversion); must be zero if pounds provided.

## Outputs
- **Output Pounds** (`OLBS`, 9.0 numeric): Converted pounds (for gallons-to-pounds) or input pounds (for pounds-to-gallons).
- **Output Gallons** (`OGAL`, 9.0 numeric): Converted gallons (for pounds-to-gallons) or input gallons (for gallons-to-pounds).
- **Product Description** (`PRDS`, 30-character): Description of the product from `GSPROD`.
- **Error Code** (numeric): Indicates success (0) or error (1: Invalid company, 2: Invalid product, 3: Both pounds and gallons provided).
- **Error Message** (35-character): Descriptive error message if applicable.

## Process Steps
1. **Validate Company Number**:
   - Perform a lookup in the `INCONT` file using `KCO`.
   - If not found, return error code 1 and message "INVALID COMPANY NO".

2. **Validate Product Code**:
   - Construct a 6-byte key (`KCO + KPROD`) and perform a lookup in the `GSPROD` file.
   - If not found, return error code 2 and message "INVALID PRODUCT CODE".
   - Retrieve product description (`TPDESC`) and specific gravity (`TPGRAV`) from `GSPROD`.

3. **Validate Input Quantities**:
   - If both `KLBS` and `KGAL` are non-zero, return error code 3 and message "ENTER EITHER POUNDS OR GALLONS".

4. **Determine Specific Gravity**:
   - If `KGRAV` is zero, use `TPGRAV` from `GSPROD`.
   - If `KGRAV` is non-zero, use the provided value.

5. **Perform Conversion**:
   - **If `KLBS` is non-zero (pounds-to-gallons)**:
     - Calculate specific gravity: `SG = 141.5 / (131.5 + KGRAV)`.
     - Calculate gallons: `OGAL = (KLBS / (SG * 999.01599999999996)) * 119.8`.
     - Set `OLBS = KLBS`.
   - **If `KGAL` is non-zero (gallons-to-pounds)**:
     - Calculate specific gravity: `SG = 141.5 / (131.5 + KGRAV)`.
     - Calculate pounds: `OLBS = (KGAL * (SG * 999.01599999999996)) / 119.8`.
     - Set `OGAL = KGAL`.

6. **Return Results**:
   - Return `OLBS`, `OGAL`, `PRDS`, error code (0 for success), and empty error message if no errors.

## Business Rules
1. **Company Validation**: The company number must exist in the `INCONT` file.
2. **Product Validation**: The product code must exist in the `GSPROD` file for the specified company.
3. **Input Exclusivity**: Either pounds or gallons can be provided, but not both.
4. **Specific Gravity Source**: Use the provided `KGRAV` if non-zero; otherwise, use `TPGRAV` from `GSPROD`.
5. **Conversion Accuracy**: Use the API-standard formula for conversions, applicable to petroleum or similar liquids.
6. **Error Handling**: Return specific error codes and messages for invalid inputs to guide corrective action.

## Calculations
- **Specific Gravity**: `SG = 141.5 / (131.5 + API Gravity)`.
- **Pounds to Gallons**: `Net Gallons = (Pounds / (SG * 999.01599999999996)) * 119.8`.
- **Gallons to Pounds**: `Pounds = (Gallons * (SG * 999.01599999999996)) / 119.8`.
- **Constants**: `999.01599999999996` and `119.8` are industry-specific adjustments for API gravity-based conversions.

## Dependencies
- **Files**:
  - `INCONT`: Inventory control file for company validation.
  - `GSPROD`: Product master file for product validation, description, and specific gravity.
- **External Modules**: None (conversion logic is self-contained, adapted from `MINLBGL1`).

</xaiArtifact>

### Additional Notes
- **Use Case Selection**: The function requirement document focuses on the primary use case (bidirectional conversion), combining both pounds-to-gallons and gallons-to-pounds into a single function for simplicity. The validation use case is incorporated as part of the input validation steps.
- **Non-Interactive Design**: The function eliminates screen interactions, taking all inputs as parameters and returning results directly, suitable for batch processing or integration with other systems.
- **Calculation Details**: The formulas are retained from `MINLBGL1`, with constants (`999.01599999999996`, `119.8`) specific to the API standard, likely for petroleum products.
- **Error Handling**: The function includes robust error handling to ensure data integrity, aligning with the interactive program’s validation logic.
- **Assumptions**: The document assumes the function is called with valid numeric formats and that the `INCONT` and `GSPROD` files are accessible in the same library as specified in the OCL procedure.

If you need further refinements or additional use cases documented as separate functions, please let me know!