# AP945 (RPGLE) Program Details

## Overview
`AP945.rpgle` provides a subfile-based user interface for adding, editing, deleting, or inquiring about 'T' (Transmitter), 'A' (Payer), and 'B' (Payee) records in the `AP1099` file.

## Process Steps
1. **Initialization**:
   - Accepts `MODE` parameter ('INQ' or 'MNT').
   - Sets inquiry mode (`*IN70` on) if `MODE = 'INQ'`.
   - Positions `AP1099I` to first 'A' record.

2. **Main Loop**:
   - Displays subfile control (`SFLCTL`) with 'A', 'B', 'T' records via `@LOAD`.
   - Handles function keys:
     - F03: Exit.
     - F04: Prompt for company number (`PROMPT`).
     - F06: Add record (`SRADD`).
     - F12: Clear and reposition subfile (`@CLRSF`, `@REPOS`).

3. **Subfile Processing**:
   - Processes user selections (`SELIO`):
     - Option 2: Edit/add record (`SFL02`, formats `FMT01`/'T', `FMT02`/'B', `FMT03`/'A').
     - Option 3: Remove delete flag from 'B' records (`SFL03`, sets `A3DEL = ' '`).
     - Option 4: Flag 'B' records for deletion (`SFL04`, sets `A3DEL = 'Y'`).
     - Option 5: Display record in inquiry mode (`SFL05`).

4. **Add/Edit**:
   - Validates inputs (`C$TY`, `C$CTL`, `C$TIN`) and checks for duplicates (`SRADD`).
   - Updates/adds records via `SRUPDT` (`UPDREC` or `ADDREC`).

5. **Error Handling**:
   - Displays error messages (e.g., "Invalid Type Entered") via `ADDMSG`, `WRTMSG`, `CLRMSG`.

6. **Termination**:
   - Closes files and ends program.

## Business Rules
- Maintains 'T', 'A', 'B' records; 'C' and 'F' are not editable.
- Validates non-blank record type, control number, TIN; prevents duplicates.
- Flags 'B' records for deletion (`A3DEL = 'Y'`) or unflagging (`A3DEL = ' '`).
- Supports inquiry (`MODE = 'INQ'`) and maintenance (`MODE = 'MNT'`) modes.
- Ensures IRS 1099 field compliance (e.g., `A3PAY1–A3PAYG`).

## External Programs Called
- None.

## Tables Used
- **AP1099I** (or `<&FGRP>AP1099I`): Index file for reading/updating records, 824 bytes.
- **AP945D**: Display file with subfile (`SFL`) and formats (`SFLCTL`, `FMT01`, `FMT02`, `FMT03`, `MSGCTL`, `MSGCLR`).
- **Library**: `QS36F`.