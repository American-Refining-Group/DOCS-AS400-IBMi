The call stack consists of the following programs, which together implement the EDI (Electronic Data Interchange) outbound Bill of Lading (BOL) tracking maintenance workflow on an IBM midrange system (e.g., AS/400 or IBM i):

1. **EDI811P.ocl36.txt** (OCL): Initiates the workflow, sets up files, and calls the `EDI811P` RPG program and subsequently `EDI811`.
2. **EDI811P.rpg36.txt** (RPG): Validates user-entered company and BOL numbers, writes validated records to `EDIBOL`.
3. **EDI811.ocl36.txt** (OCL): Loads and runs the `EDI811` RPG program and calls `EDI404SD`.
4. **EDI811.rpg36.txt** (RPG): Formats validated BOL data into EDI 404-compliant records in `EDIOUT` and `EDIRSI`.
5. **EDI404SD.ocl36.txt** (OCL): Checks for records, calls `EDI404` for validation and `EDI404FTP` for transmission, and archives/clears files.
6. **EDI404.rpgle.txt** (RPGLE): Displays a screen for user validation, setting a cancellation flag.
7. **EDI404FTP.clp.txt** (CLP): Transmits EDI data via FTP and sends a success message.

Below, I’ll identify the use cases implemented by this call stack and then provide a function requirement document for a non-interactive version of the primary use case, focusing on business requirements and necessary calculations.

---

### Use Cases Implemented by the Call Stack

The call stack implements the following use cases, each representing a distinct functional goal within the EDI BOL tracking maintenance process:

1. **Validate and Store BOL Data**:
   - **Description**: Allows a user to enter a company number and up to 10 BOL numbers via a screen, validates them against master data, and stores valid records in the `EDIBOL` file.
   - **Programs Involved**: `EDI811P.ocl36`, `EDI811P.rpg36`.
   - **Key Actions**:
     - User inputs company number (`KYCO`) and BOL numbers via `EDI811FM` screen.
     - Validates `KYCO` against `BICONT` (not deleted, `BCDEL ≠ 'D'`).
     - Validates BOL numbers against `EDIBOLTX` using a constructed key (`40402` + BOL + `000`).
     - Writes valid `KYCO` and BOL numbers to `EDIBOL`.
     - Displays error messages for invalid inputs.
   - **Outcome**: Creates a validated `EDIBOL` file for further processing or cancels if invalid.

2. **Generate EDI 404-Compliant BOL Records**:
   - **Description**: Processes validated BOL records from `EDIBOL`, retrieves detailed transaction data from `EDIBOLTX`, and generates EDI 404-compliant records in `EDIOUT` and `EDIRSI`.
   - **Programs Involved**: `EDI811.ocl36`, `EDI811.rpg36`.
   - **Key Actions**:
     - Reads `EDIBOL` records (`KYCO#`, `KYBOL#`, `KYSRN#`).
     - Matches with `EDIBOLTX` records (multiple formats, C0–C8) to retrieve details (e.g., purchase orders, routing, shipment data).
     - Writes formatted records to `EDIOUT` (e.g., `40401`–`40424`, trailer `#EOT`) and `EDIRSI` (routing/shipment data).
   - **Outcome**: Produces EDI 404-compliant data for transmission.

3. **Validate EDI Transmission**:
   - **Description**: Presents a screen to the user to confirm or cancel the EDI 404 data transmission, setting a cancellation flag if the user cancels.
   - **Programs Involved**: `EDI404SD.ocl36`, `EDI404.rpgle`.
   - **Key Actions**:
     - Displays `SCREEN1` (via `EDI404SC`) for user confirmation.
     - Sets `CANCEL` field to blanks (proceed) or `'CANCEL'` (cancel) based on user input (`KE` or `KG`).
     - If canceled, triggers clearing of the `EDI404` file (`EDIOUT`) in `EDI404SD`.
   - **Outcome**: Ensures user validation before transmission or cancels the process.

