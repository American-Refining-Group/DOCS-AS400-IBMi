The RPG program `BB080.rpgle` is designed to retrieve the current carrier fuel surcharge percentage within the Bradford Order Entry/Invoices system. It is called by other programs (likely including `BB945.rpgle`) to determine the applicable fuel surcharge percentage (`p$fspc`) based on input parameters such as company, carrier code, effective date, and time. The program queries the carrier fuel surcharge header (`bbcfsh`), detail (`bbcfsd`), and National Diesel Fuel Index (`bbndfi2`) files to find the appropriate surcharge percentage. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of `BB080.rpgle`

The program operates through a series of subroutines that retrieve and validate data to return the current fuel surcharge percentage. The main steps are:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Receives Input Parameters**:
     - `p$cono`: Company number.
     - `p$caid`: Carrier code.
     - `p$efdt`: Effective date (CCYYMMDD format).
     - `p$eftm`: Effective time (HHMM format).
     - `p$fspc`: Fuel surcharge percentage (output, initially cleared).
     - `p$fgrp`: File group ('Z' or 'G' for library selection).
     - `p$flag`: Return flag (output, set to '1' if a valid surcharge is found, otherwise unchanged).
   - Initializes date and time work fields:
     - Sets the current date (`t#cymd`) and time (`t#hrmn`) using the system timestamp (`t#time`).
     - If `p$efdt` is provided (non-zero), uses it as the effective date (`w$efdt`) and time (`w$eftm`); otherwise, defaults to the current date and time.
   - Defines key lists (`klcfsh`, `klndfi`, `klcfsd`) for file access.
   - Clears the output fuel surcharge percentage (`p$fspc`).

2. **Open Database Tables (`opntbl` Subroutine)**:
   - Applies file overrides based on `p$fgrp` ('Z' or 'G') to select the appropriate library for files.
   - Executes override commands using `QCMDEXC` for the files `bbcfsd`, `bbcfsh`, and `bbndfi2`.
   - Opens the input files: `bbcfsd`, `bbcfsh`, and `bbndfi2`.

3. **Retrieve Current Carrier Fuel Surcharge (`rtvcursurch` Subroutine)**:
   - Clears the output fuel surcharge percentage (`p$fspc`).
   - **Step 1: Retrieve Carrier Fuel Surcharge Header**:
     - Uses the key list `klcfsh` (company: `p$cono`, carrier code: `p$caid`) to position the cursor in the `bbcfsh` file.
     - Reads records in reverse order (`readpe`) to find the most recent non-deleted header record (`bjdel ≠ 'D'`) with an effective date (`bjefdt`) less than or equal to the input effective date (`w$efdt`).
     - Retrieves the region code (`bjregn`) from the matching header record.
   - **Step 2: Retrieve National Diesel Fuel Index Record**:
     - Uses the key list `klndfi` (company: `p$cono`, region: `bjregn`) to position the cursor in the `bbndfi2` file.
     - Reads records in reverse order (`readpe`) to find a non-deleted record (`bndel ≠ 'D'`) that matches one of the following conditions:
       - Effective date (`bnefdt`) is less than the input effective date (`w$efdt`).
       - Effective date matches (`bnefdt = w$efdt`) and effective time (`bneftm`) is less than or equal to the input effective time (`w$eftm`).
     - Retrieves the retail price (`bnrprc`) from the matching record.
   - **Step 3: Retrieve Fuel Surcharge Percentage from Detail Record**:
     - Uses the key list `klcfsd` (company: `p$cono`, carrier code: `p$caid`) to position the cursor in the `bbcfsd` file.
     - Reads records in reverse order (`readpe`) to find a non-deleted detail record (`bkdel ≠ 'D'`) with:
       - Effective date (`bkefdt`) less than or equal to the input effective date (`w$efdt`).
       - Retail price (`bnrprc`) within the range defined by the detail record’s “from price” (`bkfrpc`) and “to price” (`bktopc`).
     - Sets the output fuel surcharge percentage (`p$fspc`) to the detail record’s surcharge percentage (`bkfspc`).
     - Sets the return flag (`p$flag = '1'`) to indicate a successful match.
     - Exits the loop upon finding a valid surcharge percentage.
   - **Exit Conditions**:
     - If no matching header, Diesel Fuel Index, or detail record is found, the program exits without updating `p$fspc` or `p$flag`.

