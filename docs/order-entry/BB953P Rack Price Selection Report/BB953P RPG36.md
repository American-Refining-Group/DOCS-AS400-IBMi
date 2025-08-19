The provided document, `BB953P.rpg36.txt`, is an RPG (Report Program Generator) program written for an IBM System/36 or AS/400 environment. It is called from the OCL (Operation Control Language) program `BB953P.ocl36.txt` and is responsible for processing prompts for a customer rack pricing list. Below, I will explain the process steps, business rules, tables (files) used, and any external programs called.

---

### **Process Steps of the RPG Program**

The RPG program `BB953P` is designed to validate user input for generating a customer rack pricing list. It processes data related to divisions, locations, product classes, containers, products, and date ranges, ensuring that the input parameters are valid before proceeding. The program uses a workstation file (`SCREEN`) for user interaction and performs validations against data stored in several files. Here’s a detailed breakdown of the process steps:

1. **Header and File Definitions**:
   - **H-Spec**: The header specification (`H    P064`) includes a program identifier (`P064`) and indicates the program is named `BB953P`.
   - **File Definitions (F-Specs)**:
     - `SCREEN`: A workstation file (512 bytes) for user interaction, likely a display file for input/output on a terminal.
     - `GSTABL`: A disk file (256 bytes, indexed with 12 keys, type `CNTRCD`), replaced by `GSCNTR1` per modification `JK01`.
     - `INLOC`: A disk file (512 bytes, indexed with 5 keys) for location data.
     - `BICONT`: A disk file (256 bytes, indexed with 2 keys) for company/control data.
     - `GSCNTR1`: A disk file (512 bytes, indexed with 3 keys), added per `JK01` to replace `GSTABL` for container data.
     - `GSPROD`: A disk file (512 bytes, indexed with 6 keys), added per `JK02` to replace product code lookups in `GSTABL`.
   - **Array Definitions (E-Specs)**:
     - `DCO`: Array of 3 elements (35 bytes each) for company data.
     - `CTR`: Array of 5 elements (3 bytes each) for container codes.
     - `LOC`: Array of 5 elements (3 bytes each) for location codes.
     - `CLS`: Array of 10 elements (3 bytes each) for product class codes.
     - `PRD`: Array of 10 elements (4 bytes each) for product codes.
     - `COM`: Array of 11 elements (40 bytes each) for error messages.
   - **Input Specifications (I-Specs)**:
     - Defines fields for the `SCREEN` file, including keys for division (`KYDIV`), locations (`KYLOSL`, `KYLOC1-5`), product classes (`KYPCSL`, `KYPC01-10`), containers (`KYCTSL`, `KYCT01-05`), products (`KYPDSL`, `KYPD01-10`), date ranges (`KYDTSL`, `KYFRDT`, `KYTODT`), job queue (`KYJOBQ`), and copy count (`KYCOPY`).
     - Defines fields for `BICONT`, `INLOC`, `GSTABL`, `GSCNTR1`, and `GSPROD` files, such as company number (`BCCO`), location code (`ILLOC`), descriptions (`TBDESC`, `TPDESC`), and delete flags (`BCDEL`, `ILDEL`, `TPDEL`).
     - User data structure (`UDS`) defines additional fields for validation and processing.

2. **Initialization (C-Specs)**:
   - **Indicator Reset**: Lines `0042-0045` use `SETOF` to turn off indicators `31-90`, which are used for error handling and control flow.
   - **Message Initialization**: Line `0046` clears the `MSG` field (40 bytes) to ensure no residual error messages.
   - **KG Condition**: If the `KG` condition (likely a program status or key indicator) is active:
     - Sets on indicators `U1` and `LR` (last record, program termination).
     - Turns off indicators `01`, `09`, and `81`.
     - Jumps to the `END` tag, terminating the program.

