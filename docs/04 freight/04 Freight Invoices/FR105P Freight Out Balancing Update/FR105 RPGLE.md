The RPG program `FR105` is a component of the Freight Invoicing system, designed for freight out bill balancing. It is called from the main program `FR105P` to handle detailed processing of freight invoice records for a specific company, order, and shipping reference number. The program uses a display file with two subfiles (`sfl1` and `sfl1b`) to allow users to manage carrier invoice details and view order details/miscellaneous charges. Below is a detailed explanation of the process steps, business rules, database tables used, and external programs called.

---

### Process Steps of FR105

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters**:
     - `p$co`: Company number.
     - `p$ord#`: Customer order number.
     - `p$srn`: Shipping reference number.
     - `p$mode`: Run mode (`MNT` for maintenance, `INQ` for inquiry).
     - `p$fgrp`: File group (`Z` or `G` for different file libraries).
     - `p$flag`: Return flag to indicate processing outcome.
   - Initializes variables:
     - Sets subfile control fields (`rrn1`, `rrnsv1`, `rrn1b`, `rrnsv1b`) to zero.
     - Defines page size (`pagsz1 = 8`) for subfile `sfl1`.
     - Initializes date and time fields using the system date (`ps#mdy`) and century logic.
     - Sets up message handling fields (`dspmsg`, `m@pgmq`, `m@key`) and key lists (`klc1s1`, `klfrbinh`, etc.).
     - Moves input parameters to format fields (`f$co`, `f$ord#`, `f$srn`) for display.
   - Sets the header (`c$hdr1`) for the display format.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on `p$fgrp` (`G` or `Z`) to redirect file access to the appropriate library (e.g., `gfrbinh` or `zfrbinh`) using `QCMDEXC`.
   - Opens input-only files: `frbinf`, `frcinhj1`, `bbcaid`, `sa5shy`, `sa5fjxd`, `sa5fjxm`, `apveny`.
   - Opens update files: `frbinh`, `frcinh`, `frcfbh`, `frcind`.

3. **Retrieve Data for Passed Parameters (`rtvdta` Subroutine)**:
   - Chains to `frbinh` using `klfrbinh` (company, order, shipping reference) to validate the input record.
     - If not found (`*in99 = *on`), sets `p$flag = '1'`, closes files, and exits.
   - Chains to `frbinf` to retrieve additional invoice details, clearing the record if not found.
   - Reformats dates (`boind8`, `boshd8`, `bocld8`) into display format (`f$imdy`, `f$smdy`, `f$cmdy`) using the date conversion data structure.
   - Retains shipped month (`w$smth`, `w$scy`, `w$sm`) for subfile input editing.
   - Calculates invoice total (`clcinvtot`) and carrier freight total (`clccarfrt`).

4. **Calculate Invoice Total (`clcinvtot` Subroutine)**:
   - Clears the invoice total (`f$itot`).
   - Reads sales history records (`sa5shy`) using `klfrbinh` to match company, order, and shipping reference.
   - Reads invoice detail (`sa5fjxd`) and miscellaneous records (`sa5fjxm`) using `klfix`:
     - For detail records (`samscd = *blanks`), calculates freight as `saqty * safrrt` and adds to `f$itot` if `s afr坠  *sfrt <> 'Y'`.
     - For miscellaneous records (`samscd <> *blanks`, `samsty = 'F'`), calculates freight as `samqty * samamt` and adds to `f$itot`.

5. **Calculate Carrier Freight Total (`clccarfrt` Subroutine)**:
   - Clears the carrier freight total (`f$cftl`).
 - Reads carrier invoice header records (`frcinhj1`) using `klfrbinh`, excluding deleted records (`frdel <> 'D'`).
     - Adds invoice amount (`frinam`) and ejected
     - Subtracts freight balancing order override total (`frfboa`) [jb06].
     - Updates `f$cftl` with the total.
   - Calculates the freight difference (`clcfrtdiff`) between `f$cftl` and expected (`cftfam`) or actual (`bftfam`) freight amounts.

6. **Process Subfile (`srsfl1` Subroutine)**:
   - **Clear Message Subfile**: Calls `clrmsg` and `wrtmsg` to manage messages.
   - **Load Subfile `sfl1b`**: Calls `sf1lodallb` to load all order detail and miscellaneous records.
   - **Position File**: Calls `sf1rep` to position the subfile based on user input.
   - **Main Loop**:
     - Displays the command line (`sflcmd1`) and message subfile.
     - Checks for existing records in `sfl1` and `sfl1b` to enable display indicators (`*in41`, `*in31`).
     - Displays subfile control (`sflctl1b`, `sflctl1`) and processes user input:
       - **F03**: Exits the program.
       - **F04**: Prompts for carrier ID input (`prompt`) and updates the subfile record.
       - **F05**: Refreshes the subfile (`repsfl = *on`).
       - **F06**: Adds a blank subfile record for a new carrier invoice.
       - **F09**: Updates the invoice date (`sf1upd`) and refreshes the subfile [jk01].
       - **F10**: Resets the cursor to the control record.
       - **F23**: Deletes a paper invoice record (`sf1del`) if not in inquiry mode and conditions are met (`s1inty = 'P'`, `s1apst = *blanks`) [jb05].
       - **PAGEDN**: Loads additional subfile records (`sf1lod`).
       - **ENTER**: Processes changed subfile records (`sf1prc`).
     - Handles control field changes (`sf1ctledt`, `sf1ctlupd`) and cursor positioning.

