The provided document is an Operational Control Language (OCL) procedure for an IBM System/36 environment, specifically the `IN873.ocl36.txt` file. This OCL script is used to execute a program named `IN873` that handles conversion between pounds and net gallons, likely for inventory or production purposes. Below, I’ll explain the process steps of the OCL program, identify any external programs called, and list the tables used, based on the provided OCL script and general knowledge of OCL and RPG programming in the System/36 environment.

### Process Steps of the OCL Program

The OCL script defines a sequence of steps to set up and execute the `IN873` program. Here’s a breakdown of the process steps based on the OCL statements:

1. **Procedure Initiation (`// SCPROCP ,,,,,,,,?9?`)**
   - The `// SCPROCP` statement indicates the start of a System/36 procedure. The `?9?` is a substitution variable, typically representing a prefix or library identifier passed to the procedure at runtime. This allows the procedure to dynamically reference files or programs in a specific library or environment.
   - **Purpose**: Initializes the procedure and sets the context for file and program references using the provided prefix.

2. **Global System/36 Year 2000 Compliance (`// GSY2K`)**
   - The `// GSY2K` statement is a System/36 command or directive, likely related to ensuring Year 2000 compliance for date handling. This was common in legacy systems to address date-related issues around the year 2000.
   - **Purpose**: Ensures the procedure operates in a Y2K-compliant mode, adjusting system date handling if necessary.

3. **Load Program (`// LOAD IN873`)**
   - The `// LOAD IN873` statement loads the `IN873` program into memory for execution. This program, likely written in RPG II (given the System/36 context), performs the core logic for converting between pounds and net gallons.
   - **Purpose**: Prepares the `IN873` program for execution by loading it into the system’s memory.

4. **File Allocation for `INCONT` (`// FILE NAME-INCONT,LABEL-?9?INCONT,DISP-SHRRM`)**
   - This statement allocates a file named `INCONT` (likely an inventory control file) for use by the `IN873` program.
   - The `LABEL-?9?INCONT` indicates that the actual disk file name is prefixed with the substitution variable `?9?` (e.g., if `?9?` is `PROD`, the file is `PRODINCONT`).
   - `DISP-SHRRM` specifies that the file is opened in shared read mode (`SHRRM`), allowing multiple programs to read the file simultaneously without exclusive locking.
   - **Purpose**: Makes the `INCONT` file available to the `IN873` program for reading data, likely containing inventory-related data such as quantities in pounds or gallons.

5. **File Allocation for `GSPROD` (`// FILE NAME-GSPROD,LABEL-?9?GSPROD,DISP-SHR`)**
   - This statement allocates a file named `GSPROD` (likely a production or product master file) for use by the `IN873` program.
   - The `LABEL-?9?GSPROD` uses the same substitution variable to form the full file name (e.g., `PRODGSPROD`).
   - `DISP-SHR` indicates shared access, allowing other programs to access the file concurrently.
   - **Purpose**: Provides access to the `GSPROD` file, which likely contains product-specific data such as conversion factors or densities values needed for the pounds-to-gallons conversion.

6. **Execute Program (`// RUN`)**
   - The `// RUN` statement triggers the execution of the `IN873` program, which has been loaded and now has access to the `INCONT` and `GSPROD` files.
   - **Purpose**: Runs the `IN873` program, which processes the data from the allocated files to perform the conversion between pounds and net gallons.

7. **End of Procedure**
   - The OCL script implicitly ends after the `// RUN` statement, as no further statements (e.g., `// RETURN` or `// END`) are specified. This suggests that the procedure terminates after the `IN873` program completes, returning control to the calling process or ending the job.
   - **Purpose**: Completes the execution of the procedure.

### External Programs Called

Based on the OCL script, the only program explicitly called is:

- **IN873**: This is the main program loaded and executed by the OCL procedure. It is likely an RPG II program (given the System/36 context) responsible for performing the conversion between pounds and net gallons. No additional external programs are referenced in the OCL script.

### Tables Used

The OCL script does not explicitly reference any tables (e.g., lookup tables or database tables) within the procedure itself. However, it allocates two files that likely serve as the data sources for the `IN873` program:

- **INCONT**: An inventory control file, likely containing transactional or quantity data (e.g., amounts in pounds or gallons). The `?9?INCONT` label suggests it resides in a library specified by the `?9?` substitution variable.
- **GSPROD**: A production or product master file, likely containing reference data such as product densities or conversion factors (e.g., pounds per gallon for specific materials). The `?9?GSPROD` label indicates it is also library-specific.

These files are not explicitly described as tables in the OCL script, but in the context of System/36 and RPG II, they are likely physical files (flat files or database files) that the `IN873` program reads to perform the conversion. The RPG program may internally define these files as tables or data structures, but no specific table names are mentioned in the OCL.

### Additional Context and Assumptions

- **Purpose of IN873**: The comment at the top of the OCL script (`** CONVERT BETWEEN POUNDS AND NET GALLONS **`) indicates that the `IN873` program performs conversions between pounds and net gallons. This typically requires density or specific gravity data for the material being converted (e.g., water, oil, or chemicals), which is likely stored in the `GSPROD` file. The `INCONT` file may contain the quantities to be converted (e.g., input in pounds or gallons).
- **System/36 Environment**: The use of OCL and references to files with `DISP-SHR` and `DISP-SHRRM` confirm this is a System/36 environment, where RPG II programs commonly process inventory and production data using flat files or simple database structures.
- **Substitution Variable (`?9?`)**: The `?9?` variable allows the procedure to be reusable across different libraries or environments, making it flexible for different datasets or company divisions.
- **No Explicit Tables**: Unlike modern database systems, System/36 files are often flat files or simple database files rather than relational tables. The `IN873` program may internally use these files as tables, but the OCL script does not provide details about their structure.

### Notes on Limitations

- The OCL script provides only the setup and execution steps, not the internal logic of the `IN873` program. Without access to the RPG II source code for `IN873`, I cannot confirm the exact conversion logic or any additional tables it might use internally.
- No external sub-procedures or additional programs are called beyond `IN873`, and no explicit table definitions (e.g., RPG arrays or data structures) are mentioned in the OCL.
- If you have access to the `IN873` RPG program or additional documentation, I can provide a more detailed analysis of its logic or data structures.

### Summary

- **Process Steps**:
  1. Initiate the procedure with a dynamic library prefix (`?9?`).
  2. Set Y2K compliance mode.
  3. Load the `IN873` program.
  4. Allocate the `INCONTChildren's Hospital, Boston, MA 02115, USA.