4. **Transmit EDI Data and Archive**:
   - **Description**: Checks for records, transmits EDI 404 data via FTP to RSI Logistics, archives `EDIRSI` records, and clears files.
   - **Programs Involved**: `EDI404SD.ocl36`, `EDI404FTP.clp`.
   - **Key Actions**:
     - Checks if `EDIRSI` has records; cancels if empty.
     - Calls `EDI404` for validation, then `EDI404FTP` to transmit `EDIOUT` (labeled `?9?EDI404`) to `B2B.RSILOGISTICS.com`.
     - Copies `EDIRSI` to `EDIRSIH` for archiving.
     - Clears `EDIRSI` and, if canceled, `EDI404` (`EDIOUT`).
     - Sends a success message for transmission.
   - **Outcome**: Completes the EDI transmission and archives data for audit purposes.

---

### Function Requirement Document for Non-Interactive Primary Use Case

The primary use case is **Generate and Transmit EDI 404-Compliant BOL Records**, combining the core functionality of validating BOL data, generating EDI 404 records, and transmitting them without interactive screen prompts. Below is a concise function requirement document, assuming the function receives all necessary inputs programmatically (e.g., company number and BOL numbers) rather than via a screen.

<xaiArtifact artifact_id="406cc4d6-3c8b-4f80-8840-9b4f810e9558" artifact_version_id="d3f97321-8695-4ba8-81ce-6cde218716a2" title="EDI404BOLProcessingRequirements.md" contentType="text/markdown">

# EDI 404 BOL Processing Function Requirements

## Purpose
The function processes a company number and a list of BOL numbers to generate and transmit EDI 404-compliant BOL records to RSI Logistics, validating inputs, formatting EDI data, and archiving records for audit purposes.

## Inputs
- **Company Number (`KYCO`)**: 2-character identifier of the company.
- **BOL Numbers**: List of up to 10 BOL numbers (7 digits each).
- **Library Prefix**: System or library prefix (e.g., `?9?`) for file access.

## Outputs
- **EDIOUT File**: EDI 404-compliant records (197 bytes) for transmission.
- **EDIRSIH File**: Archived routing/shipment records for audit.
- **Status**: Success message or error code (e.g., invalid input, no records, cancellation).

## Process Steps
1. **Validate Inputs**:
   - Check `KYCO` against `BICONT` file; ensure it exists and is not deleted (`BCDEL ≠ 'D'`).
   - For each BOL number, construct a key (`40402` + BOL + `000`) and validate against `EDIBOLTX`.
   - If any validation fails, return error code with message (e.g., "INVALID COMPANY NUMBER" or "INVALID ORDER NUMBER AT LINE X").
2. **Store Validated Data**:
   - Write valid `KYCO` and BOL numbers to `EDIBOL` (12 bytes: `KYCO` 2 bytes, `KYBOL#` 7 bytes, `KYSRN#` 3 bytes).
3. **Generate EDI 404 Records**:
   - Read `EDIBOL` records.
   - Chain to `EDIBOLTX` using `KYBOL#` and `KYSRN#` to retrieve details (e.g., purchase orders `GS03`, routing `SYSHM`, shipment data `ELH101`).
   - Write EDI 404 segments to `EDIOUT` (e.g., `40401`–`40424`, trailer `#EOT`) and routing data to `EDIRSI`.
4. **Validate Transmission Readiness**:
   - Check if `EDIRSI` contains records; if empty, return error code "NO RECORDS TO SEND".
5. **Transmit EDI Data**:
   - Use FTP to send `EDIOUT` (labeled `?9?EDI404`) to `B2B.RSILOGISTICS.com` with commands from `FTP_IN01(FTP_GIIN13)`.
   - Log FTP output to `FTP_GIOUT(ERROR_GI13)`.
