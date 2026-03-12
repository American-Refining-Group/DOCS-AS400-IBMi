The `BB1011.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is a subroutine of the `BB101` order entry program, responsible for validating container/unit of measure combinations, retrieving rack and invoice prices, validating distributor purchase quantities for resale, and performing related calculations. The program includes multiple revisions to enhance pricing logic, handle export orders, and incorporate additional validations. Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB1011` program processes order detail data to validate inputs, retrieve pricing, and perform conversions, particularly for order entry in the `BB101` system. The steps are as follows:

1. **Program Initialization**:
   - The program is defined as part of `BB101` and uses files for validation and pricing retrieval.
   - File definitions include:
     - `GSCTWT`: Contract weight table (64 bytes, indexed, key length 9).
     - `GSUMCV`: Customer master unit of measure conversion (64 bytes, indexed, key length 12).
     - `PRCNTR`: Product container file (51 bytes, indexed, key length 9).
     - `GSCTUM`: Customer master unit of measure (64 bytes, indexed, key length 12).
     - `BBPRCE`: Rack price file (128 bytes, indexed, key length 27).
     - `BBBCPR`: Billed customer price file (128 bytes, indexed, key length 32).
     - `BICUAY`: Sales agreement file (256 bytes, indexed, key length 41, added in MG16).
     - `INLOC`: Inventory location file (512 bytes, indexed, key length 5).
     - `SA5FIZD`: Sales agreement history file (1024 bytes, indexed, key length 20).
     - `GSPROD`: Product file (512 bytes, indexed, key length 6, added in JK04, replacing `GSTABL`).
     - `GSCNTR6`: Container file (512 bytes, indexed, key length 3, added in JK02, replacing `GSTABL6`).
   - Data definitions:
     - `COM`: Array of 8 error messages (40 characters each).
     - `RKPC`: Array for rack prices (9.4 format, 5 elements).
     - `RKQT`: Array for rack quantity levels (7.0 format, 5 elements, ascending).
     - `BPPC`: Array for billed customer prices (6.4 format, 5 elements).

2. **Validation of Container/Unit of Measure**:
   - Validates the product (`@PROD`), container (`@CONO`), and unit of measure (`@UNM`) combination:
     - Chains to `GSCNTR6` (replacing `GSTABL6`, JK02) to verify container code.
     - Chains to `GSCTUM` to retrieve the IMS unit of measure for `BBORTR` and `BBORDR` (VV04).
     - If the unit of measure is not 'LBS', chains to `GSUMCV` to convert quantities to gallons (`ORGAL`) using conversion factor `UCCVFA` (multiplies if `UCOPER='M'`, divides otherwise).
     - If the combination is invalid, sets error message "INVALID PRD/CNT/UMS" (COM,04) or "INVALID PRD UM CNVRSN TO 'GAL'" (COM,01).

3. **Rack Price Retrieval**:
   - Chains to `BBPRCE` to retrieve rack prices for the product/container/unit of measure combination.
   - If the ship-to is export-specific (`CSEXPO='Y'` in `SHIPTO`, DC01), uses location '999' for rack price retrieval (revised to remove special '999' logic in JB07).
   - If no rack price is found and not required (JB10), continues processing; otherwise, sets error "INVALID RACK PRICE" (COM,02).
   - For inactive rack prices (`INACTIVE='I'`), issues error "RACK PRICE IS MARKED INACTIVE" (COM,07) and prevents order acceptance (JB10).
   - For inactive but shippable rack prices (`INACTIVE='B'`), issues warning "RACK PRICE IS MARKED INACTIVE" (COM,07) and allows continuation after stock check with function key (JB10).

4. **Sales Agreement Pricing**:
   - Chains to `BICUAY` (MG16) for ship-to specific pricing, falling back to 'ALL' ship-tos if no specific record exists.
   - New key for `BICUAY` includes company, customer, location, container, bill-to PO#, and start date (JB09).
   - Validates unit of measure against sales agreement (JB06).
   - Allows zero end date for non-expired agreements (JB09).
   - Uses billed PO# in pricing logic (JB09).
   - Includes freight code in the key for pricing selection when start dates are the same (MG06).
   - Checks sales agreement with order number in PO/order# field before PO# check (MG05).
   - Compares ordered quantity to minimum/maximum quantities in `BICUAY` (`BAMNQY`, `BAMXQY`, JK01); if below minimum, sets error "QUANTITY BELOW MINIMUM-PRESS F6 TO ALLOW" (COM,03).

5. **Billed Customer Price Retrieval**:
   - Chains to `BBBCPR` to retrieve billed customer prices.
   - If no valid price is found, sets error "INVALID BILLED CUSTOMER PRICE" (COM,05) or "INVALID - MUST HAVE A PRICE" (COM,06, fixed in JB11).

6. **Distributor Purchase Validation**:
   - Checks if the distributor has purchased enough product to resell (`ORGAL` vs. `SATOTG`):
     - Sets key for `SA5FIZD` using company (`@CONO`), customer (`DICU`), product (`@PROD`), and system date (`SYCYMD`).
     - Reads `SA5FIZD` to accumulate total gallons purchased (`SATOTG`).
     - If ordered gallons (`ORGAL`) exceed purchased gallons (`SATOTG`), sets error indicator (`@IND=4`); otherwise, sets `@IND=5`.

7. **Quantity Conversion**:
   - Calls `MINLBGL1` (replacing `MINLBGL`, JB08) to calculate gallons or pounds directly, removing the need for a conversion factor (JB04).
   - If a `GSCTWT` record exists with gallons/container, uses it for conversion instead of requiring `GSUMCV` (JB22).

