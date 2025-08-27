The provided document, `GB730P.rpgle.txt`, is an RPGLE program named `GB730P`, called from the `BB955` program (via the F9 key for history inquiry) within the context of the OCL program `BB955.ocl36.txt` for "Rack Price File Maintenance." The program serves as a "Global File Tracking History Inquiry," displaying historical data for various files, including the rack price file (`bbprce`). Below, I’ll explain the process steps, business rules, tables used, and external programs called based on the provided code, integrating it with the context from `BB955.rpgle` (parts 1 and 2).

### Process Steps of the RPGLE Program (`GB730P`)

The `GB730P` program is an interactive RPGLE program designed to display historical data for multiple files, including `bbprce`, using a subfile-based display (`gb730pd`). The provided code snippet includes file definitions, data structures, and the initialization subroutine (`*inzsr`), but it is truncated, missing the main program logic and other subroutines. Based on the available code and its context, here’s an analysis of the process steps:

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Initializes the program by setting up parameters, headers, and initial field values.
   - **Steps**:
     - **Receive Parameters**: The program accepts parameters via a parameter list (`p$elist`), including:
       - `p$file` (10 characters): Specifies the file to query (e.g., `BBPRCE` for rack pricing history).
       - `p$fgrp` (1 character): File group (`G` or `Z`), determining library overrides.
       - `p$co` (2 digits): Company number.
     - Additional data structures (`bbprparm`, etc.) receive specific fields for each supported file, such as `bbcono`, `bbloc`, `bbprod`, `bbcntr`, `bbunms`, `bbdate`, and `bbtime` for `BBPRCE`.
     - **Set Header**: Uses a `SELECT` block to set the subfile control header (`C$HDR1`) based on `p$file`. For `BBPRCE`, it sets `C$HDR1` to `HDR(8)` ("Rack Pricing File Tracking"). Other files (e.g., `SHIPTO`, `ARCUPR`, etc.) have corresponding headers.
     - **Initialize Fields**:
       - Sets subfile control fields (`RRN1`, `RRNSV1`) to zero and page size (`PAGSZ1`) to 10.
       - Defines `PAGSAV` and `RCDSAV` to hold page and record format values.
       - Initializes flags (`FRSTRD`, `REPSFL`, `FMTAGN`, `DSPMSG`, `movehdrflg`) to `*OFF`.
       - Sets message handling fields: `dspmsg` to blank, `m@pgmq` to `*`, `m@key` to blanks, and `m@msgf` to `GSMSGF *LIBL`.
     - **Date and Time Setup**: Uses the program status data structure (`PSDS##`) to retrieve the job date (`JBDT##`) and user (`USER##8`). Defines fields (`r$fdat`, `r$tdat`, `h$fdat`, `h$tdat`, `h$user`, `r$user`) for date and user tracking, with revisions (`jb07`) updating definitions to use `JBDT##` and `USER##8` instead of `empmas`-specific fields.
     - **Product Field Setup**: Defines `r$prod` and `h$prod` as like `c$prod` for product-related data.

2. **File Overrides**:
   - **Purpose**: Applies library overrides based on the file group (`p$fgrp` = `G` or `Z`).
   - **Steps**:
     - For `p$fgrp = 'G'`, uses the `OVG` array to override files to libraries prefixed with `g` (e.g., `gglcont`, `gbbprce`).
     - For `p$fgrp = 'Z'`, uses the `OVZ` array to override files to libraries prefixed with `z` (e.g., `zglcont`, `zbbprce`).
     - Overrides are defined for all files listed in the file specifications, ensuring the correct library is accessed.

3. **Subfile Processing (Inferred)**:
   - **Purpose**: Loads and displays historical data in the subfile (`SFL1`) of the workstation file (`gb730pd`).
   - **Steps (Based on Context)**:
     - Likely opens files using `USROPN` and applies overrides via `QCMDEXC` (similar to `BB955`’s `opntbl`).
     - Reads historical data from the appropriate history file (e.g., `bbprcehx` for `BBPRCE`) based on the parameters (e.g., `bbcono`, `bbloc`, `bbprod`, `bbcntr`, `bbunms`, `bbdate`, `bbtime`).
     - Populates the subfile with records, formatting fields like dates (using `GSDTEDIT` via `pldted`) and descriptions from reference files (e.g., `gsprod`, `gscntr1`).
     - Handles user interaction (e.g., scrolling, exiting) via the workstation file, with indicators controlling subfile display and errors.

4. **Message Handling (Inferred)**:
   - **Purpose**: Displays error or informational messages in a message subfile.
   - **Steps**:
     - Uses `m@msgf = 'GSMSGF *LIBL'` to send messages via `QMHSNDPM` (similar to `BB955`’s `addmsg`).
     - Clears messages using `QMHRMVPM` (similar to `BB955`’s `clrmsg`).
     - The `COM` array contains at least one error message ("Cannot Enter both Sys And Program Id"), suggesting validation of input parameters.

5. **Program Termination (Inferred)**:
   - Closes all open files (`USROPN`) and returns control to `BB955`.

### Business Rules

