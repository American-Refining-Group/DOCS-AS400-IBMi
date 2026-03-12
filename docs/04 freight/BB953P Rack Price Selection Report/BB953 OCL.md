The provided document, `BB953.ocl36.txt`, is an **OCL (Operation Control Language)** program for an IBM System/36 or AS/400 environment. It orchestrates the process of printing the latest price of each product on the rack price file, likely generating a report based on user inputs validated by the `BB953P` program (previously analyzed). Below, I will explain the process steps, list the external programs called, and identify the tables (files) used.

---

### **Process Steps of the OCL Program**

The OCL program `BB953.ocl36.txt` manages the workflow for processing and printing a rack price report. It includes checks for concurrent execution, file management, data processing, sorting, and report generation. Here’s a detailed breakdown of the steps:

1. **Header and Metadata**:
   - The file begins with a comment block: `** PRINT THE LATEST PRICE OF EACH PRODUCT ON THE RACK PRICE FILE **`, indicating the program’s purpose.
   - `// GSY2K`: A comment or marker, likely indicating Year 2000 (Y2K) compliance or referencing a specific system/process. It has no direct operational impact.

2. **Concurrency Check**:
   - `// IF ACTIVE-BB953 ...`: Checks if the `BB953` program is already running on another workstation.
     - If true, it displays messages: `"RACK PRICE LIST IS BEING RUN BY ANOTHER WORKSTATION"`, `"PLEASE TRY AGAIN LATER..."`, and an empty line.
     - Prompts the user with `PAUSE 'REPLY 0 (JOB WILL CANCEL)'`, requiring a response of `0` to cancel the job.
     - Executes `RETURN`, terminating the job to prevent concurrent execution.

3. **Clear Output File**:
   - `CLRPFM FILE(?9?RKPRCE)`: Clears the physical file `?9?RKPRCE` (where `?9?` is a library or system identifier) to ensure no residual data remains in the output file before processing.

4. **Delete Temporary Files**:
   - `// GSDELETE BB953X,BB953P,BB9531,BB953S,BB9538,BB953U,BB953T,,?9?`: Deletes temporary files (`BB953X`, `BB953P`, `BB9531`, `BB953S`, `BB9538`, `BB953U`, `BB953T`) in the specified library (`?9?`) to clean up previous job data.
   - `// GSDELETE BB9532,,,,,,,,?9?`: Deletes the temporary file `BB9532` in the specified library.

5. **Create Temporary File `BB9532`**:
   - `// IF ?9?/G CRTPF FILE(QS36F/?9?BB9532) RCDLEN(169) SIZE(*NOMAX)`: If the library is `G`, creates a physical file `?9?BB9532` in the `QS36F` library with a record length of 169 bytes and unlimited size.
   - `// IFF ?9?/G CRTPF FILE(QS36FTEST/?9?BB9532) RCDLEN(169) SIZE(*NOMAX)`: If the library is not `G`, creates the file in the `QS36FTEST` library with the same specifications.
   - This step ensures a temporary file is available for processing rack price data.

6. **Build Temporary File (`BB9531` Program)**:
   - `// LOAD BB9531`: Loads the program `BB9531`.
   - **File Assignments**:
     - `BBPRCE` (label `?9?BBPRCE`, shared read mode `SHRRM`): Likely the source rack price file.
     - `GSTABL` (label `?9?GSTABL`, `SHRRM`): General table file, possibly for product class or container data.
     - `GSPROD` (label `?9?GSPROD`, `SHRRM`): Product master file.
     - `BB9531` (label `?9?BB9531`, 999,000 records, extendable): Output file for temporary data.
   - `// RUN`: Executes `BB9531`, which processes data from `BBPRCE`, `GSTABL`, and `GSPROD` to build the temporary file `BB9531`.

7. **Process Temporary File (`BB9534` Program)**:
   - `// LOAD BB9534`: Loads the program `BB9534`.
   - **File Assignments**:
     - `BB9531` (label `?9?BB9531`, shared access `SHR`): Input file from the previous step.
     - `BB9532` (label `?9?BB9532`, `SHR`): Output file for further processed data.
   - `// RUN`: Executes `BB9534`, which likely transforms or filters data from `BB9531` into `BB9532`.

