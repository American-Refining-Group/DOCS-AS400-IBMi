### Process Steps of the OCL Program

This OCL (Operations Control Language) script appears to be a batch processing routine for an IBM midrange system (e.g., AS/400 or iSeries), designed to handle order entry posting. It manages order batches, performs checks, sorts data, authorizes credits, posts orders, handles cancellations/reactivations, unlocks records, and releases batches. The script uses placeholders like `?9?` (likely a library prefix), `?20?` (batch identifier), `?13?` (mode selector: blank for standard order post, 'PP' for viscosity ASN posts, 'PM' for product moves posts), and `?USER?`, `?WS?` (user/workstation identifiers). The process is conditional and interactive, with pauses for user input and error handling.

The high-level steps are as follows:

1. **Initial Activity Check**:
   - Check if another instance of the process (ACTIVE-BB201) is running.
   - If active, display a message indicating the post is in progress, instruct the user to try again later, pause for cancellation input (press 0, Enter to cancel), and return/exit if active.

2. **Order Batch Selection**:
   - Set local variables for offsets and data (e.g., OFFSET-470 to '?13?', OFFSET-494 to '?USER?', OFFSET-502 to '?WS?', OFFSET-504 to 'P').
   - Based on `?13?` value, set a display message (e.g., '* ORDER POST *' for blank, '* VISCOSITY ASN POSTS *' for 'PP', '* PRODUCT MOVES POSTS *' for 'PM').
   - Turn off switch 1 (for deleting a batch).
   - Load program BB001.
   - Define files BBBTCH and BBBTCHX (both labeled `?9?BBBTCH`, shared disposition).
   - Run the program.
   - If input at location '121,6' is 'CANCEL', return/exit.
   - Evaluate P20 from input at '490,2'.

3. **User Option to Cancel Posting**:
   - Display 'ORDER ENTRY POST EXECUTING'.
   - Check if file `?9?BBOR?20?` exists and has data.
   - If no data, display 'NO BATCH ?20? TO POST AT THIS TIME', pause with 'PROCEDURE IS CANCELLED', and jump to the end.
   - Pause for user input: Press ATTN, 2, Enter to cancel; press 0, Enter to continue. Set attributes to cancel-no, inquiry-yes.

4. **Credit Authorization Preparation and Execution**:
   - Load utility #GSORT.
   - Define input file as `?9?BBOR?20?` (shared).
   - Define output file as `?9?BB198S` (records 999000, extendable, retain job).
   - Run the sort with specific parameters:
     - Sort on fields like 21A, 3X 512 N.
     - Include conditions (e.g., positions 144-149 > C000000, 10-12 = C000).
     - Force sequence: Company (2-3), Customer (144-149), Order (4-9), Container (121-123), Seq# (10-12).
     - Drop full records (1-256 and 257-512).
     - Note: This sorts headers first, then marks, then details due to record type positioning.
   - End the sort.
   - Load program BB198.
   - Define multiple files (e.g., BBORTR as `?9?BB198S`, BBORTA as `?9?BBOR?20?`, BBORDRH as `?9?BBORDH`, and others like BBORDRD, BBORDRO, BBORDRI, BBORDRB, BBORDRM, BICONT, ARCUST, GSCTUM, GSTABL, CUSORD, SHIPTO, GSPROD, all shared).
   - Define printer LIST on device P5, priority 0.
   - Run the program.

5. **Order Entry Posting**:
   - Load program BB201.
   - Define files (e.g., BBORTR as `?9?BBOR?20?`, BBOTHS1 as `?9?BBOTHS1`, BBOTDS1 as `?9?BBOTDS1`, BBOTA1 as `?9?BBOTA1`, BBTRTX as `?9?BBOX?20?`, and many others like BBORDRH, BBORHS1, BBORDRD, BBORDS1, BBORDRM, BBORDRO, BBORDRI, BBORDRB, BBORA1, BBORTX, BICONT, ARCUST, GSCTUM, GSTABL, CUSORD, BBORCL, BBORHHS, BBORDHS, BBORMHS, GSPROD, BBOH, BBOHMS, all shared).
   - Define printers LIST on P5 (priority 0) and LISTOUT3 on PJ (priority 0).
   - Run the program.
   - Override database files (e.g., BBORTR to `?9?BBOR?20?`, BBTRTX to `?9?BBOX?20?`, BBOTHS1 to `?9?BBOTHS1`, BBOTDS1 to `?9?BBOTDS1`, BBOTA1 to `?9?BBOTA1`).
   - Call external program BB104B with parameters `?9?` and '0'.
   - Delete all overrides.
   - If `?13?` is 'PP', skip to 'SKIP' tag; otherwise, call BB202 with parameters (,,,,,,,?9?,,,,,,,,,,,?20?) to create spool files for emailing orders on hold.

