The provided RPGLE program, `AR810U.rpgle`, is an AR (Accounts Receivable) Invoice Inquiry program written in free-format RPGLE for an IBM i system. It displays invoice details for a given company and customer in a subfile (a scrolling list on a workstation display). Below is an explanation of the process steps, followed by the external programs called and the tables (files) used.

---

### Process Steps of the AR810U Program

1. **Program Initialization (OneTime Subroutine)**:
   - **Check Genie Environment**: The program calls `GSGENIE2C` to determine if it’s running in the Genie environment (a web-based interface). If `genievar` is not 'YES', the program terminates immediately.
   - **Set Default Company**: Retrieves the default company (`gxcono`) from the `GSCONT` file. If found and non-zero, sets the company code (`cco`) and positions the cursor on the customer field (`*in11 = *ON`). Otherwise, positions the cursor on the company field (`*in11 = *OFF`).
   - **Set Default Open Invoices**: Sets the `cOPEN` flag to 'O' to display only open invoices by default.

2. **Main Processing Loop**:
   - The program enters a `DoW` (Do While) loop that continues until the `EXIT` indicator is turned on (e.g., user presses a function key to exit).
   - **Clear Subfile**: Initializes the subfile control indicators (`SFLDSP`, `SFLCLR`), clears subfile variables (`MORE`, `sftermI`, `sfamt`, `invdatI`, `invdt8I`), and writes the subfile control record (`SFLCTL1`) to clear the subfile.
   - **Check Subfile Display**: If the subfile is ready to display (`SFLDSP = *On`), the program proceeds to retrieve company and customer data.

3. **Retrieve Company and Customer Data (GetCo Subroutine)**:
   - **Retrieve AR Control Data**: Reads the `ARCONT` file using the company code (`cco`). If not found, initializes aging bucket limits (`aclmt1`, `aclmt2`, `aclmt3`, `aclmt4`) to zero.
   - **Set Aging Bucket Headings**: Defines aging bucket descriptions (`xAGE1` to `xAGE5`) based on the AR control limits (`aclmt1` to `aclmt4`). For example:
     - `xAGE1 = 'CURRENT'`
     - `xAGE2 = ' 1 - ' + aclmt1`
     - Assigns these to subfile fields (`cage1` to `cage5`).
   - **Retrieve Customer Master**: Reads the `ARCUST` file using the company (`cco`) and customer (`ccust`) keys. If not found, sets customer details (e.g., `arname`, `aradr1`, etc.) to indicate "Customer Master not found" or blanks.

4. **Load Subfile (Main Loop)**:
   - **Read AR Detail Records**: Sets the lower limit (`SetLL`) on the `ARDETL` file using company (`cco`) and customer (`ccust`) keys, then reads records in a `DoW` loop using `ReadE`.
   - **Filter Records**:
     - Skips deleted (`adDEL = 'D'`) or inactive (`adDEL = 'I'`) records.
     - Skips open payment (`ADTYPE = 'P'`) or adjustment (`ADTYPE = 'J'`) records when `cOPEN = 'O'`.
     - Skips fully paid invoices (`ADTYPE = 'I'` and `ADPART + ADPAY = ADAMT`) when `cOPEN = 'O'`.
   - **Populate Subfile (SFfill Subroutine)**: For valid records, fills subfile fields:
     - **Convert Dates**: Converts transaction (`adtym8`) and due dates (`addud8`) from `YYYYMMDD` to `MM/DD/YYYY` format for display (`trdat`, `dudat`).
     - **Handle Record Types**:
       - **Invoice ('I')**:
         - Sets `sftype = ' INVOICE'` and `sftypeI = '1'`.
         - Calculates open amount (`sfamt = adamt - adpart - adpay`) if `cOPEN = 'O'`, else uses `adamt`.
         - Retrieves terms description from `GSTABL` using the terms code (`adterm`). If not found, uses the terms code as the description.
         - Sets invoice date (`invdat`, `invdatI`) to `trdat`.
       - **Adjustment ('J')**:
         - Sets `sftype = 'ADJUST '`.
         - Uses `adamt` as `sfamt`.
         - Calls `GetDays` to calculate days between transaction and due dates.
       - **Payment ('P')**:
         - Sets `sftype = 'PAYMENT'`.
         - Uses `adamt` as `sfamt`.
         - Calls `GetDays` to calculate days.
     - **Aging Description**: Assigns the aging bucket description (`sfage`) based on the `adage` value (1 to 5, mapping to `cage1` to `cage5`).
   - **Write Subfile Record**: Increments the relative record number (`RRN`) and writes the subfile record (`SFL1`). If `RRN` exceeds 9990, sets `MORE` to indicate not all records are displayed and exits the loop.

