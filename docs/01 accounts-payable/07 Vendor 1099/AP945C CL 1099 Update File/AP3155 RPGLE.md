# AP3155 Program Details

## Overview
`AP3155.rpg` updates the sequence number (`SEQ#`) field in all records of the `AP1099` file to ensure sequential numbering.

## Process Steps
1. **Initialization**:
   - Defines `AP1099` as an update file (750 bytes, non-indexed).
   - Maps `SEQ#` field (positions 500–507, 8 digits).

2. **Update Sequence Numbers**:
   - Increments counter (`COUNT`, 11 digits) by 1.
   - Assigns counter to `SEQ#` field.
   - Updates each record (`EXCPT UPDREC`).

3. **Termination**:
   - Ends program after processing all records (implicit single-cycle processing).

## Business Rules
- Assigns sequential 8-digit numbers (starting from 1) to all records ('T', 'A', 'B', 'C', 'F').
- Ensures IRS 1099 compliance for sequence number formatting.
- No record type-specific logic or validation.

## External Programs Called
- None.

## Tables Used
- **AP1099** (or `<&FGRP>AP1099`): Data file for updating sequence numbers, 750 bytes.
- **Library**: `QS36F`.