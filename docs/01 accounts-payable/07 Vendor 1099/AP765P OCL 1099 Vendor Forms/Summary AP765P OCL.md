### List of Use Cases Implemented by the AP765 Call Stack

The call stack, consisting of `AP765P.ocl36.txt`, `AP765.ocl36.txt` or `AP765N.ocl36.txt`, `AP766.rpg36.txt`, `AP765.rpg36.txt` (for 1099-MISC) or `AP765N.rpg36.txt` (for 1099-NEC), implements the following primary use case:


1. **Generate and Print Vendor 1099 Forms**:
   - This use case involves processing vendor payment data from a specified year, sorting and consolidating the data, and printing IRS-compliant 1099 forms (either 1099-MISC or 1099-NEC) for vendors whose total payments meet or exceed a threshold amount. The forms include vendor details, payment amounts for specific boxes, and company information, with two forms per page for 1099-MISC (via `AP765`) or three forms per page for 1099-NEC (via `AP765N`).

| Program | Basic Purpose |
|---|---|
| AP765P.ocl36.txt | Controls 1099 processing path (MISC or NEC) |
| AP765.ocl36.txt | Orchestrates 1099-MISC form generation |
| AP765N.ocl36.txt | Orchestrates 1099-NEC form generation |
| AP766.rpg36.txt | Consolidates vendor data for 1099 forms |
| AP765.rpg36.txt | Prints 1099-MISC forms (two per page) |
| AP765N.rpg36.txt | Prints 1099-NEC forms (three per page) |


---

### Function Requirement Document: Generate and Print Vendor 1099 Forms



# Function Requirement Document: Generate and Print Vendor 1099 Forms

## Purpose
Generate IRS-compliant 1099 forms (1099-MISC or 1099-NEC) for vendors based on payment data for a specified year, ensuring proper sorting, consolidation, and formatting for printing.

## Inputs
- **Vendor File (`APVNYYYY`, e.g., `APVN2012`)**: Snapshot of vendor payment data from period-end process (`AP300`), containing:
  - Vendor number, name, address (lines 1-4), Tax ID, 1099 type (`M` for MISC, `N` for NEC), payment amounts, box numbers, payee names (1 and 2), name overflow indicator, state ID, state amount.
- **Year (`?10?`)**: Four-digit year for the 1099 forms (e.g., 2012).
- **1099 Type Flag (`?L'110,1'?`)**: `M` for 1099-MISC or `N` for 1099-NEC.
- **Cross-Reference File (`PA1099X`)**: Configuration data for processing.
- **Threshold Amount (`ENTAMT`)**: Minimum payment amount for including a vendor in the output.
- **Company Information**: Company name, address (lines 1-2), Tax ID, stored in User Data Structure (UDS).

## Outputs
- **Printed 1099 Forms (`AP1099`)**:
  - 1099-MISC forms (two per page) or 1099-NEC forms (three per page) on laser paper, formatted with:
    - Company name, address, Tax ID, and year.
    - Vendor/payee name, address, Tax ID, vendor number (as account number).
    - Payment amounts in boxes 1, 3, 6, or 7.
    - Totals for vendor count and box amounts (1, 3, 6, 7).
- **Temporary Files**:
  - Sorted file (`?9?AP765S`): Intermediate sorted vendor data.
  - Processed file (`?9?AP766`): Consolidated vendor data for printing.

## Process Steps
1. **Sort Vendor Data**:
   - Sort input file (`APVNYYYY`) by 1099 type, vendor number, and company code using `#GSORT`.
   - Filter records based on 1099 type (`M` or `N`) and include only valid types (`N`, `E`, `C`, `D`).
   - Output to temporary file `?9?AP765S`.

2. **Process Vendor Data**:
   - Use `AP766` to read sorted file (`?9?AP765S`) and vendor file (`APVNYYYY`).
   - Consolidate records into one per vendor, using `PA1099X` for configuration.
   - Output processed data to `?9?AP766`.

3. **Validate and Prepare for Printing**:
   - Read `?9?AP766` using `AP765` (for 1099-MISC) or `AP765N` (for 1099-NEC).
   - Calculate total payment (`TOTAMT` = `L1AMT1` + `L1AMT2`) and include vendors where `TOTAMT` >= `ENTAMT`.
   - Assign payment amounts to boxes (1, 3, 6, or 7) based on `BOX1` and `BOX2`.

4. **Format Payee and Address**:
   - If payee name 1 (`VNPYN1`) is blank, use vendor name (`VNNAME`). Otherwise, use `VNPYN1` and `VNPYN2`.
   - If name overflow (`VNNOVF = 'Y'`), use address fields to continue name on the second line.
   - Assign highest non-blank address field (`VNADD4` to `VNADD1`) to city/state/ZIP, shifting others accordingly.

5. **Print Forms**:
   - For 1099-MISC (`AP765`): Print two forms per page with company headers, year, Tax IDs, payee names, addresses, amounts, and vendor number.
   - For 1099-NEC (`AP765N`): Print three forms per page with similar details.
   - Print totals (vendor count, box amounts) on the final page.

6. **Clean Up**:
   - Delete temporary files (`?9?AP765S`, `?9?AP766`) after processing.

## Business Rules
- **Threshold**: Include vendors only if `TOTAMT` >= `ENTAMT`.
- **Form Type**:
  - If `?L'110,1'? = 'M'`, generate 1099-MISC forms (two per page).
  - If `?L'110,1'? = 'N'`, generate 1099-NEC forms (three per page).
- **Payee Names**: Use `VNPYN1` and `VNPYN2` if provided; otherwise, use `VNNAME`. Handle overflow with address fields if `VNNOVF = 'Y'`.
- **Address Formatting**: Use highest non-blank address line for city/state/ZIP, shift others for proper formatting.
- **Box Assignments**: Assign `L1AMT1` and `L1AMT2` to boxes 1, 3, 6, or 7 based on `BOX1` and `BOX2`.
- **Vendor Number**: Print as account number on forms.
- **Totals**: Accumulate vendor count and box amounts (1, 3, 6, 7). For 1099-MISC, compute grand total (`LRAMT`).
- **Special Vendor Flags**:
  - For 1099-MISC: Flag vendor `9384` with `YES = 'YES'`.
  - For 1099-NEC: Flag vendor `32800` with `YES = 'YES'`.
- **Printer Settings**: Use form type `1099`, 10 CPI, 6 LPI on laser paper.

## Calculations
- **Total Payment**: `TOTAMT = L1AMT1 + L1AMT2` (9,2 numeric).
- **Box Amounts**: Assign `L1AMT1` and `L1AMT2` to boxes (1, 3, 6, or 7) based on `BOX1` and `BOX2` for each form.
- **Totals**:
  - Increment `COUNT` for each vendor printed.
  - Accumulate `AMT1x` to `LRAMT1`, `AMT3x` to `LRAMT3`, `AMT6x` to `LRAMT6`, `AMT7x` to `LRAMT7` (10,2 numeric).
  - For 1099-MISC: Compute `LRAMT = LRAMT1 + LRAMT3 + LRAMT6 + LRAMT7` (11,2 numeric).

## Constraints
- Input file (`APVNYYYY`) must exist and contain valid vendor data.
- Temporary files are deleted before and after processing to avoid conflicts.
- Year 2000 compliance is ensured via `GSY2K`.

## Success Criteria
- All eligible vendors (meeting threshold) are printed on IRS-compliant 1099 forms (MISC or NEC).
- Forms are correctly formatted with two (MISC) or three (NEC) per page.
- Totals and vendor counts are accurate on the final page.

