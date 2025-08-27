The provided RPGLE programs, `BB931` and `MGSTABL`, form a call stack where `BB931` is the main program for managing flat freight codes, and `MGSTABL` is a utility program for table value lookups, specifically for selecting valid units of measure. Below, I’ll identify the use cases implemented by this call stack and then provide a function requirement document for one of the use cases, reimagined as a non-interactive function that processes inputs directly.

### List of Use Cases Implemented in the Call Stack

The call stack (`BB931` calling `MGSTABL`) supports the following use cases, derived from the functionality of `BB931` and its interaction with `MGSTABL`. Each use case represents a distinct user goal within the flat freight code maintenance/inquiry system.

1. **Inquire Flat Freight Codes**:
   - **Description**: Allows users to view existing flat freight codes for a specific company and optionally filter by freight code without modifying data.
   - **Details**:
     - Run mode: `INQ` (Inquiry).
     - Displays a subfile of freight code records from `BBFRTB` (or `BBFRTBRD` for reading) based on company code (`c1cono`) and optional freight code (`c1frcd`).
     - Users can reposition the subfile using control fields and navigate pages with the Page Down key.
     - Fields are protected (`*IN71 = *ON`) to prevent modifications.
     - No database updates are performed.

2. **Maintain Flat Freight Codes (Add)**:
   - **Description**: Enables users to add new flat freight codes to the database.
   - **Details**:
     - Run mode: `MNT` (Maintenance), Entry mode (`s1updt = *OFF`).
     - Users enter a new freight code (`s1frcd`), rate (`s1rate`), freight label (`s1frlb`), unit of measure (`s1frum`), and unit of measure rate (`s1frpu`) in the subfile.
     - Validates that the freight code does not already exist in `BBFRTB`.
     - Validates company code (`c1cono`) against `BICONT` and unit of measure (`s1frum`) against `GSTABL` (via `MGSTABL`).
     - Writes a new record to `BBFRTB` with the provided values.

3. **Maintain Flat Freight Codes (Update)**:
   - **Description**: Allows users to update existing flat freight codes.
   - **Details**:
     - Run mode: `MNT`, Update mode (`s1updt = *ON`).
     - Users modify non-key fields (`s1rate`, `s1frlb`, `s1frum`, `s1frpu`) for existing freight codes in the subfile.
     - Validates company code and unit of measure as above.
     - Updates the corresponding record in `BBFRTB`.

4. **Delete Flat Freight Codes**:
   - **Description**: Permits users to delete existing flat freight codes.
   - **Details**:
     - Run mode: `MNT`.
     - Users select a record with F23, confirm deletion in a window (`sfldel1`), and press F23 again to delete.
     - Deletes the record from `BBFRTB` if it exists and clears the subfile record.

5. **Look Up Unit of Measure**:
   - **Description**: Provides a lookup interface to select a valid unit of measure code from `GSTABL`, used by `BB931` for `s1frum`.
   - **Details**:
     - Called by `BB931` when F04 is pressed for the `S1FRUM` field.
     - Displays a subfile of `GSTABL` records filtered by table type (`p$type`, e.g., `BIUNMS`) and optional search string (`c$srch`).
     - Users select a record (option `1`), and the selected code (`tbcode`) is returned to `BB931` as `p$code`.
     - Validates the table type against `GSTABL` and filters descriptions if a search string is provided.

### Function Requirement Document: Add/Update Flat Freight Code Function

**Function Name**: `AddUpdateFlatFreightCode`

**Purpose**: To add a new flat freight code or update an existing one in the database based on provided inputs, ensuring data integrity through validation against company and unit of measure tables.

**Inputs**:
- **Company Code** (`companyCode`): Numeric, 2 digits, required. Identifies the company.
- **Freight Code** (`freightCode`): Character, length as defined in `BBFRTB`, required. Unique identifier for the freight code.
- **Freight Rate** (`freightRate`): Numeric, as defined in `BBFRTB`, optional. The flat freight rate.
- **Freight Label** (`freightLabel`): Numeric, as defined in `BBFRTB`, optional. Additional identifier or label.
- **Unit of Measure** (`unitOfMeasure`): Character, as defined in `GSTABL`, optional. The unit of measure code.
- **Unit of Measure Rate** (`unitOfMeasureRate`): Numeric, as defined in `BBFRTB`, optional. Rate associated with the unit of measure.
- **Mode** (`mode`): Character, 3 characters, required. Either `ADD` (add new record) or `UPD` (update existing record).
- **File Group** (`fileGroup`): Character, 1 character, required. Either `G` (use `GBBFRTB`, `GBICONT`, `GGSTABL`) or `Z` (use `ZBBFRTB`, `ZBICONT`, `ZGSTABL`).

