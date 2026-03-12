The `BB1033.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is part of the `BB101` order entry system, designed to validate and retrieve responsible area and major location information for product/container combinations, displaying results in a subfile for user interaction. Written by Jimmy Krajacic on 08/14/2019, it includes revisions to handle non-detail records (jb01) and ignore inactivated or 'I' status records in the product load files (jb02). Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB1033` program validates product/container combinations against location data in the `prdlody`, `prdlodx`, `prdlodz`, `bbortr`, and `bbortrx` files, then displays valid shipping locations in a subfile on the `bb1033d` display file for user selection. The steps are as follows:

1. **Program Initialization**:
   - **File Definitions**:
     - `prdlody`: Input-only file for product load data (keyed, user-opened).
     - `prdlodx`: Input-only file for product load data with renamed record format `prdlodrx` (keyed, user-opened).
     - `prdlodz`: Input-only file for product load data with renamed record format `prdlodrz` (keyed, user-opened).
     - `bbortr`: Input-only file for order transactions (512 bytes, indexed, key length 11, starting at position 2).
     - `bbortrx`: Input-only file for order transaction extensions (512 bytes, indexed, key length 11, starting at position 2).
     - `bb1033d`: Work station file (display file) with a subfile `sfl1` and control format `sflctl1`.
   - **Data Structures and Fields**:
     - `time12`: Data structure for time conversion (not used in the provided code snippet).
     - `dspf_ds`: Display file data structure for feedback (record name, key, cursor location, page RRN).
     - `m@80`: Array for message data (80 elements, 1 character each).
     - `ovg` and `ovz`: Arrays for file override commands (14 elements, 80 characters each) for `prdlody`, `prdlodx`, `prdlodz` in libraries `gprdlody`, `gprdlodx`, `gprdlodz` (ovg) and `zprdlody`, `zprdlodx`, `zprdlodz` (ovz).
     - `com`: Array for error messages (6 elements, 40 characters each).
     - `RaOver`: Data structure with 10 responsible area fields (`ra01` to `ra10`, 5 characters each).
     - `Ra`: Array overlay for responsible areas (10 elements, 5 characters each).
     - Prefixes: `f$` (panel fields), `c$` (subfile control), `s1` (subfile fields), `s2` (unused), `k$` (key lists), `p$` (input parameters), `o$` (output parameters), `r$` (reposition subfile), `w$` (work fields).
   - **Indicators**:
     - 19: Panel format input change.
     - 21-39, 50-69: Screen errors.
     - 40: Subfile clear.
     - 41: Subfile display control.
     - 42: Subfile display.
     - 43: Subfile end and next change.
     - 49: Message subfile display/control/initialize/end.
     - 70-79: Input field protect.
     - 80: Primary file chain.
     - 88: Read subfile.
     - 90-99: Secondary file chain/read.

2. **Parameter Input**:
   - Receives input parameters (assumed via `*ENTRY PLIST`, not shown in the snippet) including product, container, company, order number, and other order details (inferred from `p$msg1`, `p$msg2`, `p$msg3`).

3. **Subfile Initialization (`srinit` Subroutine)**:
   - Clears the subfile (`*in40`) and initializes the relative record number (`rrn1`).
   - Opens product load files (`prdlody`, `prdlodx`, `prdlodz`).
   - Chains to `bbortr` and `bbortrx` using the company/order key to retrieve order details (indicator 80, jb01).
   - Reads product load files (`prdlody`, `prdlodx`, `prdlodz`) to find valid shipping locations, ignoring records with inactive status or 'I' status (jb02).
   - For each valid record:
     - Validates responsible area and major location.
     - Formats a subfile line (`sf1fmt` subroutine) with messages (`p$msg1`, `p$msg2`, `p$msg3`).
     - Writes to the subfile (`sfl1`) and increments `rrn1`.
     - Sets `*in43` (subfile end/next change).