5. **Display Subfile**:
   - Sets `SFLDSP = *On` and executes `ExFmt` to display the subfile control record (`SFLCTL1`) on the workstation screen, allowing user interaction (e.g., scrolling, selecting records, or exiting).

6. **Repeat or Exit**:
   - After displaying the subfile, the program loops back to check if `EXIT` is turned on. If not, it repeats the process (e.g., clears subfile, reloads data if user changes input). If `EXIT` is on, the program sets `*InLr = *On` and terminates.

---

### External Programs Called

1. **GSGENIE2C**:
   - **Purpose**: Checks if the program is running in the Genie environment by setting the `genievar` variable to 'YES' or 'NO'.
   - **Parameters**:
     - `genievar` (char(3)): Output parameter indicating Genie environment status.

2. **GSDTCLC2**:
   - **Purpose**: Calculates the difference in days between two dates.
   - **Parameters**:
     - `p#dat1` (Packed(8:0)): First date in `CCYYMMDD` format.
     - `p#dat2` (Packed(8:0)): Second date in `CCYYMMDD` format.
     - `p#fmt` (char(1)): Format of difference ('D' for days).
     - `p#diff` (Packed(10:2)): Output for the calculated difference.
     - `p#err` (char(1)): Error flag.

---

### Tables (Files) Used

1. **AR810UD**:
   - **Type**: Workstation file (display file).
   - **Usage**: External (`*Ext`), handled by Profound UI (`PROFOUNDUI(HANDLER)`).
   - **Description**: Contains the subfile (`SFL1`) and control record (`SFLCTL1`) for displaying invoice inquiry data on the screen.

2. **ARDETL**:
   - **Type**: Disk file (database file).
   - **Usage**: Input, keyed.
   - **Description**: Stores AR transaction details (e.g., invoices, payments, adjustments). Keyed by company (`cco`) and customer (`ccust`). Fields include `adDEL`, `ADTYPE`, `ADPART`, `ADPAY`, `ADAMT`, `adtym8`, `addud8`, `adage`, `adterm`.

3. **GSCONT**:
   - **Type**: Disk file.
   - **Usage**: Input, keyed.
   - **Description**: Stores general system control data, including the default company code (`gxcono`). Keyed by `gskey`.

4. **ARCONT**:
   - **Type**: Disk file.
   - **Usage**: Input, keyed.
   - **Description**: Stores AR control data, including aging bucket limits (`aclmt1`, `aclmt2`, `aclmt3`, `aclmt4`). Keyed by company (`cco`).

5. **ARCUST**:
   - **Type**: Disk file.
   - **Usage**: Input, keyed.
   - **Description**: Stores customer master data (e.g., name, address). Keyed by company (`cco`) and customer (`ccust`). Fields include `arname`, `aradr1`, `aradr2`, `aradr3`, `aradr4`, `arzip5`.

6. **GSTABL**:
   - **Type**: Disk file.
   - **Usage**: Input, keyed.
   - **Description**: Stores general system table data, including terms descriptions (`TBDESC`). Keyed by a composite key (`ARTERM` + terms code).

---

### Summary

The `AR810U` program is designed to display AR invoice details for a specific company and customer in a subfile, filtering out deleted records and optionally showing only open invoices. It retrieves data from multiple files, calculates aging buckets, and formats dates for display. The program interacts with the user through a workstation display and terminates when the user exits. It relies on two external programs (`GSGENIE2C` and `GSDTCLC2`) for environment checks and date calculations, and uses five database files (`ARDETL`, `GSCONT`, `ARCONT`, `ARCUST`, `GSTABL`) for data retrieval, along with the `AR810UD` display file for user interaction.