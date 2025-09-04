The RPG program `BI9078.rpgle` is part of the Brandford Order Entry / Invoices system and is designed for copying customer and ship-to product description records from one customer/ship-to combination to another within the `ARCUPR` file. It is likely called from the main OCL program `BI890.ocl36.txt`, as it integrates with the ship-to master maintenance workflow (`BI890`, `BI890P`, `BI8903`, `GB730P`, `BB800E`, `BI907AC`, `BI907`). The program generates a printed report of the copied records using the `QSYSPRT` printer file. Due to the truncation of the source code (5,279 characters omitted), some logic is inferred from the provided declarations, partial code, and context from related programs. Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### **Process Steps of the RPG Program**

The `BI9078` program is a batch-oriented RPGLE application that copies records from the `ARCUPR` file (customer product descriptions) based on input parameters and produces a printed report of the copy operation. It does not use a display file or subfile, unlike other programs in the workflow. Here’s a step-by-step breakdown of the process based on the provided code and context:

1. **Initialization (`*inzsr` Subroutine)**:
   - **Parameters**: Receives six input parameters via `*entry`:
     - `p$cono` (2 digits): Company number.
     - `p$cust` (6 digits): Target customer number (to copy to).
     - `p$ship` (3 digits): Target ship-to number (to copy to).
     - `p$cpyfcst` (6 digits): Source customer number (to copy from).
     - `p$cpyfshp` (3 digits): Source ship-to number (to copy from).
     - `p$fgrp` (1 character): File group (`G` or `Z`) for file overrides.
   - **Test Data**: Includes commented-out test values (e.g., `p$cono=10`, `p$cust=000095`, `p$ship=901`, `p$cpyfcst=000099`, `p$cpyfshp=901`, `p$fgrp='Z'`) for debugging.
   - **Header Setup**: Sets `c$hdr1` to "Copy Customer Product Descriptions" (`hdr(01)`).
   - **Date/Time**: Captures the current date and time (`time` to `t#time`, mapped to `t#cymd` for timestamping records).
   - **Key Lists**:
     - `klship`: `p$cono`, `p$cpyfcst`, `p$cpyfshp` (for source records in `arcup3`).
     - `klshipcpyf`: `p$cono`, `p$cust`, `p$ship`, `cpyf_cpprod`, `cpyf_cpcnty` (for target records in `arcupr`).
   - **Print Control**: Initializes `prtovr='1'` to control printer output and `str` array to `' * '` for report formatting (likely separator lines).

2. **Open Files (`openprtf` and `opntbl` Subroutines)**:
   - **Printer File**:
     - Opens `qsysprt` (printer file, 184 characters wide) with `USROPN` and overflow indicator `*inof`.
     - Applies overrides from `ovr` array using `QCMDEXC` (implied):
       - `OVRPRTF`: Sets page size (68 lines, 184 characters), lines per inch (8), characters per inch (15), overflow at line 62, output queue (`*JOB`), hold (`*YES`), save (`*YES`).
   - **Database Files**:
     - Applies overrides from `ovg` (for `G` group) or `ovz` (for `Z` group) arrays using `QCMDEXC` (implied) for three files: `arcup3`, `arcupr`, `arcuphs`. Redirects to `g*` (e.g., `garcup3`, `garcupr`) or `z*` (e.g., `zarcup3`, `zarcupr`) files based on `p$fgrp`.
     - Opens files in user-controlled mode (`USROPN`):
       - `arcup3` (input, read-only, renamed to `arcuprlf` with prefix `cpyf_`): Source customer product file.
       - `arcupr` (update/add): Target customer product file for copying records.
       - `arcuphs` (output): History file to log copied records.

3. **Copy Records (`WriteArcupr` Subroutine)**:
   - **Read Source Records**: Reads `arcup3` (source file) using `klship` (`p$cono`, `p$cpyfcst`, `p$cpyfshp`) to retrieve product records for the source customer/ship-to.
   - **Copy to Target**: For each source record:
     - Maps fields from `arcup3` (`cpyf_` prefix, e.g., `cpyf_cpprod`, `cpyf_cpcnty`, `cpyf_cpcpds`) to `arcupr` (`cpyt_` prefix).
     - Checks if a record exists in `arcupr` using `klshipcpyf` (`p$cono`, `p$cust`, `p$ship`, `cpyf_cpprod`, `cpyf_cpcnty`).
     - If no record exists, writes a new record to `arcupr` with the target customer/ship-to (`p$cust`, `p$ship`), setting `newrcd='1'`.
     - If a record exists, updates it or skips it (logic not shown in truncated code).
     - Logs the copy operation to `arcuphs` for history tracking.
   - **Report Output**: Writes to `qsysprt` using format `dtl01` for each copied record, including:
     - `cpyt_cpcono` (company), `cpyt_cpcust` (customer), `cpyt_cpship` (ship-to), `cpyt_cpprod` (product), `cpyt_cpcnty` (container type), `cpyt_cpcpds` (customer product description), `newrcd` (new record flag).
     - Header (`hdr01`) includes company name ("American Refining Group"), job details (`ps#jobn`, `ps#pgm`, `ps#usr`), date (`t#mdcy`), time (`t#hms`), file group (`p$fgrp`), and column headers.

