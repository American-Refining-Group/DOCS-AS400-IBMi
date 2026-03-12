The provided document is an RPG (Report Program Generator) program, `AP761.rpg36.txt`, which is called by the OCL program `AP760.ocl36.txt` as part of the vendor 1099 register process on an IBM AS/400 or iSeries system. This RPG program preprocesses the 1099 data to create a consolidated work file (`AP761`) with one record per vendor, regardless of multiple company affiliations, for use in generating the final 1099 register report. Below, I’ll explain the process steps, business rules, tables used, and any external programs called, based on the provided RPG code and its integration with the OCL program.

---

### Process Steps of the AP761 RPG Program

The `AP761` RPG program reads sorted 1099 data from a temporary file (`AP760S`) and the original 1099 file (`APVEND`), aggregates amounts for each vendor, adjusts for dual 1099 box amounts, and writes a consolidated record to the output file (`AP761`). Here’s a step-by-step breakdown of the process:

1. **Initialization**:
   - At the start of each level 1 break (`L1`, triggered by a change in the control field defined in `AP760S`), the program initializes accumulators and fields:
     - `L1AMT1` (primary 1099 amount): Set to zero (line 0025).
     - `L1AMT2` (secondary 1099 box amount): Set to zero (line 0026).
     - `BOX1` (first 1099 box number): Set to zero (line 0027).
     - `BOX2` (second 1099 box number): Set to zero (line 0028).

2. **Read Input Files**:
   - **Primary File (`AP760S`)**:
     - Defined as an input file (`IR`), externally described (`EDISK`), with a 30-byte record and a 3-byte key (lines 0007, 0009).
     - Sorted by 1099 type, vendor, and company (as defined in `AP760.ocl36.txt`).
     - Drives the program cycle, triggering processing for each record.
   - **Secondary File (`APVEND`)**:
     - Defined as the primary input file (`IP`), 579 bytes, internally described (line 0006).
     - Matched with `AP760S` records to retrieve detailed vendor data (e.g., name, amounts, 1099 codes).

3. **Accumulate 1099 Amounts**:
   - **Determine Year**:
     - Check the `CURLST` field (from User Data Structure at offset 141, set by `AP760.ocl36.txt` as `'C'` or `'L'`) to determine whether to use current or last year’s year-to-date (YTD) paid amount (line 0033).
     - Regardless of `CURLST`, the program adds `VNYTDP` (current year YTD paid, positions 242–247) to `L1AMT1` (lines 0034–037). Note: The code comment suggests a future change to use `VNLYDP` (last year YTD paid, positions 248–253) or an individual year file (`APVNXXXX`), but currently, it uses `VNYTDP` for both cases.
   - **Secondary Box Amount**:
     - Add `VNB2AM` (second 1099 box amount, positions 280–285) to `L1AMT2` (line 0039).

4. **Set 1099 Box Numbers**:
   - **First Box (`BOX1`)**:
     - If `BOX1` is zero, set it to `VNBOX1` (first 1099 box number, positions 276–277) (lines 0041–0043).
   - **Second Box (`BOX2`)**:
     - If `BOX2` is zero, set it to `VNBOX2` (second 1099 box number, positions 278–279) (lines 0045–0047).
   - **Default Box (`BOX1`)**:
     - At level 1 (`L1`), if `BOX1` is still zero, set it to 7 (default 1099 box, e.g., for miscellaneous income) (lines 0049–0051).

5. **Adjust Amounts for Dual Boxes**:
   - At level 1 (`L1`), if `L1AMT2` (secondary box amount) is greater than zero and `BOX2` is non-zero, subtract `L1AMT2` from `L1AMT1` (lines 0052–0056). This ensures the total YTD amount is split between the two 1099 boxes, as `VNYTDP` includes both amounts.

