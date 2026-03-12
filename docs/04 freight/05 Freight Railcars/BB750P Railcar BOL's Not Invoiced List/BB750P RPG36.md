The provided document, `BB750P.rpg36.txt`, is an RPG/36 program (used on IBM System/36 or AS/400 systems) called from the OCL program `BB750P.ocl36.txt`. This RPG program handles prompts for an **Open Railcar Bill of Lading (BOL) Listing by Order**, performing validations and setting parameters for report generation. Below, I’ll explain the process steps, business rules, tables/files used, and any external programs called.

---

### **Process Steps of the RPG Program**

The RPG program `BB750P` is designed to validate user input for generating a BOL listing, either in summary or detail format. It interacts with a display file (`SCREEN`) for user input/output and a data file (`BICONT`) for validation. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - **Header (H) Specification (Line 0001)**: Declares the program name (`BB750P`) and possibly a parameter or option (`P064`), which may indicate a program option or version.
   - **File Specifications (F) (Lines 0005-0006)**:
     - `SCREEN`: A workstation file (display file) for user interaction, with a capacity of 500 bytes.
     - `BICONT`: An indexed disk file (256 bytes, 2-byte key, indexed with `2AI`), opened for input (`IC`) and chained for keyed access.
   - **Array Definition (E) (Line 0007)**: Defines an array `COM` with 1 occurrence, 5 elements, each 40 bytes, used to store error messages.
   - **Indicator Initialization (Lines 0025-0027)**:
     - `SETOF 303132`: Clears indicators 30, 31, and 32.
     - `SETOF 3334`: Clears indicators 33 and 34.
     - `SETOF 8190`: Clears indicators 81 and 90.
     - `MOVEL*BLANKS MSG 40`: Clears the `MSG` field (40 bytes) used for error messages.

2. **Conditional Logic for Screen Input (Lines 0008-0013, 0029-0042)**:
   - **Input Specifications (I) (Lines 0008-0013)**:
     - Defines the `SCREEN` file with record format `NS 01` (screen input) and fields:
       - `KYCO` (positions 3-4): Company number.
       - `KYDETL` (position 5): Detail flag (`Y` or `N` for detail/summary report).
       - `KYJOBQ` (position 6): Job queue flag (`Y` or `N` for queued/direct execution).
       - `KYCOPY` (positions 7-8): Number of copies.
     - `NS 09`: Likely an initial screen display or default record format.
   - **Logic (Lines 0029-0042)**:
     - If record format `09` is active (`C 09 DO`):
       - Executes the `ONETIM` subroutine to set default values (Line 0031).
       - Jumps to `END` tag, bypassing further processing (Line 0032).
     - If `KG` (likely a key group or error condition) is active:
       - Clears indicators 01, 09, and 81, sets `U8` (user switch 8) and `LR` (last record), and jumps to `END` (Lines 0036-0038).
     - If record format `01` is active:
       - Executes the `S1` subroutine for input validation (Line 0041).
       - Jumps to `END` (Line 0042).

3. **S1 Subroutine (Input Validation) (Lines 0046-0072)**:
   - **Company Number Validation (Lines 0048-0052)**:
     - `KYCO CHAINBICONT 30`: Chains (reads) the `BICONT` file using `KYCO` as the key. Indicator 30 is set on if the record is not found.
     - If not found (`N30`) and `BCDEL` (delete flag in `BICONT`, position 1) equals `'D'`:
       - Sets indicator 30.
       - Moves error message `COM,1` ("INVALID COMPANY NUMBER ENTERED") to `MSG`.
       - Sets indicators 81 and 90 (for error display).
       - Jumps to `ENDS1`.
   - **Detail Flag Validation (Lines 0054-0059)**:
     - Compares `KYDETL` to `*BLANKS`, `'N'`, or `'Y'`. If invalid (not one of these values):
       - Sets indicator 33.
       - Moves error message `COM,4` ("INVALID PARAMETER ENTERED - ' ', N OR Y") to `MSG`.
       - Sets indicators 81 and 90.
       - Jumps to `ENDS1`.
   - **Job Queue Flag Validation (Lines 0061-0066)**:
     - Compares `KYJOBQ` to `*BLANKS`, `'N'`, or `'Y'`. If invalid:
       - Sets indicator 34.
       - Moves error message `COM,4` to `MSG`.
       - Sets indicators 81 and 90.
       - Jumps to `ENDS1`.
   - **Copy Number Validation (Lines 0068-0070)**:
     - If `KYCOPY` is zero (`IFEQ *ZEROS`):
       - Sets `KYCOPY` to 1 (default number of copies).
   - **End of Subroutine (Line 0072)**: Returns to the main program.

4. **ONETIM Subroutine (Default Values) (Lines 0074-0083)**:
   - Sets default values for input parameters:
     - `Z-ADD10 KYCO`: Sets company number to 10.
     - `MOVEL'Y' KYDETL`: Sets detail flag to `'Y'` (detail report).
     - `MOVEL'N' KYJOBQ`: Sets job queue flag to `'N'` (direct execution).
     - `Z-ADD1 KYCOPY`: Sets number of copies to 1.
   - Sets indicator 81 for screen output.
   - Ends the subroutine.