4. **Program Termination**:
   - Closes all files (`close *all`).
   - Sets `*inlr` to `*on` and returns to the caller (likely the OCL).

---

### **Business Rules**

The program enforces the following business rules, based on the code and context:
1. **Copy Operation**:
   - Copies product description records from a source customer/ship-to (`p$cpyfcst`, `p$cpyfshp`) to a target customer/ship-to (`p$cust`, `p$ship`) within the same company (`p$cono`).
   - Only copies records that don’t already exist in the target `arcupr` file, marking new records with `newrcd='1'`.
   - Logs all copy operations to `arcuphs` for history tracking.

2. **File Group Handling**:
   - Supports `G` or `Z` file groups via `p$fgrp`, redirecting file access to `g*` or `z*` files (e.g., `garcup3` or `zarcup3`).

3. **Report Generation**:
   - Produces a printed report (`qsysprt`) with details of copied records, including company, customer, ship-to, product code, container type, product description, and new record flag.
   - Report is held (`HOLD=*YES`) and saved (`SAVE=*YES`) in the job’s output queue for later review.
   - Includes job, user, date, time, and file group information in the header.

4. **Data Integrity**:
   - Ensures copied records are written to `arcupr` only if they don’t exist, preventing duplicates.
   - Maintains history in `arcuphs` for auditing.

5. **Validation** (Inferred):
   - Likely validates source (`p$cpyfcst`, `p$cpyfshp`) and target (`p$cust`, `p$ship`) customer/ship-to combinations against files like `arcust` or `shipto` (not declared in `BI9078` but common in the system, e.g., `BI907`).

---

### **Tables (Files) Used**

The program uses the following files, as declared in the RPGLE code and aligned with the OCL (`BI890.ocl36.txt`):
1. **qsysprt** (Printer File, `O`):
   - Printer file (184 characters wide) for generating the copy report, with formats `hdr01` (header) and `dtl01` (detail).
2. **arcup3** (Input, `IF`, `garcup3` or `zarcup3`, renamed to `arcuprlf`, prefix `cpyf_`):
   - Source customer product file, used to read product records for copying.
3. **arcupr** (Update/Add, `UF A`, `garcupr` or `zarcupr`, prefix `cpyt_`):
   - Target customer product file, where copied records are written or updated.
4. **arcuphs** (Output, `O`, `garcuphs` or `zarcuphs`):
   - Customer product history file, used to log copy operations.

The OCL declares additional files (e.g., `arcust`, `shipto`, `gsprod`, `gstabl`, `bicon`, `cuadr`, `bbshsa1`, `trrtcd`, `shipths`), but only the above are explicitly used in `BI9078`.

---

### **External Programs Called**

The program likely calls the following external program, inferred from the code:
1. **QCMDEXC** (Implied):
   - Used in the `openprtf` and `opntbl` subroutines to execute file override commands (`ovg`, `ovz`, `ovr`) for redirecting file access (`arcup3`, `arcupr`, `arcuphs`) and configuring the printer file (`qsysprt`).

No other programs are directly called by `BI9078`. The OCL references `BI907AC`, `BI907`, `BI9002`, and `BB800E`, but `BI9078` is likely invoked independently for the copy function, possibly related to `BI8903` (ship-to copy program).

---

### **Summary**

The `BI9078` program is a batch RPGLE application for copying customer and ship-to product description records, likely called from the OCL (`BI890.ocl36.txt`). It:
- Copies records from `arcup3` (source customer/ship-to) to `arcupr` (target customer/ship-to), logging changes to `arcuphs`.
- Generates a printed report (`qsysprt`) detailing the copied records.
- Supports `G` or `Z` file groups via `p$fgrp` and enforces rules for non-duplicate record copying and history tracking.
- Uses parameters for company, source/target customer, and ship-to numbers.

**Tables Used**:
- `qsysprt` (printer file for reporting)
- `arcup3` (`garcup3` or `zarcup3`, source file)
- `arcupr` (`garcupr` or `zarcupr`, target file)
- `arcuphs` (`garcuphs` or `zarcuphs`, history file)

**External Programs Called**:
- `QCMDEXC` (implied, for file and printer overrides)

Due to the truncation, some subroutines (e.g., full `WriteArcupr` logic) are not fully visible. If you have the complete code or need further analysis of its integration with the system (e.g., relation to `BI8903` for ship-to copying), let me know!