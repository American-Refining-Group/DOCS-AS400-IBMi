The provided RPG program, `BB951P.rpg36.txt`, is an RPG/36 program designed for IBM System/36, called by the `BB951P.ocl36.txt` OCL program to manage rack price file updates, specifically tracking the last two price changes. Below, I’ll explain the **process steps**, **business rules**, **tables/files used**, and any **external programs called**, based on the RPG code.

---

### Process Steps of the RPG Program

The RPG program `BB951P` processes user input from a workstation screen, validates it against various files, and prepares data for further processing (e.g., updating price files). Here’s a detailed breakdown of the process steps:

1. **File and Data Structure Definitions**:
   - **Files**:
     - `SCREEN`: A workstation file (display file) for user interaction, 512 bytes.
     - `BICONT`: A disk file (256 bytes, indexed, shared read/modify) for company control data.
     - `INLOC`: A disk file (512 bytes, indexed, shared read/modify) for location data.
     - `GSCNTR1`: A disk file (512 bytes, indexed, shared read-only) for container control data (replaced `GSTABL` per revision JK01).
     - `GSPROD`: A disk file (512 bytes, indexed, shared read/modify) for product data (replaced `GSTABL` for product codes per revision JK02).
   - **Data Structures**:
     - `DCO`: An array (3 elements, 35 bytes each) to store company data.
     - `COM`: An array (7 elements, 40 bytes each) for error messages.
     - Input specifications define fields from the screen (`SCREEN`) and files (`BICONT`, `INLOC`, `GSCNTR1`, `GSPROD`), such as company number (`KYCO`), location ranges (`KYLOFR`, `KYLOTO`), product ranges (`KYPRFR`, `KYPRTO`), container ranges (`KYCNFR`, `KYCNTO`), job queue flag (`KYJOBQ`), copy flag (`KYCOPY`), and date ranges (`KYFRDT`, `KYTODT`).
     - `UDS` (User Data Structure) holds key fields passed from the OCL program.

2. **Initialization (Cycle Start)**:
   - **Indicator Setup** (`C*` lines 0024–0026): Resets indicators (32, 38, 40–48, 81, 90) to `OFF` to clear prior states.
   - **Message Clearing** (`MOVEL*BLANKS MSG`): Initializes the message field to blanks.
   - **KG (End of File) Condition** (`C* KG` lines 0029–0031):
     - If the end-of-file condition (`KG`) is met, sets indicators `U1` (update) and `LR` (last record), clears indicators 01, 09, 81, and jumps to the `END` tag, terminating the program.
   - **Screen Input Processing**:
     - If indicator `09` is on (initial screen read), the `ONETIM` subroutine is executed, and the program jumps to `END`.
     - If indicator `01` is on (subsequent screen read), the `S1` subroutine is executed, and the program jumps to `END`.

3. **ONETIM Subroutine (Lines 0073–0106)**:
   - **Purpose**: Initializes data and populates the company array (`DCO`) from the `BICONT` file.
   - **Steps**:
     - Clears the `DCO` array (`MOVEL*BLANKS DCO`).
     - Initializes index `X` to 1 and limit `BILIM` to 0.
     - Positions the file pointer at the beginning of `BICONT` (`SETLLBICONT`).
     - Reads `BICONT` records in a loop (`AGNCO` to `ENDCO`):
       - If a record’s `BCDEL` field is `'D'` (deleted), skips to the next record.
       - Copies company number (`BCCO`) and name (`BCNAME`) to the `DCO` array at index `X`.
       - Increments `X` until it exceeds 3 or the end of `BICONT` is reached (`20` indicator).
     - Resets `KYCO` to zeros, sets `KYJOBQ` to `'N'`, and sets `KYCOPY` to 1.
     - Sets indicator `81` to signal output to the screen.
   - **Output**: Writes to the `SCREEN` file with format `BB951PFM`, displaying fields like `KYCO`, `DCO`, `KYLOFR`, `KYLOTO`, `KYPRFR`, `KYPRTO`, `KYCNFR`, `KYCNTO`, `KYJOBQ`, `KYCOPY`, `MSG`, `KYFRD8`, and `KYTOD8`.

