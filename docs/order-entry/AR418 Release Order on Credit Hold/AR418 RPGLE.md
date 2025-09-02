The provided document is an **RPGLE program** named `AR418.rpgle`, which is called from the `AR418.ocl36` OCL program in an IBM System/36 or AS/400 environment. This program handles the **Order Authorization Credit Limit Update** process, allowing authorized users to override credit limits for orders, place orders on credit hold, or unauthorize them. It includes functionality to send email notifications to customer service representatives (CSRs) and salesmen. Below, I will explain the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the AR418.rpgle Program**

The RPGLE program is designed to interact with a workstation screen (`ar418fm`) and process user inputs to manage order authorizations. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - **Header Specifications**:
     - `DFTACTGRP(*NO)`: Ensures the program does not run in the default activation group, allowing for better control in an ILE environment.
     - `DFTNAME(ar418)`: Sets the default program name to `AR418`.
     - `FIXNBR(*ZONED:*INPUTPACKED)`: Ensures zoned and packed decimal fields are handled correctly during input.
   - **File Declarations**:
     - Declares files for workstation interaction (`ar418fm`), input files (`bicont`, `bborcl`, `bbcsr`, `bbslsm`, `arcust`, `gscont`), and output printer files (`cremal`, `smemal`).
   - **Data Structures**:
     - Defines a data area (`UDS`) to store `kyco` (company number) and `kyuser` (user ID).
     - Defines arrays `com` (11 elements) for error messages and `msga` (1 element) for a prompt message.

2. **Workstation File Read**:
   - Checks if the control field `qsctl` is blank:
     - If blank, sets indicators `*in09` (on) and `*in01` (off), sets `qsctl` to `'R'`, and proceeds.
     - If not blank, sets `*in09` (off), `*in01` (on), reads the workstation file (`ar418fm`), and returns if the last record is read (`lr`).

3. **Indicator Setup**:
   - Resets indicators `*in30`, `*in31`, `*in32`, `*in33`, `*in34`, `*in90` to off to clear error or status flags.
   - Clears message fields `msg` and `msga1`.

4. **One-Time Setup (Subroutine `onetim`)**:
   - **Access Restriction**:
     - Checks if the user (`kyuser`) is one of the authorized users (`BRIANZ`, `OTTO`, `MARTY`, `FRANCO`, `GSSZTEST`, `JLBZTEST`, `JBTEST`, `MIDLA`, `SPAHR`). If so, skips time-based restrictions.
     - If not an authorized user, checks the current time (`hour`):
       - If the time is between 06:00 and 20:00 (business hours), sets `*in81` and `*in90` on, displays error message `com(11)` ("CANNOT RUN DURING NORMAL BUSINESS HOURS"), and exits.
   - **Company Number Setup**:
     - Chains to `gscont` with key `'00'` to retrieve the company number (`gxcono`).
     - If found and not zero, sets `kyco` to `gxcono`.
     - Sets `*in81` and `*in91` on to prepare for screen display.

5. **Main Processing Loop**:
   - **Indicator `*in09` Check**:
     - If `*in09` is on, executes the `onetim` subroutine and exits to the `end` tag.
   - **Indicator `kg` Check**:
     - If `kg` is on, resets indicators `*in01`, `*in09`, `*in81`, sets `*inlr` (last record) on, and exits.
   - **Indicator `*in01` Processing**:
     - Chains to `arcust` using `kyco` and `kycust` to retrieve customer data.
     - If `kyplhd = 'Y'` (place on hold):
       - Chains to `bborcl` with `orky14` (order key).
       - Captures the current time and date (`timdat`, `sytime`, `sydate`).
       - Writes to the `hold` exception (updates `bborcl` with hold status).
       - Resets indicators and clears input fields via `clear` subroutine.
     - If `kyunau = 'Y'` (unauthorize):
       - Chains to `bborcl` with `orky14`.
       - Retrieves CSR and salesman email addresses from `bbcsr` and `bbslsm` using `bltkby` and `blslmn`.
       - Writes to the `unau` exception (clears authorization fields in `bborcl`).
       - Resets indicators and clears input fields.
     - If `kyunau <> 'Y'` (authorize):
       - Resets indicators, sets `*in35` and `*in81` on, and clears input fields.
   - **Screen Processing**:
     - If `*in01`, `*in91`, and `*in92` are on, calls the `screen1` subroutine to validate input and display results.

