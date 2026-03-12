The RPG program `FR710P` is a prompt program for the Freight Out Accrual Report, designed to collect user input (company number and date) and validate it before calling a print program. Below is a detailed explanation of the process steps, followed by the external programs called and the tables used.

### Process Steps of the FR710P Program

1. **Program Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Initializes the program environment and sets up initial values.
   - **Steps**:
     - Receives an input parameter `p$fgrp` (file group, either 'G' or 'Z') to determine which file override to apply.
     - Sets the `fmtagn` flag to `*on` to control the main processing loop for the display file.
     - Loads the header text for the display panel (`c$hdr1`) from the `hdr` array.
     - Captures the current system date and time using the `time` operation and populates the `t#cymd` field with the current date in century-year-month-day format (`CCYYMMDD`).
     - Clears the `w$rdat` work field for the report date.

2. **Open Database Tables (`opntbl` Subroutine)**:
   - **Purpose**: Opens the necessary database files with appropriate overrides based on the file group parameter.
   - **Steps**:
     - Checks the `p$fgrp` parameter to determine if it is 'G' or 'Z'.
     - If `p$fgrp` is 'G', applies the override command from the `ovg` array (e.g., `ovrdbf file(glcont) tofile(*libl/gglcont) share(*no)`).
     - If `p$fgrp` is 'Z', applies the override command from the `ovz` array (e.g., `ovrdbf file(glcont) tofile(*libl/zglcont) share(*no)`).
     - Executes the override command using the `QCMDEXC` system API, passing the override string and its length.
     - Opens the `glcont` file for use in the program.

3. **Process Display File (`srfmt` Subroutine)**:
   - **Purpose**: Manages the user interface by displaying a prompt screen (`FMT01`) and handling user input.
   - **Steps**:
     - Enters a loop controlled by the `fmtagn` flag, which continues until the user exits or input is valid.
     - Displays the `FMT01` format (likely a prompt screen for company number and date) using the `exfmt` operation.
     - Clears error indicators (`*in50` to `*in69`) to reset any previous error states.
     - Processes function keys pressed by the user:
       - **F3 (Exit)**: Sets `fmtagn` to `*off` to exit the loop and terminate the program.
       - **F4 (Prompt)**: Calls the `prompt` subroutine to assist with field input (though the subroutine is currently empty).
     - If no exit or prompt key is pressed, calls the `f01edt` subroutine to validate input fields.
     - If validation passes (`*in50` is `*off`), calls the `callpgm` subroutine to proceed with report generation and exits the loop.
     - If validation fails, the loop continues, redisplaying the screen with error messages.

4. **Field Validation (`f01edt` Subroutine)**:
   - **Purpose**: Validates user-entered data (company number, month, and year) on the prompt screen.
   - **Steps**:
     - **Company Number Validation**:
       - Clears the company name field (`f$cnam`).
       - Chains to the `glcont` file using the user-entered company number (`c1co`).
       - If the company number is invalid (`*in99` is `*on`) or zero, sets error indicator `*in50` and `*in51` to `*on` and displays an error message from `com(01)` ("Invalid Company Entered").
       - If valid, retrieves the company name (`gcname`) from the `glcont` file and stores it in `f$cnam`.
     - **Report Date Month Validation**:
       - If no prior errors (`*in50` is `*off`), checks if the entered month (`c1mm`) is between 1 and 12.
       - If invalid, sets `*in50` and `*in52` to `*on` and displays an error message from `com(02)` ("Invalid Month Entered").
     - **Report Date Year Validation**:
       - If no prior errors, checks if the entered year (`c1yy`) is non-zero.
       - If zero, sets `*in50` and `*in53` to `*on` and displays an error message from `com(03)` ("Invalid Year Entered").

5. **Call Print Program (`callpgm` Subroutine)**:
   - **Purpose**: Prepares parameters and calls the external print program to generate the report.
   - **Steps**:
     - Sets the day portion of the report date (`w$day`) to 31, assuming the report uses the last day of the month.
     - Determines the century for the report date (`w$cen`):
       - If the year (`c1yy`) is 40 or greater, sets the century to 19 (e.g., 1940).
       - Otherwise, sets the century to 20 (e.g., 2040).
     - Moves the constructed report date (`w$rdat`) and company number (`c1co`) to output parameters `o$rdat` and `o$co`.
     - Calls the external program `FR710PC`, passing:
       - `o$co` (company number, 2 characters).
       - `o$rdat` (report date, 8 digits in `CCYYMMDD` format).
       - `o$fgrp` (file group, 1 character, copied from `p$fgrp`).
     - The called program (`FR710PC`) presumably generates the Freight Out Accrual Report.

6. **Program Termination**:
   - Closes all open files using `close *all`.
   - Sets the last record indicator (`*inlr`) to `*on` to signal program completion.
   - Returns control to the caller.

### External Programs Called
- **FR710PC**: The print program called to generate the Freight Out Accrual Report, receiving parameters for company number, report date, and file group.
- **QCMDEXC**: A system API called to execute the file override command for the `glcont` file.

### Tables Used
- **glcont**: A database file used to validate the company number (`c1co`) and retrieve the company name (`gcname`). The file is opened with an override to either `gglcont` or `zglcont` based on the `p$fgrp` parameter ('G' or 'Z').
- **ffr710pd**: The display file (workstation file) used to present the prompt screen (`FMT01`) to the user for input and error messages.

### Additional Notes
- **File Overrides**: The program dynamically applies overrides to the `glcont` file to point to either `gglcont` or `zglcont` based on the `p$fgrp` parameter, allowing flexibility in accessing different data sets.
- **Error Handling**: Uses indicators (`*in50` to `*in53`) to flag validation errors and display corresponding messages from the `com` array.
- **Date Handling**: Constructs the report date assuming the last day of the month (day = 31) and determines the century based on the year input.
- **Prompt Subroutine**: The `prompt` subroutine is currently empty, suggesting it may be a placeholder for future field prompting functionality (e.g., a lookup or selection list triggered by F4).

This program serves as an interactive front-end to collect and validate user input before triggering the report generation process in `FR710PC`.