The **BB802G** RPGLE program is a component of the Customer Orders and Invoicing system, designed to update the `BB802W` work file with ship-to city and state information from the `SHIPTO` file. It is called by the **BB802C** CLP program as part of the Customer Order and Invoice Inquiry process to enhance inquiry data with location details. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the BB802G RPG Program**

The program reads records from the `BB802W` work file, retrieves corresponding ship-to data from the `SHIPTO` file based on the ship-to code (`w1ship`), and updates the work file with city (`w1dstc`) and state (`w1dsts`). The steps are focused on data enrichment with no user interaction. Here’s a breakdown of the key process steps:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Purpose**: Sets up the program environment and initializes key lists.
   - **Actions**:
     - Defines a key field `k$999` as like `w1ship` and initializes it to `999` for accessing the `SHIPTO` file in specific cases.
     - Defines two key lists for accessing the `SHIPTO` file:
       - `klord999`: Uses `w1co` (company), `w1ord#` (order number), and `k$999` (fixed ship-to code `999`).
       - `klcstshp`: Uses `w1co` (company), `w1cust` (customer), and `w1ship` (ship-to code).
     - No input parameters are received, indicating the program processes all records in `BB802W`.

2. **Main Processing Loop** (implied, not explicitly coded):
   - **Purpose**: Reads `BB802W` records sequentially and updates them with ship-to data.
   - **Actions**:
     - The program implicitly reads `BB802W` records (as it is defined as an update primary file, `up`).
     - For each record, it evaluates the `w1ship` field to determine the appropriate key list for accessing `SHIPTO`.

3. **Ship-To Data Retrieval and Update**:
   - **Purpose**: Retrieves city and state from `SHIPTO` and updates `BB802W`.
   - **Actions**:
     - **Initial Check**:
       - Performs a `SETLL` on `SHIPTO` using `klord999` (`w1co`, `w1ord#`, `k$999`) to check if a record exists for ship-to code `999`, setting indicator `*in80`.
     - **Conditional Logic** (via `SELECT`):
       - **Case 1: `w1ship` between `001` and `899`**:
         - Chains to `SHIPTO` using `klcstshp` (`w1co`, `w1cust`, `w1ship`).
         - If a record is found (`*in99 = *off`):
           - Updates `BB802W` with:
             - `w1dstc` ← `csctst` (city/state description)
             - `w1dsts` ← `csstat` (state)
           - Writes the updated record (`update bb802wpf`).
       - **Case 2: `w1ship` ≥ `900` and `klord999` record exists (`*in80 = *on`)**:
         - Chains to `SHIPTO` using `klord999` (`w1co`, `w1ord#`, `k$999`).
         - If a record is found (`*in99 = *off`):
           - Updates `BB802W` with `w1dstc` and `w1dsts` as above.
           - Writes the updated record (`update bb802wpf`).
       - **Case 3: `w1ship` between `900` and `998` and `klord999` record does not exist (`*in80 = *off`)**:
         - Chains to `SHIPTO` using `klcstshp` (`w1co`, `w1cust`, `w1ship`).
         - If a record is found (`*in99 = *off`):
           - Updates `BB802W` with `w1dstc` and `w1dsts` as above.
           - Writes the updated record (`update bb802wpf`).
     - If no matching `SHIPTO` record is found (`*in99 = *on`), the `BB802W` record is not updated.

4. **Program Termination**:
   - **Purpose**: Completes processing and exits.
   - **Actions**:
     - Ends the program after processing all `BB802W` records, closing files implicitly (as `BB802W` and `SHIPTO` are defined with `up` and `if` respectively).

---

### **Business Rules**

1. **Work File Update**:
   - Updates `BB802W` records with city (`w1dstc`) and state (`w1dsts`) from `SHIPTO` based on the ship-to code (`w1ship`).
   - Only updates records if a matching `SHIPTO` record is found.

2. **Ship-To Code Logic**:
   - Handles ship-to codes differently based on their range:
     - Codes `001`–`899`: Uses `klcstshp` (`w1co`, `w1cust`, `w1ship`) to access `SHIPTO`.
     - Codes `900`–`998`: Prefers `klord999` (`w1co`, `w1ord#`, `999`) if a record exists (`*in80 = *on`), otherwise falls back to `klcstshp`.
     - Special case for code `999`: Uses `klord999` to look up a specific ship-to record tied to the order.
   - This logic accommodates different ship-to code conventions (e.g., customer-specific vs. order-specific ship-to records).

3. **Data Integrity**:
   - Uses `SETLL` and `CHAIN` to verify the existence of `SHIPTO` records before updating `BB802W`.
   - Does not modify `BB802W` if no matching `SHIPTO` record is found (`*in99 = *on`).

4. **File Group Handling**:
   - Relies on `p$fgrp` ('G' or 'Z') set by **BB802C** to access the correct library (`GSHIPTO` or `ZSHIPTO`) via overrides in the CLP.

5. **No Revisions**:
   - No revisions are listed, indicating the program has remained stable since its creation in 2011.

---

### **Tables Used**

The program accesses the following database files:

1. **BB802W**: Work file (update primary, `up`), stores processed inquiry data in `QTEMP` with fields like `w1co`, `w1cust`, `w1ord#`, `w1ship`, `w1dstc`, `w1dsts`, etc.
2. **SHIPTO**: Ship-to master file (input file, `if`), contains ship-to details like `csctst` (city/state description) and `csstat` (state).

---

### **External Programs Called**

The program does not call any external programs. It operates independently, relying on file I/O operations and the environment set up by **BB802C** (e.g., file overrides).

---

### **Additional Notes**

- **Indicator Usage**: Uses `*in80` for `SETLL` results on `klord999`, `*in99` for `CHAIN` results, and no user interface indicators (as it’s non-interactive).
- **Field Prefixes**: Uses `w1` prefix for `BB802W` fields and `cs` prefix for `SHIPTO` fields.
- **Key Lists**:
  - `klord999`: Used for ship-to code `999`, tying ship-to data to the order (`w1co`, `w1ord#`).
  - `klcstshp`: Used for customer-specific ship-to codes (`w1co`, `w1cust`, `w1ship`).
- **Context**: Called by **BB802C** after populating `BB802W` with order and invoice data (via **BB802A**, **BB802B**, **BB802E**, **BB802F**, **BB802H**, **BB802I**, **BB802P**) to add geographic data for inquiry display.
- **Efficiency**: The program processes `BB802W` sequentially, using conditional logic to optimize `SHIPTO` lookups based on ship-to code ranges.
- **Relation to Other Programs**: Complements other **BB802** series programs by enriching `BB802W` with location data, enhancing the inquiry interface’s display capabilities.

This program is a critical backend component of the inquiry system, ensuring that ship-to city and state data are added to the `BB802W` work file for use in the **BB802** inquiry interface.