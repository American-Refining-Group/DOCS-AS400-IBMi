The provided document, `AP296.rpg36.txt`, is an RPG program (likely RPG II or RPG III) running on an IBM System/36 or AS/400, called by the OCL program `AP296.ocl36.txt` for the Accounts Payable (A/P) Voucher Status change Post process. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called.

---

### Process Steps of the AP296 OCL RPG Program

The RPG program `AP296` processes A/P Open Payable data, updating statuses and generating a report for vouchers with modified status codes (e.g., hold codes). Here’s a step-by-step breakdown of the program’s logic based on the provided code:

1. **File and Data Structure Definitions**:
   - **Input Files**:
     - `APVCTR` (Input Primary, `IP`): Work file for transactions (80 bytes, indexed by 12-byte key).
     - `APCONT` (Input Full-Procedural, `IF`): A/P control file (256 bytes, indexed by 2-byte key).
     - `APVEND` (Input Full-Procedural, `IF`): Vendor master file (579 bytes, indexed by 7-byte key).
     - `APOPNH` (Update Full-Procedural, `UF`): Open A/P transactions file (384 bytes, indexed by 16-byte key).
   - **Output File**:
     - `LIST` (Output, `O`): Printer file for generating a report (132 bytes).
   - **Data Structures**:
     - `APVCTR NS 01`: Defines fields like `ATCONOL2` (company number), `ATVENDL1` (vendor), `ATVOUC` (voucher), `ATHOLD` (hold code), `ATHLDS` (hold description), `ATOHLD` (prior hold code), `ATOHDS` (prior hold description), and `ATNKEY` (key for chaining).
     - `APCONT NS`: Defines `ACNAME` (company name).
     - `APVEND NS`: Defines `VNDEL` (delete flag), `VNCO` (company number), `VNVEND` (vendor number), `VNNAME` (vendor name).
     - `APOPNH NS`: Defines fields like `OPDEL` (delete flag), `OPCONO` (company number), `OPVEND` (vendor number), `OPVONO` (voucher number), `OPRCTY` (record type), `OPSEQN` (sequence number), and many others (e.g., amounts, dates, G/L accounts, hold flags).
     - `UDS`: User Data Structure with `Y2KCEN` (century, e.g., "19") and `Y2KCMP` (company number, e.g., "80").
   - **Array**:
     - `SEP`: A 66-element array with 2-byte entries, initialized to `"* "` for report formatting.

2. **Initialization and Setup** (Calculation Specs, `C`):
   - **Time and Date Handling**:
     ```c
     C                     TIME           TIMDAT 120
     C                     MOVELTIMDAT    SYSTIM  60
     C                     MOVE TIMDAT    SYSDAT  60
     ```
     - Retrieves the system time (`TIME`) into `TIMDAT` (12 bytes).
     - Extracts the time portion (`SYSTIM`, 6 bytes) and date portion (`SYSDAT`, 6 bytes) for report headers.
   - **Report Initialization**:
     ```c
     C                     MOVE '* '      SEP
     C                     Z-ADD*ZEROS    PAGE
     ```
     - Sets the `SEP` array to `"* "` for report separators.
     - Initializes the page number (`PAGE`) to zero.
   - **Indicator Setup**:
     ```c
     C                     SETOF                     50
     ```
     - Turns off indicator `50`, which is later used to track whether a voucher has a prior hold code.

3. **Company Validation**:
   ```c
   C           ATCONO    CHAINAPCONT               92
   ```
   - Chains (looks up) the `APCONT` file using the company number (`ATCONO`) from `APVCTR`.
   - If no record is found, indicator `92` is set on, potentially skipping further processing for invalid companies.

4. **Hold Code Check**:
   ```c
   C           ATOHLD    IFEQ *BLANKS
   C           ATOHDS    IFEQ *BLANKS
   C                     SETON                     50
   C                     END
   C                     END
   ```
   - Checks if the prior hold code (`ATOHLD`) and prior hold description (`ATOHDS`) in `APVCTR` are blank.
   - If both are blank, sets indicator `50` on, indicating no prior hold code exists for the voucher.

5. **Vendor Lookup**:
   ```c
   C                     MOVELATCONO    VENDKY  70
   C                     MOVE ATVEND    VENDKY
   C           VENDKY    CHAINAPVEND               31
   ```
   - Builds a key (`VENDKY`, 7 bytes) by combining `ATCONO` (company number) and `ATVEND` (vendor number) from `APVCTR`.
   - Chains to the `APVEND` file to retrieve vendor details (e.g., `VNNAME`).
   - If no vendor record is found, indicator `31` is set on.

6. **Open A/P Record Lookup and Update**:
   ```c
   C                     MOVELATNKEY    OPNKEY 16
   C                     MOVE 1001      OPNKEY
   C           OPNKEY    CHAINAPOPNH               90
   C  N90                EXCPT
   ```
   - Builds a key (`OPNKEY`, 16 bytes) using `ATNKEY` (from `APVCTR`) and appends a constant `1001` (likely a sequence or record type identifier).
   - Chains to the `APOPNH` file to locate the corresponding open A/P record.
   - If found (indicator `90` off), updates the `APOPNH` file with:
     ```o
     OAPOPNH  E
     O                         ATHOLD   126
     O                         ATHLDS   151
     ```
     - Writes the new hold code (`ATHOLD`) to position 126 and hold description (`ATHLDS`) to positions 127–151 in the `APOPNH` record.
   - Executes an exception output (`EXCPT`) to write to the `LIST` file (report).

