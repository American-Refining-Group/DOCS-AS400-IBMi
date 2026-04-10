Here is a structured explanation of the **RPGLE program BB900P** (Work with Billing Control Entries), based on the provided source code.

### Main Purpose
This is a classic **"Work With"** maintenance/inquiry program for a master file containing **billing control entries**.  
It displays a subfile list of companies/control records, allows filtering, positioning, and supports common options:

- 2 = Change  
- 4 = Delete / Reactivate  
- 5 = Display  
- (and 1 = Create — via direct entry or F-key)

### High-Level Process Flow

1. **Initialization (*inzsr)**
   - Receives parameters:
     - `p$mode` = 'MNT' (maintenance) or 'INQ' (inquiry only)
     - `p$fgrp` = 'G' or 'Z' (file group → determines which library/environment to use)
   - Defines key lists, indicators, subfile RRNs, page size, etc.
   - Loads compile-time arrays (`ovg`, `ovz`, `hdr`, `com`, `fky`)
   - Sets current date/time
   - Prepares message handling fields

2. **File Overrides & Opens (opntbl)**
   - Depending on `p$fgrp` ('G' or 'Z'):
     - Executes `OVRDBF` commands to point to either `gbicont` or `zbicont`
   - Opens two versions of the same physical file:
     - `bicont`   → used for update/chain (input/output)
     - `bicontrd` → read-only renamed format (`bicontpr`) used for subfile loading

3. **Main Logic – Subfile Processing (srsfl1)**
   This is the heart of the program — a long DOW loop that keeps redisplaying the screen until F3=Exit.

   Typical cycle inside the loop:

   - Clear / write message subfile if needed
   - Set folded/unfolded mode (initially folded)
   - Protect fields globally if in inquiry mode (`*in70`)
   - Write command line + message subfile
   - Evaluate subfile display indicators (`SFLDSP`, `SFLDSPCTL`, `SFLEND`, etc.)
   - EXFMT `sflctl1` → shows control record + subfile

   After user presses Enter / Function key:

   - **F3** → exit loop
   - **F4** → prompt (currently just sets *in19 — likely unimplemented or external prompt)
   - **F5** → refresh (clear position field → reposition)
   - **F8** → toggle show deleted / hide deleted records
   - **F10** → place cursor in control record
   - **PageDown** → load more records (`sf1lod`)
   - **Positioning value** entered in `c1co` → reposition subfile (`sf1rep`)
   - **Option entered in subfile** → handled after READC in `sf1prc → sf1chg`

4. **Subfile Load Logic (sf1lod)**
   - Reads next record from `bicontrd` (read-only view)
   - Skips deleted records (`bcdel = 'D'`) if user chose to exclude them (`w$del = *off`)
   - Formats subfile line (`sf1fmt`)
   - Applies color (blue for deleted records) (`sf1col`)
   - Writes to subfile until page is full or end-of-file

5. **Option Processing (when Enter pressed – sf1prc → sf1chg)**
   | Option | Action                          | Mode restriction | External program called | Notes |
   |--------|---------------------------------|------------------|---------------------------|-------|
   | 1      | Create new entry                | MNT only         | `BB900`                   | Direct entry also supported |
   | 2      | Change existing entry           | MNT only         | `BB900`                   | Cannot change if marked 'D' |
   | 4      | Delete / Reactivate             | MNT only         | `BB9004`                  | Toggle delete flag |
   | 5      | Display / View details          | Any              | `BB900` (in INQ mode)     | Inquiry mode |

   After external program returns → confirmation message is sent and subfile is usually refreshed.

6. **Direct entry in control record (d1opt + d1co)**
   - Allows fast-path create/change/delete/display without selecting from subfile
   - Same logic as subfile options → calls same external programs

7. **Message Handling**
   - Uses `QMHSNDPM` / `QMHRMVPM` for program message queue
   - Message subfile is managed separately (`wrtmsg`, `clrmsg`)

8. **End of Program**
   - Closes all files
   - `*inlr = *on` and `return`

### External Programs Called

| Program  | Purpose                              | Called from         | Mode used       | Return value (`o$flag`) |
|----------|--------------------------------------|---------------------|------------------|---------------------------|
| BB900    | Maintain / Display single record     | Options 1,2,5       | 'MNT' or 'INQ'   | '1' = record was maintained |
| BB900    | Display single record (inquiry)      | Option 5            | 'INQ'            | —                         |
| BB9004   | Delete / Undelete (toggle delete flag) | Option 4          | —                | 'D' or 'A'                |

### Database Tables / Files Used

| File name in program | Record format | Physical file (after OVRDBF) | Access mode       | Purpose                     |
|----------------------|---------------|-------------------------------|-------------------|-----------------------------|
| bicont               | (default)     | gbicont or zbicont            | Input/Output      | Chain / Update records      |
| bicontrd             | bicontpr      | gbicont or zbicont            | Input only        | Subfile loading (read-only) |

Both logical/physical files point to the same data — just different open modes and possibly different libraries depending on `p$fgrp` ('G' vs 'Z').

### Summary – Most Important Points

- Classic green-screen **Work-with style** subfile program
- Supports maintenance + inquiry modes
- Two environments/libraries selectable via parameter ('G' or 'Z')
- Delete is logical (flag = 'D')
- Delegates actual record maintenance/display/delete to **BB900** and **BB9004**
- Uses compile-time arrays for messages, headers, function key text, and OVRDBF strings

Let me know if you'd like deeper explanation of any specific subroutine (e.g. `sf1lod`, `sf1chg`, message handling, etc.).