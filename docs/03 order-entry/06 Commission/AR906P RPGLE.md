**Process Steps of RPG Program AR906P**

This is a classic **"Work With"** interactive subfile maintenance program (AR906P) that allows users to view, create, change, inactivate/reactivate, and display **commission charge entries** stored in file **ARCOMM**. It supports two modes: **MNT** (maintenance) and **INQ** (inquiry-only). The program uses a single subfile (SFL1) driven by control format SFLCTL1 and a message subfile.

Here is the high-level flow:

1. **Program Initialization (*INZSR)**  
   - Receives two parameters:  
     - `p$mode` = `'MNT'` (maintenance) or `'INQ'` (inquiry)  
     - `p$fgrp` = `'G'` or `'Z'` (determines which library set to use)  
   - Builds file overrides for G* or Z* libraries and executes them via `QCMDEXC`.  
   - Opens all database files.  
   - Sets up key lists, subfile sizes, date/time stamps, and message handling structures.

2. **Main Subfile Processing (srsfl1)**  
   - Clears the message subfile.  
   - Initializes subfile control fields and global protection (`*IN70` = on for inquiry mode).  
   - Positions the subfile at the first record (`sf1rep`).  
   - Enters the main **DO** loop that runs until F3 is pressed.

3. **Display Cycle**  
   - Writes command line (`SFLCMD1`) and message subfile.  
   - Loads/ redisplays the subfile (`SFL1`).  
   - Determines folded/unfolded mode.  
   - Executes `EXFMT SFLCTL1` (user sees the subfile and can enter options or control fields).

4. **User Input Processing** (before and after subfile read)  
   - **F3** → Exit program.  
   - **F4** → Field prompting (calls lookup programs).  
   - **F5** → Refresh (reload subfile).  
   - **F8** → Toggle **Include/Exclude Inactive** records (`w$inact`).  
   - **F15** → Call print program for Commission Charge Listing.  
   - **Direct Access fields** (in control record) → Validate and process option 1/2/4/5 immediately (`sf1dir`).  
   - **ENTER** → Process changed subfile records (`sf1prc` → `sf1chg`).

5. **Subfile Loading (sf1lod)**  
   - Reads next record from **ARCOMM** (via logical **ARCOMMRD**).  
   - Skips deleted/inactive records (`acdel = 'D'` or `'I'`) if the exclude-inactive filter is active.  
   - Formats each subfile line (`sf1fmt`).  
   - Applies color coding (`sf1col` – deleted/inactive records appear in blue).  
   - Writes record to subfile and continues until a full page is loaded or end-of-file.

6. **Subfile Option Processing (sf1chg / sf1dir)**  
   - Option **2** (Change) → calls detail program (cannot change deleted records).  
   - Option **4** (Inactivate/Reactivate) → calls AR9064.  
   - Option **5** (Display) → calls detail program in inquiry mode.  
   - Option **1** (Create) → handled only via direct access.  
   - After each action, the subfile line is refreshed from the database.

7. **Detail Maintenance Calls**  
   - All create/change/display actions call the same detail program (`AR906`) with different mode flags.  
   - Inactivate/reactivate uses a dedicated program (`AR9064`).  
   - On successful completion, a confirmation message is sent and the subfile is repositioned/reloaded.

8. **Validation & Messaging**  
   - All direct-access and control-field entries are validated against master files.  
   - Errors are sent to the message subfile using `QMHSNDPM`.  
   - Messages are cleared with `QMHRMVPM`.

9. **Cleanup**  
   - Closes all files.  
   - Returns to caller.

**External Programs Called**

| Program   | Called When                  | Purpose |
|-----------|------------------------------|---------|
| **AR906**   | Option 1 (Create), 2 (Change), 5 (Display) | Full maintenance / inquiry detail screen for a single commission charge record |
| **AR9064**  | Option 4 (In/Activate)       | Inactivate or reactivate a commission charge record |
| **AR9065**  | F15                          | Print Commission Charge Listing |
| **LARCUST** | F4 on Customer field         | Customer lookup / prompt |
| **LINLOC**  | F4 on Location field         | Location lookup / prompt |
| **LGSPROD** | F4 on Product field          | Product master lookup / prompt |
| **QCMDEXC** | Startup                      | Execute file overrides |
| **QMHSNDPM** / **QMHRMVPM** | Throughout | Program message queue handling |

**Tables / Files Used**

| File (Logical / Physical) | Usage | Description |
|---------------------------|-------|-------------|
| **ARCOMM**                | Main file (maintenance) | Commission Charge master file |
| **ARCOMMRD**              | Read-only (renamed format) | Sequential read of ARCOMM for subfile load |
| **BICONT**                | Validation | Business control / company master |
| **GSPROD**                | Validation | Product master (field `TPFIL5` renamed to `x@fil5`) |
| **ARCUST**                | Validation | Customer master |
| **INLOC**                 | Validation | Inventory / warehouse location master |

All files are **overridden** at program start to either the **G** (general) or **Z** (zone-specific) library set based on the `p$fgrp` parameter.

The program follows standard IBM i "Work With" design patterns: subfile load on demand, option processing on changed records, direct-access support, field prompting, and full message subfile support.