6. **Write Output Record**:
   - At level 1 (`L1`), if the above conditions are met, the program writes a record to the output file `AP761` using the `L1ADD` exception output (line 0057).
   - **Output Fields** (lines 0059–0068, JB01, JB02):
     - Record identification: `'A'` (position 1).
     - `VN1099` (1099 code, position 264, 1 byte, renamed from `VN1099L2`).
     - `VNVEND` (vendor number, positions 4–8, 5 bytes, renamed from `VNVENDL1`).
     - `VNID#` (1099 ID number, positions 265–275, 11 bytes).
     - `VNNAME` (vendor name, positions 9–38, 30 bytes).
     - `L1AMT1` (primary 1099 amount, 9 bytes, packed decimal).
     - `L1AMT2` (secondary 1099 box amount, 9 bytes, packed decimal).
     - `BOX1` (first 1099 box number, 2 bytes).
     - `BOX2` (second 1099 box number, 2 bytes).
     - `VNPYN1` (payee name 1, positions 300–339, 40 bytes, added by JB01).
     - `VNPYN2` (payee name 2, positions 340–379, 40 bytes, added by JB01).
     - `VNNOVF` (name overflow, position 216, 1 byte, added by JB02).
     - `VNADD1` (address line 1, positions 39–68, 30 bytes, added by JB02).

---

### Business Rules

The `AP761` RPG program enforces the following business rules for preprocessing the 1099 register:

1. **Single Record per Vendor**:
   - The program consolidates multiple records for the same vendor (across different companies) into a single output record in `AP761`, aggregating YTD amounts and handling dual 1099 boxes.

2. **Year Selection**:
   - The program uses the current year’s YTD paid amount (`VNYTDP`) for both current (`CURLST = 'C'`) and last year (`CURLST = 'L'`) processing, pending a future update to use `VNLYDP` or a year-specific file (`APVNXXXX`).

3. **1099 Box Handling**:
   - If a second 1099 box amount (`VNB2AM`) exists, it is accumulated separately (`L1AMT2`) and subtracted from the primary amount (`L1AMT1`) to split the total YTD amount between two boxes.
   - The first 1099 box (`BOX1`) defaults to 7 (e.g., miscellaneous income) if not specified.
   - Box numbers (`VNBOX1`, `VNBOX2`) are preserved if provided in the input file.

4. **Record Exclusion**:
   - Records marked as deleted (`VNDEL = 'D'`, position 1) are skipped (implicitly handled by the sort in `AP760.ocl36.txt`).

5. **Data Inclusion**:
   - The output record includes critical 1099 data: 1099 code, vendor number, ID number, name, amounts, box numbers, payee names (added by JB01), name overflow, and address line 1 (added by JB02).
   - Payee names (`VNPYN1`, `VNPYN2`) and name overflow (`VNNOVF`) support extended vendor identification and formatting for printing.

6. **Error Handling**:
   - The program assumes valid input from the sorted file (`AP760S`) and original file (`APVEND`). No explicit validation is performed, as filtering (e.g., company, type) is handled by the sort in `AP760.ocl36.txt`.

---

### Tables Used

The RPG program uses the following files (tables):

1. **APVEND**:
   - Type: Disk file (`DISK`), primary input file (`IP`), internally described.
   - Size: 579 bytes.
   - Fields:
     - `VNDEL` (position 1, 1 byte): Delete code (e.g., `'D'` for deleted).
     - `VNCO` (positions 2–3, 2 bytes): Company number.
     - `VNVENDL1` (positions 4–8, 5 bytes): Vendor number.
     - `VNNAME` (positions 9–38, 30 bytes): Vendor name.
     - `VNADD1` (positions 39–68, 30 bytes): Address line 1 (added by JB02).
     - `VNNOVF` (position 216, 1 byte): Name overflow (added by JB02).
     - `VNYTDP` (positions 242–247, packed decimal, 6 bytes): Current year YTD paid.
     - `VNLYDP` (positions 248–253, packed decimal, 6 bytes): Last year YTD paid.
     - `VN1099L2` (position 264, 1 byte): 1099 code.
     - `VNID#` (positions 265–275, 11 bytes): 1099 ID number.
     - `VNBOX1` (positions 276–277, 2 bytes): First 1099 box number.
     - `VNBOX2` (positions 278–279, 2 bytes): Second 1099 box number.
     - `VNB2AM` (positions 280–285, packed decimal, 6 bytes): Second 1099 box amount.
     - `VNPYN1` (positions 300–339, 40 bytes): Payee name 1 (added by JB01).
     - `VNPYN2` (positions 340–379, 40 bytes): Payee name 2 (added by JB01).
   - Purpose: Provides detailed vendor 1099 data, matched with `AP760S` records.

