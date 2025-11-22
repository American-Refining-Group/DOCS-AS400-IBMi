**AR500.ocl36.txt – The actual BATCH Aged Trial Balance execution procedure**  
This is the **real workhorse** OCL that runs after the interactive prompt program **AR500P** has validated everything and returned control.  
It is the classic 1990s–early 2000s multi-company, multi-criteria, multi-sort aged trial balance batch run used in hundreds of third-party A/R packages on AS/400 (MAPICS, JDE World early, PRMS, JBA, etc.).

### Overall Process Flow (Step-by-Step)

| Step | Action | Purpose |
|------|------|---------|
| 1 | `// GSY2K` | Sets library list and company-specific data library based on ?9? (company suffix) |
| 2 | Delete/clean old work files | `GSDELETE ARC500, AR500X` etc. (ensures clean start) |
| 3 | If “Summary only” flag (pos 152 = S) → clear previous summary file | `CLRPFM ?9?ARATBS` |
| 4 | **Build the LDA (Local Data Area) sort control block** | Uses `LOCAL OFFSET-n,DATA-xxx` to build a 40+ character control string that the old #GSORT program reads to decide major/minor keys. This is how they avoided passing hundreds of parameters. |
| 5 | **AR390** – Pre-ageing preparation | Reads every open item in ARDETL, calculates ageing buckets as of the user’s date, writes aged detail to a temporary file. This is the **heart of the ageing calculation**. |
| 6 | **AR5004** – Build temporary customer master extract | Copies selected customers from ARCUST → ?9?ARC500 (work customer master) applying company and customer class filters |
| 7 | **AR5007** – Build temporary A/R detail extract | Reads the aged detail from AR390, joins with ARC500, applies all selection criteria (outstanding only, NOD=Y, etc.), writes to ?9?AR500X |
| 8 | **#GSORT** – Sort the detail file AR500X → AR500D | Three possible paths depending on report sequence (pos 152):<br>• C = by Customer number<br>• N = by Customer name<br>• S = by Salesman |
| 9 | **#GSORT** again – Sort the customer master ARC500 → AR500C | Uses a dynamic sort key built in step 4 that changes depending on C/N/S sequence |
| 10 | **Final print program** | Loads one of four possible report programs depending on sequence:<br>• Customer sequence → AR750 (detail) or AR502 (summary?)<br>• Name sequence → AR502<br>• Salesman sequence → AR501 |
| 11 | Overrides printer/output queue if testing (switch 6) |  |
| 12 | `RUN` → prints the actual Aged Trial Balance report(s) |  |
| 13 | Ends with switches reset to normal |  |

### Business Rules Implemented in This OCL

| Parameter Position | Meaning | How It Changes Behaviour |
|---------------------|-------|--------------------------|
| 111–113 | Company selection (ALL/CO) | Controls which company numbers are used in filters |
| 114–119 | Up to 3 company numbers | Used in every sort INCLUDE condition |
| 130–132 | Customer class selection (ALL/SEL) | Changes sort LDA and INCLUDE logic |
| 133–150 | 3 customer class ranges | Used in every sort block |
| 151 | Outstanding invoices only (O) | Changes LDA offset 31: IAP vs I*P |
| 152 | Report sequence | C = Customer, N = Name, S = Salesman → completely different sort and print programs |
| 154–156 | Salesman selection (ALL/SEL) | Only allowed when sequence = S |
| 157–160 | Salesman from/to | Used in salesman sort |
| 161 | NOD = Y (exclude detail lines?) | Adds “NOD=Y” filter in every sort block |
| 162 | Salesman copies = Y | Sets switch bit for multiple copies per salesman |
| 163 | ? (rarely used) | Additional filter |
| 123 | D = Detail, S = Summary? | Changes final switch for detail vs summary layout |

### Physical Files / Tables Used

| File Name | Description | Role |
|----------|-------------|------|
| ?9?ARDETL | A/R Open Item Detail (transactions) | Source of all invoices/credits |
| ?9?ARCUST | Customer Master | Source for customer name, salesman, credit limit, etc. |
| ?9?ARCONT | A/R Control (one record per company) | Company name, ageing periods, etc. |
| ?9?GSTABL | General tables (ageing buckets, messages) | Used by AR390 |
| ?9?ARC500 | Work customer master (extract) | Built by AR5004 |
| ?9?AR500X | Raw aged detail extract | Built by AR5007 |
| ?9?AR500D | Sorted aged detail | Output of first #GSORT |
| ?9?AR500C | Sorted customer master extract | Output of second #GSORT |
| ?9?ARCUSP | Customer ship-to / parent file | Probably used for consolidated ageing |
| ?9?SA5SHX | Salesman master (index?) | Used when sorting by salesman |
| ?9?SHIPTO | Ship-to addresses | Optional |
| ?9?ARATBS | Summary ageing table | Only used/cleared in Summary mode |

### External Programs Called

| Program | Purpose | When Called |
|---------|--------|-------------|
| **GSY2K** | Company/library setup | First line |
| **AR390** | Calculates ageing buckets for every open item as of user date | Core ageing engine |
| **AR5004** | Builds temporary customer extract (ARC500) | Applies company & class filters |
| **AR5007** | Builds temporary transaction extract (AR500X) with selections | Applies outstanding, NOD, etc. |
| **#GSORT** (twice) | System sort utility | Sorts detail and master files |
| **AR750** | Print Aged Trial Balance – Customer sequence (detail) | When pos 152 = C |
| **AR502** | Print Aged Trial Balance – Name sequence or Summary | When pos 152 = N |
| **AR501** | Print Aged Trial Balance – Salesman sequence | When pos 152 = S |
| **AR500** (commented) | Old version, probably replaced by AR750 | — |

### Summary – What This Whole Chain Does

```
Interactive Prompt (AR500P)
        ↓ (returns validated parameters)
Batch OCL Wrapper (AR500P.ocl36)
        ↓
This OCL (AR500.ocl36) → 
   AR390 → ages every transaction
   AR5004 → extracts selected customers
   AR5007 → extracts selected aged transactions
   #GSORT ×2 → sorts everything correctly
   AR750/AR501/AR502 → prints final report(s)
```

This is a **perfect example** of a classic 1990s AS/400 third-party accounting package batch report architecture:  
- One interactive prompt program  
- One OCL driver that orchestrates everything  
- Multiple small, single-purpose RPG programs  
- Heavy use of #GSORT with INCLUDE logic in the sort specs  
- Temporary work files in the company library (?9? prefix)

Even today (2025) many companies still run this exact flow nightly or month-end without modification.