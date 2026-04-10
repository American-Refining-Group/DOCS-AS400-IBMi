Here's a clear and complete explanation of **AR9064** – the **Commission Charge Entry InActivate / ReActivate** program.

### Program Purpose
AR9064 is a **simple confirmation window** program.  
It is called from the Work With program (**AR906P**) when the user selects **Option 4** on a commission charge record.  
Its only job is to allow the user to **toggle the status** of a record between **Active ('A')** and **Inactive ('I')** by pressing **F22** or **F23**.

### Process Steps

1. **Initialization (*INZSR)**
   - Receives 7 parameters from AR906P:
     - Key fields: `p$co`, `p$cusn`, `p$loc`, `p$prod`, `p$um`
     - `p$fgrp` = `'G'` or `'Z'` (library group)
     - `p$flag` = output return flag (will be set to `'A'` or `'I'` on success)
   - Builds and executes the appropriate file override (`QCMDEXC`) for either G* or Z* libraries.
   - Opens **ARCOMM** file.
   - Moves the key fields into screen fields (`f$co`, `f$cusn`, etc.).
   - Sets `winagn = *ON` to enter the main loop.

2. **Retrieve Existing Record (`rtvdta`)**
   - Chains to **ARCOMM** using the full key (`kscomm`).
   - If the record is found:
     - Checks current `acdel` value:
       - If currently **'I'** (Inactive) → Displays **"ReActivate"** header + F22 label, turns on `*IN72` (probably blue color).
       - If currently **'A'** (Active) → Displays **"InActivate"** header + F23 label, turns on `*IN73`.
   - This dynamically changes the window title and function key text depending on current status.

3. **Main Window Processing (`prcwdw`)**
   - Loops while `winagn = *ON`.
   - Displays the confirmation window (`ACTWDW`).
   - Processes user input:

     | Key   | Action |
     |-------|--------|
     | **F12** | Cancel / Exit (no change) |
     | **F22** | ReActivate the record |
     | **F23** | InActivate the record |

4. **Edit / Validation (`winedt` → `chkact`)**
   - Currently, `chkact` is empty (no additional validation is performed).
   - The program relies on the dynamic header/F-key setup from `rtvdta`.

5. **Update Database (`winupd`)**
   - **If F22 is pressed (ReActivate):**
     - Chain to the record.
     - If it exists and `acdel = 'I'`, change `acdel` to `'A'` and `UPDATE`.
     - Set `p$flag = 'A'` (success flag returned to AR906P).

   - **If F23 is pressed (InActivate):**
     - Chain to the record.
     - If it exists and `acdel` is **not** already `'I'`, change `acdel` to `'I'` and `UPDATE`.
     - Set `p$flag = 'I'` (success flag returned to AR906P).

6. **Exit**
   - After successful update, `winagn` is turned off and the program ends.
   - The calling program (AR906P) receives the flag and displays a confirmation message ("has been InActivated" or "has been ReActivated").

### Business Rules

- **Status Field**: `acdel` in file **ARCOMM**  
  - `'A'` = Active  
  - `'I'` = Inactive (soft delete / inactive flag)
- The program **never deletes** records — it only toggles the `acdel` flag.
- It prevents redundant actions:
  - You cannot "InActivate" a record that is already Inactive.
  - You cannot "ReActivate" a record that is already Active.
- The window is **context-sensitive**: it shows the opposite action of the current status.
- No other fields are changed — only the `acdel` flag is updated.
- Success is signaled back via `p$flag` so AR906P can show the correct confirmation message and refresh the subfile.

### Tables / Files Used

| File      | Usage          | Description |
|-----------|----------------|-------------|
| **ARCOMM** | UF A E K (main) | Commission Charge master file – only file used |

Only **one** physical file is accessed. No validation against other masters (customer, product, etc.) is done here because the key was already validated in AR906P.

### External Programs Called

| Program     | Called When     | Purpose |
|-------------|-----------------|---------|
| **QCMDEXC** | Startup         | Execute file override (G* or Z* library) |
| **QMHSNDPM** / **QMHRMVPM** | Message handling | Standard program message subfile support (though no errors are currently sent from this program) |

**No other RPG programs** are called from AR9064.

### Summary – Simple & Focused Design
AR9064 is intentionally very lightweight:
- Single small confirmation window.
- No complex editing or subfiles.
- Only toggles the `acdel` flag between `'A'` and `'I'`.
- Returns a clear success flag so the Work With program (AR906P) knows what message to display.

This is a classic **"soft delete / toggle status"** pattern commonly used in IBM i applications instead of hard deletes.

Would you like me to compare the three programs (AR906P → AR906 → AR9064) side-by-side or explain how the full commission charge flow works from the user’s perspective?