3. **Main Processing Logic**:
   - **Condition `09` (Initial Run)**:
     - If indicator `09` is on (likely set by the first input screen), the `ONETIM` subroutine is executed (line `0052`), and the program jumps to `END`.
   - **Condition `01` (Main Validation)**:
     - If indicator `01` is on (likely set by a subsequent screen input), the `S1` subroutine is executed (line `0055`), and the program jumps to `END`.

4. **ONETIM Subroutine (Lines `0160-0200`)**:
   - Initializes variables and sets default values for the pricing list prompt:
     - Clears the `DCO` array (company data).
     - Sets `X` to 1 and `BILIM` to 00.
     - Positions the file pointer for `BICONT` using `SETLL` (line `0165`).
     - Reads `BICONT` records in a loop (`AGNCO` to `ENDCO`):
       - Skips records marked for deletion (`BCDEL = 'D'`).
       - Continues until the end of file (indicator `20`).
     - Sets default values:
       - `KYLOSL`, `KYDTSL`, `KYPDSL`, `KYPCSL`, `KYCTSL` to `'ALL'`.
       - `KYDIV` to `'R'` (division code).
       - `CTR` to blanks.
       - `KYCO` to 10 (company code).
       - `KYJOBQ` to `'N'` (no job queue).
       - `KYCOPY` to 1 (one copy of the report).
     - Sets indicator `81` to indicate the screen should be displayed.

5. **S1 Subroutine (Lines `0062-0156`)**:
   - Validates user input from the `SCREEN` file for generating the rack pricing list:
     - **Division Validation**:
       - Checks if `KYDIV` is `'R'`, `'L'`, or `'B'` (valid division codes). If invalid, sets indicators `81`, `90`, and `29`, moves error message `COM,2` ("DIVISION MUST BE 'R' 'L' OR 'B'") to `MSG`, and jumps to `ENDS1`.
       - Sets `KYDVNO` to `'00'` for `'R'` or `'50'` for `'L'`.
     - **Location Validation**:
       - If `KYLOSL = 'ALL'`, checks if `LOCFLD` (concatenated `LOC` array) is blank. If not blank, sets error (`COM,10`: "CANNOT BE 'ALL' IF SELECTION IS MADE").
       - If `KYLOSL = 'SEL'`, checks each `LOC,X` entry. If non-blank, chains to `INLOC` using `LOCKEY`. If the record is not found (indicator `41`), sets error (`COM,3`: "INVALID LOCATION ENTERED").
     - **Product Class Validation**:
       - If `KYPCSL = 'ALL'`, checks if `CLS,1` is blank. If not, sets error (`COM,10`).
       - If `KYPCSL = 'SEL'`, checks each `CLS,X` entry. If non-blank, chains to `GSPROD` (previously `GSTABL`) using `PRCLKY`. If not found (indicator `32`), sets error (`COM,5`: "INVALID PRODUCT CLASS ENTERED").
     - **Container Validation**:
       - If `KYCTSL = 'ALL'`, checks if `CTFLD` (concatenated `CTR` array) is blank. If not, sets error (`COM,10`).
       - If `KYCTSL = 'SEL'`, checks each `CTR,X` entry. If non-blank, chains to `GSCNTR1` using `W$CNTA`. If not found (indicator `33`), sets error (`COM,6`: "INVALID CONTAINER CODE ENTERED").
     - **Product Validation**:
       - If `KYPDSL = 'ALL'`, checks if `PRD,1` is blank. If not, sets error (`COM,10`).
       - If `KYPDSL = 'SEL'`, checks each `PRD,X` entry. If non-blank, chains to `GSPROD` using `KLPROD`. If not found (indicator `31`), sets error (`COM,4`: "INVALID PRODUCT CODE ENTERED").
     - **Date Validation**:
       - If `KYDTSL = 'ALL'`, checks if `KYFRDT` is zero. If not, sets error (`COM,10`).
       - If `KYDTSL = 'SEL'`, checks if `KYFRDT` or `KYTODT` is zero. If so, sets errors (`COM,7`: "FROM DATE CANNOT BE BLANK" or `COM,8`: "TO DATE CANNOT BE BLANK").
     - **Job Queue Validation**:
       - Checks if `KYJOBQ` is blank, `'N'`, or `'Y'`. If invalid, sets error (`COM,9`: "ENTER BLANK, N OR Y").
     - **Copy Count**:
       - If `KYCOPY` is zero, sets it to 1 (default number of copies).
     - **Date Processing**:
       - Computes `KYDAT8` (8-digit date) using `UDATE` (system date) and `Y2KCEN` (century indicator, 19 or 20 based on `UYEAR` vs. `Y2KCMP`).
   - Calls `SETIND` subroutine to set indicators based on the validation loop counter `X`.
   - Jumps to `ENDS1` if any validation fails.