5. **Screen Output (Lines 0086-0092)**:
   - Outputs to the `SCREEN` file when indicator 81 is on:
     - Writes format `K8` named `BB750PFM` (likely a display format in the screen file).
     - Outputs fields: `KYCO` (position 2), `KYDETL` (position 3), `KYJOBQ` (position 4), `KYCOPY` (position 6), and `MSG` (position 46).
   - Displays error messages or input prompts to the user.

6. **Program Termination (Line 0044)**:
   - The `END` tag marks the end of processing.
   - If `U8` and `LR` are set (from `KG` condition), the program terminates.

---

### **Business Rules**

The program enforces the following business rules:
1. **Company Number Validation**:
   - The company number (`KYCO`) must exist in the `BICONT` file.
   - If the company number is invalid or the record is marked as deleted (`BCDEL = 'D'`), an error message ("INVALID COMPANY NUMBER ENTERED") is displayed, and processing stops.
2. **Detail Flag Validation**:
   - The detail flag (`KYDETL`) must be `'Y'`, `'N'`, or blank. Any other value triggers an error ("INVALID PARAMETER ENTERED - ' ', N OR Y").
3. **Job Queue Flag Validation**:
   - The job queue flag (`KYJOBQ`) must be `'Y'`, `'N'`, or blank. Any other value triggers an error ("INVALID PARAMETER ENTERED - ' ', N OR Y").
4. **Copy Number Default**:
   - If the number of copies (`KYCOPY`) is zero, it is set to 1.
5. **Default Values**:
   - If the initial screen (`NS 09`) is displayed, the program sets defaults:
     - Company number = 10.
     - Detail flag = `'Y'` (detail report).
     - Job queue flag = `'N'` (direct execution).
     - Number of copies = 1.
6. **Error Handling**:
   - Errors set indicators 81 and 90, display an error message, and stop further processing until corrected input is provided.
7. **Screen Interaction**:
   - The program uses a display file (`SCREEN`) to prompt the user for input and display errors.
   - The display format `BB750PFM` is used to show input fields and messages.

---

### **Tables/Files Used**

1. **BICONT**:
   - Type: Indexed disk file (256 bytes, 2-byte key).
   - Usage: Input file for validating the company number (`KYCO`).
   - Key Field: Used for `CHAIN` operation to retrieve records.
   - Fields Referenced:
     - `BCDEL` (position 1): Delete flag, checked for `'D'` to indicate a deleted record.
   - Purpose: Likely stores company or railcar inventory control data.

2. **SCREEN**:
   - Type: Workstation file (display file).
   - Usage: Handles user input/output for parameters like company number, detail flag, job queue flag, and number of copies.
   - Fields:
     - `KYCO`: Company number (2 bytes).
     - `KYDETL`: Detail flag (1 byte).
     - `KYJOBQ`: Job queue flag (1 byte).
     - `KYCOPY`: Number of copies (2 bytes).
     - `MSG`: Error message (40 bytes).
   - Format: Uses `BB750PFM` for displaying prompts and errors.

3. **BBBOL** (Implied from OCL):
   - Not explicitly referenced in the RPG program but mentioned in the OCL (`BB750P.ocl36.txt`).
   - Likely used by the called programs (`BB750` or `BB751`) for BOL data processing, as it’s defined in the OCL but commented out.

---

### **External Programs Called**

The RPG program `BB750P` does **not** explicitly call any external programs within its code. However:
- The OCL program (`BB750P.ocl36.txt`) calls `BB750` or `BB751` based on the `KYDETL` flag (`Y` for detail via `BB750`, else summary via `BB751`).
- Thus, `BB750P` serves as a front-end validation program, and the actual report generation is handled by:
  1. **BB750**: Likely generates the detailed BOL listing.
  2. **BB751**: Likely generates the summary BOL listing.
- These programs are not called directly in the RPG code but are invoked by the OCL script after `BB750P` validates the input.

---

### **Summary**

- **Purpose**: The RPG program `BB750P` is a user interface and validation program for an Open Railcar BOL Listing. It prompts for and validates input parameters (company number, detail flag, job queue flag, number of copies) before passing control to `BB750` (detail report) or `BB751` (summary report) via the OCL script.
- **Process Flow**:
  1. Initializes indicators and variables.
  2. Displays an initial screen with defaults via the `ONETIM` subroutine if needed.
  3. Validates user input (company number, detail flag, job queue flag, copies) using the `S1` subroutine.
  4. Displays errors if validation fails, or proceeds to allow the OCL to call the appropriate report program (`BB750` or `BB751`).
- **Business Rules**: Enforces valid company numbers (checked against `BICONT`), valid detail and job queue flags (`Y`, `N`, or blank), and a default of 1 copy if none specified.
- **Files Used**: `BICONT` (data validation), `SCREEN` (user interface), and possibly `BBBOL` (implied by OCL for report data).
- **External Programs**: None called directly, but `BB750` and `BB751` are invoked by the OCL based on validated parameters.

If you have the code for `BB750` or `BB751`, or the file layouts for `BICONT` or `BBBOL`, I can provide further details on their integration!