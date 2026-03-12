The **BB802** RPG program is a Customer Order and Invoice Inquiry application written in RPGLE (RPG Language for IBM iSeries/AS400). It facilitates interactive querying of customer orders, invoices, and related data, utilizing a subfile for displaying results. Below is a detailed explanation of the process steps, followed by a list of external programs called and tables used.

---

### **Process Steps of the BB802 RPG Program**

The program follows a structured flow to handle user input, validate data, build queries, and display results in a subfile. The main steps are executed within the `srsfl1` subroutine, with supporting subroutines handling specific tasks. Here’s a breakdown of the key process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up initial values and prepares the program environment.
   - **Actions**:
     - Receives entry parameters: `p$sagr` (Sales/Accounts file group) and `p$fgrp` (file group: 'G' or 'Z').
     - Defines reposition subfile fields (`r$*`) to mirror control fields (`c$*`).
     - Initializes work fields, output parameters, and subfile control fields (e.g., `rrn1` for subfile record number, `pagsz1` for page size set to 32).
     - Sets up date and time fields using the program status data structure (`psds##`) to derive current date (`ps#dat`) in century-year-month-day format.
     - Configures message handling fields for error messaging.
     - Defines key lists (`klist`) for database access (e.g., `klordh`, `klinvh`, `klcust`).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens required database files with appropriate overrides based on the file group (`p$fgrp`).
   - **Actions**:
     - Applies file overrides for 'G' or 'Z' file groups using arrays `ovg` or `ovz` (e.g., overriding `arcust` to `garcust` or `zarcust`).
     - Executes `QCMDEXC` to apply overrides dynamically.
     - Opens input files: `arcust`, `bbcnh`, `bborcl`, `bbordh`, `gscntr`, `gsprod`, `gstabl`, `inloc`, `shipto`, `bbordhx`, `sa5shi`, `sa5shw`, `sa5shy`, `sa5shv`, `bbcaid`.
     - Opens work file `bb802wl1` with an override to `QTEMP/bb802wl1`.

3. **Process Subfile (`srsfl1` Subroutine)**:
   - **Purpose**: Manages the main logic loop for user interaction, subfile display, and data processing.
   - **Actions**:
     - Initializes subfile mode to folded (`sfmod1 = '1'`, `*in45 = *on`).
     - Clears message subfile (`clrmsg`) and writes it (`wrtmsg`).
     - Sets default values for control fields (e.g., `c$co = 10`, `c$ord = 'Y'`, `c$inv = 'Y'`).
     - Sets default "ship from" date to the first of the month, one year prior, using `ps#dat`.
     - Disables order selection if `p$sagr` is not blank.
     - Enters a loop (`sf1agn`) to:
       - Handle repositioning if required (`repsfl = *on`).
       - Toggle F10 display for product/quantity/container fields if applicable (`*in74`).
       - Display command line and message subfile.
       - Check for existing subfile records to set display indicator (`*in41`).
       - Set folded/unfolded mode based on `sfmod1`.
       - Display subfile control (`exfmt sflctl1`).
       - Clear message subfile and format indicators.
       - Process user input based on function keys and control fields.

4. **Handle User Input (Function Keys)**:
   - **F03 (Exit)**: Displays an exit window (`f03wdw`) and terminates the loop if confirmed.
   - **F04 (Field Prompting)**: Calls `prompt` subroutine to assist with field input (e.g., customer, ship-to, salesman).
   - **F05 (Refresh)**: Triggers subfile repositioning (`repsfl = *on`).
   - **F06 (History Inquiry)**: Calls `histinq` to invoke `GB730P` for order history if a subfile record is selected.
   - **F10 (Toggle Product/Quantity Info)**: Toggles `*in75` to switch between views and repositions the subfile.
   - **ENTER**: Processes subfile records (`sf1prc`) if the subfile is not empty.
   - **PAGEDN**: Loads additional subfile records (`sf1lod`).
   - **Control Field Changes**: If control fields (e.g., `c$co`, `c$ord#`, `c$inv#`) differ from reposition fields (`r$*`), triggers subfile repositioning (`sf1rep`).

5. **Process Subfile on ENTER (`sf1prc` Subroutine)**:
   - **Purpose**: Handles user selections in the subfile.
   - **Actions**:
     - Reads changed subfile records (`readc sfl1`) in a loop.
     - Calls `sf1chg` to process selections (e.g., option 5 for inquiries).
     - Supports customer order inquiry (`BB801`) or invoice inquiry (`SA801`) based on selection and invoice number presence.