**Outputs**:
- **Success Flag** (`success`): Boolean. Indicates whether the operation was successful.
- **Error Message** (`errorMessage`): Character, 80 characters. Contains error details if the operation fails.
- **Company Name** (`companyName`): Character, as defined in `BICONT`. Name of the validated company.
- **Unit of Measure Description** (`unitOfMeasureDesc`): Character, as defined in `GSTABL`. Description of the validated unit of measure.

**Process Steps**:
1. **Apply File Overrides**:
   - Based on `fileGroup`, apply database overrides to use the correct files (`G` or `Z` group).
   - Example: For `G`, override `BICONT` to `GBICONT`, `GSTABL` to `GGSTABL`, `BBFRTB` to `GBBFRTB`.

2. **Validate Company Code**:
   - Query `BICONT` with `companyCode`.
   - If not found or zero, return `success = false`, `errorMessage = "Invalid company code"`.
   - Retrieve `companyName` from `BICONT`.

3. **Validate Unit of Measure**:
   - If `unitOfMeasure` is provided, query `GSTABL` with `tableType = 'BIUNMS'` and `tableCode = unitOfMeasure`.
   - If not found, return `success = false`, `errorMessage = "Invalid unit of measure"`.
   - Retrieve `unitOfMeasureDesc` from `GSTABL`.
   - If `unitOfMeasure` is provided, `unitOfMeasureRate` must be non-zero, and vice versa. If inconsistent, return `success = false`, `errorMessage = "Must provide both unit of measure and rate or neither"`.

4. **Validate Input Data**:
   - Ensure at least one of `freightRate`, `freightLabel`, `unitOfMeasure`, or `unitOfMeasureRate` is provided.
   - If all are blank/zero, return `success = false`, `errorMessage = "Must provide at least one value"`.

5. **Check Freight Code Existence**:
   - For `mode = ADD`, query `BBFRTB` with `companyCode` and `freightCode`.
     - If found, return `success = false`, `errorMessage = "Freight code already exists"`.
   - For `mode = UPD`, query `BBFRTB` with `companyCode` and `freightCode`.
     - If not found, return `success = false`, `errorMessage = "Freight code does not exist"`.

6. **Perform Database Operation**:
   - For `mode = ADD`:
     - Create a new record in `BBFRTB` with `companyCode`, `freightCode`, `freightRate`, `freightLabel`, `unitOfMeasure`, and `unitOfMeasureRate`.
   - For `mode = UPD`:
     - Update the existing record in `BBFRTB` with the provided non-key fields (`freightRate`, `freightLabel`, `unitOfMeasure`, `unitOfMeasureRate`).
   - Commit the operation.

7. **Return Results**:
   - Set `success = true`, clear `errorMessage`, and return `companyName` and `unitOfMeasureDesc` if validations and operations succeed.
   - Otherwise, return `success = false` with the appropriate `errorMessage`.

**Business Rules**:
1. **Company Code Validation**: Must exist in `BICONT`. Zero or invalid codes are rejected.
2. **Unit of Measure Consistency**: If a unit of measure is provided, its rate must be non-zero, and vice versa.
3. **Non-Empty Input**: At least one non-key field (`freightRate`, `freightLabel`, `unitOfMeasure`, `unitOfMeasureRate`) must be provided.
4. **Unique Freight Code**: In `ADD` mode, the freight code must not already exist for the given company.
5. **Existing Freight Code**: In `UPD` mode, the freight code must exist for the given company.
6. **File Group Selection**: The correct file group (`G` or `Z`) must be specified to access the appropriate database files.
7. **Data Integrity**: Only non-key fields can be updated in `UPD` mode; key fields (`companyCode`, `freightCode`) are protected.

**Calculations**:
- No complex calculations are performed. The function moves input values directly to the database fields after validation.
- The unit of measure description is retrieved from `GSTABL` but involves no computation.

**Dependencies**:
- **Files**: `BICONT` (company validation), `GSTABL` (unit of measure validation), `BBFRTB` (freight code storage).
- **External Program**: None required, as the function performs direct database operations.

