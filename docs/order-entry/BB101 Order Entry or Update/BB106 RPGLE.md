The `BB106.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It calculates freight charges for orders or invoices, updating relevant files with the results, and is also called by `BB495` (Bill of Lading entry) and `BB500` (Invoice entry). The program was converted from RPG II to RPGLE on 11/11/19 and includes multiple revisions to enhance freight calculation logic, handle multi-load orders and invoices, and support specific freight table selections. Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB106` program calculates freight charges based on whether it is processing an order (`FROEI='O'`) or an invoice (`FROEI='I'`), with additional modes for calculating freight for a single table (`CALC`) or returning the top five carrier choices (`TOP5`). The steps are as follows:

1. **Program Initialization**:
   - The program is defined with `fixnbr(*zoned:*inputpacked)` to handle numeric conversions (T4A).
   - Files defined include:
     - `bbortr`: Update-capable file for order transactions (512 bytes, indexed, key length 11).
     - `bbtranin`: Update-capable file for invoice transactions (512 bytes, indexed, key length 20).
     - `frof`: Input-only file for freight table data.
     - `frofh`: Input-only file for freight table history.
     - `frofc`: Update-capable file for freight charge records (JB11).
     - `frofcl1`: Input-only file for freight class data (indicator changed to 96, JB10).
   - Parameters received via `*ENTRY PLIST` (assumed, not shown in snippet):
     - `FROEI` (1 character): 'O' for order entry, 'I' for invoice entry.
     - `FRTBL` (assumed): Freight table selection for `CALC` mode.
     - `CALC/TOP5` (JB01): Mode to calculate freight for one table (`CALC`) or return top five carriers (`TOP5`).
   - Data structures and arrays:
     - `mxc`: Array for miscellaneous charges (10 elements).
     - Fields for company (`fco`, `mcoon`), order (`fordr`), customer (`fcust`), ship-to (`fship`), quantity (`mqty`), description (`mdes`), amount (`mamt`), general ledger number (`mgl#`), status (`msty`), railcar (`frcar`), etc.

2. **Freight Calculation Mode Determination**:
   - If `FROEI='O'`, reads order transaction records from `bbortr` to calculate freight charges.
   - If `FROEI='I'`, reads invoice transaction records from `bbtranin` (using railcar in the key) and allows override freight rates.
   - If mode is `CALC` (JB01), calculates freight for a single selected freight table and updates order/invoice records.
   - If mode is `TOP5` (JB01), calculates freight for the first five freight tables with a tender sequence (JB03) and returns carrier and freight total data without updating records.

3. **Freight Calculation**:
   - Calls the `MBBFRT` program to perform the actual freight calculation for each load.
   - For multi-load orders (`FROEI='O'`, JB07):
     - Corrects quantity fields to calculate freight for one load, then converts to total quantities for all loads.
   - For multi-load invoices (`FROEI='I'`, JB08, JB09):
     - Counts detail lines and, if freight is flat rate, multiplies the freight amount by the count.
   - Skips freight calculation for detail lines with override freight rates (JB04).
   - Does not update freight records if no freight table is found (JB06).

4. **Freight Record Updates**:
   - Updates `bbortr` (orders) or `bbtranin` (invoices) with calculated freight charges.
   - Adds or updates records in `frofc` for freight charges (JB11).
   - Writes the percentage of freight total to the percentage of total field (JB12).
   - For collect (`CYY`) service charges:
     - **Orders**: Deletes miscellaneous line for collect service charge (`cyyscod`, `edel` with `N67`).
     - **Invoices**: Adds (`eadd`, `cyysci`), updates (`e`, `cyysci`), or deletes (`edel`, `cyyscid`) miscellaneous line for collect service charge, including fields like company, order, customer, ship-to, quantity, description, amount, and flags.

5. **Output Operations**:
   - **Order Transaction (`bbortr`, `cyyscod`)**:
     - Deletes miscellaneous line for collect service charge if not applicable (`N67`).
   - **Invoice Transaction (`bbtranin`, `cyysci`)**:
     - Adds a new record with collect service charge details (e.g., `fco`, `fordr`, `fcust`, `fship`, `mqty`, `mdes`, `mamt`, `mgl#`, `msty`, `mxc`, `motx`, `mcoon`, `frcar`, fixed code '943').
     - Updates existing record with updated fields if applicable (`N67`).
     - Deletes record if no longer needed (`N67`, `cyyscid`).
   - **Freight Charge File (`frofc`)**:
     - Adds or updates freight charge records (JB11).

