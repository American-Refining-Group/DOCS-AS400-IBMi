The provided document, `BI945P.rpgle.txt`, is an RPGLE (RPG IV) program used in conjunction with the previously discussed OCL program (`BI945P.ocl36.txt`) on an IBM midrange system (e.g., AS/400 or iSeries). This program processes customer sales agreement data, specifically handling data from the `PRSABLX` file and populating fields in the Local Data Area (LDA) for further processing. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BI945P` RPGLE program performs data processing tasks related to customer sales agreements. Here’s a detailed breakdown of the process steps based on the provided source code:

1. **Program Initialization**:
   - **Header Specifications**:
     - `h fixnbr(*zoned:*inputpacked)`: Ensures numeric fields in zoned decimal or packed decimal formats are handled correctly during input.
     - `h dftname(bi945p)`: Sets the default program name to `BI945P`.
   - **File Definition**:
     - `fprsablx ip f 88 24aidisk keyloc(2)`: Declares the `PRSABLX` file as the primary input file (`ip`), with a record length of 88 bytes, using an access path (`aidisk`) and a key starting at position 2 (`keyloc(2)`).
   - **Data Area Definition**:
     - Defines a User Data Structure (UDS) with numerous fields (`kystyp`, `kydiv`, `kylosl`, `kyloc1`–`kyloc5`, `kypcsl`, `kypc01`–`kypc10`, `kycssl`, `kycs01`–`kycs20`, `kypdsl`, `kypd01`–`kypd30`, etc.) to store data in the LDA. These fields represent various keys and parameters for filtering or processing sales agreement data, such as customer numbers, product codes, locations, and dates.
   - **Input Record Definition**:
     - Defines the `PRSABLX` file’s record layout with fields like `psco` (company), `psloc` (location), `pscust` (customer number), `psship` (ship-to), `psprod` (product), `psstd8` (start date), `psend8` (end date), `pscgpr` (ceiling price), `psngpr` (negotiated price), etc.

2. **One-Time Setup (`onetim` Subroutine)**:
   - Executed only if indicator `*in09` is off:
     - `c if (*in09 = *off)`: Checks if the one-time setup has not yet been performed.
     - `c exsr onetim`: Calls the `onetim` subroutine.
     - `c eval *in09 = *on`: Sets indicator `*in09` to on to prevent re-execution.
   - **Subroutine `onetim`**:
     - Captures the current system date and time:
       - `c time timdat 12 0`: Stores the system timestamp in `timdat`.
       - `c movel timdat systim 6 0`: Extracts the time portion.
       - `c move timdat sysdat 6 0`: Extracts the date portion.
     - Initializes LDA fields with default values:
       - `kylosl = 'ALL'`: Sets location selection to all.
       - `kysmsl = 'ALL'`: Sets sales method selection to all.
       - `kypdsl = 'SEL'`: Sets product selection to selective.
       - `kypcsl = 'ALL'`: Sets product category selection to all.
       - `kycssl = 'SEL'`: Sets customer selection to selective.
       - `kyumsl = 'ALL'`: Sets unit of measure selection to all.
       - `kydiv = 'L'`: Sets division to 'L' (likely a specific division code).
       - `kydvno = '50'`: Sets division number to '50'.
       - `kyjobq = 'N'`: Indicates no job queue submission.
       - `kysprd = 'Y'`: Enables special pricing.
       - `kyadda = 'Y'`: Enables additional processing.
       - `kycopy = 1`: Sets copy number to 1.
       - `kystyp = 'INCLUDE'`: Sets selection type to include.
       - `kydlch = *zeros`: Initializes a change field to zero.
     - Purpose: Sets up default parameters in the LDA for processing sales agreement data.

3. **Main Processing Logic**:
   - **Conditional Execution**:
     - `c if (*in01 = *on)`: Checks if indicator `*in01` is on (likely set when a valid record is read from `PRSABLX`).
     - `c exsr s1`: Calls the `s1` subroutine to process the record.
     - `c goto end`: Skips to the end of the program if `*in01` is on, indicating processing is complete for the current cycle.
   - **Subroutine `s1`**:
     - **Date Processing**:
       - `c z-add *zero kydat8`: Initializes the LDA date field `kydat8` to zero.
       - `c udate mult 10000.01 kydat6 6 0`: Converts the system date (`udate`) to a 6-digit format (YYMMDD) by multiplying by 10000.01.
       - `c z-add kydat6 kydat8`: Stores the converted date in `kydat8`.
       - **Y2K Adjustment**:
         - `c if uyear >= y2kcmp`: Checks if the year (`uyear`) is greater than or equal to a comparison year (`y2kcmp`, position 511–512, value 80).
         - `c z-add y2kcen ucn 2 0`: Sets the century (`ucn`) to `y2kcen` (position 509–510, value 19) if true.
         - `c else`: If the year is less than `y2kcmp`, adds 1 to `y2kcen` (e.g., 19 + 1 = 20) to handle post-2000 dates.
         - `c movel ucn kydat8`: Updates `kydat8` with the adjusted century.
     - **Customer Number Assignment**:
       - `c z-add pscust kycs01`: Copies the customer number (`pscust`) from the `PRSABLX` record to the LDA field `kycs01` (first customer field).
     - **Product Assignment**:
       - The program checks up to 30 LDA fields (`kypd01` to `kypd30`) to store unique product codes (`psprod`) from the `PRSABLX` record:
         - For each field (`kypd01` to `kypd30`), it checks if the field is blank and if the product code (`psprod`) is different from all previously assigned product codes.
         - Example for `kypd01`:
           - `c if kypd01 = *blanks`: If `kypd01` is blank, `c movel psprod kypd01` assigns `psprod` to `kypd01` and jumps to the `next` tag.
         - For subsequent fields (e.g., `kypd02`):
           - Checks if `kypd02` is blank and `psprod` is not equal to `kypd01`.
           - If true, assigns `psprod` to `kypd02` and jumps to `next`.
         - This continues up to `kypd30`, ensuring each product code is unique across the fields.
       - If a product code is already present in an earlier field, it skips to the next record (`goto next`).
       - Purpose: Builds a list of up to 30 unique product codes in the LDA for further processing.

4. **Program Termination**:
   - `c end tag`: Marks the end of the main program logic.
   - The program ends after processing all records or when the input file (`PRSABLX`) is exhausted.

---

### Business Rules

The `BI945P` RPGLE program enforces the following business rules:
1. **One-Time Initialization**:
   - The `onetim` subroutine runs only once per program execution (controlled by `*in09`) to set default values in the LDA for filtering and processing sales agreement data.
   - Defaults include selecting all locations, sales methods, and product categories, with selective customer and product filtering, and specific division and processing flags.
2. **Date Handling**:
   - Adjusts dates for Y2K compliance by determining the century (19 or 20) based on the year comparison (`uyear >= y2kcmp`).
   - Stores the adjusted date in `kydat8` for use in subsequent processing.
3. **Customer Data**:
   - Copies the customer number (`pscust`) from each `PRSABLX` record to the first customer field (`kycs01`) in the LDA.
4. **Unique Product Assignment**:
   - Ensures that up to 30 unique product codes (`psprod`) are stored in LDA fields (`kypd01` to `kypd30`).
   - Checks for duplicates by comparing each new `psprod` against previously assigned fields to prevent redundancy.
   - If a product code is already assigned, the program skips to the next record without adding it again.
5. **Conditional Processing**:
   - Only processes records when `*in01` is on, indicating a valid input record.
   - Skips to the end of the program after processing each valid record to avoid unnecessary iterations.

---

### Tables Used

The program uses the following table (file):
1. **PRSABLX**:
   - The primary input file, defined as a disk file with an access path (`aidisk`) and a key starting at position 2.
   - Contains fields such as:
     - `psco` (company, positions 1–2, numeric)
     - `psloc` (location, positions 3–5, character)
     - `pscust` (customer number, positions 6–11, numeric)
     - `psship`/`psshp#` (ship-to, positions 12–14, character/numeric)
     - `psprod` (product code, positions 15–18, character)
     - `psum` (unit of measure, positions 19–21, character)
     - `pscntr` (contract, positions 22–24, character)
     - `psstd8` (start date, positions 25–32, numeric)
     - `psend8` (end date, positions 37–44, numeric)
     - `pscgpr`, `psngpr`, `psnupr` (ceiling, negotiated, and net unit prices, positions 49–63, packed decimal)
     - Various other date and time fields for start and end dates/times.
   - This file likely contains customer sales agreement data, which the program processes to extract customer and product information.

