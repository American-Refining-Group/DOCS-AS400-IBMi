The provided document, `BB9534.rpg.txt`, is an RPG (Report Program Generator) program for an IBM System/36 or AS/400 environment. It is called by the OCL program `BB953.ocl36.txt` and serves as a filtering step in the rack price list generation process. The program processes records from the temporary file `BB9531` (created by `BB9531.rpg36.txt`) and writes a subset of records to another temporary file, `BB9532`, based on user-specified selection criteria for location, product class, container, and product. Below, I will explain the process steps, business rules, tables (files) used, and any external programs called.

---

### **Process Steps of the RPG Program**

The `BB9534` RPG program filters records from the input file `BB9531` based on user prompts (e.g., location, product class, container, product) and writes valid records to the output file `BB9532`. It uses a series of subroutines to validate each criterion, ensuring only records matching the user’s selections are included. Here’s a detailed breakdown of the process steps:

1. **Header and File Definitions**:
   - **H-Spec**: The header specification (`H/TITLE ... BB9534`) identifies the program as `BB9534` and describes its purpose: excluding data in the workfile based on sort criteria.
   - **File Definitions (F-Specs)**:
     - `BB9531`: Primary input file (169 bytes, disk-based), containing preprocessed rack price data from `BB9531.rpg36.txt`.
     - `BB9532`: Output file (169 bytes, disk-based, append mode `A`), where filtered records are written.
   - **Array Definitions (E-Specs)**:
     - `LOC`: Array of 5 elements (3 bytes each) for location codes.
     - `CLS`: Array of 10 elements (3 bytes each) for product class codes.
     - `CTR`: Array of 5 elements (3 bytes each) for container codes.
     - `PROD`: Array of 10 elements (4 bytes each) for product codes.
   - **Input Specifications (I-Specs)**:
     - `BB9531`: Defines fields like `RKCONO` (company, pos 1-2), `RKLOC` (location, pos 3-5), `RKPROD` (product code, pos 8-11), `RKCNTR` (container, pos 12-14), `RKDIV` (division, pos 163-164), `RKPRCL` (product class, pos 165-167), `RKRKRQ` (rack required flag, pos 168, added per `JB01`), `RKINAC` (inactive flag, pos 169, added per `JB01`), and `RECORD` (entire 169-byte record).
     - **User Data Structure (UDS)**: Defines fields from user prompts, including:
       - `KYDIV` (division, pos 103)
       - `KYLOSL` (location selection, pos 104-106, `'ALL'` or `'SEL'`)
       - `KYLOC1-5` (specific locations, pos 107-121, stored in `LOC` array)
       - `KYPCSL` (product class selection, pos 122-124, `'ALL'` or `'SEL'`)
       - `KYPC01-10` (specific product classes, pos 125-154, stored in `CLS` array)
       - `KYCTSL` (container selection, pos 155-157, `'ALL'` or `'SEL'`)
       - `KYCT01-05` (specific containers, pos 158-172, stored in `CTR` array)
       - `KYPDSL` (product selection, pos 173-175, `'ALL'` or `'SEL'`)
       - `KYPD01-10` (specific products, pos 176-215, stored in `PROD` array)
       - `KYDTSL` (date selection, pos 216-218)
       - `KYFRDT`, `KYTODT` (from/to dates, pos 219-230)
       - `KYJOBQ` (job queue, pos 231)
       - `KYCOPY` (copy count, pos 232-233)
       - `KYDVNO` (division number, pos 234-235)
       - `KYDAT8` (8-digit date, pos 236-243)
       - `Y2KCEN` (century, 19, pos 509-510)
       - `Y2KCMP` (comparison year, 80, pos 511-512)

2. **Main Processing Loop (Lines `0040-0054`)**:
   - **Indicator `01` (Input Cycle)**:
     - For each record read from `BB9531`, the program performs filtering based on user-specified criteria.
   - **Initialization**:
     - Turns off indicators `50` and `51` (used for validation control).
     - Sets indicator `52` on (initial assumption that the record is valid).
   - **Location Filtering**:
     - If `KYLOSL = 'SEL'` (specific locations selected):
       - Turns off indicator `52` (record is not valid until proven).
       - Executes `LOCSR` subroutine to check if `RKLOC` matches any `KYLOC1-5`.
   - **Product Class Filtering**:
     - If `KYPCSL = 'SEL'`:
       - Turns off indicator `52`.
       - Executes `PRCLSR` subroutine to check if `RKPRCL` matches any `KYPC01-10`.
   - **Product Filtering**:
     - If `KYPDSL = 'SEL'`:
       - Turns off indicator `52`.
       - Executes `PRODSR` subroutine to check if `RKPROD` matches any `KYPD01-10`.
   - **Container Filtering**:
     - If `KYCTSL = 'SEL'`:
       - Turns off indicator `52`.
       - Executes `CNTRSR` subroutine to check if `RKCNTR` matches any `KYCT01-05`.
   - **Validation Outcome**:
     - If indicator `51` is on (any validation failed), skips to `END` (record is not written).
     - If indicators `50` or `52` are on (record passed all checks or no specific selections were made), writes the record using `EXCPTGOOD`.

3. **LOCSR Subroutine (Lines `0015`)**:
   - **Purpose**: Validates if the record’s location (`RKLOC`) matches any user-specified locations (`KYLOC1-5`).
   - **Logic**:
     - Initializes indicator `50` to off (record not valid yet).
     - Loops through `LOC` array (1 to 5):
       - If `50` is on (match found), jumps to `ENDLOC`.
       - If `LOC,X = RKLOC`, sets `50` on (match found).
     - If no match (`N50`), sets `51` on (validation failed).
   - **Outcome**: Sets `50` if a match is found; otherwise, sets `51` to skip the record.

