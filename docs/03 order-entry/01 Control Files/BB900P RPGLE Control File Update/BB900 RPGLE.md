Here is a breakdown of the **RPGLE program BB915** (from the source in BB900.rpgle.txt — note the mismatch between filename and internal program name/comment "BB915").

### Main Purpose of the Program
This is a **single-record maintenance / inquiry program** for a **billing control entry** — specifically the record that controls how **freight charges** are handled for a given company (identified by `bcco` = company code).

The program title inside the source is:

> Freight Arranged By Entry Maintenance/Inquiry

It maintains one logical record per company that defines:

- The default **invoicing style** for freight
- Which **general ledger accounts** should be used for different types of freight-related postings

It is typically called from a "Work With" style program (most likely BB900P from your previous question) when the user selects **option 1 (Create)**, **2 (Change)**, or **5 (Display)** on a billing control entry.

### High-Level Execution Flow

1. **Program Start / *inzsr**
   - Receives parameters:
     - `p$co`     → Company code (key field – bcco)
     - `p$mode`   → 'MNT' = maintenance, 'INQ' = inquiry only
     - `p$fgrp`   → 'G' or 'Z' (environment/library group)
     - `p$flag`   → output return flag (set to '1' if record was created/updated)
   - Sets up keylists (`klcont`, `klglms`)
   - Initializes output parameters
   - Moves company code to screen field `f$co`

2. **File Overrides & Opens (opntbl)**
   - Applies OVRDBF depending on `p$fgrp` ('G' → gbicont/gglmast, 'Z' → zbicont/zglmast)
   - Opens:
     - `bicont`   (update-capable – physical file for billing control)
     - `glmast`   (input only – general ledger master)

3. **Retrieve Existing Record (rtvdta)**
   - Chain to `bicont` using company code
   - If found → load fields, set `w$exists = *on`
   - If not found → clear record format, `w$exists = *off`
   - Sets screen header depending on mode (Maintenance vs Inquiry)

4. **Main Screen Loop (srfmt → f01pro → exfmt FMT01 → f01sr)**
   - Only one format is active (`FMT01`)
   - Displays/edits the following main fields:

     | Field     | Meaning / Purpose                              | Required? | Validation rules |
     |-----------|------------------------------------------------|-----------|------------------|
     | BCNAME    | Company / entity name                          | Yes       | Cannot be blank |
     | BCADR1    | Address line 1                                 | Yes       | Cannot be blank |
     | BCINST    | Invoicing Style (freight related)              | Yes       | Must be 1,2,3, or 5 |
     | BCSLGL    | Sales freight G/L account                      | No        | Must exist & not 'D' or 'I' in GLMast |
     | BCFRGL    | Freight expense G/L account                    | No        | Must exist & not 'D' or 'I' |
     | BCMSGL    | Miscellaneous freight G/L                      | No        | Must exist & not 'D' or 'I' (commented error) |
     | BCINGL    | Inventory freight related G/L                  | No        | Must exist & not 'D' or 'I' (commented) |
     | BCCGGL    | Cost of Goods Sold freight G/L                 | No        | Must exist & not 'D' or 'I' (commented) |
     | BCTLGL    | Tolling freight G/L                            | No        | Must exist & not 'D' or 'I' (commented) |

   - **F-keys supported**:
     - F4  → Prompt G/L account (calls LGLMAST for most GL fields)
     - F10 → Cursor home
     - F12 → Exit / return to caller

5. **Validation (f01edt) – Business Rules**
   - Name and address line 1 are **mandatory**
   - Invoicing style **must be 1, 2, 3 or 5** (error message uses com(01) = "Invalid Response...1, 2, 3, or 5")
   - For each G/L account entered:
     - Chain to GLMast (key = company + account + sub + type 'C')
     - Record must exist **and** not be marked deleted (`gldel ≠ 'D'`) **and** not inactive (`gldel ≠ 'I'`) — **JB01 2016 change**
     - If invalid → error on corresponding indicator (54–55 used; others commented)
   - In inquiry mode → clears errors after validation (read-only)

6. **Update Logic (upddbf) – only in MNT mode**
   - If record exists → compare current vs saved → update only if changed
   - If record does not exist → create new record
   - Sets return flag `p$flag = '1'` on successful create/change

7. **Next Format Logic (f01nxt)**
   - Currently only one screen (`FMT01`)
   - If no format change (*in19=*off*) and valid → ends loop → returns to caller

8. **End**
   - Closes files
   - `*inlr = *on` + return

### Core Business Rules Summary

- One control record per company (`bcco`)
- Defines name, address and **freight-related invoicing behavior**
- Controls which **G/L accounts** receive freight-related postings (sales, expense, misc, inventory, COGS, tolling)
- All referenced G/L accounts must be **active** (not deleted 'D' or inactive 'I' per 2016 change)
- Invoicing style restricted to specific values (1,2,3,5) — exact meaning not documented in code (likely internal codes: e.g. customer pays, third party, prepaid, collect, etc.)
- Program supports full **create / change** (MNT) and **display** (INQ) modes
- Address line 1 is mandatory (probably used on invoices / documents)

### Files / Tables Used

| File in program | Library (after override) | File type       | Access mode     | Purpose                              |
|-----------------|---------------------------|-----------------|-----------------|--------------------------------------|
| bicont          | gbicont or zbicont        | Physical file   | Update/Add      | Billing control master (one rec/co)  |
| glmast          | gglmast or zglmast        | Physical file   | Input only      | General Ledger account master        |

### External Programs Called

| Program called | Purpose                              | Called from              | Parameters passed                     | Return value used? |
|----------------|--------------------------------------|--------------------------|----------------------------------------|--------------------|
| LGLMAST        | G/L account prompt / search window   | F4 prompt on GL fields   | company, zeros, zeros, fgrp            | Yes – account & sub |
| QMHSNDPM       | Send diagnostic message              | addmsg                   | (standard IBM API)                     | —                  |
| QMHRMVPM       | Remove messages from program queue   | clrmsg                   | (standard IBM API)                     | —                  |
| QCMDEXC        | Execute OVRDBF commands              | opntbl                   | override string                        | —                  |

No other maintenance or business-rule heavy programs are called (unlike BB900P which called BB900 and BB9004).

This program is a relatively simple, classic **single-record DSPF + maintenance** module focused on freight accounting setup per company.

Let me know if you want deeper explanation of any subroutine (e.g. validation, update logic, prompt handling) or comparison with BB900P.