6. **Process Subfile Record Change (`sf1chg` Subroutine)**:
   - **Purpose**: Processes individual subfile record selections.
   - **Actions**:
     - For option 5:
       - If `s1inv# = *zeros` (order inquiry), calls `BB801` with mode 'CNL' (canceled) or 'OPN' (open).
       - If `s1inv# <> *zeros` (invoice inquiry), calls `SA801` with customer, invoice, and order details.
     - Clears the selection option (`s1opt`) and updates the subfile record after processing.

7. **Reposition Subfile (`sf1rep` Subroutine)**:
   - **Purpose**: Rebuilds and repositions the subfile based on user input.
   - **Actions**:
     - Clears the subfile (`sf1clr`).
     - Validates control field inputs (`sf1cte`).
     - Rebuilds the work file (`bldwrkf`) if control fields have changed.
     - Positions the work file (`bb802wl1`) and loads subfile records (`sf1lod`).
     - Retains control field values in reposition fields (`r$*`).

8. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
   - **Purpose**: Validates user input in control fields.
   - **Actions**:
     - Ensures `c$ord`, `c$inv`, `c$cno`, `c$pmov` are 'Y' or 'N', setting error indicators if invalid.
     - Validates fields like order number (`c$ord#`), invoice number (`c$inv#`), purchase order (`c$pord`), rail car (`c$carn`), dates (`c$sfdt`, `c$stdt`), customer (`c$cust`), ship-to (`c$ship`), salesman (`c$slmn`), location (`c$loc`), product (`c$prod`), carrier ID (`c$crid`), container (`c$cntr`), and canceled reason (`c$cnrs`).
     - Increments occurrence counter (`w$occr`) for each non-blank/zero field.
     - Checks database files (e.g., `arcust`, `bbordh`, `sa5shi`) to ensure valid entries.
     - Calls `GSDTEDIT` for date validation.
     - Ensures at least one filter is selected; otherwise, sets an error.

9. **Build Work File (`bldwrkf` Subroutine)**:
   - **Purpose**: Constructs a query to populate the work file `bb802wl1` based on user filters.
   - **Actions**:
     - Builds a query selection string (`qryslt`) using predefined arrays (`qryoh`, `qryod`, `qrysh`, `qrysd`, `qrycn`, `qrysh2`, `qrysd2`) for open orders, sales history, and canceled orders.
     - Appends conditions for fields like company (`c$co`), customer (`c$cust`), order number (`c$ord#`), invoice number (`c$inv#`), etc.
     - Calls `BB802C` with mode 'SAD' (detail) or 'SAH' (header) based on whether location, product, or container is specified.
     - Includes additional filters for CSR (`c$tkby`), rush orders (`c$rush`), group by (`c$gpby`), etc.

10. **Retrieve Sales History for Original Order (`rtvsls2` Subroutine)**:
    - **Purpose**: Enhances query for sales history when searching by original order.
    - **Actions**:
      - Adds conditions to `qryslt` for original order number (`c$ord#`) using `qrysh2` or `qrysd2`.
      - Supports the same filters as `bldwrkf` for consistency.

11. **Field Prompting (`prompt` Subroutine)**:
    - **Purpose**: Assists users in selecting valid values for control fields.
    - **Actions**:
      - For fields like `c$cust`, `c$ship`, calls `LCSTSHP`.
      - For `c$slmn`, calls `LGSTABL`.
      - For `c$loc`, calls `LINLOC`.
      - For `c$prod`, calls `LGSPROD`.
      - For `c$crid`, calls `LBBCAID`.
      - For `c$cntr`, calls `LGSCNTR`.
      - For `c$cnrs`, calls `LGSTABL`.
      - Updates control fields with selected values.

12. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **addmsg**: Sends error messages to the program message queue using `QMHSNDPM`.
    - **wrtmsg**: Displays the message subfile (`msgctl`).
    - **clrmsg**: Clears the message subfile using `QMHRMVPM`.

13. **History Inquiry (`histinq` Subroutine)**:
    - **Purpose**: Calls the history inquiry program for a selected subfile record.
    - **Actions**:
      - Sets `o$file = 'BBORDH'` and calls `GB730P` with parameters from `x$ordrhist`.

14. **Program Termination**:
    - Closes all open files (`close *all`).
    - Sets `*inlr = *on` and returns.

---

### **External Programs Called**

The program interacts with several external programs to perform specific functions:

