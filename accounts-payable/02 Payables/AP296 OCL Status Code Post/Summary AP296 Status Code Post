##AP296 Summary 

# Function Requirements: Post A/P Voucher Status Code Changes

## Overview
The `PostAPVoucherStatus` function updates the hold status (hold code and description) of Accounts Payable (A/P) vouchers in the open A/P file and generates a report summarizing the changes. It processes input from a work file, validates data against company and vendor records, and ensures accurate updates and reporting for the A/P Voucher Status Code Change Process.

## Inputs
- **Work File (`APVCTR`)**: Contains voucher records with:
  - Company Number (`ATCONO`, 2 bytes)
  - Vendor Number (`ATVEND`, 5 bytes)
  - Voucher Number (`ATVOUC`, 5 bytes)
  - Hold Code (`ATHOLD`, 1 byte)
  - Hold Description (`ATHLDS`, 25 bytes)
  - Prior Hold Code (`ATOHLD`, 1 byte)
  - Prior Hold Description (`ATOHDS`, 25 bytes)
  - Transaction Key (`ATNKEY`, 12 bytes)
- **Control File (`APCONT`)**: Contains company details (e.g., `ACNAME`, company name, 30 bytes).
- **Vendor File (`APVEND`)**: Contains vendor details (e.g., `VNVEND`, vendor number; `VNNAME`, vendor name).
- **Open A/P File (`APOPNH`)**: Contains open A/P transactions (updated with new hold codes and descriptions).
- **Printer File (`LIST`)**: Output file for the A/P Voucher Status Code Change Process report.

## Outputs
- **Updated Open A/P File (`APOPNH`)**: Records updated with new hold codes and descriptions.
- **Report (`LIST`)**: A printer file containing the A/P Voucher Status Code Change Process report with:
  - Company name (`ACNAME`)
  - Page number (`PAGE`)
  - Date (`SYSDAT`) and time (`SYSTIM`)
  - Voucher details (`ATVEND`, `VNNAME`, `ATVOUC`, `ATHOLD`, `ATHLDS`, `ATOHLD`, `ATOHDS`)
  - Indicator for no prior hold code (`50` on)

## Process Steps
1. **Validate Input Data**:
   - Check if `APVCTR` contains any records.
   - If no records exist, pause with message: *"THERE ARE NO RECORDS TO POST.  PRESS 0 TO CANCEL."*
   - Cancel the process if the user presses `ATTN,2,ENTER`.

2. **Load RPG Program**:
   - Load the `AP296` RPG program.

3. **Validate Company**:
   - Chain `ATCONO` to `APCONT` to verify the company number.
   - Set indicator `92` if the company number is invalid.

4. **Validate Vendor**:
   - Chain `VENDKY` (built from `ATCONO` and `ATVEND`) to `APVEND` to verify the vendor number.
   - Set indicator `31` if the vendor number is invalid.

5. **Check Prior Hold Status**:
   - If `ATOHLD` and `ATOHDS` are blank, set indicator `50` on.
   - Otherwise, no prior hold code exists.

6. **Update Open A/P Records**:
   - For each valid voucher in `APVCTR`, chain `OPNKEY` (built from `ATNKEY` and `1001`) to `APOPNH`.
   - If found (indicator `90` off), update `ATHOLD` (position 126) and `ATHLDS` (positions 127–151) in `APOPNH`.
   - Execute exception output (`EXCPT`).

7. **Generate Report**:
   - For each processed voucher, output to `LIST`:
     - Vendor number (`ATVEND`)
     - Vendor name (`VNNAME`)
     - Voucher number (`ATVOUC`)
     - New hold code (`ATHOLD`)
     - New hold description (`ATHLDS`)
     - Prior hold code (`ATOHLD`)
     - Prior hold description (`ATOHDS`)
     - Indicator for no prior hold code (`50` on)

8. **Clean Up**:
   - Delete the processed work file (`APVCTR`) after successful posting.

## Business Rules
- **Data Validation**:
  - The company number (`ATCONO`) must exist in `APCONT`.
  - The vendor number (`ATVEND`) must exist in `APVEND`.
  - The voucher must exist in `APOPNH` with matching vendor and company records.

- **Hold Code Updates**:
  - Update the hold code (`ATHOLD`) and hold description (`ATHLDS`) in `APOPNH` only for valid vouchers.
  - Check for prior hold codes (`ATOHLD`, `ATOHDS`) and update only if they are not blank.

- **Reporting**:
  - Generate a report (`LIST`) summarizing the A/P Voucher Status Code Change Process:
    - Company name (`ACNAME`)
    - Page number (`PAGE`)
    - Date (`SYSDAT`) and time (`SYSTIM`)
    - Voucher details (`ATVEND`, `VNNAME`, `ATVOUC`, `ATHOLD`, `ATHLDS`, `ATOHLD`, `ATOHDS`)
    - Indicator for no prior hold code (`50` on)

- **Error Handling**:
  - If the company number is invalid (`92` on), skip processing for that company.
  - If the vendor number is invalid (`31` on), skip processing for that vendor.
  - If no open A/P record is found (`90` on), skip the update but generate report output.

- **Data Cleanup**:
  - Delete the processed `APVCTR` work file after successful posting.

## Calculations
- No calculations are explicitly performed in this function. It updates hold codes and descriptions based on input data and validation results.

## External Programs Called
- **AP296**: The main RPG program that processes the A/P Voucher Status Code Change Process.

## Notes
- The function assumes that the input data in `APVCTR` is accurate and complete.
- The output report (`LIST`) is formatted for printing and includes all relevant voucher details.
- The `SFACOPY` system utility is used to manage the report output in the output queue.

