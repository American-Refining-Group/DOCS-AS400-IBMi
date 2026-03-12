The RPG program `MGSTABL` is a generic table values lookup utility within the Brandford Order Entry/Invoices system, designed to allow users to select values from the `gstabl` table based on a provided table type. It is called from the `BB912` program (via the F04 key for prompting the `S1REGN` field) to facilitate the selection of valid region codes or other table values. The program uses a display file (`mgstabld`) with a single subfile (`SFL1`) to present table entries to the user for selection. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called .

### Process Steps of the MGSTABL RPG Program

1. **Initialization (*INZSR Subroutine)**:
   - **Receive Parameters**: Accepts two input parameters:
     - `p$type`: The table type code (e.g., for region codes in `BB912`).
     - `p$code`: The selected table code (output parameter, returned to the calling program).
   - **Set Defaults**: Initializes subfile control fields (`rrn1`, `rrnsv1`, `pagsz1` set to 14), flags (`sf1agn`, `repsfl`), and the header field (`c$hdr1` set to "Lookup:" concatenated with the table description or an error message).
   - **Field Definitions**: Defines work fields (`r$type`, `h$type`, `r$srch`, `h$srch`) for repositioning and search functionality, and a key list (`kltype`) for accessing the `gstabl` file.
   - **Set Protection**: Sets the global protect indicator `*IN70` to `*ON` by default, disabling input unless a valid table type is provided (`ps#prm >= 1`), in which case `*IN70` is set to `*OFF` to allow user interaction.

2. **Error Handling (*PSSR Subroutine)**:
   - Checks for parameter errors: If the program status error code (`ps#err`) is 221 (indicating a parameter mismatch), sets a return code of `*DETC` and exits.
   - Ensures the correct number of parameters is passed to prevent runtime errors.

3. **Process Parameters (@parms Subroutine)**:
   - Moves the input table type (`p$type`) to the subfile control field (`c$type`) if at least one parameter is provided (`ps#prm >= 1`).
   - Disables field protection (`*IN70 = *OFF`) to allow user input when a valid table type is provided.

4. **Process Subfile 1 (srsfl1 Subroutine)**:
   - **Position File**: Calls `sf1rep` to position the `gstabl` file based on the table type (`c$type`) and optional search string (`c$srch`).
   - **Write Overlay**: Writes the window overlay record (`wdwovr`) to set up the display.
   - **Main Loop**:
     - Repositions the subfile if requested (`repsfl = *ON`), updating `c$type` and `c$srch` from retained values (`r$type`, `r$srch`).
     - Checks for existing subfile records to enable/disable display control (`*IN41` set to `*ON` if `rrn1 > 0`, else `*OFF`).
     - Writes the window overlay (`wdwovr`) and subfile control record (`sflwdw1`), then displays the subfile control (`sflctl1`) using `EXFMT` with `*IN40` (SFLDSPCTL) set to `*ON`.
     - Clears format indicators (`*IN50` to `*IN69`) to reset error states.
     - Sets the subfile record number (`rcdnb1`) to the current page (`pagrrn`) for proper redisplay.
     - **Process User Input (Before Subfile Read)**:
       - **F12 (Return)**: Exits the subfile loop (`sf1agn = *OFF`).
       - **Page Down**: Loads additional subfile records (`sf1lod`).
       - **Enter**: Processes subfile selections (`sf1prc`).
     - **Process User Input (After Subfile Read)**:
       - **Position Request**: If the table type (`c$type`) or search string (`c$srch`) changes from retained values (`h$type`, `h$srch`), repositions the subfile (`sf1rep`).
       - **F10**: Positions the cursor to the control record by clearing `row1` and `col1` and setting `*IN69`.

5. **Process Subfile on Enter (sf1prc Subroutine)**:
   - Checks if the subfile is not empty (`rrn1 > 0`).
   - Reads changed subfile records (`readc sfl1`) and processes each changed record (`sf1chg`) until no more changes are found (`*IN81 = *OFF`).

6. **Process Subfile Record Change (sf1chg Subroutine)**:
   - Processes the subfile option field (`s1opt`):
     - **Option 1 (Select)**: Calls `sf1s01` to handle the selection of a table value.

7. **Select Value (sf1s01 Subroutine)**:
   - Moves the selected table code (`tbcode`) to the output parameter (`p$code`).
   - Exits the subfile loop (`sf1agn = *OFF`), returning control to the calling program with the selected value.