7. **Report Generation** (Output Specs, `O`):
   - The program generates a report via the `LIST` printer file with the following structure:
     - **Header (Level 2, `L2`)**:
       ```o
       OLIST    D  103   L2
       O       OR        OFNL2
       O                N92      ACNAME    30
       O                                  104 'PAGE'
       O                         PAGE  Z  108
       O                                  120 'DATE'
       O                         SYSDATY  129
       O        D  2     L2
       O       OR        OFNL2
       O                                   77 'A/P VOUCHER STATUS CODE '
       O                                   88 'MODIFY POST'
       O                                  120 'TIME'
       O                         SYSTIM   129 '  :  :  '
       ```
       - Prints the company name (`ACNAME`), page number (`PAGE`), date (`SYSDAT`), and time (`SYSTIM`).
       - Includes a title: *"A/P VOUCHER STATUS CODE MODIFY POST"*.
     - **Column Headings**:
       ```o
       O        D  1     L2
       O       OR        OFNL2
       O                                   22 'VENDOR'
       O                                   50 'VOUCHER'
       O                                   66 'STATUS'
       O                                   81 'CODE'
       O                                  112 'CURRENT'
       O        D  1     L2
       O       OR        OFNL2
       O                                    8 'VENDOR'
       O                                   20 'NAME'
       O                                   50 'NUMBER'
       O                                   64 'CODE'
       O                                   85 'DESCRIPTION'
       O                                  111 'VALUES'
       ```
       - Prints column headers for vendor, voucher, status code, description, and current values.
     - **Detail Lines**:
       ```o
       O        D  2     L2
       O       OR        OFNL2
       O                         SEP      132
       O        D  1     01
       O                         ATVEND     7
       O                         VNNAME    40
       O                         ATVOUC    49
       O                         ATHOLD    62
       O                         ATHLDS    98
       O                N50      ATOHLD   105
       O                N50      ATOHDS   132
       O                 50               122 'NO PRIOR HOLD CODE'
       ```
       - Prints detail lines for each processed voucher, including:
         - Vendor number (`ATVEND`), vendor name (`VNNAME`), voucher number (`ATVOUC`), new hold code (`ATHOLD`), and new hold description (`ATHLDS`).
         - If no prior hold code exists (indicator `50` on), prints *"NO PRIOR HOLD CODE"*.
         - If prior hold codes exist (indicator `50` off), prints prior hold code (`ATOHLD`) and description (`ATOHDS`).
       - Uses the `SEP` array for line separation.

8. **Loop Control**:
   ```c
   C   L2                DO                               B1
   C                     END                              E1
   ```
   - The program processes records in a level-2 (`L2`) loop, iterating through `APVCTR` records until completion.

---

### Business Rules

Based on the program’s logic, the following business rules apply:
1. **Data Validation**:
   - The program validates the company number (`ATCONO`) against the `APCONT` file to ensure it exists.
   - It verifies vendor numbers (`ATVEND`) in the `APVEND` file.
   - It ensures open A/P records exist in `APOPNH` for the given voucher and key.

2. **Hold Code Updates**:
   - The program updates the hold code (`ATHOLD`) and hold description (`ATHLDS`) in the `APOPNH` file for matching vouchers.
   - It checks if prior hold codes/descriptions (`ATOHLD`, `ATOHDS`) are blank to determine if a voucher previously had no hold status.

3. **Reporting**:
   - Generates a report listing modified vouchers with their vendor details, voucher numbers, new hold codes/descriptions, and prior hold codes/descriptions (if applicable).
   - Includes company name, date, time, and page number in the report header.
   - Indicates when no prior hold code exists for clarity.

4. **Error Handling**:
   - If no company record is found (`92` on), the program may skip processing or produce partial output.
   - If no vendor record is found (`31` on), the report may omit vendor names.
   - If no open A/P record is found (`90` on), no update occurs, but the program may still produce report output via `EXCPT`.

5. **Data Integrity**:
   - Updates are written only to matched `APOPNH` records, ensuring data consistency.
   - The program uses shared files (`DISP-SHR` in OCL), implying concurrent access is managed by the system.

---

### Tables (Files) Used

The RPG program uses the following files:
1. **APVCTR** (Input Primary, `IP`): Work file containing check reconciliation transactions (e.g., company number, vendor, voucher, hold codes).
2. **APCONT** (Input Full-Procedural, `IF`): Control file with company details (e.g., company name).
3. **APVEND** (Input Full-Procedural, `IF`): Vendor master file with vendor details (e.g., vendor number, name).
4. **APOPNH** (Update Full-Procedural, `UF`): Open A/P transactions file, updated with new hold codes and descriptions.
5. **LIST** (Output, `O`): Printer file for generating the reconciliation report.

---

### External Programs Called

No external RPG programs are explicitly called within `AP296.rpg36.txt`. The program relies on internal logic and system operations (e.g., `CHAIN`, `EXCPT`) to perform its tasks. The `SFACOPY` command referenced in the OCL program is a system utility, not an RPG program, and is handled outside the RPG code.

---

### Summary

The `AP296` RPG program:
- Processes A/P vouchers by reading transactions from `APVCTR`, validating company and vendor data, and updating hold codes/descriptions in `APOPNH`.
- Generates a detailed report (`LIST`) showing modified vouchers, including vendor details, voucher numbers, new and prior hold codes, and descriptions.
- Uses a loop (`L2`) to process records, with validations for company, vendor, and open A/P records.
- Applies business rules to ensure data integrity, proper hold code updates, and clear reporting.

**Tables Used**: `APVCTR`, `APCONT`, `APVEND`, `APOPNH`, `LIST`.
**External Programs Called**: None (only system utilities like `SFACOPY` in the OCL).