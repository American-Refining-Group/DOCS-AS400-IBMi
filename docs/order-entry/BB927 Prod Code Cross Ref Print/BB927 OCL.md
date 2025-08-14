### Process Steps of the RPG .OCL Program

This .OCL file (BB927.ocl36.txt) appears to be an Operations Control Language (OCL) procedure for an IBM System/36 environment, which orchestrates the execution of a sort utility followed by an RPG (Report Program Generator) program to process and print a product code cross-reference file. The procedure handles file sorting and then invokes the main RPG program for reporting or printing. Below is a step-by-step breakdown of the process:

1. **Procedure Invocation and Environment Setup**:
   - The procedure begins with `// SCPROCP ,,,,,,,,?9?`, which invokes a stored procedure named SCPROCP. The commas represent parameter placeholders, and `?9?` is likely a variable or parameter value (possibly a library or file prefix).
   - `// GSY2K` sets up or switches to a specific library or environment (GSY2K), which may contain necessary programs or files for the job.

2. **Load Sort Utility**:
   - `// LOAD #GSORT` loads the sort program `#GSORT` into memory. This is a utility program used for sorting records in the input file.

3. **Define Input and Output Files for Sorting**:
   - `// FILE NAME-INPUT,LABEL-?9?BBPRXR,DISP-SHR`: Defines the input file named INPUT, which points to the physical file `?9?BBPRXR` (shared disposition, allowing concurrent access).
   - `// FILE NAME-OUTPUT,LABEL-?9?BB927S,RECORDS-999000,EXTEND-999000,RETAIN-J`: Defines the output file named OUTPUT, which points to `?9?BB927S`. It allocates space for up to 999,000 records, allows extension by another 999,000 records if needed, and retains the file after job completion (RETAIN-J).

4. **Execute the Sort Operation**:
   - `// RUN`: Initiates the execution of the loaded `#GSORT` program.
   - Sort specifications are provided:
     - `HSORTR    32A        3X  60   N`: Defines the sort header – likely specifying a record length of 60 bytes, with alphanumeric (A) and zoned decimal (X) fields, and no sequence checking (N).
     - `I C   1   1NECD`: Input control specification – processes records starting from position 1, with 1 record, using NEC D (possibly a character set or edit code).
     - `FNC   2   3                       COMPANY #`: Sort key on positions 2-3 (company number, numeric or character).
     - `FNC  28  33                       XREF SET`: Sort key on positions 28-33 (cross-reference set).
     - `FNC   4  23                       PRODUCT XREF`: Sort key on positions 4-23 (product cross-reference).
     - `FDC   1  60`: Full data copy – copies the entire 60-byte record to the output.
   - This step sorts the input file (`BBPRXR`) by the specified keys (company #, XREF set, product XREF) and writes the sorted records to the output file (`BB927S`).
   - `// END`: Terminates the sort execution.

5. **Load Main RPG Program**:
   - `// LOAD BB927`: Loads the RPG program named `BB927` into memory. This is likely the core program responsible for processing the sorted data and generating the print output (based on the procedure's title: "PRINT PRODUCT CODE CROSS REFERENCE FILE").

6. **Define Files for the RPG Program**:
   - `// FILE NAME-BBPRXR,LABEL-?9?BB927S,DISP-SHR`: Redefines the file `BBPRXR` to point to the sorted output from the previous step (`?9?BB927S`, shared disposition). This serves as the primary input for the RPG program.
   - `// FILE NAME-BICONT,LABEL-?9?BICONT,DISP-SHR`: Defines an additional input file `BICONT` (possibly a control or index file).
   - `// FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHR`: Defines `GSTABL` (likely a table file containing lookup data).
   - `// FILE NAME-GSPROD,LABEL-?9?GSPROD,DISP-SHR`: Defines `GSPROD` (likely a product master file or table).

7. **Execute the RPG Program**:
   - `// RUN`: Initiates the execution of the loaded `BB927` program.
   - The program processes the sorted input file (`BBPRXR` alias for `BB927S`) using the additional files (`BICONT`, `GSTABL`, `GSPROD`) to generate the product code cross-reference report or printout. (The exact RPG logic isn't shown in the .OCL, but it typically involves reading records, performing calculations/lookups, and outputting formatted reports.)

The overall flow sorts the cross-reference data for efficient processing, then runs the RPG program to print or report on it. The procedure assumes a batch environment and handles file retention for potential reuse.

### External Programs Called
- `#GSORT`: A sort utility program loaded and run to sort the input file based on specified keys.
- `BB927`: The main RPG program loaded and executed to process the sorted data and produce the output (e.g., printing the cross-reference).

### Tables Used
- `GSTABL`: Explicitly defined as a file, likely a table containing static or lookup data (e.g., codes, descriptions).
- `GSPROD`: Defined as a file, possibly a product table or master file used for lookups or joins during processing.
(Note: `BICONT` and `BBPRXR`/`BB927S` are more like data or control files rather than tables, but `GSTABL` and `GSPROD` fit the naming convention for table-like structures in this context.)