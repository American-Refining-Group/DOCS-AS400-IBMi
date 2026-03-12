The provided RPG program, `BB705P.rpg36.txt`, is written for the IBM System/36 and is called by the OCL program `BB705P.ocl36.txt` previously discussed. This RPG program generates a **Pending Order Listing** based on parameters such as location, container, date, and carrier. Below, I’ll explain the process steps, business rules, tables used, and any external programs called.

### Process Steps of the RPG Program

The RPG program processes user input from a workstation screen, validates parameters, and prepares data for generating an open order listing. Here’s a step-by-step breakdown of the process:

1. **File and Data Definitions**:
   - **Files**:
     - `SCREEN`: A workstation file (display file) for user interaction, with a record format `SCREEN` used to prompt for input parameters.
     - `INLOC`: A disk file (512 bytes, indexed, shared read mode) containing location data.
     - `BICONT`: A disk file (256 bytes, indexed, shared read mode) containing company or container data.
   - **Data Structures**:
     - `COM`: A table (array) with 5 elements, each 40 characters, used to store error messages (defined in the `** COM` section at the end).
     - `UDS`: User Data Structure for holding input parameters like `KYCO` (company), `KYLOC` (location), `KYFRDT` (from date), `KYTODT` (to date), `KYZEYN` (zero date flag), `KYJOBQ` (job queue flag), `KYCOPY` (number of copies), `FRDT8` (8-digit from date), `TODT8` (8-digit to date), `Y2KCEN` (century for Y2K), and `Y2KCMP` (comparison year for Y2K).
   - **Screen Input Fields** (record format `SCREEN`, NS 01):
     - `KYCO` (company, positions 3-4)
     - `KYLOC` (location, positions 5-7)
     - `KYFRDT` (from date, positions 8-13)
     - `KYTODT` (to date, positions 14-19)
     - `KYZEYN` (zero date flag, position 20)
     - `KYJOBQ` (job queue flag, position 21)
     - `KYCOPY` (number of copies, positions 22-23)
   - **File Input Fields**:
     - `INLOC`: Fields like `ILDEL` (delete flag), `ILCONO` (company), `ILLOC` (location), `ILNAME` (location name).
     - `BICONT`: Fields like `BCDEL` (delete flag).

2. **Initialization**:
   - **Line 0040-0042**: Set indicators `30`, `31`, `32`, `33`, `34`, `81`, `90`, `99` to off to reset the program state.
   - **Line 0043**: Initialize `MSG` (error message field) to blanks.
   - **Line 0045-0048**: If indicator `09` is on (first cycle), execute the `ONETIM` subroutine to set default values and exit:
     - `ONETIM` (lines 0142-0152):
       - Sets `KYCO` to 10, `KYLOC` to '001', `KYZEYN` to 'N', `KYJOBQ` to 'N', `KYCOPY` to 1.
       - Sets indicator `81` to display the screen.
       - Exits to the `END` tag.

3. **Main Processing Logic**:
   - **Line 0050-0054**: If indicator `KG` (likely a cancel key) is on, set indicator `U8` (cancel job), set `LR` (last record), and exit to `END`.
   - **Line 0055-0057**: If indicator `01` is on (screen input received), execute the `S1` subroutine and exit to `END`.

4. **S1 Subroutine (Validation Logic)** (lines 0061-0140):
   - **Company Validation** (lines 0063-0067):
     - Chain (lookup) `BICONT` using `KYCO`. If not found (`30` on) or if `BCDEL` = 'D' (deleted record), set error message `COM,1` ("INVALID COMPANY NUMBER ENTERED") and indicators `81`, `90`. Jump to `ENDS1`.
   - **Date Conversion for Y2K** (lines 0069-0088):
     - Convert `KYFRDT` and `KYTODT` (6-digit dates, YYMMDD) to 8-digit formats (`FRDT8`, `TODT8`) by adding century:
       - Multiply dates by 10000.01 to shift format.
       - Extract year (`FYY`, `TYY`) and compare with `Y2KCMP` (comparison year, e.g., 80 for 1980).
       - If year ≥ `Y2KCMP`, use `Y2KCEN` (e.g., 19 for 1900s); else, add 1 to `Y2KCEN` (e.g., 20 for 2000s).
       - Construct `FRDT8` and `TODT8` (e.g., CCYYMMDD).
   - **Location Validation** (lines 0090-0095):
     - Build `INKEY` from `KYCO` and `KYLOC`, then chain `INLOC`. If not found (`99` on), set error message `COM,3` ("INVALID LOCATION ENTERED") and indicators `81`, `90`, `32`. Jump to `ENDS1`.
   - **Zero Date Validation** (lines 0098-0113):
     - If `KYZEYN` = 'Y' (allow zero dates), check if `FRDT` or `TODT` is non-zero. If so, set error message `COM,5` ("CANNOT ENTER DATE IF ZERO REQUEST = 'Y'") and indicators `81`, `90`, `31`. Jump to `ENDS1`.
   - **Date Range Validation** (lines 0115-0119):
     - If `FRDT8` > `TODT8`, set error message `COM,2` ("FROM DATE GREATER THAN TO DATE") and indicators `81`, `90`, `31`. Jump to `ENDS1`.
   - **Parameter Validation** (lines 0122-0135):
     - Check `KYZEYN` for valid values (' ', 'N', 'Y'). If invalid (`33` on), set error message `COM,4` ("INVALID PARAMETER ENTERED - ' ', N OR Y") and indicators `81`, `90`. Jump to `ENDS1`.
     - Check `KYJOBQ` for valid values (' ', 'N', 'Y'). If invalid (`34` on), set error message `COM,4` and indicators `81`, `90`. Jump to `ENDS1`.
   - **Copy Count Adjustment** (lines 0136-0138):
     - If `KYCOPY` is zero, set it to 1 (default number of copies).