4. **PRCLSR Subroutine (Lines `0015`)**:
   - **Purpose**: Validates if the record’s product class (`RKPRCL`) matches any user-specified product classes (`KYPC01-10`).
   - **Logic**:
     - Initializes indicator `50` to off.
     - Loops through `CLS` array (1 to 10):
       - If `50` is on, jumps to `ENDPRC`.
       - If `CLS,X = RKPRCL`, sets `50` on.
     - If no match (`N50`), sets `51` on.
   - **Outcome**: Sets `50` if a match is found; otherwise, sets `51`.

5. **CNTRSR Subroutine (Lines `0015`)**:
   - **Purpose**: Validates if the record’s container (`RKCNTR`) matches any user-specified containers (`KYCT01-05`).
   - **Logic**:
     - Initializes indicator `50` to off.
     - Loops through `CTR` array (1 to 5):
       - If `50` is on, jumps to `ENDCTR`.
       - If `CTR,X = RKCNTR`, sets `50` on.
     - If no match (`N50`), sets `51` on.
   - **Outcome**: Sets `50` if a match is found; otherwise, sets `51`.

6. **PRODSR Subroutine (Lines `0015`)**:
   - **Purpose**: Validates if the record’s product code (`RKPROD`) matches any user-specified products (`KYPD01-10`).
   - **Logic**:
     - Initializes indicator `50` to off.
     - Loops through `PROD` array (1 to 10):
       - If `50` is on, jumps to `ENDPRD`.
       - If `PROD,X = RKPROD`, sets `50` on.
     - If no match (`N50`), sets `51` on.
   - **Outcome**: Sets `50` if a match is found; otherwise, sets `51`.

7. **Output Writing (Lines `0016`)**:
   - If indicators `50` or `52` are on, executes `EXCPTGOOD` to write the entire 169-byte `RECORD` to `BB9532`.
   - The output record is identical to the input record, preserving all fields (e.g., `RKCONO`, `RKLOC`, `RKPROD`, `RKCNTR`, `RKDIV`, `RKPRCL`, `RKRKRQ`, `RKINAC`).

8. **Termination**:
   - The program ends at the `END` tag after processing each input record, either writing it to `BB9532` or skipping it based on validation.

---

### **Business Rules**

The `BB9534` program enforces the following business rules for filtering rack price records:

1. **Location Filtering**:
   - If `KYLOSL = 'ALL'`, all locations are included (no filtering).
   - If `KYLOSL = 'SEL'`, the record’s `RKLOC` must match one of the user-specified locations (`KYLOC1-5`). If no match, the record is excluded.

2. **Product Class Filtering**:
   - If `KYPCSL = 'ALL'`, all product classes are included.
   - If `KYPCSL = 'SEL'`, the record’s `RKPRCL` must match one of the user-specified product classes (`KYPC01-10`). If no match, the record is excluded.

3. **Container Filtering**:
   - If `KYCTSL = 'ALL'`, all containers are included.
   - If `KYCTSL = 'SEL'`, the record’s `RKCNTR` must match one of the user-specified containers (`KYCT01-05`). If no match, the record is excluded.

4. **Product Filtering**:
   - If `KYPDSL = 'ALL'`, all products are included.
   - If `KYPDSL = 'SEL'`, the record’s `RKPROD` must match one of the user-specified products (`KYPD01-10`). If no match, the record is excluded.

5. **Record Inclusion**:
   - A record is written to `BB9532` only if it passes all applicable filters (i.e., `50` or `52` is on).
   - If any filter fails (indicator `51` on), the record is skipped.

6. **Default Behavior**:
   - If no specific selections are made (e.g., `KYLOSL`, `KYPCSL`, `KYCTSL`, `KYPDSL` are `'ALL'`), all records are written (`52` remains on).

7. **Data Preservation**:
   - The entire input record (169 bytes) is written unchanged to `BB9532` if it passes validation, preserving fields like company, location, product, container, division, product class, rack required, and inactive flags.

---

### **Tables (Files) Used**

The RPG program uses the following files (tables):
1. **BB9531**: Primary input file (169 bytes), containing preprocessed rack price data from `BB9531.rpg36.txt`. Fields include `RKCONO`, `RKLOC`, `RKPROD`, `RKCNTR`, `RKDIV`, `RKPRCL`, `RKRKRQ`, and `RKINAC`.
2. **BB9532**: Output file (169 bytes, append mode), where filtered records are written, preserving the same structure as `BB9531`.

---

### **External Programs Called**

No external programs are explicitly called within the `BB9534` RPG program. It operates independently, processing input from `BB9531` and writing to `BB9532`. The program is invoked by the OCL program `BB953.ocl36.txt`, and its output is used by subsequent steps (e.g., `#GSORT` and `BB953`).

---

### **Summary**

The `BB9534` RPG program filters records from `BB9531` to create `BB9532` by:
- Reading each record from `BB9531`.
- Validating location (`RKLOC`), product class (`RKPRCL`), container (`RKCNTR`), and product (`RKPROD`) against user-specified selections (`KYLOSL`, `KYPCSL`, `KYCTSL`, `KYPDSL`) using subroutines `LOCSR`, `PRCLSR`, `CNTRSR`, and `PRODSR`.
- Writing valid records (entire 169-byte `RECORD`) to `BB9532` if they pass all filters or if no specific selections are made.
- Skipping records that fail any filter.

**Tables (Files) Used**:
- `BB9531` (input)
- `BB9532` (output)

**External Programs Called**:
- None

This program ensures that only records matching the user’s selection criteria are included in `BB9532`, which is then sorted by `#GSORT` and used by `BB953` to generate the final rack price report.