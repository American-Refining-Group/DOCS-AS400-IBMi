Here is a clear explanation of **BB9004** — the **Billing Control Entry Delete / Restore** program.

### Main Purpose
This is a small **confirmation window** program that performs a **logical delete** or **undelete** (restore) operation on a single **billing control entry** record.

It is called from the main "Work-with" program (most likely **BB900P**) when the user selects **option 4** on a line in the subfile.

- It does **not** physically delete the record.
- It only flips the **delete flag** (`bcdel`) between `'D'` (deleted) and `'A'` (active).

### High-Level Process Flow

1. **Program Start / *inzsr**
   - Receives parameters:
     - `p$co`    → Company code (key = `bcco`)
     - `p$fgrp`  → Environment group ('G' or 'Z')
     - `p$flag`  → Output return flag (will be set to `'D'` or `'A'`)
   - Initializes screen field `f$co` with the passed company code
   - Sets up message queue handling

2. **File Overrides & Opens (opntbl)**
   - Depending on `p$fgrp`:
     - 'G' → overrides to `gbicont` and `garcust`
     - 'Z' → overrides to `zbicont` and `zarcust`
   - Opens:
     - `bicont`   → update-capable (billing control master)
     - `arcust`  → input only (customer master — used to check for dependents)

3. **Retrieve Current Record State (rtvdta)**
   - Chain to `bicont` using company code (`f$co`)
   - If record found:
     - If already `bcdel = 'D'` → show **Restore** window
       - Header = "Billing Control Entry Restore"
       - Function key label = F22=Restore
       - Protect/condition indicators `*in72`
     - If `bcdel ≠ 'D'` (usually `'A'` or blank) → show **Delete** window
       - Header = "Billing Control Entry Delete"
       - Function key label = F23=Delete
       - Protect/condition indicators `*in73`

4. **Display Confirmation Window (prcwdw → exfmt delwdw)**
   - Small modal-like window (`delwdw`)
   - Shows company code and appropriate header / function key text
   - Supports:
     - **F12** → Cancel / exit without change
     - **F22** → Confirm **Restore** (undelete)
     - **F23** → Confirm **Delete**
   - Loops until user presses F12, F22, or F23

5. **Validation / Business Rules Check (winedt → chkact)**
   - Only performed when user presses **F23** (Delete)
   - Checks whether this company code has any assigned customers:
     - `SETLL` on `arcust` by company code
     - If any customer records exist (`*in99 = *off` after SETLL) → error
     - Displays message:  
       **"This Company Has Assigned Customers, Cannot Delete."**
     - Sets `*in50 = *on` (error condition) → prevents update

6. **Perform Update (winupd) — only if no errors**
   - **F22 = Restore**:
     - Chain to record
     - If `bcdel = 'D'`, change to `'A'`
     - Update record
     - Set return flag `p$flag = 'A'`
   - **F23 = Delete**:
     - Chain to record
     - If `bcdel ≠ 'D'`, change to `'D'`
     - Update record
     - Set return flag `p$flag = 'D'`

7. **End**
   - Close files
   - Return to caller (usually BB900P)
   - Caller sees updated status via `p$flag` and typically refreshes the subfile + shows confirmation message

### Core Business Rules Summary

- **Logical delete only** — record is never physically removed (flag `bcdel` = `'D'`)
- **Restore** is allowed unconditionally (if already deleted)
- **Delete** is **protected** by a dependency check:
  - Cannot delete if any customers (`arcust`) are assigned to this company code
  - This prevents orphaning customer records or breaking referential integrity
- Delete/restore is **immediate** — no audit trail or soft-delete history beyond the flag
- Program is **very narrow scope** — only flips one flag, no other field changes
- Called in maintenance context only (no inquiry mode)

### Files / Tables Used

| File in program | Library after override | File type      | Access mode    | Purpose                                      |
|-----------------|------------------------|----------------|----------------|----------------------------------------------|
| bicont          | gbicont or zbicont     | Physical file  | Update         | Billing control master (flip `bcdel` flag)   |
| arcust          | garcust or zarcust     | Physical file  | Input only     | Customer master — check for dependent records |

### External Programs Called

| Program       | Purpose                              | Called from     | Notes                                      |
|---------------|--------------------------------------|-----------------|--------------------------------------------|
| QCMDEXC       | Execute OVRDBF override commands     | opntbl          | Standard IBM command executor              |
| QMHSNDPM      | Send diagnostic message to program queue | addmsg       | Used for "Cannot Delete" message           |
| QMHRMVPM      | Remove/clear messages from queue     | clrmsg          | Standard message API                       |

**No other business-logic or maintenance programs** are called (unlike BB900P which called BB900 and BB9004 itself).

### Typical Call Context (from BB900P)

```text
When user presses 4 on a subfile line:
  → Call 'BB9004'
     → parm company code
     → parm file group
     → parm return flag (output)
  → If return flag = 'D' → show "has been deleted"
  → If return flag = 'A' → show "has been reactivated"
  → Refresh subfile line (blue for deleted, normal for active)
```

This is a classic, minimal **logical delete confirmation** pattern common in older IBM i / AS/400 ERP applications.

Let me know if you want a comparison of delete behavior across BB900P / BB915 / BB9004, or more detail on any part.