4. **S1 Subroutine (Lines 0043–0069)**:
   - **Purpose**: Validates user input from the screen against the files and sets error messages if invalid.
   - **Steps**:
     - **Company Validation**:
       - If `KYCO` (company number) is zero, sets indicators 32, 81, 90, moves error message `COM,5` ("ENTER COMPANY NUMBER") to `MSG`, and jumps to `ENDS1`.
       - Chains to `BICONT` using `KYCO`. If not found (`32` on) or `BCDEL` is `'D'`, sets indicators 81, 90, moves error message `COM,2` ("INVALID COMPANY NUMBER ENTERED") to `MSG`, and jumps to `ENDS1`.
     - **Date Conversion** (if `KYFRDT` is non-zero):
       - Converts `KYFRDT` and `KYTODT` (MMDDYY format) to `KYFRD8` and `KYTOD8` (YYYYMMDD format) by multiplying by 10000.01 and extracting year prefixes (`FRYY`, `TOYY`).
       - If the year is ≥ 80, prefixes with `19`; otherwise, prefixes with `20` (e.g., 23 becomes 2023, 85 becomes 1985).
     - **Location From Validation** (if `KYLOFR` is non-blank):
       - Builds key `INKEY` with `KYCO` and `KYLOFR`, chains to `INLOC`. If not found (`40` on), sets indicators 81, 90, moves error message `COM,1` ("INVALID LOCATION ENTERED") to `MSG`, and jumps to `ENDS1`.
     - **Location To Validation** (if `KYLOTO` is non-blank):
       - Similar to `KYLOFR`, chains to `INLOC` with `KYCO` and `KYLOTO`. If not found (`43` on), sets error message `COM,1` and jumps to `ENDS1`.
     - **Product From Validation** (if `KYPRFR` is non-blank):
       - Builds key `KLPROD` with `KYCO` and `KYPRFR`, chains to `GSPROD`. If not found (`41` on), sets error message `COM,3` ("INVALID PRODUCT CODE ENTERED") and jumps to `ENDS1`.
     - **Product To Validation** (if `KYPRTO` is non-blank):
       - Similar to `KYPRFR`, chains to `GSPROD` with `KYCO` and `KYPRTO`. If not found (`44` on), sets error message `COM,3` and jumps to `ENDS1`.
     - **Container From Validation** (if `KYCNFR` is non-blank):
       - Chains to `GSCNTR1` using `KYCNFR`. If not found (`42` on), sets error message `COM,7` ("INVALID CONTAINER CODE") and jumps to `ENDS1`.
     - **Container To Validation** (if `KYCNTO` is non-blank):
       - Similar to `KYCNFR`, chains to `GSCNTR1` with `KYCNTO`. If not found (`45` on), sets error message `COM,7` and jumps to `ENDS1`.
     - **Job Queue Validation**:
       - Checks if `KYJOBQ` is blank, `'N'`, or `'Y'`. If not, sets error message `COM,4` ("ENTER BLANK, N OR Y") and jumps to `ENDS1`.
     - **Copy Flag**:
       - If `KYCOPY` is zero, sets it to 1.
     - **Output**: If any validation fails, indicators 81, 90 are set, and an error message is displayed. The program loops back to display the screen for user correction.

5. **Program Termination**:
   - The `END` tag is reached after `ONETIM`, `S1`, or `KG` conditions, terminating the program cycle.
   - The program outputs validated data to the `SCREEN` file, which may be passed back to the OCL program for further processing (e.g., submitting `BB951` to a job queue).

---

### Business Rules

The RPG program enforces the following business rules for validating user input to generate a rack price file report or update:

