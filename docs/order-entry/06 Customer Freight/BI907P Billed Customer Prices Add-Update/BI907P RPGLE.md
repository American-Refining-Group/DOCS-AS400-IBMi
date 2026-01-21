The provided code is an RPGLE program (BI907P) designed to manage customer and ship-to entries within the Bradford Order Entry/Invoices system. Below is an explanation of the process steps, the external programs called, and the database tables used.

---

### Process Steps of the BI907P RPGLE Program

The BI907P program is a workstation-based interactive program that allows users to work with customer and ship-to entries using a subfile (SFL) interface. It supports both inquiry (`INQ`) and maintenance (`MNT`) modes, with functionality to create, change, or display customer/ship-to records. The program follows a structured flow, with the main logic centered around subfile processing and user interaction. Below are the key process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - Receives input parameters: `p$mode` (run mode: `MNT` for maintenance or `INQ` for inquiry) and `p$fgrp` (file group: `Z` or `G`).
   - Defines key fields, work fields, and key lists for file access.
   - Initializes variables, such as subfile relative record number (`rrn1`), page size (`pagsz1` = 28), and message handling fields.
   - Sets up the current date and time using the `TIME` operation and formats it into a data structure (`t#time`, `t#cymd`).
   - Prepares message handling by setting the program message queue (`m@pgmq`) and other message-related fields.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on the `p$fgrp` parameter (`Z` or `G`) to select the appropriate database files (e.g., `zbicont` or `gbicont`).
   - Executes override commands using the `QCMDEXC` API for files `bicont`, `arcupr`, `arcuprrd`, `gsprod`, `arcust`, and `shipto`.
   - Opens these files for input with the `usropn` option.