Additionally, the OCL program (`BI945P.ocl36.txt`) references:
- **GPRSABLO**: The table where data is uploaded before `BI945P` runs, likely the source for `PRSABLX`.
- **GPRSABLN**: An obsolete spreadsheet used for data transfer in earlier processes.

The RPGLE program itself does not directly reference `GPRSABLO` or `GPRSABLN`, but it processes data from `PRSABLX`, which is likely derived from `GPRSABLO`.

---

### External Programs Called

The RPGLE program `BI945P` does not explicitly call any external programs within the provided code. However, the associated OCL program (`BI945P.ocl36.txt`) references:
1. **BI9445**:
   - Called conditionally based on the OCL logic (`IF ?L'328,1'?/Y`), either directly or via a job queue.
   - Likely processes the LDA fields populated by `BI945P` (e.g., customer and product codes).
2. **BI942F**:
   - Referenced in the OCL comments as an obsolete procedure previously used to create data in `GPRSABLO`. Not called in the current process.

Within the RPGLE program, no external program calls are made (e.g., no `CALL` or `CALLP` operations). The program focuses on processing `PRSABLX` records and updating the LDA.

---

### Summary

The `BI945P` RPGLE program processes customer sales agreement data from the `PRSABLX` file, populating the LDA with customer numbers and up to 30 unique product codes. It includes one-time initialization to set default parameters and handles Y2K date adjustments. The business rules ensure unique product assignments and selective data processing based on input record validity.

**Process Steps**:
1. Initialize the program and define the `PRSABLX` file and LDA fields.
2. Perform one-time setup (`onetim`) to initialize LDA fields with defaults.
3. Process `PRSABLX` records in the `s1` subroutine, handling date adjustments, customer number assignment, and unique product code storage.
4. Terminate after processing each valid record or when the input file is exhausted.

**Business Rules**:
1. Run one-time setup only once.
2. Adjust dates for Y2K compliance.
3. Store the customer number in `kycs01`.
4. Store up to 30 unique product codes in `kypd01` to `kypd30`.
5. Process records only when `*in01` is on.

**Tables Used**:
- `PRSABLX` (primary input file).
- Indirectly related (via OCL): `GPRSABLO` (data source), `GPRSABLN` (obsolete).

**External Programs Called**:
- None in the RPGLE program itself.
- Via OCL: `BI9445` (active), `BI942F` (obsolete).

If you have additional details about `BI9445` or the structure of `PRSABLX` or `GPRSABLO`, I can provide further insights into their integration with `BI945P`.