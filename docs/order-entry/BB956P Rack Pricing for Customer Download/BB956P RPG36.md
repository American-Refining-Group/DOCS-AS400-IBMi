The provided document is an RPG III (or RPG/36) program, `BB956P.rpg36.txt`, called from the OCL program `BB956P.ocl36.txt`. This program handles the core logic for updating rack pricing data, including validating user input from a screen and preparing data for further processing. Below is an explanation of the process steps, business rules, tables (files) used, and any external programs called.

---

### **Process Steps of the RPG Program `BB956P`**

The `BB956P` program is an interactive program that displays a screen for user input, validates the input, and sets up data for downstream rack pricing updates. Here’s a step-by-step breakdown of the process:

1. **File and Data Structure Definitions**:
   - **Files**:
     - `SCREEN`: A workstation file (display file) with a record length of 1024 bytes for user interaction.
     - `BICONT`: Input file (256 bytes, 2-byte key, direct access) for company control data.
     - `ARCUST`: Input file (384 bytes, 8-byte key, direct access) for customer data.
     - `BB956S`: Input file (9 bytes, 8-byte key, direct access) containing the customer list (created by `BB9541`).
   - **Data Structures**:
     - `MSG`: Array of 6 elements, each 40 characters, for error messages (e.g., "INVALID DATE ENTERED").
     - `DCO`: Array of 10 elements, each 35 characters, for storing customer names.
     - `DCS`: Array of 10 elements, each 6 digits, for storing customer numbers.
     - `UDS`: User data structure for local variables, including:
       - `KYDATE` (positions 105-110): Date input.
       - `KYMO` (positions 105-106): Month portion of the date.
       - `KYYR` (positions 109-110): Year portion of the date.
       - `KYCO` (positions 114-115): Company number.
       - `KYJOBQ` (position 120): Job queue code.
       - `KYCANC` (positions 129-134): Cancel flag.
       - `KYCUST` (positions 201-206): Customer number.
       - `Y2KCEN` (positions 509-510): Century for Y2K handling (e.g., 19 for 1900s).
       - `Y2KCMP` (positions 511-512): Comparison year for Y2K logic.

2. **Initialization**:
   - Clears the `MSG40` field (40 characters) to blanks (`MOVEL*BLANKS MSG40`).
   - Turns off indicators `50`, `51`, `52`, `53`, `54`, `55`, `56`, `81`, and `90` to reset the program state.

3. **Handle Cancel Condition**:
   - `C KG SETOF 0109`:
     - If the `KG` indicator is on (likely set by the workstation file when the user presses a cancel key), clears indicators `01` and `09`.
   - `C KG MOVEL'CANCEL' KYCANC`:
     - Sets the `KYCANC` field to `'CANCEL'`.
   - `C KG SETON LR`:
     - Sets the Last Record (`LR`) indicator to terminate the program.
   - `C KG GOTO END`:
     - Jumps to the `END` tag to exit the program.

4. **One-Time Processing (`ONETIM` Subroutine)**:
   - Executed when indicator `09` is on (likely at program startup).
   - **Purpose**: Populates the `DCO` and `DCS` arrays with customer data for display on the screen.
   - **Steps**:
     - Clears the `DCO` array to blanks (`MOVEL*BLANKS DCO`).
     - Initializes index `X` to 1 and sets `SCUST` to 10000000 (likely a high starting value for customer numbers).
     - Sets the lower limit for `BB956S` using `SCUST` (`SETLLBB956S`).
     - Reads records from `BB956S` in a loop (`READ BB956S 10`):
       - If end-of-file is reached (`10 = 1`), jumps to `ENDONE`.
       - If the record is marked for deletion (`PSDEL = 'D'`), skips to the next record (`GOTO AGNONE`).
       - Moves the customer number (`PSCUST`) to the `DCS` array at index `X`.
       - Uses `PSKEY` (company + customer number) to chain to `ARCUST` and retrieve the customer name (`ARNAME`).
       - If the `ARCUST` record is found (`98 = 0`), moves `ARNAME` to the `DCO` array at index `X`.
       - Increments `X` and continues until 10 customers are processed (`X COMP 10`) or no more records are available.
     - Sets indicator `81` to trigger screen output and ends the subroutine.

