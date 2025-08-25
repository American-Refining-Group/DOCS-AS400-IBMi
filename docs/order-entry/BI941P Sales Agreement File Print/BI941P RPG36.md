The provided RPG program, `BI941P.rpg36.txt`, is an RPG III program (likely for IBM System/36 or AS/400) that generates a customer sales agreement list based on user input and validation. It is called from the `BI941P.ocl36` OCL program previously analyzed. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the BI941P RPG Program**

The RPG program processes user input from a workstation screen, validates it against files and tables, and prepares data for a customer sales agreement report. Here’s a detailed breakdown of the process steps:

1. **File and Data Structure Definitions** (Lines 0002–0039):
   - **Header (H) Specification**:
     - `P064`: Likely a program parameter or option.
     - Program name: `BI941P`.
   - **File Specifications (F)**:
     - `SCREEN`: A workstation file (512 bytes) for interactive user input/output.
     - `GSTABL`: An input chained file (256 bytes, indexed, 12-byte key) on disk, likely a general system table for reference data.
     - `BICONT`: An input file (256 bytes, indexed, 2-byte key) on disk, likely a control file for company data.
   - **Array and Data Definitions (E)**:
     - `DCO`: Array (35 elements) for storing company data.
     - `CS`: Array (10 elements, 6 digits) for customer numbers.
     - `COM`: Array (14 elements, 40 characters) for error messages.
   - **Input Specifications (I)**:
     - Defines fields for the `SCREEN`, `BICONT`, and `GSTABL` files, including keys like `KYALCO`, `KYCO1`, `KYALCS`, `CS`, `KYALCN`, `KYSORT`, `KYLOC`, `KYPROD`, `KYALSL`, `KYFRSL`, `KYBRND`, `KYENDT`, `KYEXPO`, `KYCLCD`, `KYGRCD`, `KYIVGR`, etc.
     - `UDS` (User Data Structure) defines additional fields for validation and display, such as `CLDESC`, `GRDESC`, `IVDESC` (descriptions for product class, group, and inventory group).
     - `Y2KCEN` and `Y2KCMP`: Fields for Year 2000 date handling (e.g., century and comparison year).

2. **Initialization and Control Logic** (Lines 0040–0058):
   - **Clear Indicators and Variables**:
     - `SETOF`: Clears indicators (31–45, 81, 90) to reset program state.
     - `MOVEL*BLANKS MSG`: Clears the message field.
   - **Key Press Handling**:
     - If the `KG` indicator (likely a function key, e.g., F3) is set:
       - Set `U1` (update) and `LR` (last record) indicators and go to `END` (terminates program).
   - **Subroutine Execution**:
     - If record format `01` (screen input) is processed, execute subroutine `S1` and go to `END`.
     - If record format `09` (initialization), execute subroutine `ONETIM` and go to `END`.

3. **Subroutine S1: Input Validation** (Lines 0062–0156):
   - Validates user input from the `SCREEN` file against business rules:
     - **Company Validation (KYALCO)**:
       - Must be `ALL` or `CO`. If invalid, set indicators 81/90, display error message `COM,1` ("ENTER ALL OR CO"), and go to `ENDS1`.
       - If `CO`, check `KYCO1` (company number):
         - If zero, display error `COM,5` ("ENTER COMPANY NUMBER").
         - Chain to `BICONT` to verify `KYCO1`. If not found or marked deleted (`BCDEL = 'D'`), display error `COM,2` ("INVALID COMPANY NUMBER ENTERED").
     - **Customer Validation (KYALCS)**:
       - Must be `ALL` or `SEL`. If invalid, display error `COM,3` ("ENTER ALL OR SEL").
       - If `SEL`, sum customer numbers in `CS` array (`XFOOTCS`). If zero, display error `COM,6` ("ENTER CUSTOMER NUMBER").
     - **Contract Validation (KYALCN)**:
       - Must be `ALL` or `CUR`. If invalid, display error `COM,7` ("ENTER ALL OR CUR").
       - If `CUR`, calculate current date (`UDATE * 10000.01`) and adjust for Y2K compliance to set `KYDAT8` (ending date).
     - **Sort Order Validation (KYSORT)**:
       - Must be `N` (name) or `S` (salesman). If invalid, display error `COM,8` ("REPORT SORT MUST BE 'N' OR 'S'").
     - **Salesman Validation (KYALSL, KYFRSL)**:
       - `KYALSL` must be `ALL` or `SEL`. If invalid, display error `COM,3`.
       - If `ALL` and `KYFRSL` (from salesman) is non-zero, display error `COM,10` ("CANNOT ENTER SALESMAN AND 'ALL'").
       - If `SEL` and `KYFRSL` is zero, display error `COM,9` ("SALESMAN NUMBER CANNOT BE ZERO").
       - If `SEL`, chain `KYFRSL` to `GSTABL` (table type `SLSMAN`). If not found, display error `COM,11` ("INVALID SALESMAN ENTERED").
     - **Product Class (KYCLCD)**:
       - If non-blank, chain to `GSTABL` (table type `PRODCL`). If not found, display error `COM,12` ("INVALID PRODUCT CLASS ENTERED"). Else, store description in `CLDESC`.
     - **Product Group (KYGRCD)**:
       - If non-blank, chain to `GSTABL` (table type `PRODGR`). If not found, display error `COM,13` ("INVALID PRODUCT GROUP ENTERED"). Else, store description in `GRDESC`.
     - **Inventory Group (KYIVGR)**:
       - If non-blank, chain to `GSTABL` (table type `INVGRP`). If not found, display error `COM,14` ("INVALID INVENTORY GROUP ENTERED"). Else, store description in `IVDESC`.
   - If any validation fails, indicators 81/90 are set, and the program jumps to `ENDS1` to display the error message on the screen.

