Below, I will first identify the use cases implemented by the `BB204.ocl36` OCL script and the `BB204.rpgle` RPG program, which together form a call stack for automating customer sales agreement pricing. Then, I will provide a concise function requirement document for a large function that encapsulates the primary use case, assuming it processes inputs programmatically rather than through a screen.

---

### Use Cases Implemented by the BB204 Call Stack

The `BB204.ocl36` script and `BB204.rpgle` program work together to automate the posting of customer sales agreement pricing. Based on the provided code, the following use cases are implemented:

1. **Auto-Post Customer Sales Agreement Pricing**:
   - **Description**: Processes a batch of pricing records from a work file (`bb204t`), validates the start date, updates existing pricing records in the customer agreement file (`bicuag`), creates new pricing records, and maintains a history of changes in the history file (`bicuagh`). Generates a printed report of the changes.
   - **Inputs**: Work file (`bb204t`) containing pricing records, user-entered start date (via display file `bb204d`), and file group parameter (`G` or `Z`).
   - **Outputs**: Updated `bicuag` records, new `bicuagh` history records, cleared `bb204t` and `PRSABLO` files, and a printed report via `qsysprt`.
   - **Process**: Validates the start date, checks for records in `bb204t`, updates or creates pricing records, increments sequence numbers via `bicont`, logs history, and prints a report.
   - **Business Rules**:
     - Start date must match the start date in `bb204t`.
     - Existing records are updated only if their expiration date is greater than the new start date or zero.
     - New records are created with incremented sequence numbers.
     - History records are written for both updated and new pricing records.
     - A report is generated with old and new pricing details.

2. **Validate and Prepare Work File Environment**:
   - **Description**: Prepares the environment by creating a temporary work file (`BB204W` in `QTEMP`) from a workstation file, applying file overrides, and clearing output files (`bb204t`, `PRSABLO`) after processing.
   - **Inputs**: Workstation file (`?9?BB204?WS?` or `?9?BB204WS`), file group parameter (`?9?` in OCL).
   - **Outputs**: Temporary file `BB204W`, cleared `bb204t` and `PRSABLO`, and removed overrides.
   - **Process**: Duplicates the workstation file into `QTEMP`, applies overrides, and cleans up after processing.
   - **Business Rules**:
     - The process halts if `bb204t` has no records.
     - Temporary files and overrides are removed post-processing to maintain a clean environment.

3. **Generate Pricing Change Report**:
   - **Description**: Produces a printed report detailing the pricing changes, including company, customer, location, old and new prices, sequence numbers, and dates.
   - **Inputs**: Data from `bb204t` and `bicuag` during processing.
   - **Outputs**: Printed report via `qsysprt` with headers and detail lines.
   - **Process**: Writes header and detail lines to `qsysprt`, handling overflow conditions.
   - **Business Rules**:
     - The report includes old and new pricing details, sequence numbers, and dates.
     - Headers are printed on overflow, including job, program, and user information.

4. **Maintain Pricing History**:
   - **Description**: Logs all pricing changes (both updated and new records) to a history file (`bicuagh`) for audit purposes.
   - **Inputs**: Data from `bb204t` (new records) and `bicuag` (existing records).
   - **Outputs**: Records in `bicuagh` with timestamps and user IDs.
   - **Process**: Writes records to `bicuagh` during the `postit` subroutine for both new and expired records.
   - **Business Rules**:
     - History records include all pricing fields, timestamps, and the user ID.
     - Change timestamps reflect the current date and time.

---

### Function Requirement Document

The following function requirement document describes a single, programmatic function that encapsulates the primary use case: **Auto-Post Customer Sales Agreement Pricing**. It assumes the function receives all necessary inputs programmatically (e.g., start date, file group, and work file data) rather than through a screen interface.

<xaiArtifact artifact_id="3e5a5071-8f97-443a-8f8c-0e1b71577ca0" artifact_version_id="91b162bc-b8c0-49ea-a189-7c660f31cece" title="AutoPostPricingFunction.md" contentType="text/markdown">

# Function Requirement Document: Auto-Post Customer Sales Agreement Pricing

## Function Name
`AutoPostCustomerPricing`

## Purpose
Automates the posting of customer sales agreement pricing by processing a batch of pricing records, updating or creating records in the customer agreement file, maintaining a history of changes, and generating a report.

## Inputs
- **Start Date** (`startDate`): Date in CCYYMMDD format for the new pricing records.
- **File Group** (`fileGroup`): String ('G' or 'Z') to determine the library for file overrides.
- **Pricing Records** (`pricingRecords`): Array of records containing pricing data (e.g., company number, customer number, location, price, contract number, etc.), equivalent to the `bb204t` work file.
- **Current Date/Time** (`currentDateTime`): System timestamp in CCYYMMDDHHMMSS format for record creation and updates.

## Outputs
- **Updated Customer Agreement File**: Records updated or added to the customer agreement file (`bicuag`).
- **History File Records**: Records written to the history file (`bicuagh`) for audit purposes.
- **Printed Report**: Report generated via the printer file (`qsysprt`) detailing pricing changes.
- **Sequence Number Update**: Updated sequence number in the control file (`bicont`).
- **Cleared Work Files**: Cleared `bb204t` and `PRSABLO` files.

