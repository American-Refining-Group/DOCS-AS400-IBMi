The RPGLE program `AR489` and its associated OCL program (`AR489.ocl36.txt`) implement a **Credit Limit Grouping Maintenance** system for managing customer credit limit groups in an accounts receivable system on an IBM midrange platform. Below, I’ll identify the use cases implemented by the program and then provide a concise function requirement document for a non-interactive version of the primary use case, assuming inputs are provided programmatically rather than via a screen.

### List of Use Cases Implemented

Based on the analysis of the OCL and RPGLE programs, the primary use case implemented is:

1. **Maintain Customer Credit Limit Groups**:
   - **Description**: Allows users to add, update, or delete customer records associated with a credit limit group, identified by a company number and customer number. The program validates inputs, retrieves existing group data, and persists changes to the database.
   - **Details**:
     - Validates the company number against the `ARCONT` file and ensures the customer number is non-zero.
     - Manages a group of up to 25 customers (`ars` array) with a delete code (`del`) to indicate active or deleted status.
     - Provides interactive screens (`AR489S1` and `AR489S2`) for input validation and group maintenance, with error handling for invalid inputs.
     - Supports navigation via function keys (F10 to switch screens, F3/F12 to exit).

No additional distinct use cases are explicitly implemented, as the program focuses on the single function of maintaining credit limit groups. The OCL program’s calls to `GSGENIEC`, `SCPROCP`, and `GSY2K` are preparatory steps (e.g., initialization, Y2K compliance) rather than separate use cases.

---

### Function Requirement Document

Below is a function requirement document for a non-interactive version of the **Maintain Customer Credit Limit Groups** use case, assuming inputs are provided programmatically (e.g., via function parameters) rather than through interactive screens. The document outlines the business requirements, process steps, and calculations concisely.



# Function Requirement Document: Maintain Customer Credit Limit Groups

## Purpose
The `MaintainCustomerCreditLimitGroups` function manages customer credit limit groups by adding, updating, or deleting customer assignments for a specified company and customer group, ensuring valid inputs and persisting changes to the database.

## Inputs
- **Company Number** (`cono`: 2-digit numeric): Identifies the company.
- **Customer Number** (`cust`: 6-digit numeric): Identifies the primary customer for the group.
- **Customer Array** (`ars`: Array of 25 6-digit numeric): List of customer numbers in the group.
- **Delete Code** (`del`: 1-character): Status of the group record ('A' for active, other values for deletion).
- **System Control Key** (`control_key`: 2-character): Key for accessing system control data (e.g., '00').

## Outputs
- **Success Indicator**: Boolean indicating successful processing.
- **Error Message**: String describing any validation or processing errors (e.g., "INVALID COMPANY NUMBER ENTERED").
- **Updated Group Data**: Updated customer array and delete code if applicable.

## Business Rules
1. **Company Validation**:
   - The company number (`cono`) must exist in the `ARCONT` file.
   - If invalid, return error: "INVALID COMPANY NUMBER ENTERED".
2. **Customer Validation**:
   - The primary customer number (`cust`) must be non-zero.
   - If zero, return error: "INVALID CUSTOMER".
3. **Group Management**:
   - A group is uniquely identified by a composite key (`clgrky`: `cono` + `cust`).
   - The customer array (`ars`) can hold up to 25 customer numbers.
   - The delete code (`del`) indicates the group’s status ('A' for active, other values for deletion).
4. **Data Persistence**:
   - Add a new group record if no existing record is found for `clgrky`.
   - Update an existing group record if found.
   - Ensure all changes are saved to the `ARCLGR` file.

## Process Steps
1. **Initialize**:
   - Set default values for `cono` and `cust` to 0 if not provided.
2. **Validate System Control**:
   - Read the `GSCONT` file using `control_key` (e.g., '00').
   - If found and `gxcono` is non-zero, set `cono` to `gxcono`.
3. **Validate Company**:
   - Read the `ARCONT` file using `cono`.
   - If not found, return error: "INVALID COMPANY NUMBER ENTERED".
4. **Validate Customer**:
   - Check if `cust` is non-zero.
   - If zero, return error: "INVALID CUSTOMER".
5. **Process Group**:
   - Form composite key `clgrky` (`cono` + `cust`).
   - Read the `ARCLGR` file using `clgrky`:
     - If not found, create a new record with `del = 'A'`, `cono`, `cust`, and `ars`.
     - If found, update the existing record with provided `del` and `ars`.
   - Write changes to the `ARCLGR` file.
6. **Return Results**:
   - Return success indicator and any error messages.

## Calculations
- **Composite Key Creation**:
  - `clgrky` = Concatenate(`cono`, `cust`) as a 10-character field (2 bytes for `cono`, 6 bytes for `cust`).
- **Array Processing**:
  - Iterate through the `ars` array (25 elements) to validate or update customer numbers.
  - Ensure each `ars` element is a valid 6-digit numeric value.
- **Record Status**:
  - Set `del = 'A'` for new records; use provided `del` for updates.

## Database Files
- **ARCLGR** (Update File):
  - Stores group data: `cgdel` (1 byte), `cgcono` (2 bytes), `cgcust` (6 bytes), `arc` (150 bytes for 25 customers).
- **ARCONT** (Input File):
  - Stores company data: `arname` (30 bytes).
- **GSCONT** (Input File):
  - Stores system control data: `gxdel` (1 byte), `gxcono` (2 bytes).

## Error Handling
- Return descriptive error messages for invalid inputs (company or customer).
- Ensure database operations (read/write) handle record-not-found conditions gracefully.

## Assumptions
- Inputs are provided programmatically (e.g., via API or batch process).
- No interactive screen processing is required.
- The `GSCONT` file provides default company data if needed.



---

### Notes
- The function requirement document assumes a non-interactive implementation, replacing the interactive workstation screens (`AR489S1`, `AR489S2`) with direct input parameters.
- The business rules and process steps are derived from the RPGLE program’s logic, focusing on validation, data retrieval, and persistence.
- No external programs are called in the non-interactive version, as the OCL program’s calls (`GSGENIEC`, `SCPROCP`, `GSY2K`) are assumed to be handled externally or unnecessary in a programmatic context.

If you need further refinements, such as specific input formats, additional calculations, or a different structure for the requirements, please let me know!