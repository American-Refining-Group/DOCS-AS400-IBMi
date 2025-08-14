The **BB802P** RPGLE program is a component of the Customer Orders and Invoicing system, designed to populate a work file (`BB802W`) with sales history move detail data from the `BB802PQF` file, which is a logical file joining `SA5MOVD` (sales history move details) and `SA5SHP` (sales history headers). It is called by the **BB802C** CLP program in `SAD` mode when the product move flag (`P$PMOV = 'Y'`) is set, as part of the Customer Order and Invoice Inquiry system. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the BB802P RPG Program**

The program reads records from the `BB802PQF` file, processes them based on a level break (L1), and updates or adds corresponding records to the `BB802W` work file. The steps focus on data transfer with no user interaction. Here’s a breakdown of the key process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and initializes variables.
   - **Actions**:
     - Receives the input parameter: `p$fgrp` (1 character, file group: 'G' or 'Z').
     - Defines a key list (`klwrk`) for accessing the work file `BB802W`, using fields: `saco` (company), `sacust` (customer), `saord` (order number), and `sainvn` (invoice number).

2. **Main Processing Loop**:
   - **Purpose**: Reads `BB802PQF` records and processes them based on a level break.
   - **Actions**:
     - Reads `BB802PQF` records sequentially, with level break indicators (`L1`) set on fields `saco`, `sacust`, `saord`, and `sainvn`.
     - When the level break indicator `*inl1` is on, executes the `updwrkf` subroutine to process the record.

3. **Update Work File (`updwrkf` Subroutine)**:
   - **Purpose**: Adds or updates records in the `BB802W` work file based on `BB802PQF` data.
   - **Actions**:
     - Uses the `klwrk` key list (`saco`, `sacust`, `saord`, `sainvn`) to check if a matching record exists in `BB802W` using `CHAIN` (sets `*in99` if no match).
     - If no matching record exists (`*in99 = *on`):
       - Clears the `BB802W` record format (`bb802wpf`).
       - Maps fields from `BB802PQF` to `BB802W`:
         - `w1co` ← `saco` (company)
         - `w1cust` ← `sacust` (customer)
         - `w1ord#` ← `saord` (order number)
         - `w1inv#` ← `sainvn` (invoice number)
         - `w1ship` ← `saship` (ship-to code)
         - `w1pord` ← `shpord` (purchase order)
         - `w1slmn` ← `shslmn` (salesman)
         - `w1crid` ← `shrtg1` (carrier ID)
         - `w1carn` ← `shcarn` (carrier name)
         - `w1sdat` ← `shsmd8` (ship date)
         - `w1frcd` ← `shfrcd` (freight code)
         - `w1sfrt` ← `shsfrt` (ship freight)
         - `w1cafr` ← `shcafr` (carrier freight)
         - `w1cacd` ← `shcacd` (carrier code)
         - `w1trk#` ← `shtrk#` (tracking number)
         - `w1loc` ← `saloc` (location)
         - `w1prod` ← `saprod` (product)
         - `w1cntr` ← `sacntr` (container)
         - `w1qty` ← `sactqt` (quantity, added by jk01)
         - `w1um` ← `saum` (unit of measure, added by jk01)
         - `w1orpr` ← `saorpr` (order process, added by jk01)
         - `w1oori` ← `'I'` (invoice origin indicator)
         - Additional fields (per jk02):
           - `w1tkby` ← `shtkby` (taken by)
           - `w1rush` ← `sarush` (rush order flag)
           - `w1gpby` ← `shgpby` (group by)
           - `w1racd` ← `shracd` (reason code)
           - `w1mulo` ← `shmulo` (multi-order flag)
           - `w1mlcd` ← `shmlcd` (multi-code)
           - `w1tolo` ← `shtolo` (total order)
           - `w1loda` ← `shloda` (load date)
           - `w1lovo` ← `shlovo` (load volume)
           - `w1indt` ← `shimdy` (invoice date)
           - `w1inva` ← `shitot` (invoice amount)
           - `w1fccd` ← `shfccd` (freight charge code)
       - Writes the new record to `BB802W` (`write bb802wpf`).
     - If a matching record exists (`*in99 = *off`):
       - Updates the existing record with `w1oori = 'B'` (indicating both order and invoice data).
       - Updates the `BB802W` record (`update bb802wpf`).

4. **Program Termination**:
   - **Purpose**: Completes processing and exits.
   - **Actions**:
     - Ends the program after processing all `BB802PQF` records, closing files implicitly (as `BB802PQF` and `BB802W` are defined with `ip` and `uf` respectively).