5. **Screen Output** (lines 0155-0164):
   - If indicator `81` is on, display the `SCREEN` file with:
     - Program name (`BB705PFM` in field `K8`).
     - Input fields (`KYCO`, `KYLOC`, `KYFRDT`, `KYTODT`, `KYZEYN`, `KYJOBQ`, `KYCOPY`).
     - Error message (`MSG`).

6. **Program Termination**:
   - The program ends at the `END` tag (line 0059) after processing or cancellation.
   - If validation fails, it redisplays the screen with an error message. If validation passes, it likely prepares data for the `BB705` program (called by the OCL).

### Business Rules

The RPG program enforces the following business rules for generating the pending order listing:

1. **Company Validation**:
   - The company number (`KYCO`) must exist in the `BICONT` file and not be marked as deleted (`BCDEL ≠ 'D'`).
   - Invalid company triggers error message: "INVALID COMPANY NUMBER ENTERED".

2. **Location Validation**:
   - The location (`KYLOC`) combined with `KYCO` must exist in the `INLOC` file.
   - Invalid location triggers error message: "INVALID LOCATION ENTERED".

3. **Date Range Validation**:
   - The from date (`KYFRDT`) must not be later than the to date (`KYTODT`) when converted to 8-digit format (`FRDT8`, `TODT8`).
   - Invalid date range triggers error message: "FROM DATE GREATER THAN TO DATE".

4. **Zero Date Handling**:
   - If `KYZEYN` = 'Y' (allow zero dates), both `KYFRDT` and `KYTODT` must be zero. Non-zero dates trigger error message: "CANNOT ENTER DATE IF ZERO REQUEST = 'Y'".

5. **Parameter Validation**:
   - `KYZEYN` (zero date flag) must be ' ', 'N', or 'Y'.
   - `KYJOBQ` (job queue flag) must be ' ', 'N', or 'Y'.
   - Invalid values trigger error message: "INVALID PARAMETER ENTERED - ' ', N OR Y".

6. **Copy Count**:
   - If the number of copies (`KYCOPY`) is zero, it is set to 1 to ensure at least one copy of the report is generated.

7. **Y2K Date Handling**:
   - Dates are converted from 6-digit (YYMMDD) to 8-digit (CCYYMMDD) format using a century calculation based on `Y2KCMP` and `Y2KCEN` to handle dates before and after 2000 correctly.

### Tables Used

1. **COM**:
   - Defined as an array (`E` specification, line 0007) with 5 elements, each 40 characters.
   - Stores error messages listed at the end of the program:
     - `COM,1`: "INVALID COMPANY NUMBER ENTERED"
     - `COM,2`: "FROM DATE GREATER THAN TO DATE"
     - `COM,3`: "INVALID LOCATION ENTERED"
     - `COM,4`: "INVALID PARAMETER ENTERED - ' ', N OR Y"
     - `COM,5`: "CANNOT ENTER DATE IF ZERO REQUEST = 'Y'"
   - Used to display error messages on the screen when validation fails.

2. **Files as Data Sources**:
   - `INLOC`: Contains location data (fields: `ILDEL`, `ILCONO`, `ILLOC`, `ILNAME`). Functions as a table for validating location and company combinations.
   - `BICONT`: Contains company or container data (field: `BCDEL`). Functions as a table for validating company numbers.

No additional RPG tables (e.g., `TABxxx`) are defined explicitly in the program.

### External Programs Called

- **BB705**: Referenced in the screen output (line 0156, `K8 'BB705PFM'`) and likely called by the OCL program after validation. The RPG program prepares input parameters for `BB705`, which generates the actual pending order listing.
  - Note: `BB705PFM` in the screen output suggests a display file or program name, but it aligns with the `BB705` program called in the OCL (`JOBQ ?CLIB?,BB705` or `BB705 ,,,,,,,,?9?`).

### Summary

- **Process Steps**:
  1. Initialize indicators and clear error messages.
  2. On first cycle (`09`), set default parameters via `ONETIM` and exit.
  3. If cancel key (`KG`) is pressed, cancel the job and exit.
  4. On screen input (`01`), validate parameters in the `S1` subroutine:
     - Check company (`BICONT`), location (`INLOC`), date range, zero date flag, and parameter values.
     - Convert dates to 8-digit format for Y2K compliance.
     - Set default copy count if zero.
  5. Display the screen with input fields and error messages if validation fails.
  6. If validation passes, prepare parameters for the `BB705` program called by the OCL.

- **Business Rules**:
  - Validate company, location, date range, zero date flag, and parameter inputs.
  - Ensure Y2K-compliant date handling.
  - Default to one copy if none specified.
  - Display appropriate error messages for invalid inputs.

- **Tables Used**:
  - `COM`: Array for error messages.
  - `INLOC` and `BICONT`: Files used as data tables for validation.

- **External Programs Called**:
  - `BB705`: The program that generates the pending order listing, called by the OCL after validation.

If you have further details (e.g., the `BB705` RPG code or file layouts for `INLOC`/`BICONT`), I can provide a deeper analysis. Let me know if you’d like me to search for related information or clarify anything!