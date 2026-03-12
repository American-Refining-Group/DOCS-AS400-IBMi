**AP820 – Checks by Vendor Inquiry (RPG36 program)**  
This is the **payment history / check inquiry screen** that is automatically launched from the main A/P inquiry (AP800) when the user presses the “History” option (or similar).

### Purpose in plain business terms
Shows **all checks ever paid to a specific vendor**, in **check-number order** (newest first), with:
- Check date, check number, gross amount, discount taken, net paid
- Bank G/L account
- Cleared / Void / Open status from the check register
- Running totals per check
- Ability to roll up/down and select a check to drill into voucher detail (AP825)

### How it is invoked
From AP800.ocl36 → when the user requests history, the OCL changes the LDA to **AP820** and chains to this program.  
The calling program (AP800) puts the selected **Company (ICO)** and **Vendor (IVEND)** into the LDA positions 143-149 before returning.

### Files (Tables) Used

| File       | Logical/Physical | Purpose                                                                 | Key Used                          |
|------------|------------------|-------------------------------------------------------------------------|-----------------------------------|
| **APVEND**   | Physical         | Vendor master – pulls name, address, current balance, last payment      | Vendor number                     |
| **APHSTHB**  | Logical (new 2015) | A/P History Header by Vendor & Check – main file being read            | OHK1 (Vendor) + OHKCHK (Check #)  |
| **APHSTVA**  | Logical          | Used only for one-time vendor name lookup when check has no vendor name| Key built from company + bank + check |
| **APCHKR**   | Physical         | Check Register – to determine if a check has Cleared, been Voided, etc. | Company + Bank G/L + Check number |
| **SCREEN**   | WORKSTN          | Display file (AP820S1, S2, S3, S4 formats)                              | —                                 |

### External Programs Called / Chained To

| Program   | When Called                                 | Purpose                                      |
|-----------|---------------------------------------------|----------------------------------------------|
| **AP800**   | F12 or Exit with no check selected          | Return to main vendor inquiry (alpha search) |
| **AP825**   | User positions cursor and presses Enter on a check | Voucher Detail Inquiry for the selected check |

These are written into the LDA field **NXPROG** (positions 137-142) and the OCL that launched AP820 will chain to whatever is in there when the program ends.

### Major Process Flow & Business Rules (step-by-step)

1. **First-time entry (indicator 09)**
   - Reads LDA to get Company (ICO) and Vendor (IVEND) passed from AP800
   - If not coming from AP800 → defaults company = 01 and forces rekey

2. **Screen 1 – Vendor header (AP820S1)**
   - Chains to APVEND to validate the vendor and pull name/address
   - If invalid → error message “INVALID VENDOR”
   - Sets limit key = vendor number and does **SETLL** on APHSTHB

3. **Load the subfile (AP820S2 / S3 / S4)**
   - Reads **APHSTHB** forward (newest checks first) using **READ**
   - Skips records where:
     - OHDEL = ‘D’ (deleted)
     - OHKCNL = ‘C’ (cancelled/voided check – still skipped)
     - Vendor key (OHK1) does not match the selected vendor  ← **JB03 bug fix 2024**
     - Check date is in the future beyond today’s date (prevents reading to EOF on bad date entry)
   - For each valid record calculates **Amount Due this check** = Gross – Partial Paid + Last Payment Amount
   - When check number changes → writes previous check total line (indicator 21/22)

4. **Check Register lookup (APCHKR)**
   - Builds key = Company + Bank G/L + Check number
   - Chains to APCHKR to get clear date and status (D/O/R/V)
   - Sets display field:
     - Cleared → “CLEARED” + date
     - Voided → “ VOIDED”
     - Open → “    OPEN”

5. **One-time vendor name (indicator 25 & 85)**
   - Some old checks have blank vendor but valid bank/check
   - Program builds a dummy key (‘3001’ + company + bank + check) and chains to **APHSTVA** to pull the one-time vendor name → displays in a special subfile line

6. **Rolling (PgUp / PgDn)**
   - PgDn → continues reading forward from where it left off
   - PgUp → uses **READP** (read previous) starting from the first key of the current page (saved in SVBEG) and rebuilds the subfile backwards

7. **Exit / Return logic**
   - F12 or no check selected → NXPROG = ‘AP800 ’ → back to vendor search
   - Enter on a line → puts Company, Vendor, Check# into LDA and sets NXPROG = ‘AP825 ’ → drill to voucher detail

### Key Indicators Used

| Ind | Meaning                                   |
|-----|-------------------------------------------|
| 01  | First screen (vendor header)              |
| 02  | Subfile display                           |
| 09  | First time in program                     |
| 18/19 | Roll forward / backward                  |
| 20  | New check number encountered             |
| 21/22 | Print check total line                    |
| 23  | Subfile full (21 lines)                   |
| 25  | One-time vendor line                      |
| 80/81 | Exit conditions                           |
| 82  | Subfile load in progress                  |
| 83/84/85 | Control output of various subfile formats |

### Summary – What the user sees and can do
1. Enter or arrive with a vendor number → see vendor name, address, current balance
2. Subfile shows every check ever written to that vendor (most recent first)
3. Each check line shows date, check #, gross, discount, net, bank, status
4. Roll up/down through hundreds or thousands of checks
5. Position cursor and press Enter → go to AP825 to see the actual invoices paid by that check
6. F12 → back to main A/P inquiry

Classic 5250 green-screen A/P check history inquiry from a 1990s-era third-party or in-house AS/400 accounts payable package. Still rock-solid and widely used in many companies running IBM i today.