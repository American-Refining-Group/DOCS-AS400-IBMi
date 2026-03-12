The provided document, `BB955.rpgle (2).txt`, is the second part of the RPGLE program `BB955`, which is called from the OCL program `BB955.ocl36.txt` for "Rack Price File Maintenance" in the Bradford Order Entry / Invoices system. This part includes additional subroutines and data definitions not covered in the first part. Below, I’ll explain the process steps, business rules, tables used, and external programs called, integrating this with the first part for a cohesive analysis. Since the code is still truncated, I’ll focus on the provided subroutines and reference the first part where necessary.

### Process Steps of the RPGLE Program (Part 2)

The `BB955` program is an interactive RPGLE program that manages rack price data using a subfile-based display. The second part provides subroutines for validating input, loading subfile records, handling messages, and opening database files. Below are the process steps for the subroutines in this part, continuing from the main flow described in the first part:

1. **Edit Subfile Control Input (`sf1cte` Subroutine)**:
   - **Purpose**: Validates user input in the subfile control record (`sflctl1`) to ensure valid values for filtering rack price records.
   - **Steps**:
     - Clears error indicators (`*IN50` to `*IN69`, `*IN30` to `*IN39`) using `zero20` and `zero10` constants.
     - **Company (`c1cono`)**: Chains to `bicont` to validate the company number. If invalid or zero, sets error message `ERR0000` with comment `com(01)` ("Invalid Company...F4 To View Valid Values"), sets `*IN50` and `*IN51`, and clears the company name (`c1conm`). If valid, populates `c1conm` with the company name (`bcname`).
     - **Location (`c1loc`)**: Chains to `inloc` using keylist `klloc`. If invalid or blank, sets error message `ERR0000` with `com(02)` ("Invalid Location..."), sets `*IN50` and `*IN52`, and clears the location name (`c1lcnm`). If valid, populates `c1lcnm` with `ilname`.
     - **Product (`c1prod`)**: If non-blank, chains to `gsprod` using `klprod1`. If invalid, sets error message `ERR0000` with `com(03)` ("Invalid Product..."), sets `*IN50` and `*IN53`, and clears the product description (`c1prds`). If valid, populates `c1prds` with `tpdesc`.
     - **Container (`c1cntr`)**: If non-blank, validates that the container code is between '000' and '999'. If invalid, sets error message `ERR0000` with `com(04)` ("Invalid Container..."), sets `*IN50` and `*IN54`. If valid, chains to `gscntr1` using `k$cntr`. If the chain fails, sets the same error; otherwise, populates `c1cnds` with `tcdesc`.
     - **Unit of Measure (`c1unms`)**: If non-blank, chains to `gstabl` using `klunms` (with `k$biunms = 'BIUNMS'`). If invalid, sets error message `ERR0000` with `com(05)` ("Invalid Unit Of Measure..."), sets `*IN50` and `*IN55`. If valid, populates `c1umds` with `tbdesc`.
     - **Product Class (`c1prcl`)**: If non-blank, constructs a key (`w$code`) from `c1cono` and `c1prcl`, then chains to `gstabl` using `klprcl` (with `k$prodcl = 'PRODCL'`). If invalid, sets error message `ERR0000` with `com(06)` ("Invalid Product Class..."), sets `*IN50` and `*IN56`. If valid, populates `c1pcds` with `tbdesc`.
     - **Status (`c1stat`)**: Defaults to 'CF ' (Current and Future) if blank. Validates that the status is one of 'ALL', 'CUR', 'FUT', or 'CF '. If invalid, sets error message `ERR0000` with `com(07)` ("Invalid Status..."), sets `*IN50` and `*IN57`.
     - **From Date (`c1fmdy`)**: If non-zero, calls `GSDTEDIT` to validate the date. If valid, sets `c1fdat` to the converted date (`p#cymd`). If invalid, sets error `ERR0020` ("Invalid date"), sets `*IN50` and `*IN58`. If blank, sets `c1fdat` to 99999999.
     - **To Date (`c1tmdy`)**: Similar to `c1fmdy`, validates using `GSDTEDIT`. If valid, sets `c1tdat` to `p#cymd`. If invalid, sets error `ERR0020`, sets `*IN50` and `*IN59`. If blank, sets `c1tdat` to zero.
     - **Inactive Status (`c1inac`)**: Validates as blank (Active), 'I' (Inactive, no orders), or 'B' (Inactive, orders allowed). If invalid, sets error `ERR0000` with `com(08)` ("Invalid Inactive..."), sets `*IN50` and `*IN60`.
     - **No Rack Price (`c1rkrq`)**: Validates as blank or 'N' (No rack price). If invalid, sets error `ERR0000` with `com(09)` ("Invalid No Rack Price..."), sets `*IN50` and `*IN61`.
     - **Division (`c1dvsn`)**: Validates as blank (All), 'R' (Refinery), or 'L' (Blended Lube). If invalid, sets error `ERR0000` with `com(10)` ("Invalid Division..."), sets `*IN50` and `*IN62`.

