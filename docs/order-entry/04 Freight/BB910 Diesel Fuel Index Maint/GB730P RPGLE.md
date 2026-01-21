The **GB730P** RPG program is designed for **Global File Tracking History Inquiry**, providing a user interface to display historical changes for various files in the system. It is called from the **BB910** program (specifically in the `srsfl2` subroutine when the F09 key is pressed) to show history records for a selected file, such as the National Diesel Fuel Index (`BBNDFI`). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called .

---

### Process Steps of the GB730P RPG Program

1. **Program Initialization (*INZSR)**:
   - **Receives Entry Parameters**:
     - `p$elist`: A 125-character data structure containing:
       - `p$file`: File name (e.g., `BBNDFI`, `BBPRCE`, etc.).
       - `p$fgrp`: File group (`Z` or `G` for file overrides).
       - `p$co`: Company number (2 digits).
       - Additional fields specific to the file (e.g., for `BBNDFI`: effective date `pbbndefdt`, time `pbbndeftm`, region `pbbndregn`).
   - **Calls Subroutines**:
     - `getpgmparm`: Parses the `p$elist` into specific data structures based on the file name (e.g., `bbndparm` for `BBNDFI`).
     - `opntbl`: Applies file overrides (`OVG` or `OVZ`) based on `p$fgrp` and opens all relevant files.
     - `GetStartDt`: Retrieves the most recent change date and time for the specified file.
     - `setlimit`: Sets the subfile page size (`PAGSZ1`) to 10 and initializes other control fields.
   - **Sets Header**:
     - Assigns a header (`C$HDR1`) from the `HDR` array based on `p$file` (e.g., "National Diesel Fuel Index Tracking" for `BBNDFI`).
     - Sets indicator `*IN75` for specific files (`ARCUPR`, `BICUAG`, `BICUFR`) to control display behavior.
   - **Initializes Fields**:
     - Sets subfile relative record number (`RRN1`) and saved record number (`RRNSV1`) to zero.
     - Initializes flags (`FRSTRD`, `REPSFL`, `FMTAGN`, `DSPMSG`, `movehdrflg`) to `*OFF`.
     - Defines message handling fields (`m@pgmq`, `m@key`) and date validation parameters (`pldted`).

2. **Open Database Tables (opntbl)**:
   - Applies file overrides based on `p$fgrp`:
     - For `G`, overrides point to files like `gglcont`, `ggbfmsg`, `gbicuag`, etc.
     - For `Z`, overrides point to files like `zglcont`, `zgbfmsg`, `zbicuag`, etc.
   - Executes the `QCMDEXC` system command to apply overrides for all files.
   - Opens all input files (e.g., `glcont`, `shiptz`, `bbndfi`, `bbndfihx`, etc.) with `USROPN`.

3. **Get Starting Date (GetStartDt)**:
   - Retrieves the most recent change date (`sfcfdt`) and time (`sfcftm`) for the specified file by reading the corresponding history file (e.g., `bbndfihx` for `BBNDFI`).
   - Converts the change date to MM/DD/YY format using `%editc` for display.
   - Positions the file to the lowest key (`*LOVAL`) for subsequent subfile loading.

4. **Main Subfile Processing (srsfl1)**:
   - **Clear Message Subfile**: Calls `clrmsg` to clear messages and `wrtmsg` to display the message subfile.
   - **Initial Display**:
     - Clears error indicators (`*IN50`-`*IN69`) on the first display (`FRSTRD`).
     - Repositions the subfile (`sf1rep`) to load initial records.
   - **Main Loop**:
     - Displays the command line (`SFLCMD1`) and message subfile if needed (`DSPMSG`).
     - Checks if subfile records exist to enable display (`*IN41`).
     - Displays the subfile control record (`SFLCTL1`) using `EXFMT`.
     - Determines cursor location (`CSRLOC`) and sets the subfile record number (`RCDNB1`) to the lowest displayed record (`PAGRRN`).
     - Processes user input:
       - **F03 (Exit)**: Exits the program.
       - **F05 (Refresh)**: Resets positioning fields (`r$emdy`, `c$emdy`, etc.) and repositions the subfile.
       - **Page Down**: Loads more subfile records (`sf1lod`).
       - **Enter**: Processes user positioning if the date (`c$emdy`) changes (`sf1rep`).
       - **F10**: Clears cursor position (`row1`, `col1`) to reposition to the control record.
     - Continues until the user exits (`FMTAGN = *OFF`).

