The RPG program `BI9444.rpg` is an RPG/400 program invoked by the OCL program `BI944.ocl36.txt` to filter records from the input file `BICUAGNW` based on user-specified selection criteria (e.g., locations, product classes, products, and units of measure) and produce a filtered output file `BICUAGN2`. It works in conjunction with a prior sort operation (`#GSORT`) to exclude or include records as needed for the Customer Sales Agreement Master File Listing. Below, I’ll explain the process steps, business rules, tables used, and any external programs called.

### Process Steps of the RPG Program

The program reads records from `BICUAGNW`, applies selection criteria stored in the User Data Structure (UDS) from the Local Data Area (LDA), and writes valid records to `BICUAGN2`. The processing involves checking locations, product classes, products, and units of measure against user inputs, with subroutines to handle each validation.

1. **File and Data Structure Initialization** (Lines 0004–0025):
   - **Input File**:
     - `BICUAGNW` (Primary Input, 263 bytes, disk): Contains sales agreement records with fields like company, customer, location, product codes, dates, and additional data (division, product class, salesman) added by `BI9443`.
   - **Output File**:
     - `BICUAGN2` (Output, 263 bytes, disk): Receives filtered records that pass the selection criteria.
   - **Arrays**:
     - `LOC` (5 elements, 3 bytes each): Stores location codes (`KYLOC1`–`KYLOC5`).
     - `CLS` (10 elements, 3 bytes each): Stores product class codes (`KYPC01`–`KYPC10`).
     - `PRD` (10 elements, 4 bytes each): Stores product codes (`KYPD01`–`KYPD10`).
     - `PROD` (10 elements, 4 bytes each): Stores product codes from `BICUAGNW` (`BAPR01`–`BAPR10`).
   - **Input Specifications for `BICUAGNW`**:
     - Fields include `BACONO` (company, 2–3), `BACOCU` (company/customer, 2–9), `BACUST` (customer number, 4–9), `BALOC` (location, 10–12), `BAPR01`–`BAPR10` (product codes, 13–52), `BASTDT` (start date, 53–59), `BASTTM` (start time, 60–63), `BAENDT` (end date, 64–70), `BAENTM` (end time, 71–74), `BAPRCE` (price, 76–80, packed), `BAOFFP` (off-price, 82–86, packed), `BAMNGL` (minimum gallons/month, 87–93), `BAMXGL` (maximum gallons/month, 94–100), `BAPPD` (prepaid sale, 101), `BAALSH` (apply to all ship-to, 102), `BAPRIM` (pricing method, 106), `BASTD8` (start date CYMD, 118–125), `BAEND8` (end date CYMD, 126–133), `BADELV` (delivery flag, 134), `BAFRCD` (freight code, 135), `BACRDT` (creation date, 136–143), `BACRTM` (creation time, 144–149), `BALUDT` (last update date, 150–157), `BALUTM` (last update time, 158–163), `BACNTR` (container code, 164–166), `BASHIP` (ship-to number, 167–169), `BACNT#` (contract number, 170–180), `BAUNMS` (unit of measure, 181–183), `BADIV` (division, 258), `BAPRCL` (product class, 259–261), `BASLS#` (salesman, 262–263), `REC1` (1–256), `REC2` (257–263).
   - **User Data Structure (UDS)**:
     - Defines selection criteria from the LDA: `KYDIV` (division), `KYLOSL` (location selection), `KYLOC1`–`KYLOC5` (locations), `KYPCSL` (product class selection), `KYPC01`–`KYPC10` (product classes), `KYCSSL` (customer selection), `KYCS01`–`KYCS20` (customers), `KYPDSL` (product selection), `KYPD01`–`KYPD10` (products), `KYSMSL` (salesman selection), `KYFRSM`–`KYTOSM` (salesman range), `KYJOBQ` (job queue flag), `KYCOPY` (copy count), `KYDVNO` (division number), `KYDAT8` (system date), `KYUMSL` (unit of measure selection), `KYUMYP` (unit of measure type), `KYUM01` (unit of measure), `Y2KCEN` (century), `Y2KCMP` (comparison year).