4. **Program End**:
   - Closes all files (`bbcfsd`, `bbcfsh`, `bbndfi2`).
   - Sets `*inlr = *on` and returns control to the calling program with the output parameters (`p$fspc`, `p$flag`).

---

### Business Rules

The program enforces the following business rules to ensure accurate retrieval of the carrier fuel surcharge percentage:
1. **Effective Date and Time Matching**:
   - The program retrieves the most recent non-deleted carrier fuel surcharge header record (`bbcfsh`) with an effective date (`bjefdt`) less than or equal to the input effective date (`w$efdt`).
   - For the National Diesel Fuel Index (`bbndfi2`), it selects a non-deleted record with either:
     - An effective date before the input effective date.
     - The same effective date and an effective time less than or equal to the input effective time.
   - The detail record (`bbcfsd`) must have an effective date less than or equal to the input effective date and a price range (`bkfrpc` to `bktopc`) that includes the retail price from the Diesel Fuel Index record.

2. **Deletion Handling**:
   - Records marked as deleted (`bjdel = 'D'` in `bbcfsh`, `bndel = 'D'` in `bbndfi2`, `bkdel = 'D'` in `bbcfsd`) are excluded from consideration.

3. **Price Range Validation**:
   - The retail price (`bnrprc`) from the Diesel Fuel Index record must fall within the “from price” (`bkfrpc`) and “to price” (`bktopc`) of the carrier fuel surcharge detail record to retrieve the surcharge percentage (`bkfspc`).

4. **Default Date and Time**:
   - If no effective date (`p$efdt`) is provided, the program uses the current system date and time (`t#cymd`, `t#hrmn`) to determine the effective date and time for the query.

5. **Output Handling**:
   - If a valid surcharge percentage is found, `p$fspc` is set to the value from the detail record (`bkfspc`), and `p$flag` is set to '1'.
   - If no valid record is found, `p$fspc` remains cleared (zero), and `p$flag` is not updated.

6. **Data Type for Fuel Surcharge**:
   - Per revision JB01 (06/17/2024), the fuel surcharge percentage (`p$fspc`, `bkfspc`) is stored as a packed decimal with 5 digits and 2 decimal places (previously 3 digits with 1 decimal place), allowing for more precise percentage values.

---

### Tables (Files) Used

The program accesses the following files, with overrides applied based on `p$fgrp` ('Z' or 'G'):
1. **bbcfsd**: Carrier fuel surcharge detail file (input only).
2. **bbcfsh**: Carrier fuel surcharge header file (input only).
3. **bbndfi2**: National Diesel Fuel Index file (input only).

---

### External Programs Called

The program calls the following external program:
1. **QCMDEXC**: Executes file override commands to apply the appropriate library overrides based on `p$fgrp`.

---

### Summary

The `BB080.rpgle` program is a utility program within the Bradford Order Entry/Invoices system, designed to retrieve the current carrier fuel surcharge percentage based on company, carrier code, effective date, and time. It queries the carrier fuel surcharge header (`bbcfsh`) to get the region, the National Diesel Fuel Index (`bbndfi2`) to get the retail price, and the carrier fuel surcharge detail (`bbcfsd`) to find the surcharge percentage within the applicable price range. The program enforces strict validation rules to ensure only non-deleted records with matching effective dates and price ranges are considered. The revision (JB01, 06/17/2024) enhances the precision of the fuel surcharge percentage field. The program is lightweight, with no display file or user interaction, and returns the surcharge percentage and a success flag to the calling program (e.g., `BB945.rpgle`).