5. **Main Screen Processing (`S1` Subroutine)**:
   - Executed when indicator `01` is on (likely after a screen interaction).
   - **Purpose**: Validates user input from the screen and prepares data for further processing.
   - **Steps**:
     - **Validate Company Number**:
       - Chains `KYCO` to `BICONT` to verify the company number.
       - If not found (`97 = 1`), sets indicators `81` and `90`, moves error message `MSG,4` ("INVALID COMPANY NUMBER ENTERED") to `MSG40`, and jumps to `ENDS1`.
     - **Validate Customer Number**:
       - Performs a lookup of `KYCUST` in the `DCS` array.
       - If not found (`40 = 0`), sets indicators `81` and `90`, moves error message `MSG,2` ("INVALID CUSTOMER NUMBER ENTERED") to `MSG40`, and jumps to `ENDS1`.
     - **Validate Month**:
       - Checks if `KYMO` (month) is between 01 and 12.
       - If invalid (`KYMO < 01` or `KYMO > 12`), sets indicators `81` and `90`, moves error message `MSG,1` ("INVALID DATE ENTERED") to `MSG40`, and jumps to `ENDS1`.
     - **Date Processing (Y2K Handling)**:
       - Converts `KYDATE` to an 8-digit format by multiplying by 10000.01 and moving to `BEGDT` (6 digits).
       - Extracts the year portion (`BYY`) and compares it to `Y2KCMP` (likely a threshold year, e.g., 80 for 1980).
       - If `BYY >= Y2KCMP`, sets century (`BCN`) to `Y2KCEN` (e.g., 19 for 1900s).
       - Otherwise, adds 1 to `Y2KCEN` (e.g., 20 for 2000s).
       - Combines century and date into an 8-digit format (`BEGDT8`).
     - **Success Case**:
       - If all validations pass, sets indicator `U1` to indicate successful validation.
     - Ends the subroutine (`ENDS1`).

6. **Screen Output**:
   - `OSCREEN D 81`:
     - Displays the screen format `BB956PS1` when indicator `81` is on.
     - Outputs fields:
       - Screen title: "CUSTOMER RACK PRICING EXTRACT".
       - `KYDATE` (date) at position 46.
       - `KYCO` (company number) at position 48.
       - `KYCUST` (customer number) at position 54.
       - `DCO` (customer names array) at position 404.
       - `MSG40` (error message) at position 444.
       - `DCS` (customer numbers array, 1 to 10) at positions 450–504 (zero-suppressed).

7. **Program Termination**:
   - If the `ONETIM` subroutine is executed (`09 = 1`), the program jumps to `END`.
   - If the cancel condition is met (`KG = 1`), the program jumps to `END`.
   - The `END` tag marks the program’s termination point.

---

### **Business Rules**

The `BB956P` program enforces the following business rules:
1. **User Input Validation**:
   - **Company Number**: Must exist in the `BICONT` file. If invalid, displays "INVALID COMPANY NUMBER ENTERED".
   - **Customer Number**: Must exist in the `DCS` array (populated from `BB956S`). If invalid, displays "INVALID CUSTOMER NUMBER ENTERED".
   - **Month**: Must be between 01 and 12. If invalid, displays "INVALID DATE ENTERED".
   - **Date**: Processed with Y2K logic to determine the correct century (1900s or 2000s) based on a comparison year (`Y2KCMP`).

2. **Customer List Display**:
   - Displays up to 10 customer numbers (`DCS`) and names (`DCO`) from `BB956S` and `ARCUST`.
   - Skips deleted records (`PSDEL = 'D'`) in `BB956S`.

3. **Cancel Handling**:
   - If the user cancels (via a function key setting `KG`), sets `KYCANC` to `'CANCEL'` and exits.

4. **Error Handling**:
   - Displays error messages on the screen and prevents further processing if validation fails.
   - Uses indicator `81` to refresh the screen with error messages and `90` to indicate an error state.

5. **Purpose**:
   - Validates user input for company, customer, and date to ensure valid data for rack pricing updates.
   - Prepares a formatted date (`BEGDT8`) for downstream processing.
   - Sets up a list of customers for display and selection.

---

### **Tables (Files) Used**

The program interacts with the following files:
1. **Input Files**:
   - `BICONT`: Company control file containing company number (`BCCO`), name (`BCNAME`), delete flag (`BCDEL`), and rack price password (`BCRKPW`).
   - `ARCUST`: Customer file containing customer number (`ARCUST`), company number (`ARCO`), name (`ARNAME`), address (`ARADR1`), and delete flag (`ARDEL`).
   - `BB956S`: Customer list file (created by `BB9541`) containing company number (`PSCONO`), customer number (`PSCUST`), composite key (`PSKEY`), and delete flag (`PSDEL`).
2. **Workstation File**:
   - `SCREEN`: Display file for user interaction, using format `BB956PS1` to show input fields, customer list, and error messages.

---

### **External Programs Called**

The `BB956P` program does not explicitly call any external programs. It performs all processing within its own logic, relying on subroutines (`ONETIM` and `S1`) and file I/O operations.

---

### **Summary**

The `BB956P` RPG program is an interactive program that validates user input for rack pricing updates. It reads a customer list from `BB956S`, retrieves customer names from `ARCUST`, and validates company numbers against `BICONT`. The program displays up to 10 customers on a screen, checks the entered company, customer, and date (with Y2K handling), and sets indicator `U1` if validations pass. Error messages are displayed for invalid inputs, and the program supports a cancel option. It uses files `BICONT`, `ARCUST`, `BB956S`, and `SCREEN`, with no external program calls.