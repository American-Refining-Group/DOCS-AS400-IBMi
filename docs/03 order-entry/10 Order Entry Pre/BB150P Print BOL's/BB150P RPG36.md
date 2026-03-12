The `BB150P.rpg36.txt` RPG program is a prompt program for the Bill of Lading (BOL) printing process, designed to run on an IBM System/36 or AS/400 environment. It is called by the main OCL (Operation Control Language) procedure to gather and validate user input before invoking the main BOL printing program (`BB150`). The program handles user input for company selection, order numbers, and shipment reference numbers (SRNs) for single and multi-load orders, ensuring valid data is passed to `BB150`. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called.

---

### **Process Steps of the BB150P RPG Program**

The `BB150P` program serves as an interactive interface to collect and validate input parameters for BOL printing. It uses a workstation file (`SCREEN`) to prompt the user and validates inputs against data in the `BICONT` and `BBORTR` files. The program also supports multi-load orders by validating SRNs (`JB01`). Here’s a detailed breakdown of the process steps:

1. **Initialization**:
   - The program initializes by setting up the workstation file (`SCREEN`) for user interaction and clearing error message fields (`MSG35`) via the `EDIT` subroutine.
   - It sets indicators (`01`, `U8`, `LR`) to control program flow and ensure proper termination.
   - If indicator `10` is off and `11` is off, it calls the `ONETIM` subroutine to set default values for company (`KYCO`), selection type (`KYALSL`), job queue (`KYJOBQ`), and copy counts (`KYCOPY`, `KYCOPB`).
   - Default values include:
     - `KYCO = '10'` (company code).
     - `KYALSL = 'SEL'` (selective order processing).
     - `KYJOBQ = 'N'` (no job queue).
     - `KYCOPY = 01`, `KYCOPB = 01` (one copy for each output).

   **Purpose**: Prepares the program environment and sets default parameters for first-time execution.

2. **User Input Collection**:
   - Displays the `SCREEN` file (format `BB150PFM`) to prompt the user for:
     - Company code (`KYCO`, positions 1–2).
     - Selection type (`KYALSL`, positions 3–5, either `ALL` or `SEL`).
     - Up to 10 order numbers (`KYORD1`–`KYORD0`, positions 6–65).
     - Job queue flag (`KYJOBQ`, position 66, `Y`/`N`/`blank`).
     - Copy counts (`KYCOPY`, positions 67–68; `KYCOPB`, positions 69–70).
     - From and to SRNs (`KYFS`, `KYTS`, positions 71–100 and 101–130 for multi-load orders, `JB01`).
   - Reads user input into the `SCREEN` file (NS 01) and stores it in the Local Data Area (LDA, positions 101–300) for coordination with `BB150` and `BB152P`.

   **Purpose**: Collects user-specified parameters for BOL printing.

3. **Input Validation (EDIT Subroutine)**:
   - **Company Code Validation**:
     - Chains to `BICONT` using `KYCO` to verify the company code exists.
     - If invalid, sets indicator `20` and displays error message `MSG,2` ("INVALID COMPANY #").
   - **Selection Type Validation**:
     - Checks `KYALSL` for `ALL` or `SEL`.
     - If neither, sets indicator `20` and displays error message `MSG,1` ("INVALID SELECTION ALL / SEL").
     - If `ALL`, skips order number validation and proceeds to job queue checks.
   - **Order Number Validation**:
     - For `KYALSL = 'SEL'`, iterates through the 10 order number fields (`KYO,1` to `KYO,10`).
     - For each non-zero order number:
       - Constructs an 11-character key (`KY11`) by combining `KYCO` and the order number (`KYO,X`) with trailing zeros (`000`).
       - Chains to `BBORTR` to verify the order exists.
       - If invalid, sets indicator `20` and displays error message `MSG,3` ("INVALID ORDER #").
   - **SRN Validation for Multi-Load Orders (`JB01`)**:
     - Checks `BOMULO` (multi-load flag) in `BBORTR`:
       - **Single-Load Orders** (`BOMULO ≠ 'Y'`):
         - Ensures both `KYFS,X` and `KYTS,X` are zero.
         - If either is non-zero, sets indicator `20` and displays error messages `MSG,4` ("FROM SRN MUST BE ZERO") or `MSG,5` ("TO SRN MUST BE ZERO").
       - **Multi-Load Orders** (`BOMULO = 'Y'`):
         - Ensures both `KYFS,X` and `KYTS,X` are either zero or both keyed.
         - If one is zero and the other is not, sets indicator `20` and displays error message `MSG,6` ("BOTH FROM / TO SRN MUST BE KEYED").
         - Checks that `KYFS,X` and `KYTS,X` do not exceed the total loads (`BOTOLO`).
         - If exceeded, sets indicator `20` and displays error message `MSG,7` ("FROM/TO MUST BE LESS THAN LOADS").
         - Ensures `KYFS,X` is not greater than `KYTS,X`.
         - If true, sets indicator `20` and displays error message `MSG,8` ("FROM SRN MUST BE LESS THAN TO SRN").
   - **Job Queue Validation**:
     - Checks `KYJOBQ` for valid values (`Y`, `N`, or blank).
     - If invalid, sets indicator `20` but does not display a specific error message (logic assumes program termination).
   - **Copy Count Validation**:
     - If `KYCOPY` is zero, sets it to `01`.
     - If `KYCOPB` is zero, sets it to `01`.

   **Purpose**: Validates user inputs to ensure they are correct before passing them to `BB150`.

