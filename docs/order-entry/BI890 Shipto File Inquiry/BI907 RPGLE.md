The `BI907.rpgle.txt` is an RPGLE (RPG IV) program named `BI907`, designed for the Bradford Order Entry / Invoices system on an IBM AS/400 (iSeries) platform. It provides an interactive interface for maintaining and inquiring about customer and ship-to product descriptions and freight terms, using a subfile (SFL) to display and manage records. The program is called by the CLP program `BI907AC`, which sets up a temporary work file (`BI907W`). Below, I explain the process steps, business rules, tables used, and external programs called, based on the provided RPGLE source code.

### Process Steps

The program follows a structured flow to manage customer and ship-to product data, organized into subroutines and screen formats. Due to the truncation of the source code, some details are inferred from the provided sections, context from `BI907AC`, and related programs like `BI890`.

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives five input parameters (passed from `BI907AC`):
     - `p$cono` (2 characters): Company number.
     - `p$cust` (6 characters): Customer number.
     - `p$ship` (3 characters): Ship-to number.
     - `p$mode` (3 characters): Operation mode ('INQ' for inquiry, 'MNT' for maintenance).
     - `p$fgrp` (1 character): File group identifier ('G' or 'Z') for file overrides.
   - **File Setup**: Opens input files (`gstabl`, `bicont`, `gsprod`, `shipto`, `arcust`), input/output file (`arcupr`, `bi907w`), and output file (`arcuphs`) with `usropn`.
   - **Key Lists**: Defines key lists for file access:
     - `kls1s1`, `kls1r1`: Company, customer, ship-to (`c1cono`, `c1cust`, `c1ship`).
     - `kls1s2`, `klsfl1`: Company, customer, ship-to, product (`c1prod`, `s1prod`).
     - `kls1s3`: Company, customer (`c1cono`, `s3cust`).
     - `kls1s4`: Company, customer, ship-to (`c1cono`, `s3cust`, `s3ship`).
     - `klbi907`, `kl2bi907`: Company, customer, ship-to, product, container type (`cpcono`, `cpcust`, `cpship`, `cpprod`, `w$cnty`).
     - `klsfl2`: Company, product (`c1cono`, `s1prod`).
     - `klcust`: Company, customer (`c1cono`, `c1cust`).
     - `klship`: Company, customer, ship-to (`c1cono`, `c1cust`, `c1ship`).
     - `klctyp`: Container type (`k$ctyp = 'CNTRTY'`).
   - **Mode Initialization**:
     - Chains to `arcupr` using `kls1s1` to check if a record exists.
     - If found (`*in99 = *off`), sets update mode (`*in87 = *on`, `s1updt = *on`, `s1f10d = 'F10=Update Mode'`, `c1mode = 'Update Mode'`).
     - If not found and not in inquiry mode (`*in71 = *off`), sets all mode (`s1updt = *on`, `*in87 = *off`, `s1f10d = 'F10=All Mode'`, `c1mode = 'All Mode'`).
     - If in inquiry mode, sets add mode (`s1updt = *on`, `*in87 = *off`, `s1f10d = 'F10=Add Mode'`, `c1mode = 'All Mode'`).
     - Sets `updmode = *on` for default update mode.
   - **Timestamp**: Captures current date/time (`t#time`) and formats it into `t#cymd` (CCYYMMDD).
   - **Data Structures**: Defines structures for time/date conversion, message handling, and display file (`dspf_ds`), but `wkds01` (for `gbbcfsh`) is commented out, suggesting it may not be used.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides for `gstabl`, `arcupr`, `bicont`, `gsprod`, `arcust`, `shipto`, and `arcuphs` based on `p$fgrp` ('G' for `g*` files, 'Z' for `z*` files) using the `QCMDEXC` API.
   - Opens all files with `usropn` for input (`gstabl`, `bicont`, `gsprod`, `shipto`, `arcust`), input/output (`arcupr`, `bi907w`), and output (`arcuphs`).

3. **Process Subfile** (inferred from context and partial code):
   - Displays a subfile (`sfl1`) via the display file `bi907d`, controlled by `sflctl1`.
   - Populates subfile with `arcupr` records (customer pricing data) for the specified company, customer, and ship-to.
   - Supports three modes:
     - **Update Mode**: Displays existing `arcupr` records for editing (`*in87 = *on`).
     - **All Mode**: Displays all relevant product records (`*in87 = *off`).
     - **Add Mode**: Allows adding new `arcupr` records (inquiry mode off, `*in71 = *off`).
   - Handles user interactions via function keys:
     - **F03**: Exit.
     - **F04**: Prompt for valid codes (e.g., product, freight code).
     - **F05**: Refresh.
     - **F10**: Toggle between Update/Add/All modes.
     - **F12**: Cancel/return.
     - **F14-F16**, **F20**: Additional functionality (e.g., delete, copy, specific actions).
   - Validates input fields (e.g., `c1prod`, `s1prod`) against `gsprod` and freight codes against `gstabl`.

4. **Data Maintenance**:
   - **Add Records**: In Add Mode, ensures at least one product code is entered (`err(01)`).
   - **Update Records**: Updates `arcupr` fields like `cpglcd` (gallons to bill), `cpcstk` (stock number), `cpfrcd` (freight code), `cpsfrt` (separate freight), `cpcafr` (calculate freight).
   - **Delete/Reactivate**: Marks records as deleted (`err(10)`) or reactivates them (`err(11)`).
   - **Copy Records**: Supports copying records (`err(12)`).
   - Writes to `bi907w` (temporary work file) and `arcuphs` (history file) as needed.