---

### **Business Rules**

1. **Work File Population**:
   - The program adds or updates records in `BB802W` from `BB802PQF` (joining `SA5MOVD` and `SA5SHP`) for sales history move details, using the invoice number (`sainvn`) as a key component.
   - If no matching record exists in `BB802W`, a new record is created with `w1oori = 'I'` (indicating invoice origin).
   - If a matching record exists, it is updated with `w1oori = 'B'` to indicate combined order and invoice data.

2. **Level Break Processing**:
   - Processes `BB802PQF` records on a level break (`L1`) defined by `saco` (company), `sacust` (customer), `saord` (order number), and `sainvn` (invoice number).
   - Ensures records are processed only when the level break occurs, optimizing performance.

3. **Field Mapping**:
   - Maps fields from `BB802PQF`, combining move detail data (`SA5MOVD`: e.g., `saloc`, `saprod`, `sactqt`) and header data (`SA5SHP`: e.g., `shpord`, `shslmn`, `shsmd8`).
   - Includes additional fields added in revisions jk01 (`w1qty`, `w1um`, `w1orpr`) and jk02 (`w1tkby`, `w1rush`, `w1gpby`, etc.) to support enhanced inquiry capabilities (e.g., quantity, invoice date, freight charge code).

4. **File Group Handling**:
   - Relies on `p$fgrp` ('G' or 'Z') set by **BB802C** to access the correct library (`GSA5MOVD`, `GSA5SHP` or `ZSA5MOVD`, `ZSA5SHP`) via overrides in the CLP.

5. **Product Move Support**:
   - Specifically designed for product move scenarios (`P$PMOV = 'Y'` in **BB802C**), processing `SA5MOVD` instead of `SA5FILD` to handle move-specific sales history details.

6. **Data Integrity**:
   - Uses `CHAIN` to check for existing records in `BB802W`, ensuring no duplicates are created unnecessarily.
   - Clears the record format before writing new records to prevent residual data.
   - Updates existing records to reflect combined data (`w1oori = 'B'`) when applicable.

7. **Revision-Specific Rules**:
   - **jk01**: Added `w1qty` (quantity), `w1um` (unit of measure), and `w1orpr` (order process) to `BB802W` to include sales move detail quantities and processing status.
   - **jk02**: Added fields to `BB802W` (`w1tkby`, `w1rush`, `w1gpby`, `w1racd`, `w1mulo`, `w1mlcd`, `w1tolo`, `w1loda`, `w1lovo`, `w1indt`, `w1inva`, `w1fccd`) to capture additional sales history data (e.g., taken by, invoice date, invoice amount).

---

### **Tables Used**

The program accesses the following database files:

1. **BB802PQF**: Logical file (input primary, `ip`), joins `SA5MOVD` (sales history move details) and `SA5SHP` (sales history headers), containing fields like `saco`, `sacust`, `saord`, `sainvn`, `saloc`, `saprod`, etc.
2. **BB802W**: Work file (update file, `uf`), stores processed inquiry data in `QTEMP` with fields like `w1co`, `w1cust`, `w1ord#`, `w1inv#`, `w1qty`, etc.

---

### **External Programs Called**

The program does not call any external programs. It operates independently, relying on file I/O operations and the environment set up by **BB802C** (e.g., file overrides and open query).

---

### **Additional Notes**

- **Indicator Usage**: Uses `*inl1` for level break processing and `*in99` for file access control (`CHAIN` operation).
- **Field Prefixes**: Uses `w1` prefix for `BB802W` fields, aligning with the work file structure.
- **Key List**: The `klwrk` key list ensures unique identification of records in `BB802W` using `saco`, `sacust`, `saord`, and `sainvn`.
- **Context**: Called by **BB802C** in `SAD` mode with `P$PMOV = 'Y'` (sales history move detail processing), where `BB802PQF` is queried with a selection string (`QRYSLT`) to filter records.
- **Efficiency**: The program uses a simple, non-interactive approach, focusing on data transfer to the work file, with no user interface or message handling.
- **Update Logic**: Like **BB802E** and **BB802F**, it updates existing records with `w1oori = 'B'` to indicate combined data, reflecting integration with other inquiry data.
- **Relation to BB802F**: This program is nearly identical to **BB802F**, but processes `SA5MOVD` instead of `SA5FILD`, supporting product move scenarios introduced in **BB802C** revision JK01.

This program is a critical backend component of the inquiry system, ensuring that sales history move detail data is efficiently transferred to the `BB802W` work file for further processing by the **BB802** inquiry interface.