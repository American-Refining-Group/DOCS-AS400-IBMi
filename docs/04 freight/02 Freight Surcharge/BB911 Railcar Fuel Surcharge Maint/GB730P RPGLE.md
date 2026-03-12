The **GB730P** RPG program is designed for **Global File Tracking History Inquiry**, allowing users to view historical changes to various master and transaction files within the system. It is called from the main OCL (Operating Control Language) and uses a display file with a subfile (SFL1) to present historical data. The program supports inquiries for multiple file types, each with specific key fields and data structures, and uses overrides to access files in different libraries based on the file group ('G' or 'Z'). Below is a detailed explanation of the process steps, business rules, tables used, and external programs called .

---

### **Process Steps of GB730P**

The program follows a structured flow with subroutines to initialize parameters, open files, set date limits, and process the subfile for displaying historical data. Here’s a breakdown of the main process steps based on the provided code:

1. **Initialization (*inzsr Subroutine)**:
   - **Purpose**: Sets up the program's environment and initializes variables.
   - **Steps**:
     - Receives entry parameters via the `p$elist` data structure: `p$file` (file name, e.g., 'BBNDFI'), `p$fgrp` (file group, 'G' or 'Z'), and `p$co` (company code).
     - Calls `getpgmparm` to process program parameters and map them to appropriate data structures based on the file name.
     - Calls `opntbl` to open database files with overrides.
     - Calls `GetStartDt` to determine the starting date for the history inquiry.
     - Calls `setlimit` to set limits for the subfile display.
     - Defines key lists (e.g., `msgkey`, `shiptkey`, `arcupkey`, etc.) for accessing various files based on the file name.
     - Sets up generic work fields: `RRN1` and `RRNSV1` (relative record numbers) to zero, `PAGSZ1` (page size) to 10, and flags (`FRSTRD`, `REPSFL`, `FMTAGN`, `DSPMSG`, `movehdrflg`) to their initial states.
     - Sets up message handling fields (`m@pgmq`, `m@key`) and parameters for date validation (`pldted`).
     - Assigns a header (`C$HDR1`) from the `HDR` array based on the file name (e.g., 'National Diesel Fuel Index Tracking' for 'BBNDFI').
     - Defines fields for date and user handling using `*LIKE` definitions (e.g., `r$fdat`, `h$user`).

2. **Get Program Parameters (getpgmparm Subroutine)**:
   - **Purpose**: Maps the input parameters to the appropriate data structure based on the file name (`p$file`).
   - **Steps**:
     - Uses a `SELECT` structure to match `p$file` to one of the supported files (e.g., 'SHIPTO', 'ARCUPR', 'BBNDFI', etc.).
     - Moves the input parameter string (`p$elist`) to the corresponding data structure (e.g., `bbndparm` for 'BBNDFI'), which contains specific fields like company code, effective date, effective time, and region for 'BBNDFI'.
     - For certain files (e.g., 'ARCUPR', 'BICUFR'), sets indicator `*in75` to enable specific display features.

3. **Open Database Tables (opntbl Subroutine)**:
   - **Purpose**: Opens the required database files with appropriate overrides based on the file group.
   - **Steps**:
     - Uses the `OVG` or `OVZ` arrays (depending on `p$fgrp` = 'G' or 'Z') to apply `OVRDBF` commands via `QCMDEXC` for files like `glcont`, `gbfmsg`, `bbndfi`, etc., redirecting them to the appropriate library (e.g., `*LIBL/gbbndfi` for 'G', `*LIBL/zbbndfi` for 'Z').
     - Opens all files defined in the file specification (F-specs), marked as `USROPN`, including master files (e.g., `glcont`, `shiptz`) and their history counterparts (e.g., `shipthx`, `bbndfihx`).

