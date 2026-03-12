The `BB104A.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is part of the Customer Order Entry system, designed to manage the cancellation or reactivation of open orders, tracking the reasons for these actions. Written by Dave Capo on 04/20/2014, it facilitates user interaction via a display file to enter or update cancellation/reactivation details. Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB104A` program processes open order cancellations or reactivations by validating input, updating the `bbcnor` file with tracking information, and displaying results via the `bb104ad` display file. The steps are as follows:

1. **Program Initialization**:
   - **File Definitions**:
     - `bb104ad`: Work station file (display file) with subfile support, user-opened.
     - `arcust`: Input-only file for customer data (keyed, user-opened).
     - `bbordh`: Input-only file for order headers (keyed, user-opened).
     - `gstabl`: Input-only file for table data (keyed, user-opened).
     - `bbcnor`: Update-capable file for order cancellation/reactivation tracking (keyed, user-opened).
   - **Data Structures and Fields**:
     - `m@80`: Array for message data (80 elements, 1 character each).
     - `ovg`, `ovz`: Arrays for file overrides (4 elements, 80 characters each) for `arcust`, `bbordh`, `gstabl`, and `bbcnor` in libraries `garcust`, `gbbordh`, `ggstabl`, `gbbcnor` (ovg) or `zarcust`, `zbbordh`, `zgstabl`, `zbbcnor` (ovz).
     - `hdr`: Array for format headers (2 elements, 80 characters each: "Open Order Cancelation - Enter Reason", "Open Order Cancelation - Update Reason").
     - `t#time`: Data structure for time conversion (e.g., `t#hms`, `t#mdcy`).
     - `d#cymd`: Data structure for date conversion (e.g., `d#cym`, 8 digits).
     - Field prefixes: `f$` (panel fields), `c$` (subfile control), `s1`/`s2` (subfile fields, unused), `k$` (key lists), `p$` (input parameters), `o$` (output parameters), `r$` (reposition subfile), `w$` (work fields).
     - Work fields: `w$wk54` (5.4), `w$wk92` (9.2), `fmtagn`, `winagn`, `delagn` (1 character flags).
     - Message fields: `dspmsg`, `m@pgmq`, `m@key`.
   - **Key Lists**:
     - `klcnor`: For `bbcnor` (company `f$co`, order number `f$ordn`).
     - `klcust`: For `arcust` (company `f$co`, customer `bocust`).
     - `klcnrs`: For `gstabl` (type `k$bborcn='BBORCN'`, code `k$cnrs`).
   - **Indicators**:
     - 19: Panel format input change.
     - 21-39, 50-69: Screen errors.
     - 40: Subfile clear.
     - 41: Subfile display control.
     - 42: Subfile display.
     - 43: Subfile end and next change.
     - 49: Message subfile display/control/initialize/end.
     - 70: Global protect in inquiry mode.
     - 80: Primary file chain.
     - 88: Read subfile.
     - 90-99: Secondary file chain/read.

2. **Parameter Input**:
   - Receives input parameters via `*ENTRY PLIST` (assumed, not shown) including:
     - `a$co` (2 characters): Company number.
     - `a$ordn` (order number, length not specified).
   - Moves input parameters to display file fields (`f$co`, `f$ordn`).

3. **Initialization (`srbeg` Subroutine)**:
   - Opens files (`arcust`, `bbordh`, `gstabl`, `bbcnor`).
   - Initializes output parameters (`o$co`, `o$fgrp`, `o$mode`, `o$flag`) to blanks.
   - Clears subfile (`*in40`) and message fields (`dspmsg`, `m@pgmq`).
   - Defines work fields (`w$wk54`, `w$wk92`) and flags (`fmtagn`, `delagn`).

4. **Order Validation and Data Retrieval**:
   - Chains to `bbordh` using `klcnor` (company `f$co`, order number `f$ordn`) to retrieve order header details (indicator 80).
   - Chains to `arcust` using `klcust` (company `f$co`, customer `bocust`) to validate customer data.
   - Chains to `gstabl` using `klcnrs` (type `BBORCN`, code `k$cnrs`) to retrieve cancellation/reactivation reason codes.

