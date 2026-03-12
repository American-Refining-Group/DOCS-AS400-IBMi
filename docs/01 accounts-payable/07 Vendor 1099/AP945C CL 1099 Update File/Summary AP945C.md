### List of Use Cases Implemented by the Program

Based on the full call stack (`AP945C.clp`, `AP945.ocl36`, `AP945.rpgle`, `AP315.rpg`, `AP3155.rpg`), the programs collectively implement a single primary use case:

1. **Maintain IRS 1099 File**:
   - This use case involves maintaining the `AP1099` file to ensure compliance with IRS 1099 tax form requirements. It supports both live and test environments, handles programmatic record updates (add, edit, delete 'T', 'A', and 'B' records), calculates payment totals, resequences records, and preserves data integrity through backup and reorganization.

---

### Function Requirement Document



# Function Requirement Document: IRS 1099 File Maintenance

## Function Overview
The `Maintain1099File` function processes and maintains the `AP1099` file to ensure compliance with IRS 1099 tax form requirements. It supports live and test environments, updates payment totals, resequences records, and preserves data integrity through backup and reorganization, using programmatic inputs instead of screen interactions.

## Inputs
- **Mode**: String (3 characters, e.g., 'MNT' for maintenance, 'INQ' for inquiry).
- **FileGroup**: String (1 character, 'G' for live files, other values for test files, e.g., 'T').
- **AP1099Data**: File containing 1099 records ('T' for Transmitter, 'A' for Payer, 'B' for Payee, 'C' for Control, 'F' for End) in library `QS36F`.
- **RecordUpdates**: Array of updates for 'T', 'A', 'B' records, including record type, control number, TIN, payment amounts, and other IRS-required fields (e.g., names, addresses).

## Outputs
- **Updated AP1099 File**: File with updated records, totals, resequenced records, and maintained data in `QS36F`.
- **AP1099SAV File**: Backup file containing pre-maintenance data (if Mode = 'MNT').

## Process Steps
1. **Determine File Names**:
   - If FileGroup = 'G', use file names: `AP1099`, `AP1099I` (index), `AP1099SAV` (save).
   - If FileGroup ≠ 'G', prefix files with FileGroup (e.g., `TAP1099`, `TAP1099I`, `TAP1099SV`).
   - Set library to `QS36F`.

2. **Create Index File**:
   - Build index file (`AP1099I` or `<FileGroup>AP1099I`) for `AP1099` with key at position 1 (length 1), allowing duplicate keys, and additional fields at positions 7 (length 4) and 12 (length 9).

3. **Perform Maintenance (if Mode = 'MNT')**:
   - Process RecordUpdates array:
     - **Add**: Validate record type ('T', 'A', 'B'), control number, and TIN for non-blank values; check for duplicates; add new records to `AP1099I`.
     - **Edit**: Update existing 'T', 'A', or 'B' records with provided fields (e.g., payment amounts, names, addresses).
     - **Delete**: Flag 'B' records with `A3DEL = 'Y'` (position 45) or unflag (`A3DEL = ' '`) for removal/restoration.
   - If Mode = 'INQ', skip updates and return record data.

4. **Backup and Reorganize (if Mode = 'MNT')**:
   - Copy `AP1099` to `AP1099SAV`, replacing existing records.
   - Copy records back to `AP1099` in order:
     - 'T' records (replace existing).
     - 'A' records (add).
     - 'B' records where position 45 ≠ 'Y' and record type = 'B' (add).
     - 'C' records (add).
     - 'F' records (add).

5. **Update Totals**:
   - For 'B' records:
     - Sum payment amounts (`A3PAY1` to `A3PAYG`, positions 55–246, 12 digits, 2 decimals).
     - Count total payees.
   - Update 'C' record:
     - Set total payment amounts (`A4TAM1` to `A4TAMG`, positions 19–303) to sums from 'B' records.
     - Set payee count (`A4CNT`, positions 2–9) to total payee count.
   - Update 'T' record:
     - Set total payees (`A1TPAY`, positions 296–303) to total payee count.

6. **Update Sequence Numbers**:
   - Assign sequential numbers (starting from 1) to all records in `AP1099` at positions 500–507 (8 digits).

7. **Clean Up**:
   - Delete index file (`AP1099I` or `<FileGroup>AP1099I`).

## Business Rules
- **IRS Compliance**: Ensure 'T', 'A', 'B', 'C', 'F' records are in correct order and format per IRS 1099 specifications (e.g., `A3PAY1–A3PAYG`, `A1TIN`, `A2TIN`).
- **Environment Support**: Handle live (FileGroup = 'G') and test (FileGroup ≠ 'G') files with dynamic naming.
- **Data Integrity**: Back up `AP1099` to `AP1099SAV` before changes; copy records in specified order.
- **Record Updates**:
   - Validate non-blank record type ('T', 'A', 'B'), control number, and TIN; prevent duplicates.
   - Only 'B' records can be flagged for deletion (`A3DEL = 'Y'`) or unflagged (`A3DEL = ' '`).
- **Calculations**:
   - Sum 'B' record payment amounts (12 digits, 2 decimals) for 'C' record totals (`A4TAM1–A4TAMG`).
   - Count 'B' records for payee totals in 'C' (`A4CNT`) and 'T' (`A1TPAY`) records.
   - Assign 8-digit sequential numbers to all records.
- **Record Filtering**: Exclude 'B' records where position 45 = 'Y' during reorganization.
- **File Management**: Use `QS36F` library; create and delete index file for performance.

## Assumptions
- Input `AP1099` file exists in `QS36F` with valid IRS 1099 record formats.
- `RecordUpdates` array contains valid data for 'T', 'A', 'B' records, including required fields.
- Sufficient disk space for `AP1099SAV` and index files.

## Constraints
- Sequence numbers are 8 digits, limiting to 99,999,999 records.
- Payment amounts are 12 digits with 2 decimals, aligning with IRS field lengths.
- No validation of input record data beyond record type, control number, TIN, and position 45 checks.
- 'C' and 'F' records are not editable; only updated programmatically by totals and sequence steps.

