### Process Steps of the RPGLE Program BB114

This RPG IV (RPGL E) program (indicated by the .rpgle extension and syntax) is an interactive inquiry program for customer order confirmations in a Customer Orders system on an IBM midrange system (e.g., IBM i or AS/400). It uses a display file with a subfile (SFL1) to list and query open order headers based on user-specified criteria, allowing selections for further actions like viewing confirmations. The program is called from the main OCL with a parameter (likely a library prefix or file group indicator). It handles user inputs via function keys, validates searches, applies filters via dynamic queries (using OPNQRYF implied by query select arrays), and integrates with message queues for errors/feedback. Below is a step-by-step breakdown of the program's execution based on the provided code structure and subroutines:

1. **Program Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: p$fgrp (file group, e.g., 'G' or 'Z' for library overrides).
   - Sets up work fields, dates, and times using system data structure (PSDS): Converts current date (from UDATE or *DATE) to various formats (e.g., CCYYMMDD, YYMMDD) for filtering, defaulting "from" date to the 1st of the month one year prior.
   - Defines reposition fields (r$*) mirroring control fields (c$*) for subfile positioning.
   - Initializes output parameters (o$*) with defaults (e.g., o$mode = blanks, o$dstc = 'N').
   - Sets subfile control variables: RRN1 = 0, page size = 24, folded mode off initially.
   - Prepares message handling: Sets program message queue (m@pgmq = '*'), clears data.
   - Defines key lists (KLISTs) for chaining to files (e.g., KLORDH for order headers by company/order#).
   - Prepares arrays for query selects (qryoh for order headers, qryod for details), comments/errors (com), and file overrides (ovr, ovg, ovz).

2. **Open Database Tables (OPNTBL Subroutine)**:
   - Applies file overrides via QCMDEXC API based on p$fgrp ('G' or 'Z'): Redirects files to specific libraries (e.g., GARCUST for 'G', ZARCUST for 'Z') and sets share(*NO).
   - Overrides work file BB114WL1 to QTEMP/BB114WL1.
   - Opens all input files (ARCUST, BBORCL, BBORDH, etc.) and the work file.

3. **Main Subfile Processing Loop (SRSFL1 Subroutine)**:
   - Initializes subfile mode to folded ('1' for first load), clears message subfile, positions to first record (company = 10).
   - Defaults "PU/Ship From Date" to 1 year ago.
   - Calls SF1REP to reposition/load the subfile based on search criteria.
   - Enters a loop (SF1AGN = *ON) for user interaction:
     - Writes command line, displays/clears message subfile.
     - Conditions subfile display (*IN41 = *ON if RRN1 > 0).
     - Toggles folded/unfolded mode (*IN45).
     - Displays subfile control (EXFMT SFLCTL1).
     - Clears format indicators (21-39, 50-69 for errors).
     - Determines cursor position for next display.
     - Sets subfile record number (RCDNB1) to current page RRN for redisplay.

4. **Handle User Inputs Before Subfile Read**:
   - Processes function keys and positioning:
     - F3: Displays exit confirmation window (EXFMT F03WDW); if confirmed, sets SF1AGN = *OFF to exit loop.
     - F4: Determines cursor position, calls PROMPT subroutine (truncated, but likely displays field-specific prompts, e.g., for customer search via MCSTSHP using x$cstshp parms).
     - F5: Refreshes (sets ONETIME = *ON, REPSFL = *ON) to reload subfile.
     - Positioning: If control fields (c$co, c$ord#, etc.) differ from reposition fields (r$*), calls SF1REP to rebuild subfile.
     - Page Down: Calls SF1LOD (truncated, but likely loads next page of records into subfile).
   - Continues loop (ITER) if not ENTER.

5. **Process Subfile on ENTER (SF1PRC and SF1CHG Subroutines)**:
   - If subfile not empty (RRN1 > 0), reads changed subfile records (READC SFL1 until *IN81 = *ON).
   - For each changed record:
     - Clears option field (S1OPT1 = *OFF).
     - Processes selection (SELECT on S1OPT):
       - Option 6: Calls external program BB114SC (Customer Order Confirmation) passing company, order#, file group; updates fields like A$ORD (truncated).
     - Handles window prompts based on record/field (e.g., 'SFL1' and 'S1CRLM' displays CRDWDW for credit limit).
   - Post-processing: Handles other keys like F10 (positions cursor to control record).

6. **Reposition and Load Subfile (SF1REP and SF1LOD Subroutines - Truncated but Inferred)**:
   - Clears control fields if repositioning.
   - Builds dynamic query select string (QRYSLT) using qryoh/qryod arrays for filtering open orders (e.g., not deleted, specific status, date ranges, company/customer/order#/etc.).
   - Uses OPNQRYF (implied) on BBORDH (headers) and possibly details to select records (e.g., BODEL ≠ 'D', BORQD8 in range).
   - Chains to files using KLISTs (e.g., KLORDH for BBORDH, KLCUST for ARCUST) to validate/fetch descriptions (e.g., customer name C$CSNM).
   - Loads records into subfile (WRITE SFL1), updates RRN1.
   - Handles "no results" (adds message from com(02)).

7. **Field Prompting (PROMPT Subroutine - Truncated but Inferred)**:
   - Based on cursor location (ROW1/COL1) and field (e.g., C$CUST), calls utilities like MCSTSHP for customer/ship-to search.
   - Validates inputs (e.g., chains to GSTABL for SLSMAN/BBORCN/BBCAID codes).
   - Updates control fields and repositions subfile.

8. **Message and Error Handling (ADDMSG, WRTMSG, CLRMSG Subroutines)**:
   - Adds messages to program queue via QMHSNDPM API (e.g., for errors like "Must Enter At Least One Selection Value" from com(01)).
   - Writes/clears message subfile (WRITE MSGCTL or MSGCLR), toggles *IN49 for display.
   - Removes messages via QMHRMVPM API.
   - Sets DSPMSG = *ON for general errors.

9. **Program Termination**:
   - Closes all files (*ALL).
   - Sets *INLR = *ON and returns to caller.

The program uses a "load-on-demand" approach for the subfile (via Page Down), not "load all," and loops until exit. Truncated sections (e.g., SF1REP, PROMPT) likely include detailed query building, file chains, and additional calls/validations.

### Business Rules

- **Order Filtering and Inquiry**: Queries open order headers (BBORDH) that are not deleted (BODEL ≠ 'D'), in specific statuses (e.g., BOORPR in 'P'), and match user criteria (company, order#, PO#, customer, ship-to, salesman, location, product, carrier ID, country). Date range defaults to PU/Ship from 1 year ago. Includes rules for including/excluding canceled orders or invoices.
- **Validation and Required Inputs**: Must enter at least one selection value (error com(01)). Cannot search for tracking# if invoices excluded (com(05)) or canceled reason if not including canceled orders (com(06)).
- **Credit Limit Handling**: Displays credit limit window (CRDWDW); checks for over-limit (com(03)/04), though protection on subfile option removed (revision jk03).
- **Subfile Options and Actions**: Option 6 triggers confirmation view/send via BB114SC; confirms if sent (com(07)). Orders in batch noted (com(08)).
- **File Group Overrides**: Uses 'G' or 'Z' to switch libraries (e.g., GBBORDH vs. ZBBORDH), allowing multi-company/environment support.
- **Data Lookups**: Chains to tables for descriptions/names (e.g., ARCUST for customer name, GSPROD for product desc, GSTABL for codes like SLSMAN/BBORCN, BBCAID for carriers - revision jk04).
- **User Interface Rules**: Supports folded/unfolded subfile, positioning, prompting (F4), refresh (F5). Exit requires confirmation. Errors protect fields (*IN70-79) and display on message subfile.
- **Date Handling**: Validates/converts dates via GSDTEDIT (parms defined but call truncated). Uses system date for defaults.
- **Security/Authorization**: Implied credit limit authorization; no direct user auth in visible code.
- **Error and No-Results Handling**: Displays "No Results" if subfile empty; uses message array for standardized feedback.

### Tables/Files Used

These are database files (physical/logical) and display/printer files explicitly defined or referenced:

- **Display File (Workstation)**: BB114D (CF, with subfile SFL1; used for interactive I/O, formats like SFLCTL1, MSGCTL, F03WDW, CRDWDW).

- **Input Files (IF)**:
  - ARCUST (customers; keyed, USROPN).
  - BBORCL (order control; keyed, USROPN).
  - BBORDH (order headers; keyed, USROPN).
  - GSCNTR (countries; keyed, USROPN).
  - GSPROD (products; keyed, USROPN; field TPFIL5 aliased as X@FIL5).
  - GSTABL (general tables; keyed, USROPN; used for codes like SLSMAN, BBORCN).
  - INLOC (inventory locations; keyed, USROPN).
  - SHIPTO (ship-to addresses; keyed, USROPN).
  - BBCAID (carrier IDs; keyed, USROPN; added in revision jk04).

- **Work File (IF)**: BB114WL1 (keyed, USROPN; overridden to QTEMP).

No printer files are used in the visible code (PRTF_OPN DS defined but unused).

### External Programs Called

- **BB114SC**: Called for subfile option 6 (Customer Order Confirmation); passes parameters like company, order#, file group; receives updates (e.g., mode, flag).
- **QCMDEXC**: System API; called to execute file override commands (OVRDBF).
- **QMHSNDPM**: System API; called to send messages to program message queue.
- **QMHRMVPM**: System API; called to remove messages from program message queue.

Truncated sections likely include additional calls, such as:
- MCSTSHP (inferred from x$cstshp DS; for customer/ship-to search in prompting).
- GSDTEDIT (parms defined in PLDTED; for date validation).