4. **Get Starting Date (GetStartDt Subroutine)**:
   - **Purpose**: Determines the most recent change date and time for the specified file to initialize the subfile.
   - **Steps**:
     - Uses a `SELECT` structure to handle each file type (e.g., 'BBNDFI', 'BBORDH', etc.).
     - For each file, positions the history file (e.g., `bbndfihx` for 'BBNDFI') using the appropriate key list (e.g., `bbndfikey`) with `SETGT`.
     - Reads the most recent record (`READPE`) and checks if a record exists (`*in69 = *off`).
     - Converts the change date (`lhchd8`, `mhchd8`, etc.) from `YYYYMMDD` to `MMDDYY` format using multiplication by `100.0001` and stores it in `sfcfdt` (subfile change date) with `%editc(datmdy: 'Y')`.
     - Sets the change time (`sfcftm`) from the history record’s time field (e.g., `lhchtm` for 'BBNDFI').
     - Resets the file pointer to the beginning (`*LOVAL` with `SETLL`) for subsequent processing.

5. **Set Limit (setlimit Subroutine)**:
   - **Purpose**: Configures the subfile page size and display limits.
   - **Steps**:
     - Sets `PAGSZ1` to 10 (subfile page size).
     - Initializes other control fields as needed (not fully detailed in the provided code snippet).

6. **Main Processing Loop (Not Explicitly Shown but Implied)**:
   - **Purpose**: Displays and manages the subfile (SFL1) for history inquiry.
   - **Steps** (based on typical RPG subfile processing):
     - Clears the subfile (`SFLCLR`) and loads historical records based on the file and key specified in `p$elist`.
     - Displays the subfile control record (`SFLCTL`) using `EXFMT` to present the history data to the user.
     - Processes user inputs (e.g., F3 to exit, F5 to refresh, page down to load more records).
     - Formats each subfile record with fields like change date (`sfcfdt`), change time (`sfcftm`), and other file-specific data (e.g., region and price for 'BBNDFI').
     - Handles navigation and repositioning based on user actions.

7. **Program Termination**:
   - Closes all open files (`CLOSE *ALL`).
   - Sets `*INLR = *ON` and returns control to the calling OCL.

---

### **Business Rules**

The program enforces the following business rules based on the code and context:

1. **File-Specific Inquiry**:
   - The program supports history inquiries for multiple files (e.g., 'BBNDFI', 'BBORDH', 'BBORDD', etc.), each with its own key structure and data fields.
   - The file to be queried is specified via the `p$file` parameter, and the program dynamically adjusts its behavior (e.g., key lists, headers) based on this input.

2. **File Group Overrides**:
   - Files are accessed in either the 'G' or 'Z' library group, determined by `p$fgrp`. The program applies `OVRDBF` commands to ensure the correct library is used (e.g., `gbbndfi` for 'G', `zbbndfi` for 'Z').

3. **Date Validation**:
   - The program uses the `GSDTEDIT` external program to validate dates (via the `pldted` parameter list) when necessary, ensuring that change dates are in a valid format.

4. **History Tracking**:
   - For each file, the program retrieves the most recent change date and time from the corresponding history file (e.g., `bbndfihx` for 'BBNDFI') to initialize the subfile.
   - Historical data includes fields like change date (`lhchd8`), change time (`lhchtm`), user (`lhuser`), and file-specific fields (e.g., region and price for 'BBNDFI').

5. **Subfile Display**:
   - The subfile displays up to 10 records per page (`PAGSZ1 = 10`).
   - Users can navigate through records using page down (`PAGEDN = X'F5'`) and other function keys (e.g., F3 to exit, F5 to refresh).

6. **Message Handling**:
   - Errors, such as invalid system and program IDs entered together, are handled via the message file (`gbfmsg`) and displayed in the message subfile (`msgctl`).
   - The error message 'Cannot Enter both Sys And Program Id' is defined in the `COM` array.

7. **Security**:
   - The program is proprietary to WCI International Company, and unauthorized use is prohibited, as noted in the security statement.

---

### **Tables Used**

The program interacts with the following database files, all defined with `USROPN` for manual control and `SHARE(*NO)` for exclusive access. Each file has a corresponding history file suffixed with 'hx' (except `gbfmsg`):