2. **Edit Subfile Control Input for New Records (`sf1ctenew` Subroutine)**:
   - **Purpose**: Validates additional locations and effective date for new rack price entries.
   - **Steps**:
     - Clears error indicators (`*IN50` to `*IN69`, `*IN30` to `*IN39`).
     - **Additional Locations (`loc` array)**: Loops through 10 locations (`loc` array). For each non-blank location:
       - Validates by checking `inloc` using `klloc2`. If invalid, sets error `ERR0000` with `com(02)` ("Invalid Location..."), sets `*IN50` and an indicator (`*IN30` to `*IN39`) based on the index.
       - Checks that the location is not the same as the current location (`c1loc`). If it is, sets error `ERR0000` with `com(20)` ("Additional Location Cannot Be Same..."), sets `*IN50` and the corresponding indicator.
       - Sets `w$exis` to indicate locations are entered.
     - **Effective Date (`c1emdy`)**: If non-zero, validates using `GSDTEDIT`. If valid, sets `c1edat` to `p#cymd`. If invalid, sets error `ERR0020`, sets `*IN50` and `*IN63`. If blank, clears `c1edat`.
     - **Location and Effective Date Check**: If new records are being added (`w$newrecs`) and locations are entered (`w$exis`) but `c1emdy` is zero, sets error `ERR0000` with `com(16)` ("Must Enter Effective Date..."), sets `*IN50` and `*IN63`.
     - If no errors (`*IN50 = *OFF`), saves `c1emdy` to `r$emdy` and `loc` to `locr` for repositioning.

3. **Load All Records to Subfile (`sf1lodall` Subroutine)**:
   - **Purpose**: Populates the subfile (`sfl1`) with records from the work file (`bb955w`).
   - **Steps**:
     - Resets the multiple quantities flag (`w$mltqty`) and subfile folding (`*IN45`, `sfmod1`, `*IN79`).
     - Sets the relative record number (`rrn1`) to the saved value (`rrnsv1`) to append records correctly.
     - Sets the subfile record number (`rcdnb1`) to `rrn1 + 1` for proper display.
     - Reads the work file (`bb955w`) starting from the keylist `kls1s3` (`c1cono`, `c1ppos`).
     - For each record:
       - Skips deleted records (`w1del = 'D'`) if `s1showdel` is off.
       - Formats the subfile line (`sf1fmt`).
       - Determines the subfile record color (`sf1rtvclr`).
       - Applies protection schemes (`sf1pro`).
       - Sets `w$mltqty` if any quantity (`s1qt(01)`) is non-zero.
       - Writes the record to the subfile and increments `rrn1`.
     - Sets `*IN43` (SFLEND) if the end of the file is reached.

4. **Add Message (`addmsg` Subroutine)**:
   - **Purpose**: Sends error messages to the message subfile.
   - **Steps**:
     - Calls `QMHSNDPM` to send the message using fields `m@id`, `m@msgf` ('GSMSGF'), `m@data`, `m@l`, `m@type` ('*DIAG'), `m@pgmq`, `m@scnt`, `m@key`, and `m@errc`.
     - Clears message data fields (`m@data`, `m@l`).
     - Sets the `dspmsg` flag to display the message subfile.

