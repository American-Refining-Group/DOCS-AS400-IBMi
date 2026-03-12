The RPG program `FR711P` is a prompt program for the Freight Out Reconciliation Report, designed to collect and validate user input before calling a print program. Below is an explanation of the process steps, external programs called, and tables used, based on the provided source code.

---

### Process Steps of the RPG Program (`FR711P`)

The program follows a structured process to prompt the user for input, validate it, and call a print program (`FR711PC`) to generate the report. Here's a breakdown of the steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up initial variables and environment.
   - **Actions**:
     - Receives an input parameter `p$fgrp` (file group: 'G' or 'Z') to determine which database files to override.
     - Sets the `fmtagn` flag to `*ON` to enable the main processing loop for the display file.
     - Loads the header text (`c$hdr1`) from the `hdr` array for the display file.
     - Captures the current date and time into `t#time` and converts it into a date format (`t#cymd`) for use in the report.
     - Defines a key list (`key1`) for database access, using fields `wkco` (company) and `c1loc` (location).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens the necessary database files with appropriate overrides.
   - **Actions**:
     - Checks the `p$fgrp` parameter to determine whether to apply overrides for 'G' (general) or 'Z' (alternate) files.
     - Executes `QCMDEXC` to apply overrides for `glcont` and `inloc` files:
       - For 'G': Overrides `glcont` to `gglcont` and `inloc` to `ginloc`.
       - For 'Z': Overrides `glcont` to `zglcont` and `inloc` to `zinloc`.
     - Opens the `glcont` and `inloc` files for reading.

3. **Process Display File (`srfmt` Subroutine)**:
   - **Purpose**: Displays the prompt screen and processes user input.
   - **Actions**:
     - Enters a loop (`fmtagn = *ON`) to repeatedly display the `FMT01` format of the display file (`fr711pd`) using `EXFMT`.
     - Clears error indicators (50-69) to reset validation states.
     - Checks for function key presses:
       - **F3 (Exit)**: Sets `fmtagn` to `*OFF` to exit the loop and terminate the program.
       - **F4 (Field Prompting)**: Calls the `prompt` subroutine to assist with field input (though the subroutine is empty in the provided code).
     - Validates input fields by calling the `f01edt` subroutine.
     - If validation passes (`*in50 = *OFF`), calls the `callpgm` subroutine to invoke the print program and exits the loop.
     - If validation fails, redisplays the screen with error messages.

4. **Validate Input Fields (`f01edt` Subroutine)**:
   - **Purpose**: Validates user-entered data on the prompt screen.
   - **Actions**:
     - **Company (`c1co`)**:
       - Chains to the `glcont` file to verify the company code.
       - If invalid or blank (`*in99 = *ON` or `c1co = *zeros`), sets error indicator `*in50` and `*in51` and displays error message `com(01)` ("Invalid Company Entered").
       - If valid, retrieves the company name (`gcname`) into `f$cnam`.
     - **Report Date Month (`c1mm`)**:
       - Checks if the month is between 1 and 12.
       - If invalid (`c1mm < 1` or `c1mm > 12`), sets `*in50` and `*in52` and displays error message `com(02)` ("Invalid Month Entered").
     - **Report Date Year (`c1yy`)**:
       - Checks if the year is non-zero.
       - If invalid (`c1yy = 0`), sets `*in50` and `*in53` and displays error message `com(03)` ("Invalid Year Entered").
     - **Location (`c1loc`)**:
       - If non-blank, chains to the `inloc` file using `key1` (company and location).
       - If invalid (`*in99 = *ON`), sets `*in50` and `*in54` and displays error message `com(04)` ("Invalid Loc Entered").
     - **Sort Option (`c1sort`)**:
       - Validates that the sort code is either 'C' or 'L'.
       - If invalid, sets `*in50` and `*in56` and displays error message `com(05)` ("Invalid Sort Entered, Valid Codes: C/L").
     - **Month Selection (`c1selmo`)** (added in revisions `jb01` and `jb02`):
       - Validates that the month selection is one of 'M', 'O', 'C', 'A', or blank.
       - If invalid, sets `*in50` and `*in56` and displays error message `com(06)` ("Invalid Selected Month, Valid Codes: M/O/C/A/' '").
     - If any validation fails, `*in50` is set to `*ON`, causing the screen to redisplay with error messages.

5. **Call Print Program (`callpgm` Subroutine)**:
   - **Purpose**: Prepares parameters and calls the print program.
   - **Actions**:
     - Sets the report date (`w$rdat`) by determining the century (`w$cen`):
       - If `c1yy >= 40`, sets century to 19 (e.g., 1940–1999).
       - Otherwise, sets century to 20 (e.g., 2000–2039).
     - Moves input fields (`c1co`, `w$rdat`, `c1loc`, `c1caid`, `c1sort`, `c1selmo`) to output parameters (`o$co`, `o$rdat`, `o$loc`, `o$caid`, `o$sort`, `o$selmo`).
     - Calls the external program `FR711PC`, passing the following parameters:
       - `o$co` (Company, 2 characters)
       - `o$rdat` (Report Date, 8 digits)
       - `o$loc` (Location, 3 characters)
       - `o$caid` (Carrier ID, 6 characters)
       - `o$sort` (Sort option, 1 character)
       - `o$selmo` (Month selection, 1 character)
       - `o$fgrp` (File group, 1 character, passed from input `p$fgrp`)
     - Sets the report date day to 31 (`w$day`), assuming the last day of the month for the report.

6. **Program Termination**:
   - Closes all open files (`close *all`).
   - Sets `*inlr = *ON` to indicate the program is ending.
   - Returns control to the caller.

---

### External Programs Called

- **FR711PC**: The print program called by the `callpgm` subroutine to generate the Freight Out Reconciliation Report based on the validated input parameters.
- **QCMDEXC**: A system program used to execute override commands for the `glcont` and `inloc` files in the `opntbl` subroutine.

---

### Tables Used

- **Display File**:
  - `fr711pd`: The display file used for the user interface, with format `FMT01` for prompting and displaying errors.

- **Database Files**:
  - `glcont`: Input-only file for company data, overridden to either `gglcont` or `zglcont` based on the `p$fgrp` parameter.
  - `inloc`: Input-only file for location data, overridden to either `ginloc` or `zinloc` based on the `p$fgrp` parameter.

---

### Additional Notes

- **Month Selection Options** (from revisions `jb01` and `jb02`):
  - `M`: Selected month only, including all (open and closed) records.
  - `O`: Selected month, open records only.
  - `C`: Selected month, closed records only.
  - `A`: All open records from previous and selected months.
  - Blank: Previous month open records and selected month all records.

- **Indicator Usage**: The program uses indicators (19, 21–69, 70–99) for screen control, error handling, and database operations, as detailed in the header comments.

- **Field Prefixes**:
  - `f$`: Display file fields (input/output).
  - `k$`: Key list fields.
  - `s$`: Selected subfile values.
  - `p$`: Input parameters.
  - `o$`: Output parameters.
  - `w$`: Work fields.

- **Compile Notes**: The program is compiled using PDM options `c1` or `c2`, indicating specific compilation configurations.

This program serves as a front-end interface for collecting and validating user input, ensuring data integrity before passing it to the report generation program (`FR711PC`).