1. **BB801**: Customer Order Inquiry program, called when a subfile record with option 5 and no invoice number is selected.
   - Parameters: `o$co`, `o$ord#`, `o$mode`, `o$fgrp`, `o$flag`.
2. **SA801**: Customer Invoice Inquiry program, called when a subfile record with option 5 and an invoice number is selected.
   - Parameters: `o$co`, `o$cust`, `o$inv#`, `o$ord#`, `o$sagr`, `o$pmov`, `o$fgrp`, `o$flag`.
3. **BB802C**: Builds the work file `bb802wl1` based on query selection criteria.
   - Parameters: `o$mode` ('SAD' or 'SAH'), `o$dstc`, `o$pmov`, `o$fgrp`, `qryslt`.
4. **GSDTEDIT**: Validates and converts date fields.
   - Parameters: `p#mdy`, `p#cymd`, `p#err`.
5. **LCSTSHP**: Assists with customer and ship-to selection.
   - Parameters: `x$cstshp` (data structure with `x$co`, `x$srch`, `x$cust`, `x$ship`, `x$flag`, `x$fgrp`).
6. **LGSTABL**: Assists with table-based field selection (e.g., salesman, canceled reason).
   - Parameters: `k$slsman`/`k$bborcn`, `k$slmn`/`k$cnrs`, `o$fgrp`.
7. **LINLOC**: Assists with location selection.
   - Parameters: `o$co`, `o$loc`, `o$fgrp`.
8. **LGSPROD**: Assists with product selection.
   - Parameters: `o$co`, `o$prod`, `o$fgrp`.
9. **LBBCAID**: Assists with carrier ID selection.
   - Parameters: `o$co`, `o$caid`, `o$fgrp`.
10. **LGSCNTR**: Assists with container selection.
    - Parameters: `k$cntr`, `o$fgrp`.
11. **GB730P**: Order history inquiry program, called via F06.
    - Parameters: `x$ordrhist` (data structure with `o$file`, `o$fgrp`, `c$co`, `s1cust`, `s1ord#`, `o$seq#`).
12. **QCMDEXC**: Executes override commands for file access.
    - Parameters: `dbov##`, `dbol##`.
13. **QMHSNDPM**: Sends error messages to the program message queue.
    - Parameters: `m@id`, `m@msgf`, `m@data`, `m@l`, `m@type`, `m@pgmq`, `m@scnt`, `m@key`, `m@errc`.
14. **QMHRMVPM**: Removes messages from the program message queue.
    - Parameters: `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv`, `m@errc`.

---

### **Tables Used**

The program accesses the following database files, all defined with `usropn` for manual opening and overrides:

1. **arcust**: Customer master file.
2. **bbcnh**: Canceled orders header file.
3. **bborcl**: Order control file.
4. **bbordh**: Order header file.
5. **gscntr**: Container master file.
6. **gsprod**: Product master file.
7. **gstabl**: General table file (used for salesman, canceled reason, etc.).
8. **inloc**: Location master file.
9. **shipto**: Ship-to master file.
10. **bbordhx**: Order header file (renamed format `bbordpx`).
11. **sa5shi**: Sales history header file (renamed format `sa5shppi`).
12. **sa5shw**: Sales history header file (renamed format `sa5shppw`).
13. **sa5shy**: Sales history header file (renamed format `sa5shppy`).
14. **sa5shv**: Sales history header file (renamed format `sa5shppv`, added by jk01).
15. **bbcaid**: Carrier ID file (added by jk10).
16. **bb802wl1**: Work file in `QTEMP` for storing query results.

---

### **Additional Notes**

- **Indicator Usage**: The program uses indicators (19, 21–39, 40–43, 49, 50–69, 70–79, 80, 88, 90–99) to control screen behavior, subfile operations, and error handling.
- **Field Prefixes**: Fields are prefixed (e.g., `f$` for display file fields, `c$` for subfile control, `s1` for subfile fields) to organize data usage.
- **Revisions**: The program has been updated multiple times (e.g., jk01 for original order search, jk03 for F10 toggle, jk09 for file override to `QTEMP`, jk10 for carrier ID table change).
- **Query Selection**: The program dynamically builds SQL-like query selection strings (`qryslt`) using predefined arrays to filter data based on user input.
- **Subfile Management**: The subfile (`sfl1`) supports folded/unfolded views, pagination, and user selections for inquiries.

This program is a robust inquiry tool designed for flexibility in querying order and invoice data, with extensive validation and user assistance features.