7. **Process Subfile on ENTER (`sf1prc` Subroutine)**:
   - Reads changed records in `sfl1` using `readc`.
   - Processes each changed record (`sf1chg`) and updates the subfile.

8. **Process Subfile Record Change (`sf1chg` Subroutine)**:
   - Edits subfile input (`sf1edt`).
   - Updates the database (`sf1upd`) if no errors (`*in50 = *off`) and not in inquiry mode.
   - Applies processing logic (`sf1pro`).

9. **Edit Subfile Input (`sf1edt` Subroutine)**:
   - Validates carrier ID (`s1caid`):
     - Chains to `bbcaid` (formerly `gstabl` [jk03]) to verify the carrier ID.
     - If invalid or blank, sets error message `ERR0010` and indicators (`*in50`, `*in61`).
     - Retrieves carrier name (`cicanm`) if valid.
   - Validates vendor linkage (`apveny`) for the carrier ID [jk02]:
     - If not linked, sets error message `ERR0037`.

10. **Load Subfile Records (`sf1lod` Subroutine)**:
    - Loads up to `pagsz1` (8) records into `sfl1` from `frcinhj1`, excluding deleted records (`frdel = 'D'`).
    - Formats (`sf1pro`) and writes records to the subfile.

11. **Load All Subfile Records (`sf1lodallb` Subroutine)**:
    - Clears `sfl1b` and loads sales history (`sa5shy`), invoice detail (`sa5fjxd`), and miscellaneous records (`sa5fjxm`) for the order.
    - Filters miscellaneous records to exclude blank `samscd` and include freight-related records (`samsty = 'F'`, excluding `samscd = 'F'` [jb03]).
    - Formats and writes records to `sfl1b`.

12. **Subfile Processing (`sf1pro` Subroutine)**:
    - Applies formatting and validation logic for subfile records (details omitted due to truncation).

13. **Field Prompting (`prompt` Subroutine)**:
    - Handles prompting for carrier ID (`s1caid`) by calling `LBBCAID` (formerly `LGSTABL` [jk03]) with parameters for company, carrier ID, and file group.
    - Updates `s1caid` if a valid value is returned.
    - Sets `*in19` for panel input change.

14. **Message Handling (`addmsg`, `wrtmsg`, `clrmsg` Subroutines)**:
    - **addmsg**: Sends messages to the program message queue using `QMHSNDPM`.
    - **wrtmsg**: Writes the message subfile control (`msgctl`).
    - **clrmsg**: Clears the message subfile using `QMHRMVPM`.

15. **Program Termination**:
    - Closes all files (`close *all`).
    - Sets `*inlr = *on` and returns.

---

### Business Rules

1. **Input Validation**:
   - Validates carrier ID against `bbcaid` (formerly `gstabl` [jk03]) and ensures it is linked to a vendor in `apveny` [jk02].
   - Displays error messages (`ERR0010`, `ERR0037`) for invalid or unlinked carrier IDs.
   - Validates invoice dates using `GSDTEDIT` [assumed from `pldted` parameter list].

2. **Freight Calculations**:
   - Calculates invoice total (`f$itot`) by summing freight amounts from detail (`saqty * safrrt`) and miscellaneous records (`samqty * samamt`) where `sasfrt <> 'Y'` and `samsty = 'F'`.
   - Calculates carrier freight total (`f$cftl`) from `frcinhj1`, excluding deleted records, and adjusts for freight balancing order override (`frfboa`) [jb06].
   - Computes the difference between actual and expected freight amounts (`f$diff`).

3. **Subfile Filters**:
   - Excludes deleted carrier invoice records (`frdel = 'D'`) [jb05].
   - Filters miscellaneous records to exclude `samscd = 'F'` (separate freight) [jb03].
   - Only lists detail and miscellaneous records for the specified order and shipping reference number [jb01, jb02].

4. **Mode Restrictions**:
   - In inquiry mode (`p$mode = 'INQ'`), input fields are protected (`*in70-79`), and updates/deletes are disabled.
   - In maintenance mode (`p$mode = 'MNT'`), allows updates and deletes of paper invoices (`s1inty = 'P'`, `s1apst = *blanks`) [jb05].

5. **Deletion Rules**:
   - Only paper invoices (`s1inty = 'P'`) with no A/P invoice status (`s1apst = *blanks`) can be deleted using F23 [jb05].

6. **Multi-File Logical Support**:
   - Uses multi-file logical files `sa5fjxd`, `sa5fjxm`, and `frcinhj1` to include product move records alongside invoice details [jb06].