6. **SETIND Subroutine (Lines `0160-0156`)**:
   - Sets indicators (`21-30`, `51-60`, `61-65`, `42-46`) based on the value of `X` (loop counter) for product, product class, container, or location errors, respectively.
   - Ensures the correct error indicators are set for display on the screen.

7. **Output Specifications (O-Specs)**:
   - Writes validated or default input fields to the `SCREEN` file (format `BB953PFM`) if indicator `81` is on.
   - Outputs fields like `KYDIV`, `KYLOSL`, `KYLOC1-5`, `KYPCSL`, `KYPC01-10`, `KYCTSL`, `KYCT01-05`, `KYPDSL`, `KYPD01-10`, `KYDTSL`, `KYFRDT`, `KYTODT`, `KYJOBQ`, `KYCOPY`, and `MSG`.

8. **Termination**:
   - The program terminates at the `END` tag (line `0058`) after processing the `ONETIM` or `S1` subroutines, ensuring all validations are complete and the screen is updated with results or errors.

---

### **Business Rules**

The RPG program enforces the following business rules for the customer rack pricing list prompt:

1. **Division (`KYDIV`)**:
   - Must be `'R'`, `'L'`, or `'B'`. Invalid values trigger error message `"DIVISION MUST BE 'R' 'L' OR 'B'"`.
   - Sets `KYDVNO` to `'00'` for `'R'` or `'50'` for `'L'`.

2. **Location (`KYLOSL`, `KYLOC1-5`)**:
   - If `KYLOSL = 'ALL'`, no specific locations (`KYLOC1-5`) can be entered (error: `"CANNOT BE 'ALL' IF SELECTION IS MADE"`).
   - If `KYLOSL = 'SEL'`, at least one location (`KYLOC1-5`) must be non-blank and must exist in the `INLOC` file (error: `"INVALID LOCATION ENTERED"`).

3. **Product Class (`KYPCSL`, `KYPC01-10`)**:
   - If `KYPCSL = 'ALL'`, no specific product classes (`KYPC01-10`) can be entered (error: `"CANNOT BE 'ALL' IF SELECTION IS MADE"`).
   - If `KYPCSL = 'SEL'`, at least one product class must be non-blank and must exist in `GSPROD` (error: `"INVALID PRODUCT CLASS ENTERED"`).

4. **Container (`KYCTSL`, `KYCT01-05`)**:
   - If `KYCTSL = 'ALL'`, no specific containers (`KYCT01-05`) can be entered (error: `"CANNOT BE 'ALL' IF SELECTION IS MADE"`).
   - If `KYCTSL = 'SEL'`, at least one container must be non-blank and must exist in `GSCNTR1` (error: `"INVALID CONTAINER CODE ENTERED"`).

5. **Product (`KYPDSL`, `KYPD01-10`)**:
   - If `KYPDSL = 'ALL'`, no specific products (`KYPD01-10`) can be entered (error: `"CANNOT BE 'ALL' IF SELECTION IS MADE"`).
   - If `KYPDSL = 'SEL'`, at least one product must be non-blank and must exist in `GSPROD` (error: `"INVALID PRODUCT CODE ENTERED"`).