## Process Steps
1. **Validate Inputs**:
   - Verify that `startDate` is a valid date (1–12 for month, valid days per month, leap year handling).
   - Confirm that `pricingRecords` is not empty; if empty, return an error.
   - Ensure `startDate` matches the start date in each `pricingRecords` entry.

2. **Apply File Overrides**:
   - Apply file overrides for `bicuag`, `bicont`, `bicuagh`, and `bb204t` based on `fileGroup` ('G' for `gbicuag`, etc.; 'Z' for `zbicuag`, etc.).
   - Configure the printer file (`qsysprt`) with specified attributes (e.g., 68 lines, 184 characters, 8 LPI, 15 CPI).

3. **Process Pricing Records**:
   - For each record in `pricingRecords`:
     - **Check Existing Record**:
       - Query `bicuag` using company number (`bacono`) and sequence number (`baseqn`).
       - If found:
         - If the existing record’s expiration date (`baend8`) is greater than `startDate` or zero, update it to `startDate - 1 day` at 23:59.
         - Update the last updated date (`baludt`) and time (`balutm`) with `currentDateTime`.
         - Write the original record to `bicuagh` with the current timestamp and user ID.
       - If not found, initialize old values to zero/blank.
     - **Create New Record**:
       - Retrieve the next sequence number from `bicont` for the company number, increment it, and update `bicont`.
       - Create a new `bicuag` record with fields from `pricingRecords` (e.g., `bacust`, `baloc`, `baprce`, etc.).
       - Set creation/update timestamps (`bacrdt`, `bacrtm`, `baludt`, `balutm`) to `currentDateTime`.
       - Set start date (`bastdt`, `bastd8`) to `startDate` at 00:00.
       - Set expiration date (`baendt`, `baend8`) to a default (e.g., 2079/12/31) or as specified.
       - Write the new record to `bicuagh`.
     - **Generate Report Line**:
       - Write a detail line to `qsysprt` with company, customer, location, old/new prices, sequence numbers, and dates.
       - Handle overflow by printing headers as needed.

4. **Clean Up**:
   - Clear `bb204t` and `PRSABLO` files.
   - Remove file overrides.
   - Close all files.

## Business Rules
1. **Start Date Validation**:
   - Must be a valid date (correct month, day, leap year handling).
   - Must match the start date in each `pricingRecords` entry; otherwise, return an error.

2. **Expiration Date Calculation**:
   - For existing `bicuag` records, update the expiration date to `startDate - 1 day` at 23:59 if the current expiration is greater than `startDate` or zero.
   - New records use a default expiration date (e.g., 2079/12/31) unless specified.

3. **Sequence Number Management**:
   - Retrieve the last sequence number from `bicont` for the company number, increment by 1, and update `bicont`.

4. **History Logging**:
   - Write both updated (expired) and new records to `bicuagh` with all pricing fields, `currentDateTime` as the change timestamp, and the user ID.

5. **Reporting**:
   - Generate a report with headers (company, program, user, date, time, file group) and detail lines (company, customer, location, old/new prices, sequence numbers, dates).
   - Handle overflow by printing headers when the page limit (62 lines) is reached.

6. **Error Handling**:
   - Return errors for invalid start dates or empty `pricingRecords`.
   - Log errors to a message queue for traceability.

## Calculations
- **Expiration Date**:
  - `expirationDate = startDate - 1 day`, calculated using a date arithmetic function (e.g., equivalent to `GSDTCLC1`).
  - Example: If `startDate = 20251201`, then `expirationDate = 20251130`.
- **Sequence Number**:
  - `newSequenceNumber = lastSequenceNumber (from bicont) + 1`.
- **Date Formatting**:
  - Convert `startDate` to CCYYMMDD with century indicator (1 for 20xx, 0 for 19xx).
  - Set start time to 00:00 and end time to 23:59 for new records.

## Dependencies
- **Files**:
  - `bicuag`: Customer agreement file (update/add).
  - `bicont`: Control file for sequence numbers (update).
  - `bicuagh`: History file (output).
  - `bb204t`: Work file with pricing records (input).
  - `PRSABLO`: Output file to be cleared.
  - `qsysprt`: Printer file for reporting.
- **External Function**:
  - Date arithmetic function (e.g., `GSDTCLC1`) to calculate `startDate - 1 day`.

## Error Handling
- Return error codes/messages for:
  - Invalid or mismatched start date.
  - Empty `pricingRecords`.
  - File access or override failures.
- Log errors to a message queue for audit purposes.

</xaiArtifact>

---

### Notes
- The primary use case (Auto-Post Customer Sales Agreement Pricing) is the core functionality, while the other use cases (environment preparation, reporting, and history maintenance) are supporting functions integrated into the main process.
- The function requirement document assumes a programmatic interface, eliminating the need for the display file (`bb204d`) and user interaction (e.g., F09 key press). All inputs are provided upfront, making the process fully automated.
- The document focuses on business requirements (validation, updates, history, reporting) and includes calculations (e.g., expiration date, sequence number) as specified in the RPG program.