8. **Reposition Subfile (sf1rep Subroutine)**:
   - Clears the subfile (`sf1clr`) and resets `rrn1` and `rrnsv1`.
   - Validates control record input (`sf1cte`):
     - Chains to `gstabl` using `kltype` to verify the table type (`c$type`). If invalid (`*IN50 = *ON`), sets the header to "Lookup: Invalid Table Type Code" (`com(01)`). If valid, sets the header to "Lookup:" concatenated with the table description (`tbdesc`).
   - Retains control fields (`c$type` to `r$type` and `h$type`, `c$srch` to `r$srch` and `h$srch`) for repositioning.
   - Calculates the length of the search string (`c$srch`) using `%len(%trim(c$srch))` for filtering.
   - Applies protection schemes (`sf1pro`).
   - Loads subfile records (`sf1lod`).

9. **Subfile Protection Schemes (sf1pro Subroutine)**:
   - Sets `*IN76` to `*ON` if no parameters are passed (`ps#prm <= 0`), protecting fields in error scenarios. Otherwise, sets `*IN76` to `*OFF` to allow input.

10. **Load Subfile Records (sf1lod Subroutine)**:
    - Sets `rrn1` to the last subfile record number (`rrnsv1`) to append new records.
    - Sets `rcdnb1` to `rrn1 + 1` to ensure the new page is displayed with the cursor on the first record.
    - Loops up to the page size (`pagsz1 = 14`):
      - Reads the next `gstabl` record for the table type (`c$type`) using `reade`.
      - Checks for end of file (`*IN43 = *ON`) and adjusts `rcdnb1` if needed.
      - Filters records if a search string (`c$srch`) is provided, using `scan` to match against the table description (`tbdesc`). Skips non-matching records.
      - Formats the subfile record (`sf1fmt`), clearing the option field (`s1opt`).
      - Writes the record to the subfile (`sfl1`) and increments `rrn1`.
    - Saves the last subfile record number (`rrnsv1 = rrn1`).
    - Clears cursor position fields (`row1`, `col1`).

11. **Clear Subfile (sf1clr Subroutine)**:
    - Resets `rrn1` and `rrnsv1` to zero.
    - Sets `*IN40` (SFLDSPCTL) and `*IN41` (SFLDSP) to `*OFF` and `*IN42` (SFLCLR) to `*ON`, writes the subfile control record (`sflctl1`), then clears `*IN42`.

12. **Program Termination**:
    - Closes all files and sets `*INLR = *ON` to end the program.

### Business Rules
1. **Inquiry-Only Mode**:
   - The program operates in inquiry mode, allowing users to select a table value but not modify `gstabl` records. Field protection (`*IN70`) is enabled unless a valid table type is provided.

2. **Table Type Validation**:
   - The table type (`c$type`) must exist in `gstabl`. If invalid, an error message ("Invalid Table Type Code") is displayed in the header, and no records are loaded.

3. **Search Functionality**:
   - Users can filter table records by entering a search string (`c$srch`), which is matched against the table description (`tbdesc`) using a `scan` operation. Only matching records are loaded into the subfile.

4. **Selection**:
   - Users select a table value by entering option 1 (`s1opt = 1`) in the subfile, which returns the selected code (`tbcode`) to the calling program via `p$code`.

5. **Navigation**:
   - **F12**: Returns to the calling program without selecting a value.
   - **F10**: Positions the cursor to the control record for entering a new table type or search string.
   - **Page Down**: Loads additional table records.
   - **Enter**: Processes selections or repositions the subfile based on updated control fields (`c$type`, `c$srch`).

6. **Subfile Display**:
   - The subfile displays up to 14 records per page (`pagsz1`).
   - Records are filtered by table type and optionally by a search string.
   - The header dynamically reflects the table description or an error message.

### Tables Used
1. **gstabl**: Input file (read-only) containing table type codes and descriptions, used to populate the subfile with valid values (e.g., region codes for `BB912`).

### External Programs Called
- None explicitly called. The program does not reference external programs like `QCMDEXC`, `QMHSNDPM`, or `QMHRMVPM` for file overrides or message handling, unlike `BB912` or `BB910`. It also does not use `GSDTEDIT` for date validation, as date processing is minimal.

### Additional Notes
- The program is tightly integrated with `BB912`, specifically for prompting region codes (`S1REGN`) via the F04 key, returning the selected region code to the calling program.
- The simplicity of the program (no external program calls, single file access) makes it a lightweight lookup utility, focused on user selection of predefined table values.
- The use of a window overlay (`wdwovr`) suggests a popup-style interface, enhancing user experience by displaying the lookup within the context of the calling program’s screen.

This program provides a user-friendly interface for selecting valid table values, ensuring accurate data entry in the calling program (`BB912`) with robust validation and filtering capabilities.