5. **Write Message Subfile (`wrtmsg` Subroutine)**:
   - **Purpose**: Displays the message subfile.
   - **Steps**:
     - Sets `*IN49` to enable subfile display and control, writes `msgctl`, then resets `*IN49`.

6. **Clear Message Subfile (`clrmsg` Subroutine)**:
   - **Purpose**: Clears the message subfile.
   - **Steps**:
     - Resets `dspmsg` flag.
     - Saves the current record format (`rcdnam`) and page RRN (`pagrrn`).
     - Calls `QMHRMVPM` to remove messages using `m@pgmq`, `m@scnt`, `m@rmvk`, `m@rmv` ('*ALL'), and `m@errc`.
     - Restores `rcdnam` and `pagrrn`.

7. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens all database files with appropriate overrides.
   - **Steps**:
     - If the file group parameter (`p$fgrp`) is 'G' or 'Z', applies overrides from arrays `ovg` or `ovz` (respectively) using `QCMDEXC` for files `bicont`, `gscntr1`, `gsctum`, `gsprod`, `gstabl`, `inloc`, `bbprcw`, `bbprce`, and `bbprceh`.
     - Applies an override for `bb955w` to `QTEMP/bb955w` (revision `jk04`) using `ovr` array.
     - Opens files: `bicont`, `gscntr1`, `gsctum`, `gsprod`, `gstabl`, `inloc`, `bbprcw`, `bbprce`, `bbprceh`, and `bb955w`.

8. **Initialization Subroutine (`*inzsr`)**:
   - **Purpose**: Initializes program variables and parameters.
   - **Steps**:
     - Receives the file group parameter (`p$fgrp`) from the OCL call.
     - Initializes date and time fields using the program status data structure (`psds##`): sets `ps#mdy`, `ps#hms`, and converts to century (`ps#cen`) and date formats (`ps#ymd`, `ps#dat`).
     - Initializes subfile control fields (`rrn1`, `rrnsv1`, `k$rrn`, `pagsz1`), message handling fields (`dspmsg`, `m@pgmq`, `m@key`), and work fields.
     - Sets `o$file` to 'BBPRCE' and clears control fields (`c1cono`, `s1date`, `s1time`).

### Business Rules

The business rules in this part focus on input validation and subfile management, complementing those from the first part:
- **Input Validation**:
  - **Mandatory Fields**: Company (`c1cono`), location (`c1loc`), product (`c1prod`), container (`c1cntr`), unit of measure (`c1unms`), and product class (`c1prcl`) must exist in their respective files (`bicont`, `inloc`, `gsprod`, `gscntr1`, `gstabl`). Invalid entries trigger error `ERR0000` with specific comments (`com(01)` to `com(06)`).
  - **Container Range**: Container codes (`c1cntr`) must be between '000' and '999'.
  - **Status Values**: Status (`c1stat`) must be 'ALL', 'CUR', 'FUT', or 'CF ' (defaults to 'CF ' if blank).
  - **Date Validation**: From and to dates (`c1fmdy`, `c1tmdy`) must be valid, checked via `GSDTEDIT`. Invalid dates trigger `ERR0020`.
  - **Inactive Status**: Must be blank, 'I', or 'B' (`c1inac`).
  - **No Rack Price**: Must be blank or 'N' (`c1rkrq`).
  - **Division**: Must be blank, 'R', or 'L' (`c1dvsn`).
  - **Additional Locations**: Must exist in `inloc` and not match the current location (`c1loc`). Errors trigger `com(02)` or `com(20)`.
  - **Effective Date for New Records**: Required if additional locations are entered (`com(16)`).
- **Subfile Loading**:
  - Excludes deleted records (`w1del = 'D'`) unless `s1showdel` is on.
  - Sets multiple quantities flag (`w$mltqty`) if quantities are non-zero.
  - Uses `sf1fmt`, `sf1rtvclr`, and `sf1pro` to format, color, and protect subfile records.