6. **Remove Order Lockout Fields**:
   - Display 'UNLOCKING OPEN ORDERS'.
   - Load program BB215.
   - Define files BBORTR as `?9?BBOR?20?` and BBORDRH as `?9?BBORDH` (both shared).
   - Run the program to unlock records in the open orders file.

7. **Release Batch and Cleanup**:
   - Jump to 'END' tag.
   - Set local OFFSET-475 to '?F'A,?9?BBOR?20?'?' (likely checking file attributes).
   - Load program BB005.
   - Define file BBBTCH as `?9?BBBTCH` (shared).
   - Run the program to release the batch.
   - Blank all local variables.
   - Delete files BBOR?20? and BBOX?20? using GSDELETE utility from library ?9?.

### External Programs Called

The script loads and runs several programs (via `LOAD` and `RUN`), calls others directly (via `CALL`), and references one in a comment. Here's the list:

- BB001: Loaded and run for order batch selection.
- #GSORT: Loaded and run as a sorting utility for credit authorization preparation.
- BB198: Loaded and run for credit authorization processing.
- BB201: Loaded and run for the main order entry post.
- BB104B: Called (with parameters ?9? and '0') for open order cancellation or reactivation, archiving canceled orders.
- BB202: Called (with parameters ,,,,,,,?9?,,,,,,,,,,,?20?) conditionally if ?13? is not 'PP', to create spool files for emailing orders on hold.
- BB215: Loaded and run to remove order lockout fields.
- BB005: Loaded and run to release the batch.
- BB117: Referenced in a comment ("FOR CALLED PGM: BB117") but not explicitly called or loaded in this script.

### Tables (Files) Used

The script defines numerous files (tables/datasets) across different steps, all with shared disposition (`DISP-SHR`) unless noted. Many are prefixed with `?9?` (library) and use `?20?` for batch-specific naming. Here's a comprehensive list, grouped by usage context for clarity:

#### Batch Selection:
- BBBTCH (label: ?9?BBBTCH)
- BBBTCHX (label: ?9?BBBTCH)

#### Credit Authorization (Sort):
- INPUT (label: ?9?BBOR?20?)
- OUTPUT (label: ?9?BB198S, records: 999000, extend: 999000, retain: J)

#### Credit Authorization (Main):
- BBORTR (label: ?9?BB198S)
- BBORTA (label: ?9?BBOR?20?)
- BBORDRH (label: ?9?BBORDH)
- BBORDRD (label: ?9?BBORDD)
- BBORDRO (label: ?9?BBORDO)
- BBORDRI (label: ?9?BBORDI)
- BBORDRB (label: ?9?BBORDB)
- BBORDRM (label: ?9?BBORDM)
- BICONT (label: ?9?BICONT)
- ARCUST (label: ?9?ARCUST)
- GSCTUM (label: ?9?GSCTUM)
- GSTABL (label: ?9?GSTABL)
- CUSORD (label: ?9?CUSORD)
- SHIPTO (label: ?9?SHIPTO)
- GSPROD (label: ?9?GSPROD)

#### Order Entry Post:
- BBORTR (label: ?9?BBOR?20?)
- BBOTHS1 (label: ?9?BBOTHS1)
- BBOTDS1 (label: ?9?BBOTDS1)
- BBOTA1 (label: ?9?BBOTA1)
- BBTRTX (label: ?9?BBOX?20?)
- BBORDRH (label: ?9?BBORDH)
- BBORHS1 (label: ?9?BBORHS1)
- BBORDRD (label: ?9?BBORDD)
- BBORDS1 (label: ?9?BBORDS1)
- BBORDRM (label: ?9?BBORDM)
- BBORDRO (label: ?9?BBORDO)
- BBORDRI (label: ?9?BBORDI)
- BBORDRB (label: ?9?BBORDB)
- BBORA1 (label: ?9?BBORA1)
- BBORTX (label: ?9?BBORTX)
- BICONT (label: ?9?BICONT)
- ARCUST (label: ?9?ARCUST)
- GSCTUM (label: ?9?GSCTUM)
- GSTABL (label: ?9?GSTABL)
- CUSORD (label: ?9?CUSORD)
- BBORCL (label: ?9?BBORCL)
- BBORHHS (label: ?9?BBORHHS)
- BBORDHS (label: ?9?BBORDHS)
- BBORMHS (label: ?9?BBORMHS)
- GSPROD (label: ?9?GSPROD)
- BBOH (label: ?9?BBOH)  // For called PGM BB117
- BBOHMS (label: ?9?BBOHMS)  // For called PGM BB117

#### Remove Order Lockout:
- BBORTR (label: ?9?BBOR?20?)
- BBORDRH (label: ?9?BBORDH)

#### Release Batch:
- BBBTCH (label: ?9?BBBTCH)

Note: Files like BBOR?20? and BBOX?20? are temporary/batch-specific and are deleted at the end via GSDELETE. Printers (LIST, LISTOUT3) are also defined but are output devices, not tables.