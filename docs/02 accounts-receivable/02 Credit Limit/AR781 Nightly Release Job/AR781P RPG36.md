This document contains the source code for **AR781P**, an **RPG II** program (running in IBM System/36 or AS/400 compatibility mode). It is the **first** of the three programs executed in the OCL procedure previously analyzed (AR781P → AR781 → AR781A).

### Purpose of AR781P
The program’s header comment states:  
**"CREDIT HOLD ELIGIBLE FOR RELEASE SCREEN"**

Its job is to **pre-screen open order lines** from the backlog file (BBORCL) and identify customers whose orders are potentially eligible for release from credit hold. It creates a small intermediate selection file (AR780I) containing only the **group customer numbers** (parent/grouping customers) that have at least one open order line meeting specific criteria.

This intermediate file (AR780I) is then used by the main program AR781 for detailed credit limit calculations and reporting.

### Process Steps (Execution Flow)

1. **File Declarations**:
   - Input:
     - **BBORCL**: Open order lines (backlog) file – primary input, read sequentially.
     - **ARCUST**: Customer master file – used for chaining to get customer details.
   - Output:
     - **AR780I**: Work/selection file (updated, record length 64) – receives one record per qualifying group customer.

2. **Main Logic (in subroutine ALL, executed for each record in BBORCL)**:
   - The program reads every record in BBORCL (indicator 01 turned on for each input record).
   - It checks four conditions nested with `IF` statements:

     ```rpg
     IF BLORDR ≠ *ZEROS          (order number is present/not blank)
       IF BLOVCL = 'Y'           (order line flagged as over credit limit)
         IF BLAUIN = *BLANKS     (no authorization initials entered)
           IF BLUSID = *BLANKS   (no user ID entered – meaning no manual override)
     ```

     All four must be true for the record to qualify.

   - If the record qualifies:
     - Chain to **ARCUST** using the order line’s customer number (BLCOCU) to retrieve the full customer record.
     - If found (indicator 99 off):
       - Build a key (ARKEY) consisting of:
         - Company/group code (ARGCO, positions 1–2)
         - Grouping customer number (ARGCUS, positions 3–8)
       - Chain to **AR780I** using this composite key.
       - Regardless of whether the record already exists in AR780I:
         - Output a **DETAIL** record via `EXCPT` to AR780I (add or replace).
         - The output record contains:
           - Position 1: literal 'A'
           - Positions 2–3: ARGCO (group company)
           - Positions 4–9: ARGCUS (group customer number)

3. **Result**:
   - AR780I ends up with **one unique record per group customer** (parent billing customer) that has at least one open order line:
     - Marked as over credit limit (`BLOVCL = 'Y'`)
     - Not yet manually authorized/released (no initials or user ID)

### Business Rules Implemented

The program identifies customers **eligible for credit hold release review** based on these rules:

1. Only consider **actual open orders** (order number present).
2. Only consider orders already **flagged as exceeding credit limit** (`BLOVCL = 'Y'` – this flag is presumably set by a prior credit check process).
3. Exclude orders that have already been **manually reviewed and authorized** (i.e., someone has entered authorization initials or a user ID to override the hold).
4. Use **group/parent customer billing structure**:
   - Many companies bill at a parent (group) level while allowing orders at child/ship-to level.
   - The program consolidates to the **group customer** (ARGCO + ARGCUS) so that the subsequent report (AR781) evaluates credit at the correct billing entity level.

This is a **pre-filter** to dramatically reduce the number of customers processed in the main credit check program (AR781), making the batch run faster.

### Tables/Files Used

| File    | Type | Description                                      | Access Mode       | Key/Usage |
|---------|------|--------------------------------------------------|-------------------|-----------|
| **BBORCL**  | Input  | Open order lines (backlog)                       | Sequential read   | Primary driver file |
| **ARCUST**  | Input  | Customer master file                             | Random chain      | Retrieve group/parent info |
| **AR780I**  | Update | Intermediate selection file (work file)          | Random chain + add| Receives group customer keys |

### External Programs Called

- **None**.  
  AR781P does **not** call any external programs (no `CALL` statements).  
  It is a standalone processing program that reads BBORCL, looks up ARCUST, and writes to AR780I.

### Summary

AR781P is a **filtering/pre-selection program** that scans all open order lines and builds a compact list (AR780I) of **parent/group customers** who:
- Have at least one open order exceeding the credit limit,
- Have not yet had that hold manually overridden.

This list is then used by the main program AR781 to perform detailed credit calculations and generate the final "Credit Hold Eligible for Release" report. This two-step design (pre-filter + detailed processing) was common in older batch systems to improve performance.