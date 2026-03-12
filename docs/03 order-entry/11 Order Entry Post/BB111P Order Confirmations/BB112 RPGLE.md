### Process Steps of the RPG Program BB112

This RPGLE program (BB112) is an interactive inquiry tool for customer order confirmations in a customer orders system. It provides a subfile-based screen where users can enter selection criteria to view and interact with open orders (e.g., filtering by company, order number, customer, ship-to, etc.). The program supports prompting, validation, message handling, and actions like sending confirmations. It uses overrides to access files from different library groups (e.g., G or Z) based on input parameters. The main flow is initialization, file opening, subfile processing in a loop (display, input handling, repositioning/loading), and termination. Below is a step-by-step breakdown based on the code structure and subroutines:

1. **Program Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: `P$FGRP` (file group: 'G' or 'Z' for library overrides).
   - Defines work fields, reposition fields (e.g., `R$CO`, `R$ORD#`), output parameters (e.g., `O$CO`, `O$ORD#`), and defaults (e.g., `O$MODE` to blanks, `O$FLAG` to '0').
   - Sets subfile control variables (e.g., `RRN1` to 0, page size `PAGSZ1` to 24).
   - Calculates current date/time from program status data structure (`PSDS##`), derives century (`PS#CEN`), year/month/day formats (`PS#YMD`, `PS#DAT`), and handles Y2K logic (e.g., if year >=40, century=19; else 20).
   - Initializes message handling (e.g., `DSPMSG` off, program message queue to '*').
   - Defines keylists (e.g., `KLORDH` for order header chaining by company/order#, `KLCUST` for customer, etc.) for file access.

2. **Open Database Tables (OPNTBL Subroutine)**:
   - Applies file overrides via `QCMDEXC` API based on `P$FGRP`:
     - If 'G' or 'Z', loops through arrays `OVG` or `OVZ` (8 overrides each) for files like ARCUST, BBORCL, etc., redirecting to libraries like *LIBL/GARCUST or *LIBL/ZARCUST.
     - Additionally, overrides BB112WL1 to QTEMP/BB112WL1 (from `OVR` array, added in revision jk01).
   - Opens all declared files with USROPN (user-controlled open): ARCUST, BBORCL, BBORDH, GSCNTR, GSPROD, GSTABL, INLOC, SHIPTO, and BB112WL1.

3. **Process Subfile (SRSFL1 Subroutine - Main Interactive Loop)**:
   - Sets initial subfile mode to folded (`#FOLD='1'`, indicator 45 on).
   - Clears and writes message subfile.
   - Defaults company (`C$CO`) to 10, repositions to first record, and sets ship-from date (`C$SFDT`) to the 1st of the month one year ago (using date conversion DS).
   - Calls SF1REP to reposition/load the subfile initially.
   - Enters a loop (`SF1AGN=*ON`) for user interaction:
     - If reposition needed (`REPSFL=*ON`), clears selection fields and calls SF1REP.
     - Writes command line (SFLCMD1), displays messages if any.
     - Conditions subfile display (indicator 41 on if `RRN1>0`), sets fold/unfold (indicator 45).
     - Displays subfile control (EXFMT SFLCTL1, indicator 40 on).
     - Clears messages and format indicators (50-69 off, 21-39 off).
     - Determines cursor position from INFDS (`CSRSRLOC`).
     - Initializes subfile record number (`RCDNB1`) with page RRN for redisplay.
     - Processes user input BEFORE subfile read:
       - F3: Displays exit confirmation window (EXFMT F03WDW); if confirmed, sets `SF1AGN=*OFF` to exit loop.
       - F4: Calls PROMPT subroutine to handle field-specific prompting (e.g., for order#, customer, etc., using keylists and chains).
       - F5: Sets `ONETIME=*ON` and `REPSFL=*ON` for refresh.
       - Position changes (e.g., `C$CO <> R$CO`): Calls SF1REP to reload subfile.
       - Page Down: Calls SF1LOD to load more records (commented as "fill subfile" for non-load-all method).
     - If Enter pressed, calls SF1PRC to process subfile.
     - Processes user input AFTER subfile read:
       - F10: Positions cursor to control record.
   - Loop continues until exit (F3 confirmed).

4. **Subfile Reposition/Load (SF1REP Subroutine)**:
   - Clears subfile (indicator 40 on for SFLCLR).
   - Builds query select strings (`QRYSLT`) using arrays QRYOH (for order header) and QRYOD (for order detail), incorporating selection criteria (e.g., company, customer, order status not 'D'eleted, date ranges).
   - Positions files (e.g., SETLL on BBORDH using keylists).
   - Loads subfile records by reading files (e.g., BBORDH, BBORCL), chaining for details (e.g., customer name from ARCUST, ship-to from SHIPTO), and writing to SFL1.
   - Handles "no results" message if no records loaded.
   - Updates reposition fields (e.g., `R$CO = C$CO`).

5. **Process Subfile on Enter (SF1PRC Subroutine)**:
   - If subfile not empty (`RRN1>0`), reads changed records (READC SFL1 until EOF, indicator 81).
   - For each changed record, calls SF1CHG.

6. **Process Subfile Record Change (SF1CHG Subroutine)**:
   - Processes option field (`S1OPT`):
     - If 6 (Customer Order Confirmation): Calls external program BB112SC with parameters (company, order# , file group); sets message "Confirmation has been sent for Order # [order#]" using COM array, calls ADDMSG.
   - Updates subfile record (CHAIN and UPDATE SFL1).
   - Clears option field.

7. **Field Prompting (PROMPT Subroutine)**:
   - Determines cursor row/column and field/record name.
   - Selects based on record/field:
     - For SFLCTL1 fields (e.g., `C$ORD#`): Chains to files (e.g., BBORDH for order#, ARCUST for customer), prompts windows if needed, validates, and sets values.
     - For SFL1: Displays credit window (EXFMT CRDWDW) for credit limit field.
   - Handles commented-out calls (e.g., to LGSTABL for carrier reason).

8. **Message Handling**:
   - ADDMSG: Builds message data, calculates length, sends via QMHSNDPM API, sets `DSPMSG=*ON`.
   - WRTMSG: Writes message control (MSGCTL), indicators 49 on for display.
   - CLRMSG: Removes messages via QMHRMVPM API, restores record/page, sets `DSPMSG=*OFF`.

9. **Program Termination**:
   - Closes all files.
   - Sets `*INLR=*ON` and returns.

The program uses a "load-on-demand" approach for the subfile (via Page Down), with dynamic queries for filtering open orders.

### Business Rules

- **Selection Criteria**: Users must enter at least one value (error message from COM(01) if not). Supports filters like company (default 10), order#, PO#, customer, ship-to, salesman, location, product, carrier ID, country. Validates inputs (e.g., chains to ensure existence).
- **Order Scope**: Focuses on open orders (status not 'D'eleted, order print flag 'C' or 'R' for confirmation/reprint). Includes date ranges (e.g., requested date >=1 year ago default).
- **Credit Limits**: Displays indicators for over-credit-limit (unauthorized/authorized, messages from COM(03/04)).
- **Restrictions**: Cannot search tracking# if invoices excluded (COM(05)); cannot search cancel reason without including canceled orders (COM(06)).
- **Subfile Interaction**: Supports fold/unfold (indicator 45), options (e.g., 6 for confirmation send), reposition on changes.
- **File Overrides**: Based on `P$FGRP` ('G' or 'Z'), redirects files to group-specific libraries (e.g., G for production, Z for test/archive). Work file BB112WL1 always to QTEMP.
- **Messages/Errors**: Uses GSMSGF message file. Displays "No Results" (COM(02)) if empty subfile. Logs actions like confirmation sent (COM(07)) or order in batch (COM(08)).
- **Validation**: Date validation via PLDTED (prepared for GSDTEDIT module, but not called). Ensures valid keys via chains (indicators 80/90-99 for chain failures).
- **Security/Interactivity**: User-controlled opens, interactive with function keys (F3 exit confirm, F4 prompt, F5 refresh, F10 cursor position). Protects fields (indicators 70-79).

### Tables Used

The program declares and uses the following files (tables). All are keyed (K) disk files except the display file. Overrides may redirect to different libraries (e.g., G/Z prefixed):

- `BB112D`: Workstation display file (CF WORKSTN) with subfile SFL1 for interactive inquiry screen (input/output).
- `ARCUST`: Input-only (IF) for customer data (e.g., chaining for names).
- `BBORCL`: Input-only (IF) for order control/details.
- `BBORDH`: Input-only (IF) for order headers (primary for positioning/reading orders).
- `GSCNTR`: Input-only (IF) for country data.
- `GSPROD`: Input-only (IF) for product data (renames TPFIL5 to X@FIL5).
- `GSTABL`: Input-only (IF) for table data (e.g., salesmen, carriers).
- `INLOC`: Input-only (IF) for location/inventory data.
- `SHIPTO`: Input-only (IF) for ship-to addresses.
- `BB112WL1`: Input-only (IF) work file (temporary in QTEMP for query results or intermediate data).

No update/output files; all are read-only except potential subfile updates.

### External Programs Called

The program calls the following external programs/procedures (via CALL opcode). System APIs are included as they are external:

- `BB112SC`: Called for option 6 (order confirmation) with parameters for company, order#, file group.
- `QCMDEXC`: System API to execute override commands (multiple times in OPNTBL).
- `QMHSNDPM`: System API to send program messages (in ADDMSG).
- `QMHRMVPM`: System API to remove messages (in CLRMSG).

Commented-out calls (not executed): `LGSTABL` (for table lookup in PROMPT) and `GSDTEDIT` (prepared PLIST but no CALL). No other user programs in visible code.