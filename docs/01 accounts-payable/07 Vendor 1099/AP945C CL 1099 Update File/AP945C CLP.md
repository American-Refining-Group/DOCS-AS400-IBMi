# AP945C Program Details

## Overview
`AP945C.clp` is a CL program that orchestrates the maintenance of the IRS 1099 file (`AP1099`), ensuring compliance through index creation, user-driven record updates, reorganization, total calculations, and sequence number updates.

## Process Steps
1. **Retrieve Parameters**:
   - Accepts `&MODE` (e.g., 'MNT' for maintenance) and `&FGRP` (e.g., 'G' for live, 'T' for test).
   - Sets library to `QS36F` and file names (`&FILE`, `&SAVE`, `&INDEX`) based on `&FGRP` (e.g., `AP1099`, `AP1099SAV`, `AP1099I` or prefixed with `&FGRP`).

2. **Create Index**:
   - Calls `AP945` (OCL) to create index file `&INDEX` for `&FILE`.

3. **Maintenance**:
   - Overrides `AP1099` to `&FILE` and calls `AP945` (RPGLE) with `&MODE` for user-driven record maintenance (add/edit/delete/inquire).

4. **Backup and Reorganize (if &MODE = 'MNT')**:
   - Copies `&FILE` to `&SAVE`, replacing existing records.
   - Copies records back to `&FILE` in order: 'T' (replace), 'A' (add), 'B' (add, excluding position 45 = 'Y'), 'C' (add), 'F' (add).

5. **Update Totals**:
   - Calls `AP315` to update 'C' and 'T' record totals based on 'B' records.

6. **Update Sequence Numbers**:
   - Calls `AP3155` to assign sequential numbers to all records in `&FILE`.

7. **Clean Up**:
   - Deletes index file `&INDEX`.

## Business Rules
- Ensures IRS 1099 compliance with record order and format.
- Supports live (`&FGRP = 'G'`) and test environments via dynamic file naming.
- Maintains data integrity with backup to `&SAVE`.
- Excludes deleted 'B' records (position 45 = 'Y') during reorganization.

## External Programs Called
- `AP945` (OCL Procedure): Creates index file.
- `AP945` (RPGLE Program): Handles user maintenance/inquiry.
- `AP315` (RPG Program): Updates totals in 'C' and 'T' records.
- `AP3155` (RPG Program): Updates sequence numbers.

## Tables Used
- **AP1099** (or `<&FGRP>AP1099`): Main 1099 data file, updated and reorganized.
- **AP1099I** (or `<&FGRP>AP1099I`): Index file, created and deleted.
- **AP1099SAV** (or `<&FGRP>AP1099SV`): Backup file for pre-maintenance data.
- **Library**: `QS36F`.