3. **Subfile Processing (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Clears any existing messages in the message subfile using the `clrmsg` subroutine and writes the message control record (`wrtmsg`).
   - **Initialize Subfile**: Sets the initial subfile mode to folded (`sfmod1 = '1'`, `*in45 = *on`), clears subfile control fields, and sets the company code (`c1co = 10`).
   - **Set Protection Mode**: If `p$mode = 'MNT'`, input fields are editable (`*in70 = *off`); otherwise, in inquiry mode, fields are protected (`*in70 = *on`).
   - **Position File**: Calls `sf1rep` to position the file cursor based on user input (company, customer, or ship-to).
   - **Main Loop (`sf1agn`)**:
     - Displays the command line and message subfile.
     - Checks if the subfile contains records (`rrn1 > 0`) to enable display control (`*in41`).
     - Toggles between folded/unfolded subfile display based on `sfmod1`.
     - Writes and formats the subfile control record (`sflctl1`) using `exfmt`.
     - Processes user input based on function keys or direct access:
       - **F03**: Exits the program.
       - **F04**: Prompts for field input (e.g., customer or ship-to selection).
       - **F05**: Refreshes the subfile by clearing positioning fields and repositioning.
       - **F08**: Toggles inclusion/exclusion of deleted records (`w$del`).
       - **PAGEDN**: Loads additional subfile records (`sf1lod`).
       - **ENTER**: Processes subfile selections (`sf1prc`).
       - **F10**: Positions the cursor to the control record.
       - Direct access or repositioning via control fields (`c1co`, `c1cust`, `c1ship`, `c1prod`).
     - Handles cursor location and subfile record number for redisplay.

4. **Subfile Record Processing (`sf1prc` Subroutine)**:
   - Reads changed subfile records (`readc sfl1`) if the subfile is not empty (`rrn1 > 0`).
   - Calls `sf1chg` to process each changed record based on the option selected (`s1opt`).

5. **Process Subfile Record Change (`sf1chg` Subroutine)**:
   - Stores selected customer (`s1cust`) and ship-to (`s1ship`) values.
   - Processes options:
     - **Option 2** (Change): If in maintenance mode (`p$mode = 'MNT'`), calls `sf1s02` to modify the record.
     - **Option 5** (Display): Calls `sf1s05` to display the customer order.
   - Updates the subfile record after processing by chaining to the `arcupr` file, formatting the record (`sf1fmt`), applying color coding (`sf1col`), and updating the subfile.

6. **Reposition Subfile (`sf1rep` Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets the relative record number (`rrn1`).
   - Validates control input (`sf1cte`) to ensure the company code is valid.
   - Positions the file cursor based on user input (`c1co`, `c1cust`, `c1ship`) using key lists (`kls1s1`, `kls1s2`, `kls1s3`).
   - Loads the subfile with records (`sf1lod`).
   - Retains control fields for repositioning.

7. **Validate Subfile Control Input (`sf1cte` Subroutine)**:
   - Validates the company code (`c1co`) by chaining to `bicont`.
   - Sets an error message (`ERR0010`) if the company code is invalid or zero.
   - Updates the company name (`c1conm`) if valid.

8. **Load Subfile Records (`sf1lod` Subroutine)**:
   - Sets the subfile record number to the last saved value (`rrnsv1`).
   - Loads up to `pagsz1` (28) records into the subfile.
   - Reads records from `arcuprrd` based on the key list (`kls1s1`, `kls1s2`, or `kls1s3`).
   - Filters out deleted records if `w$del = *off` and `cpdel = 'D'`.
   - Applies product filters if `c1prod` is specified.
   - Formats each subfile line (`sf1fmt`) and applies color coding (`sf1col`).
   - Writes the record to the subfile and increments `rrn1`.

9. **Format Subfile Line (`sf1fmt` Subroutine)**:
   - Clears the subfile record.
   - Populates fields (`s1cust`, `s1ship`, `s1prod`, `s1frcd`, `s1sfrt`, `s1cafr`, `s1cnty`, `s1cpds`, `s1del`) from the `arcupr` file.
   - Retrieves the product description (`s1prds`) from `gsprod` if a product code is specified.

10. **Subfile Color Coding (`sf1col` Subroutine)**:
    - Sets the color to blue (`*in72 = *on`) for deleted records (`s1del = 'D'`).

11. **Direct Access Processing (`sf1dir` Subroutine)**:
    - Validates direct input (`d1cust`, `d1ship`) for option 1 (create):
      - Ensures customer and ship-to are specified.
      - Validates customer (`arcust`) and ship-to (`shipto`) existence.
      - Checks for duplicate records in `arcupr`.
    - Processes options (1, 2, or 5) if no errors, calling `sf1s01`, `sf1s02`, or `sf1s05`.
    - Clears input fields and triggers subfile repositioning if successful.

12. **Create Record (`sf1s01` Subroutine)**:
    - Calls the external program `BI907C` in maintenance mode (`MNT`) with parameters for company, customer, ship-to, and file group.

13. **Change Record (`sf1s02` Subroutine)**:
    - Validates that the record is not deleted.
    - Calls `BI907C` in maintenance mode with the same parameters.

14. **Display Customer Order (`sf1s05` Subroutine)**:
    - Calls `BI907C` in inquiry mode (`INQ`) to display the record.

15. **Field Prompting (`prompt` Subroutine)**:
    - Handles field prompting for `C1CUST`, `C1SHIP`, `D1CUST`, `D1SHIP`, and `C1PROD` by calling external programs:
      - `LARCUST` for customer selection.
      - `LCSTSHP` for ship-to selection.
      - `LGSPROD` for product selection.
    - Updates the respective fields with selected values.

16. **Message Handling**:
    - **Add Message (`addmsg`)**: Sends error messages to the program message queue using `QMHSNDPM`.
    - **Write Message Subfile (`wrtmsg`)**: Displays the message subfile.
    - **Clear Message Subfile (`clrmsg`)**: Clears messages using `QMHRMVPM`.

17. **Program Termination**:
    - Closes all open files.
    - Sets `*inlr = *on` and returns.

---

### External Programs Called

The program calls the following external programs:
1. **BI907C**:
   - Purpose: Handles creation, modification, or display of customer/ship-to entries.
   - Called in: `sf1s01` (create), `sf1s02` (change), `sf1s05` (display).
   - Parameters: `qqco` (company), `qqcust` (customer), `qqship` (ship-to), `o$mode` (run mode: `MNT` or `INQ`), `o$fgrp` (file group: `Z` or `G`).

2. **LARCUST**:
   - Purpose: Prompts for customer selection.
   - Called in: `prompt` subroutine for fields `C1CUST` and `D1CUST`.
   - Parameters: `o$co` (company), `o$cust` (customer), `o$fgrp` (file group).

3. **LCSTSHP**:
   - Purpose: Prompts for ship-to selection.
   - Called in: `prompt` subroutine for fields `C1SHIP` and `D1SHIP`.
   - Parameters: `x$cstshp` (data structure with company, search, customer, ship-to, flag, file group).

4. **LGSPROD**:
   - Purpose: Prompts for product selection.
   - Called in: `prompt` subroutine for field `C1PROD`.
   - Parameters: `o$co` (company), `o$prod` (product), `o$fgrp` (file group).

5. **QCMDEXC**:
   - Purpose: Executes file override commands.
   - Called in: `opntbl` subroutine.
   - Parameters: `dbov##` (override command), `dbol##` (command length).

6. **QMHSNDPM**:
   - Purpose: Sends messages to the program message queue.
   - Called in: `addmsg` subroutine.
   - Parameters: `m@id` (message ID), `m@msgf` (message file), `m@data` (message data), `m@l` (data length), `m@type` (message type), `m@pgmq` (program queue), `m@scnt` (stack counter), `m@key` (message key), `m@errc` (error code).

7. **QMHRMVPM**:
   - Purpose: Removes messages from the program message queue.
   - Called in: `clrmsg` subroutine.
   - Parameters: `m@pgmq` (program queue), `m@scnt` (stack counter), `m@rmvk` (message key), `m@rmv` (remove option), `m@errc` (error code).

---

### Database Tables Used

The program uses the following database files, opened with the `usropn` option and overridden based on the file group (`Z` or `G`):
1. **bicont**:
   - Purpose: Stores company information.
   - Used in: `sf1cte` to validate company code (`c1co`) and retrieve company name (`bcname`).
   - Override: `gbicont` (G group) or `zbicont` (Z group).

2. **arcupr**:
   - Purpose: Stores customer/ship-to cross-reference data.
   - Used in: `sf1chg`, `sf1dir` to chain and validate records.
   - Override: `garcupr` (G group) or `zarcupr` (Z group).

3. **arcuprrd**:
   - Purpose: Same as `arcupr` but with a renamed record format (`arcuprpr`).
   - Used in: `sf1rep`, `sf1lod` to position and read records for subfile loading.
   - Override: `garcupr` (G group) or `zarcupr` (Z group).

4. **gsprod**:
   - Purpose: Stores product information.
   - Used in: `sf1fmt` to retrieve product descriptions (`tpdesc`) for subfile display.
   - Override: `ggsprod` (G group) or `zgsprod` (Z group).

5. **arcust**:
   - Purpose: Stores customer master data.
   - Used in: `sf1dir` to validate customer numbers.
   - Override: `garcust` (G group) or `zarcust` (Z group).

6. **shipto**:
   - Purpose: Stores ship-to data.
   - Used in: `sf1dir` to validate ship-to codes.
   - Override: `gshipto` (G group) or `zshipto` (Z group).

---

### Summary

The BI907P program is a robust interactive application for managing customer and ship-to entries, with support for creating, modifying, and displaying records in a subfile interface. It handles user input via function keys, validates data, and integrates with external programs for prompting and record maintenance. The program leverages multiple database files with dynamic overrides based on the file group, ensuring flexibility in accessing different data sets. The message handling system provides user feedback for errors and validations, enhancing the user experience.