7. **Function Key Actions**:
   - **F03**: Exits the program.
   - **F04**: Prompts for carrier ID input.
   - **F05**: Refreshes the subfile.
   - **F06**: Adds a blank carrier invoice record.
   - **F09**: Accepts invoice date changes [jk01].
   - **F10**: Positions cursor to the control record.
   - **F23**: Deletes a paper invoice record under specific conditions.

---

### Database Tables Used

1. **frbinf**:
   - Input-only file for freight invoice details.
   - Used in `rtvdta` to retrieve invoice data.

2. **frbinh**:
   - Update file for freight invoice headers.
   - Used in `rtvdta` to validate input parameters.

3. **frcinh**:
   - Update file for carrier invoice headers.
   - Used in `sf1upd` to update carrier invoice records.

4. **frcfbh**:
   - Update file for freight billed balance headers [jb06].
   - Used in `sf1upd` to update freight balancing records.

5. **frcind**:
   - Update file for carrier invoice details.
   - Used in `sf1del` and `sf1upd` for deleting and updating records.

6. **frcinhj1**:
   - Input-only multi-file logical file (replaces `frcinh1` [jb06]) for carrier invoice headers and freight billed balance headers.
   - Used in `sf1lod` and `clccarfrt` to load subfile records and calculate totals.

7. **bbcaid**:
   - Input-only file for carrier ID validation (replaces `gstabl` [jk03]).
   - Used in `sf1edt` to validate `s1caid` and retrieve carrier name (`cicanm`).

8. **sa5shy**:
   - Input-only file for sales history.
   - Used in `sf1lodallb` and `clcinvtot` to load order details and calculate invoice totals.

9. **sa5fjxd**:
   - Input-only multi-file logical file (replaces `sa5fixd` [jb06]) for invoice and product move detail records.
   - Used in `sf1lodallb` and `clcinvtot` to calculate freight amounts.

10. **sa5fjxm**:
    - Input-only multi-file logical file (replaces `sa5fixm` [jb06]) for miscellaneous and product move records.
    - Used in `sf1lodallb` and `clcinvtot` to calculate freight amounts.

11. **apveny**:
    - Input-only file for vendor data.
    - Used in `sf1edt` to ensure carrier ID is linked to a vendor [jk02].

---

### External Programs Called

1. **QCMDEXC**:
   - Called in `opntbl` to execute file override commands (`ovrdbf`).
   - Parameters:
     - `dbov##`: Override command string (80 characters).
     - `dbol##`: Command length (15.5).

2. **QMHSNDPM**:
   - Called in `addmsg` to send messages to the program message queue.
   - Parameters:
     - `m@id`: Message ID.
     - `m@msgf`: Message file (`GSMSGF`).
     - `m@data`: Message data.
     - `m@l`: Message data length.
     - `m@type`: Message type (`*DIAG`).
     - `m@pgmq`: Program message queue (`*`).
     - `m@scnt`: Stack counter.
     - `m@key`: Message key.
     - `m@errc`: Error code.

3. **QMHRMVPM**:
   - Called in `clrmsg` to clear messages from the program message queue.
   - Parameters:
     - `m@pgmq`: Program message queue.
     - `m@scnt`: Stack counter.
     - `m@rmvk`: Message key for removal.
     - `m@rmv`: Removal option (`*ALL`).
     - `m@errc`: Error code.

4. **LBBCAID**:
   - Called in `prompt` to handle carrier ID prompting (replaces `LGSTABL` [jk03]).
   - Parameters:
     - `o$co`: Company number.
     - `o$caid`: Carrier ID (input/output).
     - `o$fgrp`: File group.

5. **GSDTEDIT** (Assumed):
   - Implied by the `pldted` parameter list in `*inzsr` for date validation.
   - Parameters:
     - `p#mdy`: Date in MMDDYY format.
     - `p#cymd`: Date in CCYYMMDD format.
     - `p#err`: Error flag.

---

### Additional Notes
- The program is tightly integrated with `FR105P`, which calls `FR105` to handle detailed freight invoice processing (e.g., `sf1s01` in `FR105P`).
- Revisions enhance functionality:
  - **jb01**: Filters order detail/misc records by shipping reference number and improves freight calculations.
  - **jb02**: Includes both detail and miscellaneous records in `sfl1b`.
  - **jb03**: Excludes `samscd = 'F'` to prevent doubling of separate freight charges.
  - **jk02**: Ensures carrier IDs are linked to vendors.
  - **jb05**: Fixes deletion of carrier invoice detail records.
  - **jb06**: Supports multi-file logical files for product moves and freight balancing headers, adjusting freight calculations.
  - **jk03**: Replaces `gstabl` with `bbcaid` for carrier ID validation.
  - **jk04**: Renames duplicate fields in `s5movdpf`.
- The program uses two subfiles:
  - `sfl1`: Displays carrier invoice records.
  - `sfl1b`: Displays order detail and miscellaneous records.
- The program supports maintenance and inquiry modes, with restrictions on updates/deletes in inquiry mode.
- Error handling includes validation of carrier IDs, vendor linkage, and invoice dates, with appropriate error messages displayed in the message subfile.

This program is a critical component for managing freight out bill balancing, providing detailed control over carrier invoices and order details within an AS/400 environment.