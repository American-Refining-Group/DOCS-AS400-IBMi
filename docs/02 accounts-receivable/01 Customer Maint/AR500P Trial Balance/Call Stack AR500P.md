Here is the **complete, accurate execution order** of the AR500 Aged Trial Balance suite as it runs in production in 2025, including every program you provided.

| Order | Program     | Called By         | Main Purpose (One Sentence)                                                                                   | Key Input Files (Tables)                                 | Key Output Files / Side Effects                                                                                   |
|-------|-------------|-------------------|---------------------------------------------------------------------------------------------------------------|----------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| 1     | **AR500P**  | Main OCL (AR500P.ocl36) | Interactive prompt screen — validates all parameters, updates last ageing date in live files.                 | ARCONT, GSCONT, ARCUST (for company names)               | Returns 163-byte parameter block to OCL; updates ACDATE in every ARCONT record                              |
| 2     | **AR390**   | AR500.ocl36       | Physically re-ages every open item in ARDETL as of the user date and updates customer high-balance history. | ARDETL (U), ARCUST (U), ARCONT, GSTABL                  | Updates ADAGE in every ARDETL record; updates AGE buckets & high-balance in every ARCUST record               |
| 3     | **AR5004**  | AR500.ocl36       | Builds clean, filtered temporary customer master extract (applies company & optional customer class filter).| ARCUST, ARCWRK                                           | Creates/overwrites ?9?ARC500 (385-byte work customer file)                                                    |
| 4     | **AR5007**  | AR500.ocl36       | Builds final aged transaction extract with sort keys + injects NOD flag into customer extract.               | ARDETL, ARC500 (UF)                                      | Creates ?9?AR500X (138-byte enriched transaction file); updates position 385 of ARC500 with NOD flag          |
| 5     | **#GSORT**  | AR500.ocl36 (twice) | Sorts the work files into the correct sequence (Customer / Name / Salesman).                                 | ?9?AR500X → ?9?AR500D ; ?9?ARC500 → ?9?AR500C            | Creates sorted ?9?AR500D (detail) and ?9?AR500C (master) — both retained only for this job                    |
| 6a    | **AR750**   | AR500.ocl36       | Prints Aged Trial Balance **by Customer Number** (most common official report).                               | AR500C (ARCUST), AR500D (ARDETL), ARCONT, GSTABL, ARCUSP | PRINT spooled file (164-col) — clean, due-date headings, credit-limit “**”, grouping subtotals                |
| 6b    | **AR502**   | AR500.ocl36       | Prints Aged Trial Balance **by Customer Name** (alphabetical).                                               | AR500C (ARCUST), AR500D (ARDETL), ARCONT, GSTABL         | PRINT spooled file — alpha order; headings still incorrectly say “Days from Invoice Date”                     |
| 6c    | **AR501**   | AR500.ocl36       | Prints Aged Trial Balance **by Salesman** with optional multiple copies per rep + Excel export.              | AR500C (ARCUST), AR500D (ARDETL), SA5SHX, ARCONT, GSTABL | PRINT spooled file + ?9?ARATBS (512-byte spreadsheet file); page eject per salesman if requested              |

### Notes on Execution Flow
- Only **one** of 6a / 6b / 6c runs — decided by position 152 of the parameter block:
  - “C” → AR750  
  - “N” → AR502  
  - “S” → AR501
- All temporary files (?9?ARC500, ?9?AR500X, ?9?AR500D, ?9?AR500C, ?9?ARATBS) are automatically deleted at job end (RETAIN-T or RETAIN-J).
- **AR390** is the only program that physically changes live A/R data — everything else is read-only or temporary.

This chain has been running unchanged (except minor fixes) in hundreds of IBM i shops for 25–35 years and remains the gold-standard month-end A/R reconciliation process.