6. **Program Termination**:
   - Sets the last record indicator (`LR`) to exit the program.
   - Returns freight totals and carrier data (for `TOP5` mode) or updated order/invoice records to the calling program.

---

### Business Rules

The program enforces the following business rules:

1. **Freight Calculation Modes**:
   - For `FROEI='O'`, processes order transactions (`bbortr`) and calculates freight for orders (called by `BB101`).
   - For `FROEI='I'`, processes invoice transactions (`bbtranin`) with railcar in the key and allows override freight rates (called by `BB495`, `BB500`).
   - For `CALC` mode (JB01), calculates freight for a single freight table and updates records.
   - For `TOP5` mode (JB01), calculates freight for up to five freight tables with tender sequences (JB03) and returns carrier/freight data without updates.

2. **Multi-Load Handling**:
   - For orders, calculates freight for one load, then converts quantities to total for all loads (JB07).
   - For invoices, multiplies flat-rate freight by the count of detail lines (JB08, JB09).

3. **Override Freight Rates**:
   - Skips freight calculation for detail lines with override freight rates (JB04).

4. **Freight Table Validation**:
   - Does not update freight records if no freight table is found in `frof` or `frofh` (JB06).
   - Corrects duplicate records in `frofh` (JB01).

5. **Collect Service Charges**:
   - Adds, updates, or deletes miscellaneous lines for collect (`CYY`) service charges in `bbortr` (orders) or `bbtranin` (invoices).
   - Uses fixed code '943' for invoice service charge records.

6. **Customer Price Fields (JB13)**:
   - Defines pricing for freight scenarios:
     - **Freight Collect**: Customer price = 1.00, product price = 1.00, freight price = 0.00.
     - **Freight PPD&ADD**: Customer price = 1.00, product price = 1.00, freight price = 0.25 (freight is separate).
     - **Freight Prepaid with Rack Price**: Customer price = 1.00, product price = 1.00, freight price = 0.25 (rack price excludes freight).
     - **Freight Prepaid with Sales Agreement**: Customer price = 1.00, freight price varies.

7. **Error Handling**:
   - No error messages are defined in the provided snippet (COM,01-04 are placeholders), but the program relies on the calling program to handle errors.

---

### Tables (Files) Used

The program interacts with the following files:

1. **bbortr**: Update-capable file for order transactions (512 bytes, indexed, key length 11).
   - Fields: `fco` (company), `fordr` (order), `fcust` (customer), `fship` (ship-to), `mqty` (quantity), `mdes` (description), `mamt` (amount), `mgl#` (GL number), `msty` (status), `mxc` (miscellaneous charges), `motx` (tax flag), `mcoon` (company, repeated).
2. **bbtranin**: Update-capable file for invoice transactions (512 bytes, indexed, key length 20).
   - Fields: Same as `bbortr`, plus `frcar` (railcar) and fixed code '943' for service charges.
3. **frof**: Input-only file for freight table data.
   - Contains freight rate information.
4. **frofh**: Input-only file for freight table history.
   - Tracks historical freight data.
5. **frofc**: Update-capable file for freight charge records (JB11).
   - Stores calculated freight charges.
6. **frofcl1**: Input-only file for freight class data (indicator 96, JB10).
   - Provides freight classification details.

---

### External Programs Called

The program calls the following external program:

1. **MBBFRT**:
   - Called to perform the actual freight calculation for each load.
   - Returns freight totals used to update order/invoice records or populate top five carrier data.

---

### Summary

The `BB106` RPGLE program, called from `BB101.ocl36.txt` (and also `BB495` and `BB500`), calculates freight charges for orders (`FROEI='O'`) or invoices (`FROEI='I'`), supporting single-table calculations (`CALC`) or top-five carrier selections (`TOP5`). It processes multi-load orders and invoices, skips override freight rates, and updates `bbortr`, `bbtranin`, and `frofc` with freight data, including collect service charges. Business rules enforce mode-specific calculations, multi-load adjustments, and customer pricing logic. The program interacts with six files (`bbortr`, `bbtranin`, `frof`, `frofh`, `frofc`, `frofcl1`) and calls `MBBFRT` for freight calculations, ensuring accurate freight processing for order and invoice entry.