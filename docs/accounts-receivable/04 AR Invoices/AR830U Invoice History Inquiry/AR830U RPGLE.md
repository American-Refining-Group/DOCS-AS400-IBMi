The provided code is an RPGLE program (`AR830U.rpgle`), not an OCL program. It implements an AR (Accounts Receivable) Invoice History Inquiry for displaying invoice history records based on user-specified criteria. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used.

---

### **Process Steps of the AR830U RPGLE Program**

The program is designed to run in a Genie environment (a web-based interface for IBM i applications) and displays invoice history records in a subfile (a scrollable list on a workstation display). Here’s a step-by-step breakdown of its processing:

1. **Program Initialization**:
   - **Control Specifications**: The program uses free-format RPGLE with `Ctl-Opt DftActGrp(*No)`, indicating it runs in a named activation group for better resource management.
   - **File Declarations**: Declares:
     - `AR830UD`: A workstation file (display file) with a subfile (`SFL1`) for user interaction, handled by Profound UI (a web interface tool).
     - `ARHIST`, `GSCONT`, `ARCUST`, `GSTABL`: Input files (tables) for invoice history, company control, customer master, and table data, respectively, accessed with keys.
   - **Variable Declarations**: Defines variables for processing, including subfile record number (`RRN`), date calculations, and temporary fields for invoice data.
   - **Check Genie Environment**: Calls `GSGENIE2C` to retrieve the `genievar` variable. If `genievar` is not `'YES'`, the program terminates (sets `*InLr = *On` and returns), ensuring it only runs in the Genie interface.
   - **Default Company Retrieval**: Retrieves the default company number (`gxcono`) from the `GSCONT` file using key `gskey = *zero`. If found and valid, sets the company number (`cco`) and positions the cursor at the customer field (`*in11 = *ON`); otherwise, positions it at the company field.
   - **Set Default Date**: Initializes the invoice date filter (`cinvdat`) to today’s date minus 2 years (changed from 5 years per LMS01 modification on 8/8/24).
   - **Indicator Initialization**: Sets `REFRESH` and `EXIT` indicators to `*Off` and clears subfile display/control flags (`SFLDSP`, `SFLCLR`).

2. **Main Processing Loop** (`DoW EXIT = *Off`):
   - **Clear Subfile**:
     - Sets `SFLCLR = *On`, writes to the subfile control record (`SFLCTL1`) to clear it, then resets `SFLCLR = *Off`.
     - Resets subfile record number (`RRN = 0`) and clears `MORE` and `sftermI`.
   - **Retrieve Customer Information**:
     - If the subfile is to be displayed (`SFLDSP = *On`), retrieves customer details from `ARCUST` using keys `cco` (company) and `ccust` (customer).
     - If no customer record is found, sets default values (e.g., `arname = 'Customer Master not found'`).
   - **Convert Date Format**:
     - Converts the user-entered invoice date (`cinvdat`, MM/DD/YYYY) to `cinvcymd` (YYYYMMDD) for comparison with invoice history dates.
   - **Load Subfile with Invoice History**:
     - Sets the lower limit (`SetLL`) on `ARHIST` using keys `cco` and `ccust`.
     - Reads records (`ReadE`) from `ARHIST` matching the company and customer.
     - For each record:
       - **Skip Deleted Records**: If `ahDEL` is `'D'` or `'I'`, skips the record (`iter`).
       - **Date Filter**: If the invoice date (`AHIND8`, YYYYMMDD) is earlier than `cinvcymd`, skips the record.
       - **Populate Subfile Record**: Calls subroutine `SFfill` to format the record data.
       - **Write to Subfile**: Increments `RRN` and writes the record to `SFL1`.
       - **Limit Check**: If `RRN` exceeds 9990, sets `MORE = 'More history...not all included...'` and exits the read loop.
   - **Display Subfile**:
     - Sets `SFLDSP = *On` and executes `ExFmt SFLCTL1` to display the subfile to the user.
   - **Loop Continuation**: Continues the loop until the user exits (`EXIT = *On`).

