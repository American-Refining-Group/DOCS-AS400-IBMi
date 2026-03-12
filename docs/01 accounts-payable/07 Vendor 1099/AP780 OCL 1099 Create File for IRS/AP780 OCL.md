The provided document is an Operations Control Language (OCL) program, specifically `AP780.ocl36.txt`, designed to create an IRS 1099 file for Accounts Payable (A/P) processing, primarily for 1099-MISC forms. Below, I’ll break down the process steps, list the external programs called, and identify the tables used.

### Process Steps of the AP780 OCL Program

The OCL program automates the creation of an IRS 1099 file by performing a series of file manipulations, data validations, and processing steps. Here’s a detailed explanation of the steps:

1. **Initialization and Setup**:
   - The program begins with comments explaining its purpose: to create an A/P 1099 file for IRS electronic processing, specifically for 1099-MISC forms. Other 1099 types would require program modifications.
   - It prompts for two parameters:
     - `?10?`: The four-digit year for the 1099 file (e.g., 2012).
     - `?13?`: The 1099 file name (e.g., `APVN2012`).
   - A programmer note indicates that the file specified in `?13?` (e.g., `APVN2012`) is created in a prior process (`AP300`) and contains vendor data before monthly and yearly totals are cleared.
   - The program sets the `SWITCH` to `00000000` and clears local variables (`LOCAL BLANK-*ALL`).
   - It initializes a data structure with hardcoded company information for American Refining Group Inc., including name, address, tax ID, year (`?10?`), contact name, and phone number.

2. **File Name Assignment**:
   - If `?9?` (likely a test environment flag) equals `G`, the program sets `P13` to `APVN?10?` (e.g., `APVN2012`).
   - Otherwise, it sets `P13` to `?9?VN?10?` (e.g., `TESTVN2012` for a test environment).
   - If the file specified by `?13?` does not exist (`DATAF1-?13?`), the program pauses with an error message (`?10? NOT FOUND ( ?13? )`) and cancels execution.

3. **User Update Option**:
   - The program pauses to ask if a Data File Utility (DFU) update should be moved to a separate menu option.
   - If `?9?` equals `G`, it updates the `?13?` file using format `APFMT15`.
   - If in a test environment (`?9?` not equal to `G`), it pauses with a message indicating that DFU does not work in the test environment.

4. **Run AP780P Program**:
   - Loads and runs the `AP780P` program.
   - Opens a file `GSTABL` (or `?9?GSTABL` based on the environment) with shared access (`DISP-SHR`).
   - If `SWITCH1-1` is set, the program jumps to the `END` tag, terminating execution.

5. **File Deletion**:
   - Displays a message indicating that the program is creating the A/P 1099 file for IRS electronic processing.
   - Deletes several files if they exist, depending on the environment (`?9?/G`):
     - `AP1099I`, `?9?AP1099I`, `AP1099`, `?9?AP1099`, `IRSTAX`, `?9?IRSTAX`, and `?9?AP781`.

6. **Sorting Input File (GSORT)**:
   - Loads the `#GSORT` program to sort the input file specified by `?13?` (e.g., `APVN2012`).
   - Input file: `?13?` with shared access (`DISP-SHR`).
   - Output file: `?9?AP780S` with 999,000 records, extendable by 999,000, and retained as a job file (`RETAIN-J`).
   - Sorting parameters:
     - `HSORTA 22A 3X N`: Sorts ascending on a 22-character field, with additional logic.
     - Conditions (`O C`, `I*`, `I C`, etc.) define sorting and filtering logic based on fields like 1099 type, vendor, and company.
     - Fields extracted include:
       - 1099 type (positions 264–264).
       - Vendor (positions 4–8).
       - Company (positions 2–3).

7. **Processing with AP781**:
   - Loads the `AP781` program.
   - Input files:
     - `APVEND` (label `?13?`, shared access).
     - `AP780S` (label `?9?AP780S`).
   - Output file:
     - `AP781` (label `?9?AP781`, 1,000 records, extendable by 500, temporary retention `RETAIN-T`).
   - Runs the program to process the sorted data.

8. **Creating the Final 1099 File (AP780)**:
   - Loads the `AP780` program.
   - Input file: `AP781` (label `?9?AP781`, shared access).
   - Output file:
     - If `?9?/G`, creates `AP1099` (1,000 records, extendable by 500).
     - Otherwise, creates `?9?AP1099` (e.g., `TESTAP1099`).
   - Runs the program to generate the final 1099 file.

9. **Building Index**:
   - If `?9?/G` and `AP1099I` exists, builds an index (`AP1099I`) for `AP1099` with keys at positions 7 (4 bytes) and 12 (9 bytes), allowing duplicate keys.
   - If not `?9?/G` and `?9?AP1099I` exists, builds an index (`?9?AP1099I`) for `?9?AP1099` with the same key structure.

10. **Copying Data**:
    - Deletes the existing file `AP10?10?` (e.g., `AP102012`) or `?9?AP1?10?` (e.g., `TESTAP12012`) if it exists.
    - Copies data:
      - If `?9?/G`, from `AP1099` to `AP10?10?`.
      - Otherwise, from `?9?AP1099` to `?9?AP1?10?`.

11. **Cleanup and Termination**:
    - Jumps to the `END` tag.
    - Resets the `SWITCH` to `00000000` and clears local variables (`LOCAL BLANK-*ALL`).
    - Deletes the temporary file `?9?AP781` if it exists.

### External Programs Called

The program invokes the following external programs:
1. **AP780P**: Likely a preprocessing program that sets up or validates data before the main 1099 file creation.
2. **#GSORT**: A sorting utility to sort the input file (`?13?`) and produce a sorted output (`?9?AP780S`).
3. **AP781**: Processes the sorted data to prepare it for the final 1099 file.
4. **AP780**: Generates the final 1099 file (`AP1099` or `?9?AP1099`).

### Tables Used

The program references the following files (tables):
1. **GSTABL** (or `?9?GSTABL`): A table file opened with shared access, likely containing configuration or reference data.
2. **APVEND** (label `?13?`, e.g., `APVN2012`): The input vendor file containing A/P data before totals are cleared.
3. **AP780S** (label `?9?AP780S`): A temporary sorted output file from the `#GSORT` program.
4. **AP781** (label `?9?AP781`): A temporary file used by the `AP781` program for intermediate processing.
5. **AP1099** (or `?9?AP1099`): The final 1099 output file.
6. **AP1099I** (or `?9?AP1099I`): An index file for `AP1099` or `?9?AP1099`.
7. **IRSTAX** (or `?9?IRSTAX`): A file that may store tax-related data, deleted if it exists.
8. **AP10?10?** (or `?9?AP1?10?`, e.g., `AP102012` or `TESTAP12012`): The final copied 1099 data file.

### Summary

The `AP780` OCL program orchestrates the creation of an IRS 1099 file by:
- Validating input parameters and files.
- Allowing user updates via DFU.
- Sorting vendor data, processing it through intermediate steps, and generating the final 1099 file.
- Managing file deletions and indexing for efficient data access.
- Supporting both production (`?9?/G`) and test environments.

The program relies on external programs (`AP780P`, `#GSORT`, `AP781`, `AP780`) and multiple files for data storage and processing, ensuring the output is suitable for IRS electronic submission.