6. **Screen Processing (Subroutine `screen1`)**:
   - **Validate Company Number**:
     - Chains to `bicont` with `kyco`. If not found or marked as deleted (`bcdel = 'D'`), sets `*in81`, `*in90` on, displays `com(1)` ("INVALID COMPANY NUMBER ENTERED"), and exits.
   - **Validate Order**:
     - Chains to `bborcl` with `orky14` (constructed from `kyco`, `kycust`, `kyord#`).
     - If not found, sets `*in81`, `*in90`, `*in31` on, displays `com(2)` ("INVALID CO/CUST/ORDER# ENTERED"), and exits.
     - If marked as deleted (`bldel = 'D'`), sets `*in81`, `*in90`, `*in31` on, displays `com(8)` ("ORDER HAS BEEN DELETED FROM FILE"), and exits.
   - **Authorization Checks**:
     - If `kyplhd = 'Y'` (place on hold) and `blovcl = 'Y'` (over credit limit) and `blauin` is blank, displays `com(10)` ("ORDER IS ALREADY ON HOLD").
     - If `kyplhd <> 'Y'` (not on hold) and `blovcl <> 'Y'` (not over limit) and `blauin` is not blank, displays `com(6)` ("ORDER DOES NOT NEED AUTHORIZATION").
     - If `kyauin` is blank (no authorization initials), displays `com(3)` ("MUST ENTER AUTHORIZATION INITIALS").
     - If `kyplhd <> 'Y'`, `blauin` and `blusid` are not blank, displays `com(5)` ("ORDER HAS ALREADY BEEN AUTHORIZED") and `msga(1)` ("TO UNAUTHORIZE THIS ORDER ENTER A 'Y'").
   - **Process Valid Record**:
     - Retrieves CSR email (`edicu`) from `bbcsr` using `bltkby` (order taken by), defaulting to `'SJM'` if not found.
     - Retrieves salesman email (`edism`) from `bbslsm` using `blslmn`.
     - Retrieves notification email (`ediar`) from `bbcsr` using `kyauin` or `blauin`.
     - Captures current time and date.
     - If `kyplhd <> 'Y'`, writes to the `auth` exception (updates `bborcl` with authorization details) and displays `com(7)` ("ORDER HAS BEEN AUTHORIZED").
     - If `kyplhd = 'Y'`, writes to the `hold` exception and displays `com(9)` ("ORDER HAS BEEN PLACED ON HOLD").
     - Clears input fields via `clear` subroutine.

7. **Output Handling**:
   - Writes to `cremal` and `smemal` printer files for email notifications:
     - For `auth`: Outputs "RELEASED FROM CREDIT HOLD" with order details, customer info, credit limit, and payment history.
     - For `unau`: Outputs "PLACED ON CREDIT HOLD" or "UNAU DENIED" with similar details.
   - The `JBLIST` printer file is commented out, indicating it is no longer used for tracking changes.

8. **Program Termination**:
   - If `*in81` is off, sets `*inlr` on to end the program.
   - Writes to the workstation file (`ar418fm`) to display the updated screen.

---

### **Business Rules**

1. **Access Control**:
   - Only specific users (`BRIANZ`, `OTTO`, `MARTY`, `FRANCO`, `GSSZTEST`, `JLBZTEST`, `JBTEST`, `MIDLA`, `SPAHR`) can run the program during business hours (06:00–20:00). Others are restricted to non-business hours to prevent disruptions.

2. **Order Validation**:
   - The company number (`kyco`) must exist in `bicont` and not be marked as deleted.
   - The order (`kyco`, `kycust`, `kyord#`) must exist in `bborcl` and not be marked as deleted.
   - Authorization initials (`kyauin`) are required to process an order.

3. **Authorization Logic**:
   - If an order is over the credit limit (`blovcl = 'Y'`) and not yet authorized (`blauin` is blank), it can be placed on hold (`kyplhd = 'Y'`).
   - If an order does not need authorization (`blovcl <> 'Y'` and `blauin` is not blank), the user is notified it’s already authorized.
   - To unauthorize an order, the user must enter `kyunau = 'Y'`.

