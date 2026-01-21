The call stack consists of two programs: the RPGLE program `SA717P` and the CLP program `SA717C`, which work together to build a sales analysis file based on user input. Below, I will identify the use cases implemented by this call stack and then provide a concise function requirement document for a non-interactive version of the primary use case, focusing on business requirements and calculations.

---

### Use Cases Implemented by the Call Stack

The call stack implements the following use cases, derived from the functionality of `SA717P` and `SA717C`:

1. **Interactive Sales Analysis Report Generation**:
   - **Description**: Allows a user to interactively specify a date range and data type ('S' for sales or 'P' for product moves) via a display file (`SLS1` format), validates the input, constructs a query selection string, and generates a sales analysis report by calling `SA717C` to process and query the data.
   - **Inputs**: 
     - Group identifier (`p$fgrp`: 'G' or 'Z').
     - Transaction from date (`c$trfd`).
     - Transaction to date (`c$trtd`).
     - Data type (`c$type`: 'S' or 'P').
   - **Process**: 
     - Validates dates and data type, builds a query string, and calls `SA717C` to join files and run the `SA717Q` query.
   - **Output**: A sales analysis report generated via the `SA717Q` query, stored in `SA717QRYF`.
   - **Key Features**:
     - Interactive screen input with error handling for invalid dates or data types.
     - Dynamic file overrides based on group (`G` or `Z`).
     - Message subfile for error display.
     - Support for sales or product movement data.

2. **Field Prompting for User Input**:
   - **Description**: Provides field-level prompting when the user presses F04, allowing them to get assistance with input fields on the `SLS1` screen.
   - **Inputs**: Cursor location (`csrloc`) to determine the field being prompted.
   - **Process**: Captures cursor position for return after prompting (minimal logic implemented, likely a placeholder for a window or help function).
   - **Output**: Updated cursor position for the next screen display.
   - **Key Features**:
     - Supports user interaction for field-level assistance.
     - Limited implementation, suggesting future expansion.

3. **Batch Sales Analysis File Creation**:
   - **Description**: Processes sales or product movement data in a batch mode by joining multiple files (`SA5FILD`/`SA5MOVD`, `SA5SHP`, `SHIPTO`) based on a query selection string, creating a temporary work file, and running a predefined query (`SA717Q`) to generate the sales analysis report.
   - **Inputs** (from `SA717P` to `SA717C`):
     - Group identifier (`&P$FGRP`: 'G' or 'Z').
     - Query selection string (`&QRYSLT`).
     - Data type (`&P$TYPE`: 'S' or 'P').
   - **Process**: 
     - Selects appropriate files based on group and data type.
     - Copies query format file to `QTEMP`.
     - Joins files using `OPNQRYF`, applies the query selection, and outputs results to `SA717QRYF`.
     - Runs the `SA717Q` query to produce the report.
   - **Output**: Sales analysis report in `SA717QRYF`.
   - **Key Features**:
     - Dynamic file selection based on group and data type.
     - Multi-file join with outer join support (`JDFTVAL(*YES)`).
     - Error handling for file copy operations.

---

### Function Requirement Document: Batch Sales Analysis File Creation

This function requirement document assumes the primary use case (Batch Sales Analysis File Creation) is implemented as a non-interactive function that takes all required inputs programmatically, rather than through a screen. The document focuses on business requirements, process steps, and necessary calculations.

<xaiArtifact artifact_id="18acc830-4665-4106-b6f1-f94118a8eb2a" artifact_version_id="b5d7cb97-21bf-4fc5-8976-3b365221527a" title="Batch_Sales_Analysis_Function_Requirements.md" contentType="text/markdown">

# Function Requirement Document: Batch Sales Analysis File Creation

## Overview
The Batch Sales Analysis File Creation function generates a sales analysis report by processing sales or product movement data based on provided inputs. It joins multiple database files, applies a query selection string, and produces a report using a predefined query (`SA717Q`). The function operates non-interactively, taking all inputs as parameters.

## Business Requirements
1. **Input Parameters**:
   - **Group Identifier** (`group_id`, 1 character): Specifies the dataset group ('G' for production, 'Z' for development).
   - **Query Selection String** (`query_selection`, 1024 characters): Defines the data filter (e.g., excluding deleted records, specific freight/carrier codes, date range).
   - **Data Type** (`data_type`, 1 character): Specifies the data to process ('S' for sales, 'P' for product moves).
   - **Transaction From Date** (`tran_from_date`, 8 digits, CCYYMMDD): Start date of the transaction range.
   - **Transaction To Date** (`tran_to_date`, 8 digits, CCYYMMDD): End date of the transaction range.

2. **Output**:
   - A sales analysis report stored in the `SA717QRYF` file, accessible via the library list (`*LIBL`).
   - The report is generated by the `SA717Q` query in the `GSSQRY` library.

3. **Business Rules**:
   - The function must support two datasets:
     - `G` group: Uses files in the `DATA` library (`GSA5FILD`, `GSA5SHP`, `GSHIPTO`).
     - `Z` group: Uses files in the `DATADEV` library (`ZSA5FILD`, `ZSA5SHP`, `ZSHIPTO`).
   - Data type determines the primary file:
     - `S`: Uses `SA5FILD` (sales data).
     - `P`: Uses `SA5MOVD` (product movement data).
   - The query selection string must include conditions to:
     - Exclude deleted records (`SADEL *NE "D"`).
     - Filter by freight codes (`SHFRCD *NE "C"`) and carrier codes (`SHCACD *EQ %VALUES("TT" "RC" "MC" "SL")`).
     - Apply a date range on `SAIND8` if provided.
   - Dates must be valid (CCYYMMDD format) and `tran_from_date` must not be greater than `tran_to_date`.
   - Joins must match records across files on company (`SACO`/`SHCO`/`CSCO`), customer (`SACUST`/`SHCUST`/`CSCUST`), invoice (`SAINV`/`SHINV`), order (`SAORD`/`SHORD`), and ship-to (`SASHIP`/`CSSHIP`).
   - Use outer joins (`JDFTVAL(*YES)`) to include records with missing join matches.
   - Overwrite the output file `SA717QRYF` with each execution.