4. **Output and Program Termination**:
   - If validation fails (indicator `20` is on), redisplays the `SCREEN` with the error message (`MSG35`) and input fields for correction.
   - If validation succeeds, writes validated inputs to the LDA and sets indicator `11` to indicate completion.
   - Outputs to `SCREEN` (format `BB150PFM`) with fields:
     - `KYCO`, `KYALSL`, `KYORD1`–`KYORD0`, `KYJOBQ`, `KYCOPY`, `KYCOPB`, `KYFS`, `KYTS`, and `MSG35`.
   - Sets `U8` and `LR` (Last Record) indicators to terminate the program and pass control back to the OCL.

   **Purpose**: Communicates validated parameters to the main program or prompts the user to correct errors.

---

### **Business Rules**

The `BB150P` program enforces the following business rules:
1. **Company Code**:
   - Must exist in the `BICONT` file.
   - Invalid company codes trigger an error message ("INVALID COMPANY #").
2. **Selection Type**:
   - Must be `ALL` (process all orders for the company) or `SEL` (process specific orders).
   - Invalid selections trigger an error message ("INVALID SELECTION ALL / SEL").
3. **Order Numbers**:
   - For `SEL`, at least one order number (`KYO,1`–`KYO,10`) must be valid and exist in `BBORTR`.
   - Invalid order numbers trigger an error message ("INVALID ORDER #").
4. **Shipment Reference Numbers (SRNs)** (`JB01`):
   - For single-load orders (`BOMULO ≠ 'Y'`):
     - Both `KYFS` and `KYTS` must be zero.
     - Non-zero values trigger errors ("FROM SRN MUST BE ZERO" or "TO SRN MUST BE ZERO").
   - For multi-load orders (`BOMULO = 'Y'`):
     - Both `KYFS` and `KYTS` must be zero or both must be keyed.
     - Non-zero SRNs must not exceed the total loads (`BOTOLO`).
     - `KYFS` must not be greater than `KYTS`.
     - Violations trigger errors ("BOTH FROM / TO SRN MUST BE KEYED", "FROM/TO MUST BE LESS THAN LOADS", or "FROM SRN MUST BE LESS THAN TO SRN").
5. **Job Queue**:
   - Must be `Y`, `N`, or blank.
   - Invalid values prevent program continuation.
6. **Copy Counts**:
   - `KYCOPY` and `KYCOPB` default to `01` if zero, ensuring at least one copy is printed.
7. **LDA Coordination**:
   - Input parameters are stored in the LDA (positions 101–300) to ensure consistency with `BB150` and `BB152P`.

---

### **Tables (Files) Used**

The program interacts with the following files:
1. **SCREEN** (CP, 500 bytes, WORKSTN):
   - Workstation file for user input/output, using format `BB150PFM`.
   - Fields: `KYCO`, `KYALSL`, `KYORD1`–`KYORD0`, `KYJOBQ`, `KYCOPY`, `KYCOPB`, `KYFS`, `KYTS`, `MSG35`.
2. **BICONT** (IC, 256 bytes):
   - Billing container file, used to validate the company code (`KYCO`).
3. **BBORTR** (IC, 512 bytes):
   - Order transaction file, used to validate order numbers (`KYO,X`) and check multi-load status (`BOMULO`, `BOTOLO`).

**Key Fields**:
- `KYCO` (company code), `KYALSL` (selection type), `KYORD1`–`KYORD0` (order numbers), `KYFS`/`KYTS` (from/to SRNs), `KYJOBQ` (job queue), `KYCOPY`/`KYCOPB` (copy counts), `BOMULO` (multi-load flag), `BOTOLO` (total loads).

---

### **External Programs Called**

The `BB150P` program does not explicitly call any external programs. It is designed to collect and validate input, then pass control back to the OCL, which likely invokes `BB150` to perform the actual BOL printing.

---

### **Summary**

The `BB150P` RPG program is a prompt and validation program that:
- Collects user input for company code, selection type, order numbers, SRNs, job queue, and copy counts via the `SCREEN` workstation file.
- Validates inputs against `BICONT` (company) and `BBORTR` (orders), with specific logic for single and multi-load orders (`JB01`).
- Enforces business rules for valid company codes, selection types, order numbers, SRNs, and copy counts.
- Stores validated inputs in the LDA for use by `BB150` and potentially `BB152P`.
- Displays error messages and redisplays the screen if validation fails.

**Files Used**: `SCREEN`, `BICONT`, `BBORTR`.
**External Programs**: None.

If you need further details (e.g., specific field mappings, screen format details, or interaction with `BB150`), let me know!