8. **Date Validation**:
   - Validates that the entered date is within one year of the last sold date (JK03):
     - Chains to `PRCNTR` to check the last sold date.
     - Converts Gregorian date (`$MDY`, `$CN`) to Julian format (`G$JD`) using subroutine `@DTE1`.
     - If the date is more than one year since the last sold date, sets error "MORE THAN 1 YEAR SINCE LAST SOLD" (COM,08).
     - Allows brand-new product/container combinations to bypass the one-year check if entered on the same day (MG04).

9. **Hazardous Material Code**:
   - Retrieves and passes back the hazardous material code (`HAZM`) to the calling program (JB17, JB18).

10. **Output and Termination**:
    - Sets indicator `LR` to end the program.
    - Returns validation results, prices, and error indicators to the calling program (`BB101`).

---

### Business Rules

The program enforces numerous business rules to ensure valid order entry data:

1. **Container/Unit of Measure Validation**:
   - Product, container, and unit of measure combinations must exist in `GSCNTR6` and `GSCTUM` (errors 01, 04).
   - Non-'LBS' units are converted to gallons using `GSUMCV` conversion factors (error 01 if invalid).
   - `GSCTWT` records with gallons/container bypass `GSUMCV` requirements (JB22).

2. **Pricing Rules**:
   - Rack prices are retrieved from `BBPRCE`, but not required for products with special sales agreements (JB10).
   - Inactive rack prices (`INACTIVE='I'`) prevent order acceptance; `INACTIVE='B'` allows shipping until stock is depleted with a warning (COM,07, JB10).
   - Sales agreement prices from `BICUAY` prioritize ship-to specific records over 'ALL' ship-tos (MG16) and must match the unit of measure (JB06).
   - Sales agreements use company, customer, location, container, bill-to PO#, and start date, with zero end date allowed (JB09).
   - Freight code is included in `BICUAY` key for same start dates (MG06).
   - Billed customer prices from `BBBCPR` are mandatory (errors 05, 06).
   - Sales agreements with order number in PO/order# field are checked first (MG05).

3. **Quantity Validation**:
   - Ordered quantities must meet minimum/maximum thresholds in `BICUAY` (`BAMNQY`, `BAMXQY`, JK01, error 03).
   - Distributor buyback quantities (`ORGAL`) must not exceed previously purchased gallons (`SATOTG`, error indicator 4).

4. **Export Pricing**:
   - For export ship-tos (`CSEXPO='Y'` in `SHIPTO`, DC01), special pricing logic applies (special '999' location removed in JB07).

5. **Date Validation**:
   - Order dates must be within one year of the last sold date in `PRCNTR` (JK03, error 08).
   - Brand-new product/container combinations entered on the same day bypass the one-year check (MG04).

6. **Customer-Owned Products**:
   - If customer-owned product flag is 'Y', uses a zero rack price (JB05).

7. **Inactive Records**:
   - Inactive records in `GSCTUM`, `GSCTWT`, and `GSUMCV` are treated as deleted (JB16).
   - Certain units of measure ('KG', 'LI', 'ML', 'OZ') do not require `GSUMCV` records (JB13, JB15).

8. **Hazardous Material**:
   - The hazardous material code (`HAZM`) is retrieved and returned to the calling program (JB17, JB18).

---

### Tables (Files) Used

The program interacts with the following files:

1. **GSCTWT**: Contract weight table (64 bytes, indexed, key length 9).
   - Used for gallons/container conversion (JB22).
2. **GSUMCV**: Customer master unit of measure conversion (64 bytes, indexed, key length 12).
   - Used for unit of measure conversions (not required for 'KG', 'LI', 'ML', 'OZ', JB13, JB15).
3. **PRCNTR**: Product container file (51 bytes, indexed, key length 9).
   - Used for last sold date validation (JK03, MG04).
4. **GSCTUM**: Customer master unit of measure (64 bytes, indexed, key length 12).
   - Retrieves IMS unit of measure for `BBORTR` and `BBORDR` (VV04).
5. **BBPRCE**: Rack price file (128 bytes, indexed, key length 27).
   - Retrieves rack prices.
6. **BBBCPR**: Billed customer price file (128 bytes, indexed, key length 32).
   - Retrieves billed customer prices.
7. **BICUAY**: Sales agreement file (256 bytes, indexed, key length 41, MG16).
   - Retrieves ship-to specific and 'ALL' sales agreement prices.
8. **INLOC**: Inventory location file (512 bytes, indexed, key length 5).
   - Validates location data.
9. **SA5FIZD**: Sales agreement history file (1024 bytes, indexed, key length 20).
   - Accumulates purchased gallons for distributor validation.
10. **GSPROD**: Product file (512 bytes, indexed, key length 6, JK04).
    - Replaces `GSTABL` for product code validation.
11. **GSCNTR6**: Container file (512 bytes, indexed, key length 3, JK02).
    - Replaces `GSTABL6` for container code validation.

---

### External Programs Called

The program calls the following external program:

1. **MINLBGL1** (JB08):
   - Called to calculate gallons or pounds directly, replacing `MINLBGL` (JB04).
   - Used for weight-to-volume conversions, improving accuracy over the previous factor-based approach.

---

### Summary

The `BB1011` RPGLE program, called from the `BB101.ocl36.txt` OCL program, is a critical component of the `BB101` order entry system. It validates container/unit of measure combinations, retrieves rack and billed customer prices, checks distributor purchase quantities, and performs conversions. Key processes include validating data against multiple files, retrieving pricing from `BBPRCE`, `BBBCPR`, and `BICUAY`, converting quantities using `MINLBGL1`, and validating dates and purchase quantities. Business rules enforce valid pricing, quantity thresholds, export-specific logic, and date restrictions, with accommodations for customer-owned products and inactive records. The program interacts with 11 files and calls `MINLBGL1` for conversions, ensuring robust order entry validation and pricing.