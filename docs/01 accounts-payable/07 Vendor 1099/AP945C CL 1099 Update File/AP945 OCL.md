# AP945 (OCL) Program Details

## Overview
`AP945.ocl36` is an S/36 OCL procedure that builds an index file (`AP1099I` or `<&FGRP>AP1099I`) for the `AP1099` file to enable efficient record access.

## Process Steps
1. **Check File Group**:
   - Evaluates `?9?` (9th parameter, `&FGRP` from `AP945C`).

2. **Build Index**:
   - If `?9? = 'G'`, creates `AP1099I` for `AP1099` with key at position 1 (length 1), duplicate keys allowed, and fields at positions 7 (length 4) and 12 (length 9).
   - If `?9? ≠ 'G'`, creates `<&FGRP>AP1099I` for `<&FGRP>AP1099` with the same key structure.

## Business Rules
- Creates index for efficient access by record type (position 1).
- Supports live (`&FGRP = 'G'`) and test environments with dynamic file naming.
- Ensures duplicate keys are allowed for multiple records of the same type (e.g., 'B' records).

## External Programs Called
- None.

## Tables Used
- **AP1099** (or `<&FGRP>AP1099`): Input data file for index creation.
- **AP1099I** (or `<&FGRP>AP1099I`): Output index file.
- **Library**: `QS36F`.