1. **Company Number (`KYCO`)**:
   - Must be non-zero and exist in the `BICONT` file.
   - Must not be marked as deleted (`BCDEL ≠ 'D'`).
   - If invalid, displays "INVALID COMPANY NUMBER ENTERED" or "ENTER COMPANY NUMBER".

2. **Location Range (`KYLOFR`, `KYLOTO`)**:
   - If non-blank, must exist in the `INLOC` file for the given company (`KYCO`).
   - If invalid, displays "INVALID LOCATION ENTERED".

3. **Product Range (`KYPRFR`, `KYPRTO`)**:
   - If non-blank, must exist in the `GSPROD` file for the given company (`KYCO`).
   - If invalid, displays "INVALID PRODUCT CODE ENTERED".

4. **Container Range (`KYCNFR`, `KYCNTO`)**:
   - If non-blank, must exist in the `GSCNTR1` file.
   - If invalid, displays "INVALID CONTAINER CODE".

5. **Job Queue Flag (`KYJOBQ`)**:
   - Must be blank, `'N'`, or `'Y'`.
   - If invalid, displays "ENTER BLANK, N OR Y".

6. **Date Range (`KYFRDT`, `KYTODT`)**:
   - If provided, converted from MMDDYY to YYYYMMDD format.
   - Years ≥ 80 are prefixed with `19` (e.g., 1985); years < 80 are prefixed with `20` (e.g., 2023).
   - Stored in `KYFRD8` and `KYTOD8` for output.

7. **Copy Flag (`KYCOPY`)**:
   - If zero, automatically set to 1 to ensure at least one copy is processed.

8. **Company Data Population**:
   - Up to three non-deleted companies from `BICONT` are loaded into the `DCO` array for display on the screen.

9. **Error Handling**:
   - Validation failures set indicators 81 (screen output) and 90 (error condition), display an error message, and prompt the user to correct input.

---

### Tables/Files Used

The RPG program uses the following files (some of which are tables or data files):
1. **SCREEN**: Workstation file for user input/output, using format `BB951PFM`.
2. **BICONT**: Company control file (256 bytes, indexed), containing company number (`BCCO`), name (`BCNAME`), and deletion flag (`BCDEL`).
3. **INLOC**: Location file (512 bytes, indexed), containing company (`ILCONO`), location code (`ILLOC`), name (`ILNAME`), and deletion flag (`ILDEL`).
4. **GSCNTR1**: Container control file (512 bytes, indexed), used for validating container codes (`KYCNFR`, `KYCNTO`).
5. **GSPROD**: Product file (512 bytes, indexed), containing product description (`TPDESC`), deletion flag (`TPDEL`), and other fields (e.g., `TPFLCD`).

**Note**: The `GSTABL` file is commented out (per revisions JK01 and JK02), replaced by `GSCNTR1` for container data and `GSPROD` for product data.

---

### External Programs Called

The RPG program **does not explicitly call any external programs**. It interacts with the `SCREEN` file for user input/output and performs file operations (e.g., `CHAIN`, `READ`, `SETLL`) on the defined files. The OCL program (`BB951P.ocl36.txt`) calls `BB951`, which is likely this RPG program, but no additional programs are invoked within the RPG code itself.

---

### Summary

- **Process Overview**: The `BB951P` RPG program handles user input for generating or updating a rack price file, validating company, location, product, container, and job queue parameters against respective files. It initializes company data, performs date conversions, and enforces strict validation rules, displaying error messages for invalid inputs. The validated data is output to the screen and likely passed back to the OCL program for further processing (e.g., job queue submission).
- **Business Rules**: Ensures valid company, location, product, and container codes, correct job queue flags, and proper date formatting. Automatically sets copy count and handles up to three companies.
- **Files/Tables**: `SCREEN`, `BICONT`, `INLOC`, `GSCNTR1`, `GSPROD`.
- **External Programs**: None called directly by the RPG program.

If you need further details (e.g., specific file structures, screen format `BB951PFM`, or additional context about the OCL interaction), please provide them, and I can refine the explanation. Alternatively, I can search for related information if you’d like!