1. **glcont**: Input file for general ledger control data.
2. **shiptz**: Input file for ship-to master data.
3. **shipthx**: Input file for ship-to history data.
4. **arcup31**: Input file for customer product code data.
5. **arcuphx**: Input file for customer product code history.
6. **cuadrx**: Input file for customer ship-to address data.
7. **cuadrhx**: Input file for customer ship-to address history.
8. **arcusz**: Input file for A/R customer master data.
9. **arcushx**: Input file for A/R customer master history.
10. **arcuspx**: Input file for A/R customer supplemental data.
11. **arcusphx**: Input file for A/R customer supplemental history.
12. **bicuag**: Input file for customer sales agreement data.
13. **bicuaghx**: Input file for customer sales agreement history.
14. **bbprce**: Input file for rack pricing data.
15. **bbprcehx**: Input file for rack pricing history.
16. **bbndfi**: Input file for national diesel fuel index data.
17. **bbndfihx**: Input file for national diesel fuel index history.
18. **bbcfsh**: Input file for carrier fuel surcharge header data.
19. **bbcfshhx**: Input file for carrier fuel surcharge header history.
20. **bbcfsd**: Input file for carrier fuel surcharge detail data.
21. **bbcfsdhx**: Input file for carrier fuel surcharge detail history.
22. **bicuf2**: Input file for customer freight data.
23. **bicufrhx**: Input file for customer freight history.
24. **gsprod**: Input file for product code data.
25. **gsprodhx**: Input file for product code history.
26. **gshazm**: Input file for hazmat/shipping description data.
27. **gshazmhx**: Input file for hazmat/shipping description history.
28. **gsumcv**: Input file for product unit of measure conversion data.
29. **gsumcvhx**: Input file for product unit of measure conversion history.
30. **gsctum**: Input file for container unit of measure conversion data.
31. **gsctumhx**: Input file for container unit of measure conversion history.
32. **gsctwt**: Input file for container weight data.
33. **gsctwthx**: Input file for container weight history.
34. **biprt2**: Input file for product tax code data.
35. **biprtxhx**: Input file for product tax code history.
36. **gscntr1**: Input file for container code data.
37. **gscntrhx**: Input file for container code history.
38. **bbordh**: Input file for open order header data (renamed to `bbordhsr`).
39. **bborhhsx**: Input file for open order header history.
40. **bbordd**: Input file for open order detail data.
41. **bbordhsx**: Input file for open order detail history.
42. **bbordm**: Input file for open order miscellaneous data (renamed to `bbormhsr`).
43. **bbormhsx**: Input file for open order miscellaneous history.
44. **gbfmsg**: Input file for message data, used for error message handling.

**Note**: The `empmas` and `emphisz` files were removed in revision `jb07` (06/19/2024), so they are no longer used.

---

### **External Programs Called**

The program calls the following external programs:
1. **QCMDEXC**: Executes `OVRDBF` commands to override file mappings to the correct library based on the file group ('G' or 'Z').
2. **GSDTEDIT**: Validates dates via the `pldted` parameter list, taking a date in `MMDDYY` format (`p#mdy`), returning a converted date in `YYYYMMDD` format (`p#cymd`), and setting an error flag (`p#err`) if invalid.

---

### **Summary**

The **GB730P** program is a versatile RPG application for inquiring into the history of changes across multiple master and transaction files, including the National Diesel Fuel Index (`BBNDFI`) used by **BB911**. It initializes by processing input parameters, applying file overrides, and determining the starting date for history records. The program uses a subfile to display historical data, with file-specific key lists and headers tailored to each file type. Business rules ensure proper file access, date validation, and error handling, with a focus on displaying change dates, times, and user details. The program interacts with a wide range of files (master and history) and relies on `QCMDEXC` for file overrides and `GSDTEDIT` for date validation, making it a critical component for auditing changes in the system.