2. **Main Processing Loop** (Lines 0040–C-specs):
   - **Record Processing** (Indicator 01, primary file `BICUAGNW`):
     - Reads each record from `BICUAGNW` sequentially.
     - Initializes indicators 50 and 51 to off (`SETOF 50 51`) and sets indicator 52 on (`SETON 52`) to assume the record is valid unless proven otherwise.
   - **Location Validation (`LOCSR` Subroutine)**:
     - If `KYLOSL = 'SEL'`, calls `LOCSR` to check if `BALOC` matches any of `KYLOC1`–`KYLOC5` in the `LOC` array.
     - Loops through 5 locations (`X = 1 to 5`).
     - If a match is found, sets indicator 50 on (valid record).
     - If no match (`N50`), sets indicator 51 on (invalid record) and branches to `END`.
   - **Product Class Validation (`PRCLSR` Subroutine)**:
     - If `KYPCSL = 'SEL'`, calls `PRCLSR` to check if `BAPRCL` matches any of `KYPC01`–`KYPC10` in the `CLS` array.
     - Loops through 10 product classes (`X = 1 to 10`).
     - If a match is found, sets indicator 50 on.
     - If no match (`N50`), sets indicator 51 on and branches to `END`.
   - **Product Validation (`PRODSR` Subroutine)**:
     - If `KYPDSL = 'SEL'`, calls `PRODSR` to check if any of `BAPR01`–`BAPR10` in the `PROD` array matches any of `KYPD01`–`KYPD10` in the `PRD` array.
     - Nested loops through 10 products (`X = 1 to 10`, `Y = 1 to 10`).
     - If a match is found for non-blank `PROD,Y`, sets indicator 50 on.
     - If no match (`N50`), sets indicator 51 on and branches to `END`.
   - **Unit of Measure Validation (`UOMSR` Subroutine)**:
     - If `KYUMSL = 'SEL'`, calls `UOMSR` to check `BAUNMS` against `KYUM01` based on `KYUMYP`:
       - If `KYUMYP = 'INCLUDE'`, `BAUNMS` must equal `KYUM01` to set indicator 50 on.
       - If `KYUMYP = 'EXCLUDE'`, `BAUNMS` must not equal `KYUM01` to set indicator 50 on.
     - If no match (`N50`), sets indicator 51 on and branches to `END`.
   - **Output Decision**:
     - If indicator 50 (valid record) or 52 (no selection criteria applied) is on, writes the record to `BICUAGN2` using the `GOOD` format (`EXCPT GOOD`).
     - The output includes `REC1` (1–256) and `REC2` (257–263) from the input record.
   - **End Processing**:
     - Branches to `END` if indicator 51 is on (invalid record) or after writing a valid record.
     - Continues reading `BICUAGNW` until end-of-file.

### Business Rules

The program enforces the following business rules to filter records:
1. **Location Selection (`KYLOSL`)**:
   - If `KYLOSL = 'SEL'`, the record’s location (`BALOC`) must match one of the user-specified locations (`KYLOC1`–`KYLOC5`) in the `LOC` array.
   - If no match, the record is excluded (indicator 51 on).
2. **Product Class Selection (`KYPCSL`)**:
   - If `KYPCSL = 'SEL'`, the record’s product class (`BAPRCL`) must match one of the user-specified product classes (`KYPC01`–`KYPC10`) in the `CLS` array.
   - If no match, the record is excluded.
3. **Product Selection (`KYPDSL`)**:
   - If `KYPDSL = 'SEL'`, at least one of the record’s product codes (`BAPR01`–`BAPR10`) must match one of the user-specified product codes (`KYPD01`–`KYPD10`) in the `PRD` array.
   - If no match, the record is excluded.
4. **Unit of Measure Selection (`KYUMSL`)**:
   - If `KYUMSL = 'SEL'`:
     - For `KYUMYP = 'INCLUDE'`, the record’s unit of measure (`BAUNMS`) must equal `KYUM01`.
     - For `KYUMYP = 'EXCLUDE'`, the record’s unit of measure must not equal `KYUM01`.
   - If the condition is not met, the record is excluded.
