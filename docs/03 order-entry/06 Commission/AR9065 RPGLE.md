Here's a clear and complete breakdown of **AR9065 – Print Commission Charge Listing**.

### Program Purpose
AR9065 is a **batch-style report program** that prints a simple listing of all **Commission Charge** records from the **ARCOMM** file. It is called from the Work With program (**AR906P**) when the user presses **F15**.

### Overall Process Flow

1. **Initialization (*INZSR)**
   - Receives one parameter: `p$fgrp` (`'G'` or `'Z'`) to determine which library set to use.
   - Sets up report header line 1.
   - Turns on `prtovr` flag so the first page will print headings.
   - Calls `opntbl` to open the data file with proper overrides.

2. **Open Printer File (`openprtf`)**
   - Builds and executes an override command for the printer file **QSYSPRT** using the `ovr` compile-time array:
     - Sets page size, lines per inch, characters per inch, overflow line, output queue, hold/save options.
   - Opens the printer file `QSYSPRT`.

3. **Main Report Processing (`prtlist`)**
   - Reads **ARCOMM** sequentially (full file scan) until end-of-file (`*INLR`).
   - For every record:
     - Checks for printer overflow (`ovrflo`).
     - Writes the detail line using **EXCEPT DTL01**.
   - After all records are processed, closes the printer file.

4. **Overflow Handling (`ovrflo`)**
   - When the overflow indicator `*INOF` turns on:
     - Prints the three header lines (`HDR01`).
     - Resets the overflow flag.
   - This ensures proper page headings on every new page.

5. **Close Printer File (`closprtf`)**
   - Closes `QSYSPRT`.
   - Executes `DLTOVR FILE(QSYSPRT)` to clean up the printer override.

6. **File Overrides (`opntbl`)**
   - Applies the correct library override for **ARCOMM** (`GARCOMM` or `ZARCOMM`) based on `p$fgrp`.

### Report Layout (What Gets Printed)

The report has a professional-looking multi-line header and a clean detail line:

**Page Header (printed on every page):**
- Line 1: "American Refining Group" + "Commission Charge Listing by Co#/Cust/Loc/Prod/U/M" + Page number
- Line 2: Job name / Program name + Date
- Line 3: User + File Group (G/Z) + Time
- Separator line
- Column headings (two lines):
  - Co | Cust # | Loc | Prod | U/M | Vend# | Rate Per Qty | Pct Per Qty | Rate Per Sales $ | Pct Per Sales $ | Prod Class | Prod Grp | Inv Grp

**Detail Lines:**
One line per commission charge record showing:
- Company (CO)
- Customer Number (CUSN)
- Location (LOC)
- Product (PROD)
- Unit of Measure (UM)
- Vendor Number (VEND)
- Rate Amount (RAMT)
- Rate Percent (RPCT)
- Sales Amount (SAMT)
- Sales Percent (SPCT)
- Product Class (PCLS)
- Product Group (PGRP)
- Inventory Group (IGRP)

### Business Rules

- Prints **all records** in the ARCOMM file with no selection criteria or filters.
- Includes **both active and inactive** records (no `acdel` check).
- Reads the file in arrival sequence (physical order, not keyed).
- Uses traditional RPG **EXCEPT** output specifications (very common in older-style report programs).
- Printer settings: 68 lines per page, 164 characters wide, 8 LPI, 15 CPI.
- Spool file is created with **HOLD(*YES)** and **SAVE(*YES)** so it doesn't print automatically and can be reviewed later.
- No subtotals, grand totals, or breaks — it is a straightforward flat listing.

### Files / Tables Used

| File       | Usage              | Description |
|------------|--------------------|-----------|
| **ARCOMM** | Input only (IF E K) | Commission Charge master file – the only data file read |
| **QSYSPRT**| Output (Printer)   | Standard IBM i printer file for the report |

### External Programs Called

| Program    | Called When      | Purpose |
|------------|------------------|-------|
| **QCMDEXC** | Multiple times   | Execute overrides for data file and printer file |

No other RPG programs are called.

### Compile-Time Data Arrays

- **`ovg`** / **`ovz`**: Overrides for ARCOMM (G vs Z library)
- **`ovr`**: Printer overrides (OVRPRTF + DLTOVR)
- **`hdr`**: Report title

### Summary – Simple Report Program

AR9065 is a **classic, no-frills RPG report program**:
- Reads the entire ARCOMM file sequentially.
- Prints a clean columnar listing with proper page headings.
- Uses standard printer overrides and overflow handling.
- Designed to be called directly from the Work With screen (F15).

It complements the online programs (AR906P, AR906, AR9064) by providing a printable view of all commission charge entries.

---

Would you like me to summarize the **complete commission charge system flow** (how AR906P → AR906 → AR9064 → AR9065 work together)?