6. **Archive and Clean Up**:
   - Copy `EDIRSI` to `EDIRSIH` (append mode, no format check).
   - Clear `EDIRSI` and, if canceled, `EDIOUT`.
7. **Return Status**:
   - Return success message ("FTP TRANSMITTED SUCCESSFULLY") or error code.

## Business Rules
1. **Validation**:
   - `KYCO` must exist in `BICONT` and not be deleted.
   - Each BOL must exist in `EDIBOLTX` with a valid key (`40402` + BOL + `000`).
   - Up to 10 BOLs are processed; empty or zero BOLs are skipped.
2. **EDI 404 Compliance**:
   - Records in `EDIOUT` follow EDI 404 format (e.g., `40401`–`40424` segments, fixed positions for `KYBOL#`, `KYSRN#`, constants like `000`, `TKR`).
   - Trailer record (`#EOT`) marks the end of the transaction set.
3. **Transmission**:
   - FTP transmission targets `B2B.RSILOGISTICS.com`.
   - `EDIRSI` must contain records to proceed; else, the process aborts.
4. **Archiving**:
   - `EDIRSI` records are archived to `EDIRSIH` before clearing.
   - `EDIOUT` is cleared only if the process is canceled.
5. **Error Handling**:
   - Return specific error messages for invalid inputs or empty files.
   - Ignore minor FTP errors to ensure transmission attempts complete.
6. **Environment**:
   - Clear local variables and files (`EDIRSI`, `FTP_GIOUT`) after processing to prevent data leakage.

## Calculations
- **Key Construction for Validation**:
  - BOL key = `'40402' + BOL (7 digits) + '000'`.
  - Example: For BOL `1234567`, key = `404021234567000`.
- **Record Formatting**:
  - `EDIOUT` records (197 bytes) use fixed positions (e.g., `KYBOL#` at 6–12, `KYSRN#` at 195–197).
  - `EDIRSI` records (1970 bytes) include detailed fields (e.g., `EN9011`, `ELH101`).
- **Serial Number (`KYSRN#`)**:
  - 3-digit serial number from `EDIBOL` and `EDIBOLTX` aligns across files to ensure data consistency.

## Files Used
- **Input**:
  - `BICONT`: Validates company number (`KYCO`).
  - `EDIBOLTX`: Validates BOL numbers and provides EDI details.
  - `EDIBOL`: Stores validated BOL records.
  - `EDIRSI`: Contains routing/shipment data; checked for records.
  - `FTP_IN01(FTP_GIIN13)`: FTP commands.
- **Output**:
  - `EDIOUT` (labeled `?9?EDI404`): EDI 404 records.
  - `EDIRSIH`: Archived `EDIRSI` records.
  - `FTP_GIOUT(ERROR_GI13)`: FTP logs/errors.

## Assumptions
- Inputs are provided programmatically (e.g., via API or file) rather than a screen.
- Cancellation is handled by an external parameter or flag, not user input.
- FTP credentials and commands are preconfigured in `FTP_IN01`.

</xaiArtifact>

---

### Notes on Use Case Selection and Non-Interactive Design

- **Primary Use Case**: The function requirement document focuses on the end-to-end process of generating and transmitting EDI 404 BOL records, combining use cases 1, 2, and 4 (validation, EDI generation, transmission/archiving). Use case 3 (interactive validation via `EDI404`) is omitted, as the non-interactive requirement eliminates screen prompts, assuming validation is handled programmatically or via input data checks.
- **Simplifications**: The document consolidates the workflow into a single function, removing interactive elements (e.g., `EDI404SC` screen, `KE`/`KG` keys) and assuming inputs are provided upfront. Cancellation is handled via a parameter or error condition rather than user input.
- **Context Integration**: The document leverages the entire call stack to ensure all business rules (e.g., validation, EDI formatting, archiving) and file interactions are covered, aligning with the original program’s functionality.

If you need further refinements or additional details (e.g., specific field mappings for EDI 404 segments), please provide additional context or requirements.