- **Message Handling**: Errors are displayed in a message subfile (`GSMSGF`) with specific IDs and comments. Messages are cleared and redisplayed as needed.
- **File Overrides**: Supports two file groups ('G' or 'Z') for library-specific file access, with `bb955w` overridden to `QTEMP` (revision `jk04`).
- **Currency**: Assumes U.S. dollars (`c#US$` from part 1).

### Tables Used

The tables used are consistent with the first part, with their usage clarified in this part:
1. **bb955d**: Workstation file with subfile `sfl1` for user interaction.
2. **bicont**: Input file for validating company numbers (`c1cono`) and retrieving company names (`bcname`).
3. **gscntr1**: Input file for validating container codes (`c1cntr`) and retrieving descriptions (`tcdesc`).
4. **gsctum**: Input file, likely for customer or unit master data (usage not detailed in this part, but noted as inactive in `JB05`).
5. **gsprod**: Input file for validating product codes (`c1prod`) and retrieving descriptions (`tpdesc`).
6. **gstabl**: Input file for validating unit of measure (`c1unms`, `BIUNMS`) and product class (`c1prcl`, `PRODCL`), retrieving descriptions (`tbdesc`).
7. **inloc**: Input file for validating locations (`c1loc`, `loc` array) and retrieving names (`ilname`).
8. **bbprcw**: Input file (renamed `bbprcepf`) for reading price data.
9. **bb955w**: Update file in `QTEMP` for staging rack price changes.
10. **bbprce**: Update file for primary rack price data.
11. **bbprceh**: Output file for logging price history.

### External Programs Called

This part introduces additional external programs:
1. **GSDTEDIT**: Called to validate dates (`c1fmdy`, `c1tmdy`, `c1emdy`). Parameters include `p#mdy` (input date), `p#cymd` (converted date), and `p#err` (error flag).
2. **QMHSNDPM**: System API to send messages to the message subfile.
3. **QMHRMVPM**: System API to clear messages from the message subfile.
4. **QCMDEXC**: System API to execute file override commands for `bb955w`, `bicont`, `gscntr1`, `gsctum`, `gsprod`, `gstabl`, `inloc`, `bbprcw`, `bbprce`, and `bbprceh`.

From the first part:
5. **GB730P**: Called for history inquiry (F9).
6. **BB955AC**: Called to create/clear the work file (`bb955w`).

### Integration with First Part

The subroutines in part 2 support the main flow from part 1:
- **sf1cte** and **sf1ctenew** validate inputs for filtering (`srsfl1`, `sf1rep`) and adding new records (`sf1f07`, `addadlloc`), ensuring data integrity before database updates.
- **sf1lodall** populates the subfile in `srsfl1` and `sf1rep`, handling record filtering and display.
- **addmsg**, **wrtmsg**, and **clrmsg** manage error display in `srsfl1`, `sf1cte`, and `sf1ctenew`.
- **opntbl** is called at program start to open files, supporting all database operations.
- **`*inzsr`** initializes the program, setting up parameters from the OCL call (`'?9?'`) and date/time fields.

### Additional Notes

- **Truncated Code**: Subroutines like `sf1fmt`, `sf1rtvclr`, `sf1pro`, `bldwrkf`, and `writehist` are missing, limiting details on record formatting, coloring, protection, work file building, and history logging. If you have these, I can provide further analysis.
- **Revisions**: Part 2 references `jk04` (override `bb955w` to `QTEMP`) and indirectly supports `JB05` (inactive records in `gsctum` treated as deleted).
- **Parameter `'?9?'`**: Likely influences file group selection ('G' or 'Z') in `opntbl`, but its exact role is unclear without `rtvdta`.
- **Error Messages**: The `com` array provides detailed error descriptions, enhancing user feedback.

If you have the remaining code or specific subroutines, please share them for a complete analysis. Alternatively, I can use DeepSearch to gather context on AS/400 RPG conventions or similar systems. Let me know how to proceed!