5. **Default Inclusion**:
   - If no selection criteria are applied (`KYLOSL`, `KYPCSL`, `KYPDSL`, `KYUMSL` not `'SEL'`) or if a record passes all applicable criteria, it is written to `BICUAGN2`.
6. **Record Exclusion**:
   - A record is excluded (not written) if it fails any of the above selection checks (indicator 51 on).

### Tables (Files) Used

The program uses the following files, as defined in the File Specification (F-spec) and Input Specification (I-spec):
1. **BICUAGNW** (Primary Input, 263 bytes):
   - Fields: `BACONO`, `BACOCU`, `BACUST`, `BALOC`, `BAPR01`–`BAPR10`, `BASTDT`, `BASTTM`, `BAENDT`, `BAENTM`, `BAPRCE`, `BAOFFP`, `BAMNGL`, `BAMXGL`, `BAPPD`, `BAALSH`, `BAPRIM`, `BASTD8`, `BAEND8`, `BADELV`, `BAFRCD`, `BACRDT`, `BACRTM`, `BALUDT`, `BALUTM`, `BACNTR`, `BASHIP`, `BACNT#`, `BAUNMS`, `BADIV`, `BAPRCL`, `BASLS#`, `REC1`, `REC2`.
   - Usage: Primary input file containing sales agreement records with added fields from `BI9443`.
2. **BICUAGN2** (Output, 263 bytes):
   - Fields: `REC1` (1–256), `REC2` (257–263).
   - Usage: Output file containing filtered records that pass selection criteria.

### External Programs Called

The RPG program `BI9444` does not explicitly call any external programs via `CALL` operations. It is invoked by the OCL program `BI944.ocl36.txt` and processes data independently, relying on the input from `BICUAGNW` (produced by `BI9443` and sorted by `#GSORT`) and writing to `BICUAGN2` for further sorting and processing by `BI944` in the OCL workflow.

### Summary

- **Process Steps**:
  1. Read each record from `BICUAGNW`.
  2. Initialize indicators (50 and 51 off, 52 on).
  3. If `KYLOSL = 'SEL'`, validate `BALOC` against `KYLOC1`–`KYLOC5` in `LOCSR`.
  4. If `KYPCSL = 'SEL'`, validate `BAPRCL` against `KYPC01`–`KYPC10` in `PRCLSR`.
  5. If `KYPDSL = 'SEL'`, validate `BAPR01`–`BAPR10` against `KYPD01`–`KYPD10` in `PRODSR`.
  6. If `KYUMSL = 'SEL'`, validate `BAUNMS` against `KYUM01` based on `KYUMYP` (`INCLUDE` or `EXCLUDE`) in `UOMSR`.
  7. Write record to `BICUAGN2` if valid (indicator 50 or 52 on) using `GOOD` format.
  8. Skip invalid records (indicator 51 on) and continue until end-of-file.

- **Business Rules**:
  - Filter records based on user-specified locations, product classes, products, and units of measure.
  - For `KYLOSL = 'SEL'`, `BALOC` must match a specified location.
  - For `KYPCSL = 'SEL'`, `BAPRCL` must match a specified product class.
  - For `KYPDSL = 'SEL'`, at least one product code must match a specified product.
  - For `KYUMSL = 'SEL'`, `BAUNMS` must match (or not match) `KYUM01` based on `KYUMYP`.
  - Include records that pass all criteria or when no specific selection is applied.

- **Tables Used**:
  - `BICUAGNW` (primary input)
  - `BICUAGN2` (output)

- **External Programs Called**:
  - None directly called within the RPG program. It is part of the OCL workflow involving `BI9443`, `#GSORT`, and `BI944`.

This program acts as a filtering step, ensuring only records meeting user-specified criteria are passed to the next stage of the Customer Sales Agreement Master File Listing process. If you need further analysis of `BI944` or other components, please provide their source code or additional details.