**Error Handling**:
- Returns specific error messages for invalid company code, unit of measure, inconsistent unit of measure/rate, missing values, or freight code existence issues.
- Ensures database operations are only performed if all validations pass.

**Notes**:
- This function replaces the interactive subfile processing in `BB931` and the unit of measure lookup in `MGSTABL` with direct input validation and database operations.
- Assumes the database files are accessible with the correct overrides applied based on `fileGroup`.

<xaiArtifact artifact_id="d56d5ace-2241-4cb5-9f8c-ada852bca4ab" artifact_version_id="85babdbc-c839-41ca-9283-3a58b989a2da" title="AddUpdateFlatFreightCode_Requirements.txt" contentType="text/markdown">

# Function Requirement Document: AddUpdateFlatFreightCode

## Purpose
Add or update a flat freight code in the database, ensuring data integrity through validation against company and unit of measure tables.

## Inputs
- **companyCode**: Numeric, 2 digits, required. Company identifier.
- **freightCode**: Character, per `BBFRTB`, required. Unique freight code.
- **freightRate**: Numeric, per `BBFRTB`, optional. Flat freight rate.
- **freightLabel**: Numeric, per `BBFRTB`, optional. Freight label.
- **unitOfMeasure**: Character, per `GSTABL`, optional. Unit of measure code.
- **unitOfMeasureRate**: Numeric, per `BBFRTB`, optional. Unit of measure rate.
- **mode**: Character, 3 chars, required. `ADD` or `UPD`.
- **fileGroup**: Character, 1 char, required. `G` or `Z` for file selection.

## Outputs
- **success**: Boolean. Operation success status.
- **errorMessage**: Character, 80 chars. Error details if failed.
- **companyName**: Character, per `BICONT`. Validated company name.
- **unitOfMeasureDesc**: Character, per `GSTABL`. Unit of measure description.

## Process Steps
1. **Apply File Overrides**: Use `fileGroup` to override to `G` (`GBICONT`, `GGSTABL`, `GBBFRTB`) or `Z` (`ZBICONT`, `ZGSTABL`, `ZBBFRTB`) files.
2. **Validate Company Code**: Query `BICONT` with `companyCode`. If invalid or zero, return `success = false`, `errorMessage = "Invalid company code"`. Set `companyName`.
3. **Validate Unit of Measure**: If `unitOfMeasure` provided, query `GSTABL` with `tableType = 'BIUNMS'`. If invalid, return `success = false`, `errorMessage = "Invalid unit of measure"`. Set `unitOfMeasureDesc`. Ensure `unitOfMeasure` and `unitOfMeasureRate` are both provided or neither.
4. **Validate Input Data**: Ensure at least one of `freightRate`, `freightLabel`, `unitOfMeasure`, or `unitOfMeasureRate` is non-blank/zero. If not, return `success = false`, `errorMessage = "Must provide at least one value"`.
5. **Check Freight Code Existence**:
   - `ADD`: If `companyCode` and `freightCode` exist in `BBFRTB`, return `success = false`, `errorMessage = "Freight code already exists"`.
   - `UPD`: If not found, return `success = false`, `errorMessage = "Freight code does not exist"`.
6. **Perform Database Operation**:
   - `ADD`: Write new record to `BBFRTB` with input values.
   - `UPD`: Update existing record’s non-key fields in `BBFRTB`.
7. **Return Results**: Set `success = true`, clear `errorMessage`, return `companyName` and `unitOfMeasureDesc` if successful.

## Business Rules
1. **Company Code**: Must exist in `BICONT`.
2. **Unit of Measure**: If provided, must exist in `GSTABL` with `tableType = 'BIUNMS'`, and `unitOfMeasureRate` must be non-zero (and vice versa).
3. **Non-Empty Input**: At least one non-key field must be provided.
4. **Unique Freight Code**: Must not exist for `ADD`, must exist for `UPD`.
5. **File Group**: `G` or `Z` determines database files.
6. **Data Integrity**: Key fields (`companyCode`, `freightCode`) are protected in `UPD`.

## Calculations
- No calculations; input values are moved to database fields after validation.
- Retrieves `companyName` and `unitOfMeasureDesc` from respective files.

## Dependencies
- **Files**: `BICONT`, `GSTABL`, `BBFRTB`.
- **External Programs**: None.

## Error Handling
- Returns specific messages for invalid company code, unit of measure, inconsistent unit of measure/rate, missing values, or freight code issues.
- Database operations only proceed if validations pass.

</xaiArtifact>