5. **Subfile Repositioning (sf1rep)**:
   - Clears the subfile (`sf1clr`) and resets `RRN1` and `RRNSV1`.
   - Validates the input date (`c$emdy`) using `GSDTEDIT`. If invalid, sets error `ERR0020` and displays it.
   - If no errors, positions the history file using the appropriate keylist (e.g., `bbndfikey` for `BBNDFI`) and loads the subfile (`sf1lod`).

6. **Subfile Loading (sf1lod)**:
   - Sets `RRN1` to the last saved record number (`RRNSV1`).
   - Loads up to `PAGSZ1` (10) records:
     - Reads records from the appropriate history file (e.g., `bbndfihx` for `BBNDFI`) using the corresponding keylist.
     - Formats each record (`sf1fmt`) based on the file type, mapping fields like change date, time, user, and specific data (e.g., region `lhregn`, price `lhrprc` for `BBNDFI`).
     - Writes the record to `SFL1` and increments `RRN1`.
   - Sets the subfile end indicator (`*IN43`) if no more records are available.
   - Saves the last record number (`RRNSV1`).

7. **Message Handling**:
   - **addmsg**: Sends error messages (e.g., `ERR0020`) to the program message queue using `QMHSNDPM`.
   - **wrtmsg**: Writes the message subfile (`MSGCTL`).
   - **clrmsg**: Clears the message subfile using `QMHRMVPM`.

8. **Program Termination**:
   - Closes all files.
   - Sets `*INLR` to `*ON` and returns.

---

### Business Rules

1. **File-Specific History Inquiry**:
   - The program supports history inquiries for multiple files (e.g., `BBNDFI`, `BBPRCE`, `BICUAG`, etc.), each with specific key fields and display formats.
   - The file to query is determined by the `p$file` parameter, and the appropriate history file (e.g., `bbndfihx`) is accessed.

2. **File Overrides**:
   - Based on `p$fgrp` (`Z` or `G`), the program overrides file names to point to the correct library (e.g., `gbbndfi` or `zbbndfi`).
   - Overrides are applied using `QCMDEXC` to ensure the correct files are accessed.

3. **Date Validation**:
   - The input date (`c$emdy`) is validated using `GSDTEDIT`. Invalid dates trigger error `ERR0020`.
   - Dates are converted to MM/DD/YY format for display using `%editc`.

4. **Subfile Display**:
   - Displays up to 10 records per page (`PAGSZ1`).
   - Records are sorted by change date and time in descending order (most recent first) using `SETGT` and `READPE`.
   - The subfile is cleared and reloaded when repositioned (`sf1rep`).

5. **User Interaction**:
   - Users can navigate using function keys:
     - **F03**: Exit the program.
     - **F05**: Refresh the subfile, clearing positioning fields.
     - **F10**: Reposition the cursor to the control record.
     - **Page Down**: Load the next page of records.
     - **Enter**: Reposition the subfile based on a new date input.
   - The program protects fields in inquiry mode, ensuring read-only access.

6. **Error Handling**:
   - Errors (e.g., invalid date) are displayed in a message subfile.
   - Only one error (`ERR0020`) is defined in the `COM` array, indicating limited error scenarios.

7. **Security**:
   - The program is proprietary to WCI International Company, with unauthorized use prohibited.
   - User information (`USER##8`) is captured for history records.

---

### Tables (Files) Used

1. **gb730pd**:
   - Type: Display file (workstation).
   - Usage: Defines the user interface with subfile `SFL1` and control record `SFLCTL1`.
   - Access: Input/Output.

2. **glcont**:
   - Type: Input file.
   - Usage: Stores company information (likely for validation).
   - Access: Read-only, user-opened.

3. **gbfmsg**:
   - Type: Input file.
   - Usage: Stores message definitions (e.g., `ERR0020`).
   - Access: Read-only, user-opened.

4. **shiptz, shipthx**:
   - Type: Input files.
   - Usage: Store ship-to information and its history.
   - Access: Read-only, user-opened.

5. **arcup31, arcuphx**:
   - Type: Input files.
   - Usage: Store customer product code data and its history.
   - Access: Read-only, user-opened.