5. **Freight Code Validation (JB01, JB02)**:
   - Validates freight codes (`cpfrcd`) in `gstabl` (keyed by `k$frty = 'BBFRCD'`).
   - Special cases:
     - **JB01 (05/27/2020)**: For freight code `C` (freight collect, non-Bradford location like Anchor), validates `cpcafr` (calculate freight) as ' ', 'Y', or 'N' (`err(08)`).
     - **JB02 (01/17/2022)**: For freight code `C`, validates `cpsfrt` (separate freight) as ' ', 'Y', or 'N' (`err(07)`). For freight code `A` (freight collect with service fee, arranged by ARG but billed by carrier), requires `cpsfrt = 'Y'` (`err(09)`), with a $100 service fee.

6. **Message Handling**:
   - Uses `QMHSNDPM` to send error messages (e.g., `err(01-12)`) to the message subfile (`msgctl`).
   - Clears messages with `QMHRMVPM`.
   - Displays errors like "Invalid Code, Valid Codes are: 'G' ' '" (`err(04)`), "Invalid Freight Code" (`err(05)`), etc.

7. **Program Termination**:
   - Closes all files (`close *all`).
   - Sets `*inlr = *on` and returns.

### Business Rules

1. **Operation Modes**:
   - **Inquiry Mode (`p$mode = 'INQ'`)**: Protects fields (`*in70`, `*in71`) for read-only access.
   - **Maintenance Mode (`p$mode = 'MNT'`)**: Allows adding, updating, deleting, or reactivating `arcupr` records.
   - **Update Mode**: Edits existing records (`*in87 = *on`).
   - **All Mode**: Displays all relevant records (`*in87 = *off`).
   - **Add Mode**: Adds new records, requiring at least one product code (`err(01)`).

2. **Validation**:
   - Validates company (`c1cono`) against `bicont`.
   - Validates customer (`c1cust`) against `arcust`.
   - Validates ship-to (`c1ship`) against `shipto`.
   - Validates product codes (`c1prod`, `s1prod`) against `gsprod`.
   - Validates freight codes (`cpfrcd`) in `gstabl` (`err(05)`).
   - Validates `cpglcd` (gallons to bill) as 'G' (gross), ' ' (blank), or 'N' (net) (`err(04)`).
   - For freight code `C`:
     - `cpsfrt` must be ' ', 'Y', or 'N' (`err(07)`, JB02).
     - `cpcafr` must be ' ', 'Y', or 'N' (`err(08)`, JB01).
   - For freight code `A`, `cpsfrt` must be 'Y' (freight collect with $100 service fee, JB02) (`err(09)`).
   - Prevents adding records if they exist or are marked deleted (`err(02)`).

3. **Subfile Management**:
   - Displays up to 32 records (`pagsz1`) in the subfile (`sfl1`).
   - Supports navigation with roll keys (`PAGEDN`) and function keys (F03, F04, F05, F10, F12, etc.).
   - Uses `bi907w` for temporary storage and `arcuphs` for history.

4. **Error Handling**:
   - Displays errors in a message subfile (e.g., "Must Enter At Least One Product Code in ADD Mode", "Invalid Freight Code").
   - Supports record deletion (`err(10)`) and reactivation (`err(11)`).

5. **Freight Terms (JB01, JB02)**:
   - Handles special freight scenarios:
     - Non-Bradford location (e.g., Anchor) with freight code `C` (JB01).
     - Freight collect with a $100 service fee for code `A` (JB02).

### Tables (Files) Used

The program uses the following database files (tables):
1. **gstabl**: General system table (input, keyed, contains freight codes and container types).
2. **bicont**: Company/contract master file (input, keyed, validates company).
3. **gsprod**: Product master file (input, keyed, validates product codes).
4. **shipto**: Ship-to master file (input, keyed, validates ship-to).
5. **arcust**: Customer master file (input, keyed, validates customer).
6. **arcupr**: Customer pricing file (input/output, keyed by company, customer, ship-to, product, container type; stores `cpglcd`, `cpcstk`, `cpfrcd`, `cpsfrt`, `cpcafr`).
7. **arcuphs**: Customer pricing history file (output, keyed, stores historical data).
8. **bi907w**: Temporary work file (input/output, keyed, in `QTEMP`, managed by `BI907AC`).
9. **bi907d**: Display file (workstation file with subfile `sfl1` and formats `sflctl1`, `msgctl`, etc.).

### External Programs Called

The program calls the following external programs (system APIs):
1. **QCMDEXC**: Executes file override commands (`ovrdbf`) for `gstabl`, `arcupr`, `bicont`, `gsprod`, `arcust`, `shipto`, and `arcuphs`.
2. **QMHSNDPM**: Sends error messages to the program message queue.
3. **QMHRMVPM**: Clears messages from the program message queue.

### Summary

The `BI907` program is an interactive tool for maintaining and inquiring about customer and ship-to product descriptions and freight terms, called by `BI907AC`. It:
- Supports inquiry (`INQ`) and maintenance (`MNT`) modes with Update, All, and Add sub-modes.
- Manages `arcupr` records, validating product codes, freight codes, and freight terms.
- Handles special freight scenarios (JB01, JB02) for non-Bradford locations and service fees.
- Uses a subfile (`sfl1`) for displaying/editing records, with robust validation against `gstabl`, `gsprod`, `bicont`, `arcust`, and `shipto`.
- Writes to `bi907w` (temporary) and `arcuphs` (history) files.
- Displays errors via a message subfile and supports navigation with function keys.

The program integrates with eight database files and three system APIs, ensuring accurate maintenance of pricing and freight data. If you need further details or analysis of specific subroutines (not fully visible due to truncation), please provide additional context or code!

System: * Today's date and time is 03:22 PM EDT on Monday, September 08, 2025.