## Process Steps
1. **Validate Inputs**:
   - Ensure `group_id` is 'G' or 'Z'. If invalid, return error code `ERR_INVALID_GROUP`.
   - Ensure `data_type` is 'S' or 'P'. If invalid, return error code `ERR_INVALID_TYPE`.
   - Validate `tran_from_date` and `tran_to_date` using a date validation routine (e.g., equivalent to `GSDTEDIT`).
     - Dates must be in CCYYMMDD format.
     - If `tran_from_date` > `tran_to_date`, return error code `ERR_INVALID_DATE_RANGE`.
   - Verify `query_selection` is non-empty and properly formatted for `OPNQRYF`.

2. **Determine File Names**:
   - Set primary file (`file01`):
     - If `data_type` = 'S', `file01` = `group_id` + `SA5FILD` (e.g., `GSA5FILD`).
     - If `data_type` = 'P', `file01` = `group_id` + `SA5MOVD` (e.g., `GSA5MOVD`).
   - Set secondary file (`file02`): `group_id` + `SA5SHP` (e.g., `GSA5SHP`).
   - Set tertiary file (`file03`): `group_id` + `SHIPTO` (e.g., `GSHIPTO`).

3. **Copy Query Format File**:
   - Copy `SA717QF` to `QTEMP/SA717QF`:
     - If `group_id` = 'G', source from `DATA/SA717QF`.
     - If `group_id` = 'Z', source from `DATADEV/SA717QF`.
   - Use `MBROPT(*REPLACE)` and `CRTFILE(*YES)`.
   - Handle copy errors (e.g., `CPF2817`) gracefully.

4. **Execute Query**:
   - Override `SA717QF` to `file01` with `SHARE(*YES)`.
   - Perform an open query (`OPNQRYF`) joining `file01`, `file02`, and `file03`:
     - Use `query_selection` for filtering.
     - Sort by `SACO` (company), `SACUST` (customer), `SASHIP` (ship-to).
     - Apply join conditions (see Business Rules).
     - Output to format `SA717QF` with `JDFTVAL(*YES)`.
   - Copy query results to `SA717QRYF` using `CPYFRMQRYF` with `MBROPT(*REPLACE)`.

5. **Run Report Query**:
   - Execute the `GSSQRY/SA717Q` query to process `SA717QRYF` and generate the sales analysis report.

6. **Cleanup**:
   - Close the open query file.
   - Delete all file overrides.

## Calculations
- **Date Validation**:
  - Use a date validation routine (e.g., equivalent to `GSDTEDIT`) to convert and validate `tran_from_date` and `tran_to_date` from CCYYMMDD format.
  - Ensure `tran_from_date` ≤ `tran_to_date` to form a valid range.
- **Query Selection String**:
  - If `tran_from_date` and `tran_to_date` are non-zero, append a date range condition to `query_selection`: `SAIND8 *EQ %RANGE(tran_from_date tran_to_date)`.
  - Combine with static conditions: `(SADEL *NE "D") *AND (SHFRCD *NE "C") *AND (SHCACD *EQ %VALUES("TT" "RC" "MC" "SL"))`.
- **File Name Construction**:
  - Concatenate `group_id` with file suffixes (`SA5FILD`, `SA5MOVD`, `SA5SHP`, `SHIPTO`) to form file names.

## Error Handling
- Return error codes for invalid inputs (`ERR_INVALID_GROUP`, `ERR_INVALID_TYPE`, `ERR_INVALID_DATE_RANGE`).
- Handle file copy errors (`CPF2817`) by logging and continuing execution.
- Ensure cleanup (file close, override deletion) even if errors occur.

## Dependencies
- **Files**:
  - `SA717QF` (format file in `DATA` or `DATADEV`).
  - `GSA5FILD`, `ZSA5FILD`, `GSA5MOVD`, `ZSA5MOVD` (primary data files).
  - `GSA5SHP`, `ZSA5SHP` (shipment files).
  - `GSHIPTO`, `ZSHIPTO` (ship-to files).
  - `SA717QRYF` (output file in `*LIBL`).
- **Query**: `GSSQRY/SA717Q`.
- **System APIs**: `CPYF`, `OPNQRYF`, `CPYFRMQRYF`, `RUNQRY`, `CLOF`, `DLTOVR`.

## Assumptions
- The `SA717Q` query is predefined and correctly configured to process `SA717QRYF`.
- Input dates are in CCYYMMDD format.
- The library list (`*LIBL`) contains the necessary libraries for output files.

</xaiArtifact>

---

### Notes
- The primary use case (Batch Sales Analysis File Creation) was chosen for the function requirement document because it encapsulates the core functionality of `SA717C` and the data processing initiated by `SA717P`. The interactive use case was not selected, as the requirement specifies a non-interactive function.
- The field prompting use case is minor and underdeveloped in the original code, so it was not prioritized for the function requirements.
- The document is concise, focusing on business requirements and necessary calculations, while maintaining technical accuracy for the IBM i environment. If you need a function requirement document for another use case or additional details, let me know!