5. **Display and User Interaction**:
   - Displays the `bb104ad` panel with format headers ("Enter Reason" or "Update Reason").
   - Allows user to enter or update cancellation/reactivation details (e.g., reason code, date).
   - In inquiry mode, protects input fields (`*in70`).
   - Validates user input, including date fields using the `dtp010r` program (via `pld010` PLIST).

6. **Update Cancellation/Reactivation Records**:
   - Chains to `bbcnor` using `klcnor` to check for existing records.
   - If no record exists, adds a new record to `bbcnor` with cancellation/reactivation details (e.g., company, order number, reason, date).
   - If a record exists, updates it with new details.
   - Writes changes to `bbcnor` (update or add operation).

7. **Program Termination**:
   - Sets the last record indicator (`LR`) to exit the program.
   - Returns output parameters (`o$co`, `o$fgrp`, `o$mode`, `o$flag`) to the calling program.

---

### Business Rules

The program enforces the following business rules:

1. **Order Cancellation/Reactivation Tracking**:
   - Tracks cancellations or reactivations of open orders by storing details in `bbcnor`.
   - Requires a valid company (`f$co`) and order number (`f$ordn`) to process.
   - Validates customer data against `arcust` and reason codes against `gstabl` (type `BBORCN`).

2. **Date Validation**:
   - Uses the `dtp010r` program to validate date fields (`p#mdy` to `p#cymd`, with error flag `p#err`).
   - Ensures entered dates are valid before updating `bbcnor`.

3. **File Overrides**:
   - Uses override arrays (`ovg`, `ovz`) to redirect files to production (`g*`) or alternate (`z*`) libraries (e.g., test environments).
   - Ensures correct file access based on the environment.

4. **Inquiry Mode**:
   - Protects input fields (`*in70`) in inquiry mode, allowing only review of existing data.

5. **Error Handling**:
   - No error messages are defined in the provided snippet, but the program relies on the display file (`bb104ad`) and `dtp010r` to handle validation errors.
   - Uses indicators (21-39, 50-69) for screen errors.

---

### Tables (Files) Used

The program interacts with the following files:

1. **bb104ad**: Work station file (display file) with subfile support.
   - Fields: `f$co` (company), `f$ordn` (order number).
2. **arcust**: Input-only customer file (keyed, user-opened).
   - Fields: `bocust` (customer number).
3. **bbordh**: Input-only order header file (keyed, user-opened).
   - Contains order header details (e.g., company, order number).
4. **gstabl**: Input-only table file (keyed, user-opened).
   - Fields: `tbtype` (`BBORCN` for cancellation/reactivation reasons), `tbcode` (reason codes).
5. **bbcnor**: Update-capable file for cancellation/reactivation tracking (keyed, user-opened).
   - Fields: Company, order number, reason code, date, etc.

---

### External Programs Called

The program calls the following external program:

1. **dtp010r**:
   - Called via the `pld010` PLIST to validate date fields.
   - Parameters:
     - `p#mdy` (6 digits): Input date in MMDDYY format.
     - `p#cymd` (8 characters): Output date in CCYYMMDD format.
     - `p#err` (1 character): Error flag.

---

### Summary

The `BB104A` RPGLE program, called from the `BB101.ocl36.txt` OCL program, manages the cancellation or reactivation of open orders in the Customer Order Entry system. It validates order and customer data, allows users to enter or update cancellation/reactivation reasons via the `bb104ad` display file, and updates the `bbcnor` file. Business rules ensure valid data, proper date validation, and inquiry mode protection. The program interacts with five files (`bb104ad`, `arcust`, `bbordh`, `gstabl`, `bbcnor`) and calls `dtp010r` for date validation, providing robust tracking of order status changes.