2. **AP760S**:
   - Type: Disk file (`DISK`), input file (`IR`), externally described (`EDISK`).
   - Size: 30 bytes, with a 3-byte key.
   - Purpose: Sorted 1099 data (by 1099 type, vendor, company) from the `#GSORT` step in `AP760.ocl36.txt`. Drives the program cycle and provides control fields for matching with `APVEND`.

3. **AP761**:
   - Type: Disk file (`DISK`), output file (`O`).
   - Size: 196 bytes.
   - Fields: As listed in the output specification (lines 0059–0068, JB01, JB02).
   - Purpose: Consolidated work file with one record per vendor, containing aggregated amounts and formatting data for the final 1099 register report.

---

### External Programs Called

The `AP761` RPG program does not explicitly call any external RPG or CL programs. It is invoked by the OCL program `AP760.ocl36.txt` and produces output for the subsequent program `AP760`. Key points:

- **Called by OCL**:
  - The OCL program `AP760.ocl36.txt` loads and runs `AP761`, passing file labels `?13?` (e.g., `APVN2025`) for `APVEND` and `?9?AP760S` (e.g., `PRODAP760S`) for `AP760S`, and defining `?9?AP761` (e.g., `PRODAP761`) as the output file.

- **No Subprogram Calls**:
  - The program uses no internal subroutines or external program calls. All processing is handled within the RPG cycle and exception output.

- **Downstream Program**:
  - The output file `AP761` is used by the `AP760` program (called later in `AP760.ocl36.txt`) to generate the final 1099 register report.

---

### Integration with OCL Program

The `AP761` RPG program integrates with `AP760.ocl36.txt` as follows:

- **Input Files**:
  - `APVEND` uses the 1099 file (`?13?`, e.g., `APVN2025`) created by `AP300` (period-end processing).
  - `AP760S` is the sorted file (`?9?AP760S`) produced by `#GSORT` in the OCL, filtered by company and 1099 type selections from `AP760P.rpg36.txt`.

- **Output File**:
  - `AP761` produces the intermediate file (`?9?AP761`), which is used as input by the `AP760` program for the final report.

- **Parameters**:
  - The `CURLST` field (offset 141 in UDS) is set by `AP760.ocl36.txt` based on `KYCRLS` (`'C'` or `'L'`) from `AP760P.rpg36.txt`, controlling whether current or last year amounts are used (though currently, `VNYTDP` is used for both).

- **Change History**:
  - **JB01 (01/30/2013)**: Added `VNPYN1` and `VNPYN2` to the output file to support printing payee names.
  - **JB02 (01/20/2014)**: Added `VNNOVF` and `VNADD1` to support name overflow and address line 1 for improved report formatting.

---

### Summary

- **Process Steps**: The program initializes accumulators, reads sorted (`AP760S`) and original (`APVEND`) 1099 data, aggregates YTD amounts per vendor, adjusts for dual 1099 boxes, sets default box numbers, and writes a consolidated record to `AP761` with vendor details, amounts, and formatting fields.
- **Business Rules**: Consolidate vendor records across companies into one record, use current YTD amounts (pending future year-specific logic), split amounts between two 1099 boxes if applicable, default `BOX1` to 7 if unspecified, and include payee names, name overflow, and address for printing.
- **Tables Used**:
  - `APVEND`: Original 1099 data (e.g., `APVN2025`).
  - `AP760S`: Sorted 1099 data (e.g., `PRODAP760S`).
  - `AP761`: Output work file (e.g., `PRODAP761`).
- **External Programs Called**: None directly; called by `AP760.ocl36.txt`, produces output for `AP760`.

The `AP761` program serves as a critical preprocessing step, consolidating and formatting 1099 data for the final report generated by `AP760`, ensuring accurate aggregation and proper handling of 1099 box amounts.