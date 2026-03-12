The `BB1018.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. Its purpose is to look up product cross-reference information in the `BBPRXR` file based on a company, long product code, and cross-reference set (`XSET`). The program was converted from RPG II to RPGLE (JB03) and includes revisions to handle duplicate cross-reference codes for different customers (MG02) and reset cross-reference data when the long product code changes (JB01). Below is a detailed explanation of the process steps, business rules, tables (files) used, and any external programs called.

---

### Process Steps of the RPGLE Program

The `BB1018` program retrieves product and container codes from the `BBPRXR` file using a key constructed from company, long product code, and cross-reference set. It returns these values along with a cross-reference code to the calling program. The steps are as follows:

1. **Program Initialization**:
   - The program defines the `BBPRXR` file as input-only (60 bytes, indexed, key length 28, starting at position 1).
   - A parameter data structure `PARMS8` is defined to receive and return data:
     - `CO` (positions 1-2, 2 digits): Company number.
     - `LPROD` (positions 3-22, 20 characters): Long product code (product cross-reference).
     - `BXPROD` (positions 23-26, 4 characters): Product code.
     - `BXCNTR` (positions 27-29, 3 characters): Container code.
     - `CROSSX` (positions 30-49, 20 characters): Product cross-reference code.
     - `XSET` (positions 50-55, 6 characters): Cross-reference set (e.g., customer-specific set).
   - Input specifications for `BBPRXR`:
     - `BXDEL` (position 1, 1 character): Delete code ('D' for deleted).
     - `BXCONO` (positions 2-3, 2 digits): Company number.
     - `BXPXRC` (positions 4-23, 20 characters): Product cross-reference code.
     - `BXPROD` (positions 24-27, 4 characters): Product code.
     - `BXXSET` (positions 28-33, 6 characters): Cross-reference set (added for duplicate codes, MG02).
     - `BXCNTR` (positions 34-36, 3 characters): Container code.
     - `BXQTTY` (position 37, 1 character): Quantity type.
   - Work fields:
     - `HLD20` (20 characters): Temporary storage for `LPROD`.
     - `HLD26` (26 characters): Temporary storage for key components.
     - `XRFKEY` (28 characters): Key for chaining to `BBPRXR`.

2. **Parameter Input**:
   - Receives input parameters via `PARMS8` (`CO`, `LPROD`, `BXPROD`, `BXCNTR`, `CROSSX`, `XSET`) through the `*ENTRY PLIST` (line 0019).

3. **Key Construction**:
   - Moves `XSET` to `HLD26` (line 0024).
   - Moves `LPROD` to `HLD20` (line 0025).
   - Constructs the 28-character key `XRFKEY`:
     - Positions 1-2: `CO` (company number, line 0026).
     - Positions 3-22: `LPROD` (from `HLD20`, line 0026).
     - Positions 23-28: `XSET` (from `HLD26`, line 0027).

4. **Retrieve Cross-Reference Record**:
   - Chains to `BBPRXR` using `XRFKEY` (company/long product/set, line 0028).
   - **If a record is found (`N77`)**:
     - Checks if the record is deleted (`BXDEL = 'D'`):
       - If deleted, clears `BXPROD` and `BXCNTR` to blanks (lines 0030-0031).
       - If not deleted, moves `BXPXRC` to `CROSSX` (line 0032).
   - **If no record is found (`77`)**:
     - Clears `BXPROD` and `BXCNTR` to blanks (line JB).
     - If `CROSSX` is blank, moves `LPROD` to `CROSSX` (JB01, line JB); otherwise, clears `CROSSX` to blanks.

5. **Program Termination**:
   - Sets the last record indicator (`LR`, line 0036) to exit the program.
   - Returns the updated `PARMS8` data structure (`BXPROD`, `BXCNTR`, `CROSSX`) to the calling program.

---

### Business Rules

The program enforces the following business rules:

1. **Cross-Reference Lookup**:
   - The program retrieves product (`BXPROD`) and container (`BXCNTR`) codes from `BBPRXR` using a 28-character key (company, long product code, cross-reference set).
   - The key includes `XSET` to allow duplicate cross-reference codes for different customers (MG02, 08/15/16).

2. **Deleted Records**:
   - If a `BBPRXR` record is marked as deleted (`BXDEL = 'D'`), the program returns blank `BXPROD` and `BXCNTR` values, indicating no valid cross-reference.

3. **Cross-Reference Reset**:
   - If no record is found and `CROSSX` is blank, the program sets `CROSSX` to the input `LPROD` to reset the cross-reference (JB01, 10/15/09).
   - If `CROSSX` is not blank when no record is found, it is cleared to blanks.

4. **No Error Messaging**:
   - The program does not define or display error messages, relying on the calling program (e.g., `BB101`) to handle invalid or missing data.

5. **Key Expansion**:
   - The key was expanded to 28 positions (from 22) to include `XSET`, enabling customer-specific duplicate cross-reference codes (MG02, 06/30/16).

---

### Tables (Files) Used

The program interacts with the following file:

1. **BBPRXR**: Input-only file for product cross-reference data (60 bytes, indexed, key length 28, starting at position 1).
   - Fields:
     - `BXDEL` (position 1, 1 character): Delete code.
     - `BXCONO` (positions 2-3, 2 digits): Company number.
     - `BXPXRC` (positions 4-23, 20 characters): Product cross-reference code.
     - `BXPROD` (positions 24-27, 4 characters): Product code.
     - `BXXSET` (positions 28-33, 6 characters): Cross-reference set.
     - `BXCNTR` (positions 34-36, 3 characters): Container code.
     - `BXQTTY` (position 37, 1 character): Quantity type.

---

### External Programs Called

The `BB1018` RPGLE program does not call any external programs. All processing is handled internally through file operations.

---

### Summary

The `BB1018` RPGLE program, called from the `BB101.ocl36.txt` OCL program, retrieves product and container codes from the `BBPRXR` file using a 28-character key (company, long product code, cross-reference set). It supports duplicate cross-reference codes for different customers (MG02) and resets the cross-reference code when the long product code changes (JB01). The program chains to `BBPRXR`, processes deleted or non-existent records, and returns updated product, container, and cross-reference data. Business rules ensure valid lookups, handle deleted records, and reset cross-references as needed. The program interacts solely with the `BBPRXR` file and does not call external programs, relying on internal logic for all operations.