8. **Sort Temporary File (`#GSORT` Program)**:
   - `// LOAD #GSORT`: Loads the system sort utility `#GSORT`.
   - **File Assignments**:
     - `INPUT` (label `?9?BB9532`, `SHRRM`): Input file from the previous step.
     - `OUTPUT` (label `?9?BB953S`, 999,000 records, extendable, retained temporarily `RETAIN-T`): Sorted output file.
   - `// RUN ... // END`: Executes the sort with the following specifications:
     - `HSORTR 20A 3X 169 N`: Defines a sort with a 20-byte ascending key, 3 additional keys, 169-byte records, and no sequence checking.
     - **Sort Keys** (ascending):
       - `FNC 1 2`: Positions 1-2 (likely company or division code).
       - `FNC 3 5`: Positions 3-5 (likely location code).
       - `FNC 163 164`: Positions 163-164 (possibly a sequence or control field).
       - `FNC 165 167`: Positions 165-167 (possibly another control field).
       - `FNC 8 11`: Positions 8-11 (possibly product code).
       - `FNC 12 14`: Positions 12-14 (possibly product class).
       - `FNC 15 17`: Positions 15-17 (possibly container code).
     - `FDC 1 169`: Includes all fields (positions 1-169) in the output.
     - `I C 1 1NECD`: Excludes records where position 1 is not equal to `'D'` (delete flag check).
   - This step sorts the `BB9532` file into `BB953S` for organized report generation.

9. **Print Rack Price Report (`BB953` Program)**:
   - `// LOAD BB953`: Loads the program `BB953`.
   - **File Assignments**:
     - `BB9531` (label `?9?BB953S`): Sorted input file from the previous step.
     - `BICONT` (label `?9?BICONT`, `SHRRM`): Company/control file.
     - `INLOC` (label `?9?INLOC`, `SHRRM`): Location file.
     - `GSTABL` (label `?9?GSTABL`, `SHRRM`): General table file.
     - `OUTFILE` (label `?9?RKPRCE`, `SHR`): Output file for the final report.
     - `GSCNTR1` (label `?9?GSCNTR1`, `SHR`): Container file.
   - `// RUN`: Executes `BB953`, which generates the rack price report using the sorted data and writes it to `?9?RKPRCE`.

10. **Cleanup Temporary Files**:
    - `// GSDELETE BB953X,BB953P,BB9531,BB953S,BB9538,BB953U,BB953T,,?9?`: Deletes the same temporary files as before to clean up.
    - `// GSDELETE BB9532,,,,,,,,?9?`: Deletes the `BB9532` file.

---

### **External Programs Called**

The OCL program invokes the following external programs:
1. **BB9531**: Builds the initial temporary file `BB9531` from `BBPRCE`, `GSTABL`, and `GSPROD`.
2. **BB9534**: Processes `BB9531` to create `BB9532`, likely performing additional transformations or filtering.
3. **#GSORT**: System sort utility to sort `BB9532` into `BB953S` based on specified keys.
4. **BB953**: Generates the final rack price report, writing to `?9?RKPRCE`.

---

### **Tables (Files) Used**

The OCL program references the following files (tables):
1. **?9?RKPRCE**: Output file for the final rack price report, cleared at the start and written to by `BB953`.
2. **BBPRCE** (label `?9?BBPRCE`): Source rack price file, used by `BB9531`.
3. **GSTABL** (label `?9?GSTABL`): General table file, used by `BB9531` and `BB953`.
4. **GSPROD** (label `?9?GSPROD`): Product master file, used by `BB9531`.
5. **BB9531** (label `?9?BB9531`): Temporary file created by `BB9531`, processed by `BB9534`, and used as input (`BB953S`) by `BB953`.
6. **BB9532** (label `?9?BB9532`): Temporary file created by `BB9534`, sorted by `#GSORT`.
7. **BB953S** (label `?9?BB953S`): Sorted temporary file, used as input by `BB953`.
8. **BICONT** (label `?9?BICONT`): Company/control file, used by `BB953`.
9. **INLOC** (label `?9?INLOC`): Location file, used by `BB953`.
10. **GSCNTR1** (label `?9?GSCNTR1`): Container file, used by `BB953`.

Additionally, temporary files `BB953X`, `BB953P`, `BB9538`, `BB953U`, and `BB953T` are referenced for deletion but not actively used in processing.

---

### **Summary**

The `BB953.ocl36.txt` OCL program orchestrates the generation of a rack price report by:
- Checking for concurrent execution of `BB953` to prevent conflicts.
- Clearing the output file `?9?RKPRCE` and deleting temporary files.
- Creating a temporary file `BB9532` based on the library context.
- Running `BB9531` to build an initial temporary file `BB9531` from source data.
- Running `BB9534` to process `BB9531` into `BB9532`.
- Sorting `BB9532` into `BB953S` using `#GSORT` with specific keys.
- Running `BB953` to generate the final report in `?9?RKPRCE`.
- Cleaning up temporary files after completion.

**External Programs Called**:
- `BB9531`
- `BB9534`
- `#GSORT`
- `BB953`

**Tables (Files) Used**:
- `?9?RKPRCE`
- `BBPRCE`
- `GSTABL`
- `GSPROD`
- `BB9531`
- `BB9532`
- `BB953S`
- `BICONT`
- `INLOC`
- `GSCNTR1`

This OCL program integrates multiple programs and files to produce a sorted, validated rack price report, leveraging the input parameters validated by `BB953P` (from the previous analysis).