4. **Subfile Processing (`srsfl1` Subroutine)**:
   - Writes an assume-overlay record (`wdwovr`).
   - Enters a loop (`sf1agn`) to process user interaction:
     - If `rrn1 > 0`, sets `*in41` (subfile display) to show the subfile; otherwise, clears it.
     - Sets the subfile record number (`rcdnb1`) to `rrn1`.
     - Writes the subfile control format (`sflwdw1`) and displays the subfile control (`sflctl1`) using `exfmt`.
     - Checks for F12 key (return, exits loop).
     - Continues until user exits (`sf1agn = *off`).

5. **Subfile Line Formatting (`sf1fmt` Subroutine)**:
   - Moves input messages (`p$msg1`, `p$msg2`, `p$msg3`) to subfile fields (`s1msg1`, `s1msg2`, `s1msg3`).
   - Sets `*in43` (subfile end/next change) to enable subfile updates.

6. **Program Termination**:
   - Sets the last record indicator (`LR`) to exit the program.
   - Returns selected responsible area and major location data (via `o$` output parameters, assumed).

---

### Business Rules

The program enforces the following business rules:

1. **Validation of Shipping Locations**:
   - Validates product/container combinations against `prdlody`, `prdlodx`, and `prdlodz` to identify valid responsible areas and major locations.
   - Ignores records in product load files that are inactivated or have a status of 'I' (jb02).
   - If the product/container cannot ship from the header location, checks alternate headers (error "Prod/Cntr Can't Ship from Hdr", COM,01; allows alternate header, COM,02).
   - If the product/container cannot ship from the major location, checks alternate major locations (error "Prod/Cntr Can't Ship From Maj", COM,04; allows alternate major, COM,05).
   - If no valid record is found, displays error "Prod/Cntr not in Prd Load File,F12 Accpt" (COM,03) and allows user to accept with F12.

2. **Non-Detail Record Handling**:
   - Handles non-detail records in `bbortr` and `bbortrx` to ensure correct order data retrieval (jb01).

3. **Subfile Display**:
   - Displays valid shipping locations in a subfile (`sfl1`) for user selection.
   - Supports user navigation with F12 to return or accept a non-valid location (COM,03).

4. **File Overrides**:
   - Uses override arrays (`ovg`, `ovz`) to redirect `prdlody`, `prdlodx`, and `prdlodz` to libraries `gprdlody`, `gprdlodx`, `gprdlodz` (production) or `zprdlody`, `zprdlodx`, `zprdlodz` (alternate, possibly test environment).

5. **Error Handling**:
   - Displays errors via the `com` array in the subfile, allowing user interaction to resolve issues (e.g., F12 to accept missing product/container records).

---

### Tables (Files) Used

The program interacts with the following files:

1. **prdlody**: Input-only product load file (keyed, user-opened).
   - Contains product/container location data.
2. **prdlodx**: Input-only product load file with renamed record format `prdlodrx` (keyed, user-opened).
   - Alternate product load data.
3. **prdlodz**: Input-only product load file with renamed record format `prdlodrz` (keyed, user-opened).
   - Additional product load data.
4. **bbortr**: Input-only order transaction file (512 bytes, indexed, key length 11, starting at position 2).
   - Contains order details (e.g., company, order number).
5. **bbortrx**: Input-only order transaction extension file (512 bytes, indexed, key length 11, starting at position 2).
   - Contains additional order details (jb01).
6. **bb1033d**: Work station file (display file) with subfile `sfl1` and control format `sflctl1`.
   - Fields: `s1msg1`, `s1msg2`, `s1msg3` (subfile messages), `c$` fields (subfile control).

---

### External Programs Called

The `BB1033` RPGLE program does not call any external programs. All processing is handled internally through file operations and subroutines (`srinit`, `sf1fmt`, `srsfl1`).

---

### Summary

The `BB1033` RPGLE program, called from the `BB101.ocl36.txt` OCL program, validates and retrieves responsible area and major location information for product/container combinations in the `BB101` order entry system. It reads `prdlody`, `prdlodx`, `prdlodz`, `bbortr`, and `bbortrx` to validate shipping locations, ignoring inactive or 'I' status records (jb02), and displays results in a subfile for user selection. Business rules ensure valid location checks, handle alternate headers/majors, and allow user overrides with F12. The program interacts with six files and does not call external programs, relying on internal subroutines for processing.