3. **Subroutine: SFfill**:
   - Formats data for a subfile record based on the invoice type (`ahtype`):
     - **Invoice (ahtype = 'I')**:
       - Sets `sftype = ' INVOICE'` and `sftypeI = '1'`.
       - Retrieves terms code (`ahterm`) and looks up its description in `GSTABL` using key `ktbcode = '    ' + sftermI`.
       - If no description is found, uses `ktbcode`; otherwise, formats as `ktbcode + '-' + TBDESC`.
     - **Adjustment (ahtype = 'J')** or **Payment (ahtype = 'P')**:
       - Sets `sftype` to `'ADJUST '` or `'PAYMENT'`, respectively.
       - Calls `GetDays` subroutine to calculate the difference between transaction date (`ahtda8`) and due date (`ahdud8`).
   - **Date Conversion**:
     - Converts invoice date (`ahind8`), due date (`ahdud8`), and transaction date (`ahtda8`) from YYYYMMDD to MM/DD/YYYY format using `%date` and `%char`.
     - Uses `monitor` blocks to handle invalid dates, setting them to `*loval` if errors occur.

4. **Subroutine: GetDays**:
   - Calls external program `GSDTCLC2` to calculate the difference in days between transaction date (`ahtda8`) and due date (`ahdud8`).
   - Sets parameters: `p#dat1 = ahtda8`, `p#dat2 = ahdud8`, `p#fmt = 'D'` (days), and clears `p#diff` and `p#err`.
   - Stores the result in `sfdays = p#diff`.

5. **Program Termination**:
   - When the user exits (`EXIT = *On`), sets `*InLr = *On` and returns, ending the program.

---

### **External Programs Called**

The program calls two external programs:
1. **GSGENIE2C**:
   - Purpose: Retrieves the `genievar` variable to check if the program is running in the Genie environment.
   - Parameter: `genievar` (char(3), output, values `'YES'` or `'NO'`).
2. **GSDTCLC2**:
   - Purpose: Calculates the difference in days between two dates.
   - Parameters:
     - `p#dat1` (Packed(8:0), input, date 1 in CCYYMMDD format).
     - `p#dat2` (Packed(8:0), input, date 2 in CCYYMMDD format).
     - `p#fmt` (char(1), input, format `'D'` for days).
     - `p#diff` (Packed(10:2), output, date difference).
     - `p#err` (char(1), output, error flag).

---

### **Tables (Files) Used**

The program uses the following files (tables):
1. **AR830UD**:
   - Type: Workstation file (display file).
   - Usage: Output/Input for user interface.
   - Subfile: `SFL1` with relative record number (`RRN`).
   - Handler: Managed by Profound UI (`PROFOUNDUI(HANDLER)`).
2. **ARHIST**:
   - Type: Physical file (database table).
   - Usage: Input, keyed access.
   - Purpose: Stores accounts receivable invoice history records.
   - Keys: `cco` (company number), `ccust` (customer number).
3. **GSCONT**:
   - Type: Physical file.
   - Usage: Input, keyed access.
   - Purpose: Stores company control information, including default company number (`gxcono`).
   - Key: `gskey` (likely a control key, set to `*zero`).
4. **ARCUST**:
   - Type: Physical file.
   - Usage: Input, keyed access.
   - Purpose: Stores customer master data (e.g., name, address).
   - Keys: `cco` (company number), `ccust` (customer number).
5. **GSTABL**:
   - Type: Physical file.
   - Usage: Input, keyed access.
   - Purpose: Stores table data, including terms descriptions (`TBDESC`).
   - Keys: Table code (`'ARTERM'`), terms code (`ktbcode`).

---

### **Summary**

The `AR830U` RPGLE program is an interactive inquiry tool that displays invoice history records for a specified company and customer, filtered by date (defaulting to the last 2 years). It uses a subfile to present data, retrieves additional details from customer and terms tables, and calculates date differences for adjustments and payments. The program ensures it runs in a Genie environment and leverages external programs for specific tasks.

If you need further clarification or details about specific parts of the program, let me know!