6. **cuadrx, cuadrhx**:
   - Type: Input files.
   - Usage: Store customer ship-to address data and its history.
   - Access: Read-only, user-opened.

7. **arcusz, arcushx**:
   - Type: Input files.
   - Usage: Store A/R customer master data and its history.
   - Access: Read-only, user-opened.

8. **arcuspx, arcusphx**:
   - Type: Input files.
   - Usage: Store A/R customer supplemental data and its history.
   - Access: Read-only, user-opened.

9. **bicuag, bicuaghx**:
   - Type: Input files.
   - Usage: Store customer sales agreement data and its history.
   - Access: Read-only, user-opened.

10. **bbprce, bbprcehx**:
    - Type: Input files.
    - Usage: Store rack pricing data and its history.
    - Access: Read-only, user-opened.

11. **bbndfi, bbndfihx**:
    - Type: Input files.
    - Usage: Store National Diesel Fuel Index data and its history (used by `BB910`).
    - Access: Read-only, user-opened.

12. **bbcfsh, bbcfshhx**:
    - Type: Input files.
    - Usage: Store carrier fuel surcharge header data and its history.
    - Access: Read-only, user-opened.

13. **bbcfsd, bbcfsdhx**:
    - Type: Input files.
    - Usage: Store carrier fuel surcharge detail data and its history.
    - Access: Read-only, user-opened.

14. **bicuf2, bicufrhx**:
    - Type: Input files.
    - Usage: Store customer freight data and its history.
    - Access: Read-only, user-opened.

15. **gsprod, gsprodhx**:
    - Type: Input files.
    - Usage: Store product code data and its history.
    - Access: Read-only, user-opened.

16. **gshazm, gshazmhx**:
    - Type: Input files.
    - Usage: Store hazmat/shipping description data and its history.
    - Access: Read-only, user-opened.

17. **gsumcv, gsumcvhx**:
    - Type: Input files.
    - Usage: Store product unit of measure conversion data and its history.
    - Access: Read-only, user-opened.

18. **gsctum, gsctumhx**:
    - Type: Input files.
    - Usage: Store container unit of measure conversion data and its history.
    - Access: Read-only, user-opened.

19. **gsctwt, gsctwthx**:
    - Type: Input files.
    - Usage: Store container weight data and its history.
    - Access: Read-only, user-opened.

20. **biprt2, biprtxhx**:
    - Type: Input files.
    - Usage: Store product tax code data and its history.
    - Access: Read-only, user-opened.

21. **gscntr1, gscntrhx**:
    - Type: Input files.
    - Usage: Store container code data and its history.
    - Access: Read-only, user-opened.

22. **bbordh, bborhhsx**:
    - Type: Input files.
    - Usage: Store open order header data and its history.
    - Access: Read-only, user-opened.

23. **bbordd, bbordhsx**:
    - Type: Input files.
    - Usage: Store open order detail data and its history.
    - Access: Read-only, user-opened.

24. **bbordm, bbormhsx**:
    - Type: Input files.
    - Usage: Store open order miscellaneous data and its history.
    - Access: Read-only, user-opened.

**Note**: The `empmas` and `emphisz` files are commented out (revision `jb07`), indicating they are no longer used.

---

### External Programs Called

1. **GSDTEDIT**:
   - Purpose: Validates the input date (`p#mdy`) and returns a converted date (`p#cymd`) or an error flag (`p#err`).
   - Called in: `sf1rep` for validating the repositioning date (`c$emdy`).

2. **QCMDEXC**:
   - Purpose: Executes file override commands (`OVG` or `OVZ`).
   - Called in: `opntbl`.

3. **QMHSNDPM**:
   - Purpose: Sends error messages to the program message queue.
   - Called in: `addmsg`.

4. **QMHRMVPM**:
   - Purpose: Clears the message subfile.
   - Called in: `clrmsg`.

---

### Summary

The **GB730P** program provides a read-only interface for viewing historical changes to various files, including the National Diesel Fuel Index (`BBNDFI`) used by `BB910`. It processes input parameters to identify the file and key fields, applies file overrides, retrieves the most recent change date, and displays history records in a subfile. Users can navigate through records, refresh the display, or reposition by date, with errors handled via a message subfile. The program supports multiple files with specific keylists and ensures data integrity through validation and proper file access.