The business rules are derived from the code and its role in the `BB955` context:
- **File-Specific History Inquiry**: The program supports history inquiries for multiple files (e.g., `BBPRCE`, `SHIPTO`, `ARCUPR`, etc.), with `BBPRCE` relevant to rack pricing. The header (`C$HDR1`) is set based on the file specified in `p$file`.
- **Parameter Validation**: Ensures valid parameters (e.g., `bbcono`, `bbloc`, `bbprod`) are passed. The `COM(01)` message suggests that system and program IDs cannot both be entered, indicating input validation.
- **File Group Overrides**: Supports two library groups (`G` or `Z`) to access different data sets, ensuring flexibility across environments (e.g., production vs. test).
- **Date Handling**: Uses `GSDTEDIT` for date validation, ensuring historical dates are correctly formatted (e.g., `d#cymd`, `d#mdy`).
- **User and Date Tracking**: Captures the user (`USER##8`) and job date (`JBDT##`) for audit purposes, with revisions (`jb07`) removing dependency on `empmas` fields.
- **Inactive Records**: Likely respects `BB955`’s rule (`JB05`) treating inactive records in `gsctum`, `gsctwt`, and `gsumcv` as deleted, filtering them from history displays unless specified.

### Tables Used

The program uses the following files, all defined with `USROPN` for explicit opening:
1. **glcont**: General ledger control file.
2. **shiptz**, **shipthx**: Ship-to master and history files.
3. **arcup31**, **arcuphx**: Customer product code master and history files.
4. **cuadrx**, **cuadrhx**: Customer ship-to address master and history files.
5. **arcusz**, **arcushx**: A/R customer master and history files.
6. **arcuspx**, **arcusphx**: A/R customer supplemental master and history files.
7. **bicuag**, **bicuaghx**: Customer sales agreement master and history files.
8. **bbprce**, **bbprcehx**: Rack pricing master and history files (key for `BB955` integration).
9. **bbndfi**, **bbndfihx**: National diesel fuel index master and history files.
10. **bbcfsh**, **bbcfshhx**: Carrier fuel surcharge header master and history files.
11. **bbcfsd**, **bbcfsdhx**: Carrier fuel surcharge detail master and history files.
12. **bicuf2**, **bicufrhx**: Customer freight master and history files.
13. **gsprod**, **gsprodhx**: Product code master and history files.
14. **gshazm**, **gshazmhx**: Hazmat/shipping description master and history files.
15. **gsumcv**, **gsumcvhx**: Product unit of measure conversion master and history files.
16. **gsctum**, **gsctumhx**: Container unit of measure conversion master and history files.
17. **gsctwt**, **gsctwthx**: Container weight master and history files.
18. **biprt2**, **biprtxhx**: Product tax code master and history files.
19. **gscntr1**, **gscntrhx**: Container code master and history files.
20. **bbordh**, **bborhhsx**: Open order header master and history files.
21. **bbordd**, **bbordhsx**: Open order detail master and history files (renamed `bbordhpf` to `bbordhsr`).
22. **bbordm**, **bbormhsx**: Open order miscellaneous master and history files (renamed `bbordmpf` to `bbormhsr`).
23. **gbfmsg**: Message file for error messages.

*Note*: The `empmas` and `emphisz` files were removed in revision `jb07`, so they are not used in the current version.

### External Programs Called

The provided code explicitly references one external program:
1. **GSDTEDIT**: Called via the `pldted` parameter list to validate dates, with parameters `p#mdy` (input date), `p#cymd` (converted date), and `p#err` (error flag).

Inferred from context and similarity to `BB955`:
2. **QCMDEXC**: Likely used to apply file overrides (`OVG`, `OVZ`), similar to `BB955`’s `opntbl`.
3. **QMHSNDPM**: System API to send messages to the message subfile.
4. **QMHRMVPM**: System API to clear messages from the message subfile.

### Integration with `BB955` and OCL

- **Context**: `GB730P` is called from `BB955` when the F9 key is pressed, passing parameters via the `x$custhist` data structure (`o$file`, `o$fgrp`, `c1cono`, `c1loc`, `s1prod`, `s1cntr`, `s1unms`, `s1date`, `s1time`). For `BBPRCE`, these map to `bbprparm` (`bbcono`, `bbloc`, `bbprod`, `bbcntr`, `bbunms`, `bbdate`, `bbtime`).
- **OCL Role**: The OCL program (`BB955.ocl36.txt`) initiates `BB955`, which calls `GB730P` for history inquiries, passing the `'?9?'` parameter that likely influences `p$fgrp` ('G' or 'Z').
- **Process Flow**:
  - `BB955` triggers `GB730P` to display `bbprcehx` records for a specific rack price entry (e.g., by company, location, product, container, unit of measure, date, and time).
  - `GB730P` validates parameters, applies overrides, loads historical data into the subfile, and displays it to the user.
  - The user can view changes (e.g., price updates logged via `BB955`’s `writehist` subroutine) and return to `BB955`.

### Additional Notes

- **Truncated Code**: The code is truncated, missing key subroutines (e.g., main subfile processing, message handling). The analysis is based on the initialization and context from `BB955`. If you have the full code, I can provide a detailed breakdown of the subfile logic.
- **Revisions**:
  - `jk01`: Moved customer data to the screen.
  - `jk02`: Expanded price fields in `BAPRCE`/`BAOFFP` to 9.4 packed.
  - `jk03`: Added `BFTSEQ` to `BICUFR` and `gscntr1` support.
  - `jk04`: Added order files (`bbordh`, `bbordd`, `bbordm`).
  - `jk05`: Renamed `BDDDEL` to `BDDEL`.
  - `jk06`: Added `BAPOOR` to `BICUAG`.
  - `jb07`/`jb08`: Updated `BBCFSD:BKFSPC` to 5.2 packed and removed `empmas`, `emphisz`.
- **Limitations**: Without the full code, I cannot detail the exact subfile loading or user interaction logic. The program likely follows a pattern similar to `BB955`’s subfile processing (`srsfl1`, `sf1lodall`).

If you have the complete `GB730P` code or specific subroutines, please share them for a more comprehensive analysis. Alternatively, I can use DeepSearch to gather additional context on AS/400 RPG history inquiry programs. Let me know how to proceed!