4. **Subroutine ONETIM: Initialization** (Lines 0160–0200):
   - **Initialize Variables**:
     - Clear `DCO` array and set `BILIM` (limit) to zero.
     - Set lower limit (`SETLL`) for `BICONT` file to prepare for reading.
   - **Read BICONT File**:
     - Loop through `BICONT` records, skipping deleted records (`BCDEL = 'D'`).
     - Store company number (`BCCO`) and name (`BCNAME`) in `DCO` array.
   - **Set Default Values**:
     - Set `KYALCO = 'ALL'`, `KYCO1 = 0`, `KYALCS = 'ALL'`, `CS = 0`, `KYALCN = 'ALL'`, `KYALSL = 'ALL'`, `KYSORT = 'N'`, `KYEXPO = 'N'`.
     - Set indicator 81 to display the screen.

5. **Output to Screen** (Lines 0203–0216):
   - Writes to `SCREEN` with record format `D` (display) if indicator 81 is on.
   - Outputs fields like `KYALCO`, `KYCO1`, `DCO`, `KYALCS`, `CS`, `MSG`, `KYALCN`, `KYSORT`, `KYLOC`, `KYPROD`, `KYALSL`, `KYFRSL`, `KYBRND`, `KYENDT`, `KYEXPO`, `KYCLCD`, `KYGRCD`, `KYIVGR` to the screen format `BI941PFM`.

6. **Program Termination**:
   - The program ends after displaying the screen or if the user presses a function key (`KG`) to exit.

---

### **Business Rules**

The program enforces the following business rules for generating the customer sales agreement list:
1. **Company Selection**:
   - Must be `ALL` (all companies) or `CO` (specific company).
   - If `CO`, a valid company number (`KYCO1`) must exist in `BICONT` and not be marked deleted.
2. **Customer Selection**:
   - Must be `ALL` (all customers) or `SEL` (selected customers).
   - If `SEL`, at least one customer number must be provided in the `CS` array.
3. **Contract Selection**:
   - Must be `ALL` (all contracts) or `CUR` (current contracts).
   - For `CUR`, the ending date (`KYDAT8`) is set based on the current date, adjusted for Y2K compliance.
4. **Report Sort Order**:
   - Must be `N` (by name) or `S` (by salesman).
5. **Salesman Selection**:
   - Must be `ALL` (all salesmen) or `SEL` (specific salesman).
   - If `ALL`, no salesman number (`KYFRSL`) can be specified.
   - If `SEL`, a valid salesman number must exist in `GSTABL` under table type `SLSMAN`.
6. **Product Class, Group, and Inventory Group**:
   - If specified (non-blank), must exist in `GSTABL` under respective table types (`PRODCL`, `PRODGR`, `INVGRP`).
   - Descriptions are retrieved for valid entries.
7. **Error Handling**:
   - Invalid inputs trigger specific error messages (stored in `COM` array) and set indicators 81/90 to display the error on the screen.
8. **Date Handling**:
   - Supports Y2K compliance by adjusting dates based on `Y2KCEN` (century) and `Y2KCMP` (comparison year).

---

### **Tables Used**

The program uses the following files/tables:
- **BICONT**:
  - A control file containing company data (e.g., company number `BCCO`, name `BCNAME`, deletion flag `BCDEL`).
  - Used to validate company numbers and populate the `DCO` array with company information.
- **GSTABL**:
  - A general system table containing reference data for:
    - Salesmen (`SLSMAN` table type).
    - Product classes (`PRODCL` table type).
    - Product groups (`PRODGR` table type).
    - Inventory groups (`INVGRP` table type).
  - Fields include `TBDEL` (deletion flag), `TBTYPE` (table type), `TBCODE` (table code), `TBDESC` (description), and `TBEIN` (EIN number).

---

### **External Programs Called**

- **None**:
  - The RPG program does not explicitly call any external programs. It is designed to handle input validation and screen output internally, likely preparing data for a report that may be generated by another program (e.g., `BI941` called in the OCL script).

---

### **Summary**

- **Process Steps**:
  1. Define files (`SCREEN`, `BICONT`, `GSTABL`) and data structures.
  2. Initialize indicators and variables.
  3. Handle function key (`KG`) to exit if pressed.
  4. Execute `ONETIM` subroutine for first-time setup (populate `DCO` from `BICONT`, set defaults).
  5. Execute `S1` subroutine to validate user input for company, customer, contract, sort order, salesman, product class, group, and inventory group.
  6. Display errors or valid data on the screen.
  7. Terminate after processing or user exit.

- **Business Rules**:
  - Enforce valid inputs for company (`ALL`/`CO`), customer (`ALL`/`SEL`), contract (`ALL`/`CUR`), sort order (`N`/`S`), salesman (`ALL`/`SEL`), and optional product classAscension

System: * Today's date and time is 11:27 AM EDT on Thursday, August 21, 2025.