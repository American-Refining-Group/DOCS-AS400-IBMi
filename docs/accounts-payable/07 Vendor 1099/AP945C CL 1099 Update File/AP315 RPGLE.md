# AP315 Program Details

## Overview
`AP315.rpg` updates 'C' (Control) and 'T' (Transmitter) records in the `AP1099` file with payment totals and payee counts based on 'B' (Payee) records.

## Process Steps
1. **Initialization**:
   - Defines `AP1099I` as an update file (750 bytes, indexed).
   - Sets `TYPE` to 'B' for reading Payee records.

2. **Read 'B' Records**:
   - Positions to first 'B' record (`SETLL`).
   - Reads 'B' records (`READE`) until end-of-file (`*IN55`).
   - Accumulates payment amounts (`A3PAY1–A3PAYC`) into `W4TAM1–W4TAMC` (15 digits, 2 decimals).
   - Increments payee counter (`W4CNT`).

3. **Update 'C' Record**:
   - Chains to 'C' record (`CHAIN`).
   - Sets total payment amounts (`A4TAM1–A4TAMC`) and payee count (`A4CNT`) from accumulated values.
   - Clears dummy fields (`DUM1–DUM12`) to zeros.
   - Updates record (`EXCPT UPDREC`).

4. **Update 'T' Record**:
   - Chains to 'T' record.
   - Sets total payees (`A1TPAY`) to `W4CNT`.
   - Updates record (`EXCPT UPDREC`).

5. **Termination**:
   - Sets `*INLR` to end program.

## Business Rules
- Processes only 'B' records for totaling payments and counting payees.
- Updates 'C' record with payment totals and payee count.
- Updates 'T' record with total payees.
- Ensures IRS 1099 compliance for field positions and formats.

## External Programs Called
- None.

## Tables Used
- **AP1099I** (or `<&FGRP>AP1099I`): Index file for reading/updating records, 750 bytes.
- **Library**: `QS36F`.