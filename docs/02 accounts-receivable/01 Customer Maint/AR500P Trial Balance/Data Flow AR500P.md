Here is a **complete cross-reference matrix** showing every program in the AR500 suite (including the ones you provided) and exactly how each program touches every physical file / table.

| Program   | ARDETL               | ARCUST               | ARCONT | GSTABL | ARCWRK | ARCUSP | SA5SHX | ?9?ARC500 | ?9?AR500X | ?9?AR500D / ?9?AR500C | ?9?ARATBS | PRINT (spooled) | Side Effects / Notes |
|-----------|----------------------|----------------------|--------|--------|--------|--------|--------|-----------|-----------|-----------------------|-----------|-----------------|----------------------|
| **AR500P** | –                    | Read (validation)    | Read + **Write** (updates ACDATE) | Read   | –      | –      | –      | –         | –         | –                     | –         | –               | Updates last ageing date in every ARCONT |
| **AR390**  | **Update** (writes ADAGE + bucket moves) | **Update** (writes AGE(1–5) packed buckets + high-balance) | Read   | Read   | –      | –      | –      | –         | –         | –                     | –         | –               | **Only program that permanently changes live A/R data** |
| **AR5004** | –                    | Read (primary cycle) | –      | –      | Read   | –      | –      | **Create + Write** (385-byte extract) | –         | –                     | –         | –               | Builds filtered customer work file |
| **AR5007** | Read (primary)       | –                    | –      | –      | –      | –      | –      | **Update** (writes NOD flag to pos 385) + Read | **Create + Write** (138-byte enriched detail) | –                     | –         | –               | Injects NOD + sort keys |
| **#GSORT** | –                    | –                    | –      | –      | –      | –      | –      | Read      | Read      | **Create + Write** (sorted versions) | –         | –               | Two sorts: detail → AR500D, master → AR500C |
| **AR750**  | –                    | Read (AR500C)        | Read   | Read   | –      | Read   | –      | Read (AR500C) | Read (AR500D) | –                     | –         | **Write**       | By Customer # – current production layout |
| **AR502**  | –                    | Read (AR500C)        | Read   | Read   | –      | –      | –      | Read (AR500C) | Read (AR500D) | –                     | –         | **Write**       | By Name (alpha) |
| **AR501**  | –                    | Read (AR500C)        | Read   | Read   | –      | –      | Read   | Read (AR500C) | Read (AR500D) | –                     | **Write** | **Write**       | By Salesman + optional Excel export |

### Legend
- **Read** = opened for input only  
- **Update** = opened for update (records modified)  
- **Create + Write** = file is cleared/re-created and written to  
- **Write** = spooled output or work file output  
- – = not touched

### Key Takeaways
- Only **AR390** permanently modifies live production data (ARDETL + ARCUST)
- **AR500P** is the only other program that writes to a live file (ARCONT.ACDATE)
- All other programs are **read-only** on live files and only write to temporary work files or spooled output
- The **?9?** files are all temporary and automatically deleted at job end

This matrix is extremely useful for impact analysis, modernisation planning, and regression testing.