6. **Date Range (`KYDTSL`, `KYFRDT`, `KYTODT`)**:
   - If `KYDTSL = 'ALL'`, no specific dates can be entered (error: `"CANNOT BE 'ALL' IF SELECTION IS MADE"`).
   - If `KYDTSL = 'SEL'`, both `KYFRDT` and `KYTODT` must be non-zero (errors: `"FROM DATE CANNOT BE BLANK"` or `"TO DATE CANNOT BE BLANK"`).

7. **Job Queue (`KYJOBQ`)**:
   - Must be blank, `'N'`, or `'Y'` (error: `"ENTER BLANK, N OR Y"`).

8. **Copy Count (`KYCOPY`)**:
   - If zero, defaults to 1.

9. **Company Data**:
   - Reads `BICONT` to skip deleted records (`BCDEL = 'D'`).
   - Ensures valid company data is available before setting defaults.

10. **Default Values**:
    - If no input is provided (initial run), sets defaults: `KYLOSL`, `KYDTSL`, `KYPDSL`, `KYPCSL`, `KYCTSL` to `'ALL'`, `KYDIV` to `'R'`, `KYCO` to 10, `KYJOBQ` to `'N'`, and `KYCOPY` to 1.

11. **Error Handling**:
    - Uses the `COM` array to store error messages (e.g., `"INVALID LOCATION ENTERED"`, `"FROM DATE CANNOT BE BLANK"`).
    - Sets indicators `81` and `90` to display errors on the screen and halt processing.

---

### **Tables (Files) Used**

The RPG program uses the following files (tables):
1. **SCREEN**: A workstation file (display file) for user input/output, defining fields like division, locations, product classes, containers, products, dates, job queue, and copy count.
2. **BICONT**: Contains company/control data (e.g., `BCCO`: company number, `BCNAME`: company name, `BCDEL`: delete flag).
3. **INLOC**: Contains location data (e.g., `ILLOC`: location code, `ILNAME`: location name, `ILDEL`: delete flag).
4. **GSTABL**: Originally used for product class/container lookups, now replaced by `GSCNTR1` and `GSPROD` (contains `TBDESC`: table description, `TBABDS`: short description).
5. **GSCNTR1**: Contains container data (e.g., `TCDESC`: container description), added per modification `JK01`.
6. **GSPROD**: Contains product data (e.g., `TPDESC`: product description, `TPPRGP`: product group code, `TPPRCL`: product class code, `TPABDS`: short description), added per modification `JK02`.

---

### **External Programs Called**

No external programs are explicitly called within the RPG program. The program operates independently, performing validations and screen updates within its own logic. However, it is invoked by the OCL program `BB953P.ocl36.txt`, which may queue or run a program named `BB953`. It’s unclear if `BB953` is a separate program or a misnomer for `BB953P`, but within the RPG code, no external program calls (e.g., via `CALL` operations) are present.

---

### **Summary**

The `BB953P` RPG program is a prompt validation program for a customer rack pricing list. It:
- Initializes variables and sets default values via the `ONETIM` subroutine.
- Validates user input for division, locations, product classes, containers, products, date ranges, job queue, and copy count via the `S1` subroutine.
- Enforces business rules to ensure valid data, checking against files like `INLOC`, `GSCNTR1`, and `GSPROD`.
- Displays errors or validated data on the `SCREEN` file.
- Uses the `SETIND` subroutine to manage error indicators.

**Tables (Files) Used**:
- `SCREEN`
- `BICONT`
- `INLOC`
- `GSTABL` (replaced by `GSCNTR1` and `GSPROD`)
- `GSCNTR1`
- `GSPROD`

**External Programs Called**:
- None explicitly called within the RPG program.

The program ensures that user inputs are valid before proceeding with the pricing list generation, likely triggering further processing (e.g., report generation) via the OCL program or a subsequent job (`BB953`).