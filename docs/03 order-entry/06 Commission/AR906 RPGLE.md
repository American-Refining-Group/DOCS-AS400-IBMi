**AR906 – Commission Charge Maintenance / Inquiry**

This is the **detail maintenance program** called from the Work With subfile program (AR906P). It displays/edits a **single commission charge record** using a simple one-panel format (`FMT01`).

### Process Steps (High-Level Flow)

1. **Initialization (*INZSR)**
   - Receives 8 parameters from AR906P:
     - Key fields: `p$co`, `p$cusn`, `p$loc`, `p$prod`, `p$um`
     - `p$mode` = `'MNT'` (maintenance) or `'INQ'` (inquiry)
     - `p$fgrp` = `'G'` or `'Z'` (library group)
     - `p$flag` = return flag (output – set to `'1'` on successful add/change)
   - Builds and executes file overrides (G* or Z* libraries) via `QCMDEXC`.
   - Opens all database files.
   - Moves key parameters into screen fields.

2. **Retrieve Data (`rtvdta`)**
   - Chain to **ARCOMM** using the full key.
   - If record **does not exist**:
     - Clear record.
     - Chain **GSPROD** and default `acpcls`, `acpgrp`, `acigrp` from product master.
     - Set `w$exists = *OFF`.
   - If record **exists**:
     - Load all fields.
     - Set `w$exists = *ON`.
   - Load descriptive fields (product description, location name, customer name) for display.
   - Set header and global protection based on mode.

3. **Main Processing Loop (`srfmt`)**
   - Displays `FMT01` panel.
   - Loops until F12 is pressed (`fmtagn` flag controls the loop).

4. **Format Processing (`f01sr`)**
   - **F4** → Field prompting (`prompt`).
   - **F10** → Position cursor to home.
   - **F12** → Exit (set `fmtagn = *OFF`).
   - **ENTER** (maintenance mode only):
     - Edit input (`f01edt`).
     - If no errors → Update database (`upddbf`) → Set return flag `'1'`.
     - Exit program (no multi-panel navigation; this is a single-screen program).

5. **Edit / Validation (`f01edt`)**
   - Validates **Vendor Code** (`ACVEND`) against **APVEND**.
   - If invalid → error message and indicator `*IN51`.

6. **Database Update (`upddbf`) – Maintenance Mode Only**
   - Chain to **ARCOMM**:
     - **Record exists** → If any field changed, `UPDATE`.
     - **Record does not exist** → `WRITE` new record with `acdel = 'A'` (Active).
   - Sets `p$flag = '1'` (signals success back to AR906P).

7. **Protection Rules (`f01pro`)**
   - Inquiry mode (`'INQ'`): All input fields protected (`*IN70`–`*IN73` on).
   - Maintenance mode:
     - Key fields protected if record already exists (`*IN71`).
     - All other fields remain editable.

8. **Cleanup**
   - Closes files.
   - Returns to AR906P with updated `p$flag`.

### Business Rules

- **Record Key**: Unique combination of  
  `Company (CO) + Customer (CUSN) + Location (LOC) + Product (PROD) + Unit of Measure (UM)`
- **Status**: New records are always created with `acdel = 'A'` (Active). Inactivation is handled by AR9064.
- **Defaults on Create**:
  - Product Class (`acpcls`), Product Group (`acpgrp`), and Inventory Group (`acigrp`) are copied from the **GSPROD** master.
- **Vendor Requirement**:
  - Vendor code (`ACVEND`) is mandatory and must exist in **APVEND**.
  - Vendor name is displayed after successful F4 prompt.
- **Inquiry Mode**: Read-only view (no updates allowed).
- **Change Detection**: Only updates the database if at least one field actually changed (avoids unnecessary I/O).
- **No Delete Here**: Deletion/inactivation is performed in the Work With program (option 4 → AR9064) or via AR906P.
- **Messages**: Uses the standard program message subfile (`GSMSGF`).

### External Programs Called

| Program     | Called When              | Purpose |
|-------------|--------------------------|-------|
| **LAPVEND** | F4 on Vendor field       | Vendor lookup / prompt window |
| **QCMDEXC** | Startup                  | Execute library overrides (G/Z files) |
| **QMHSNDPM** / **QMHRMVPM** | Throughout | Program message handling |

(No other RPG programs are called.)

### Tables / Files Used

| File       | Usage          | Description |
|------------|----------------|-------------|
| **ARCOMM** | UF A E (main)  | Commission Charge master – add, change, read |
| **APVEND** | Input only     | Vendor master (validation + name) |
| **GSPROD** | Input only     | Product master (defaults + description) |
| **INLOC**  | Input only     | Location master (name) |
| **ARCUST** | Input only     | Customer master (name) |
| **BICONT** | Input only     | Company/business control |

All files are overridden at startup to either the **G** or **Z** library set based on the `p$fgrp` parameter passed from AR906P.

This program follows the standard “maintenance detail” pattern: load → validate → update-or-write → return success flag. It is intentionally simple (single panel, no subfiles) because the complex list navigation and options are handled by the calling Work With program AR906P.