4. **Email Notifications**:
   - When an order is authorized or placed on hold, notifications are sent to:
     - The CSR (`edicu`) based on `bltkby` from `bborcl`.
     - The salesman (`edism`) based on `blslmn` from `bborcl`.
     - A credit manager or A/R clerk (`ediar`) based on `kyauin` or `blauin`.
   - Notifications include order details, customer information, and payment history.

5. **Data Updates**:
   - Authorization updates `bborcl` with user initials (`kyauin`) and user ID (`kyuser`).
   - Unauthorization clears authorization fields in `bborcl`.
   - Holding an order sets `blovcl = 'Y'` in `bborcl`.

---

### **Tables (Files) Used**

The program uses the following files:

1. **ar418fm**:
   - Type: Workstation file (CF, externally described).
   - Purpose: Displays the user interface for input and output, handled by `PROFOUNDUI(HANDLER)`.

2. **bicont**:
   - Type: Input file (IF, 256 bytes, indexed).
   - Fields: `bcdel` (delete code).
   - Purpose: Stores company control data for validation.

3. **bborcl**:
   - Type: Update file (UF, 256 bytes, indexed).
   - Fields: `bldel` (delete code), `blcono` (company), `blcust` (customer), `blordr` (order), `blbtch` (batch), `bltamt` (transaction amount), `bloamt` (order amount), `blovcl` (over limit flag), `blauin` (authorization initials), `blusid` (user ID), `blor$$` (original order amount), `blpord` (purchase order), `blprtl` (product total), `blslmn` (salesman), `bltkby` (taken by).
   - Purpose: Stores order details and authorization status.

4. **bbcsr**:
   - Type: Input file (IF, 192 bytes, indexed).
   - Fields: `crdel` (delete code), `crco` (company), `crcaid` (CSR ID), `crcanm` (CSR name), `qqcrem` (email address).
   - Purpose: Stores CSR data, including email addresses for notifications.

5. **bbslsm**:
   - Type: Input file (IF, 192 bytes, indexed).
   - Fields: `smdel` (delete code), `smco` (company), `smcaid` (CSR ID), `smcanm` (CSR name), `qqsmem` (email address).
   - Purpose: Stores salesman data, including email addresses for notifications.

6. **arcust**:
   - Type: Input file (IF, 384 bytes, indexed).
   - Fields: `ardel` (delete code), `arco` (company), `qqarcu` (customer number), `arname` (name), `aradr1`, `aradr2`, `aradr3` (address), `artotd` (total due), `arcurd` (current due), `ar0110` (1–30 days overdue), `ar1120` (31–60 days overdue), `ar2130` (61–90 days overdue), `arov30` (over 90 days overdue), `arpdat` (last payment date), `arclmt` (credit limit), `ararea` (area code), `artele` (telephone).
   - Purpose: Stores customer master data, including credit and payment details.

7. **gscont**:
   - Type: Input file (IF, 512 bytes, indexed).
   - Fields: `gxdel` (delete code), `gxcono` (company number).
   - Purpose: Stores global control data, used to retrieve the company number.

8. **cremal**:
   - Type: Output file (O, 140 bytes, printer).
   - Purpose: Generates email notifications for CSRs or credit managers with order status details.

9. **smemal**:
   - Type: Output file (O, 140 bytes, printer).
   - Purpose: Generates email notifications for salesmen with order status details.

10. **ist132**:
    - Type: Output file (O, 132 bytes, printer).
    - Purpose: Likely a legacy report file (possibly related to `JBLIST`, which is commented out).

---

### **External Programs Called**

The RPGLE program does not explicitly call any external programs via `CALL` operations. All logic is handled within the program, using subroutines (`screen1`, `onetim`, `clear`) and file operations.

---

### **Summary**

The `AR418.rpgle` program, called by the `AR418.ocl36` OCL program, manages the process of authorizing or placing orders on credit hold, with the following key features:
- Validates company, customer, and order data.
- Restricts access to specific users during business hours.
- Updates order authorization status in `bborcl`.
- Sends email notifications to CSRs, salesmen, and credit managers via `cremal` and `smemal`.
- Displays error messages and prompts via the `ar418fm` workstation file.

The program uses a robust set of business rules to ensure data integrity and proper authorization workflows, with extensive file I/O and error handling to support the credit limit override process.