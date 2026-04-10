**Use Cases Implemented (as Large Functions)**

These are the core business functions implemented by the AR906P → AR906 → AR9064 → AR9065 call stack. Each is treated as a self-contained process that accepts the required inputs and completes the full business operation (no screen interaction assumed):

1. **Manage Commission Charge Entry**  
   (Create, Change, or View a single commission charge record)

2. **Toggle Commission Charge Status**  
   (Inactivate or Reactivate a commission charge record)

3. **List / Search Commission Charges**  
   (Retrieve and display filtered list of commission charge records)

4. **Print Commission Charge Listing**  
   (Generate full printable report of all commission charge records)

---

**Function Requirement Document**  
**Commission Charge Management**

**1. Business Object**  
A Commission Charge defines how commission is calculated for a specific combination of:  
**Key** = Company (CO) + Customer (CUSN) + Location (LOC) + Product (PROD) + Unit of Measure (UM)  

**Core Fields**  
- Vendor (ACVEND)  
- Rate per Quantity (ACRAMT) + Percent per Quantity (ACRPCT)  
- Rate per Sales $ (ACSAMT) + Percent per Sales $ (ACSPCT)  
- Product Class (ACPCLS), Product Group (ACPGRP), Inventory Group (ACIGRP)  
- Status (ACDEL): 'A' = Active, 'I' = Inactive  

**2. Use Case 1 – Manage Commission Charge Entry**  
**Inputs**  
- Key fields (CO, CUSN, LOC, PROD, UM)  
- Mode: MNT or INQ  
- Optional: File group ('G' or 'Z')  

**Process Steps**  
1. Retrieve existing record or create new.  
2. On create: default Product Class, Product Group, and Inventory Group from Product Master (GSPROD).  
3. Validate Vendor exists in Vendor Master (APVEND).  
4. In MNT mode: allow update of all non-key fields.  
5. Save: Write new record (status = 'A') or update existing record only if data actually changed.  

**Business Rules**  
- Key is unique.  
- Key fields are protected after creation.  
- Vendor is mandatory and must be valid.  
- Inactive records cannot be changed (handled by calling program).  

**3. Use Case 2 – Toggle Commission Charge Status**  
**Inputs**  
- Key fields (CO, CUSN, LOC, PROD, UM)  
- File group ('G' or 'Z')  

**Process Steps**  
1. Retrieve current record.  
2. If current status = 'I' → set to 'A' (Reactivate).  
3. If current status = 'A' → set to 'I' (Inactivate).  
4. Update only the status field.  

**Business Rules**  
- Soft status toggle only — records are never deleted.  
- Prevents redundant action (cannot inactivate an already inactive record).  
- Returns success flag ('A' or 'I') to caller.  

**4. Use Case 3 – List / Search Commission Charges**  
**Inputs**  
- Optional selection criteria: CO, CUSN, LOC, PROD, UM  
- Include/exclude inactive flag  
- File group ('G' or 'Z')  

**Process Steps**  
1. Position and read ARCOMM using provided key values (partial key supported).  
2. Skip inactive records ('I' or 'D') if exclude-inactive is selected.  
3. Return list with color coding (inactive = blue).  
4. Support direct key access and subfile options for further actions.  

**Business Rules**  
- Supports both full and partial key search.  
- Inactive records are visible only when explicitly requested.  

**5. Use Case 4 – Print Commission Charge Listing**  
**Inputs**  
- File group ('G' or 'Z')  

**Process Steps**  
1. Read entire ARCOMM file sequentially.  
2. Print formatted report with standard headers (company, user, date, time, page).  
3. Include every record regardless of status.  

**Business Rules**  
- No filters or subtotals.  
- Spool file created with HOLD(*YES) SAVE(*YES).  
- Report width = 164 characters, 68 lines/page.  

**General Business Rules (All Use Cases)**  
- All functions respect File Group ('G' or 'Z') for library separation.  
- No hard deletes — only status toggle.  
- Commission rates support both quantity-based and sales-dollar-based calculation (stored as separate rate + percent pairs).  
- Product-related groups are defaulted from master on creation but can be overridden.  

This document captures the complete business requirements and logic implemented across the four programs. All calculations are simple storage of rates/percentages (no runtime commission math performed in these programs).


**Commission Charge Programs – Call Stack**

| Order | Program   | Main Purpose                                                                 | Tables / Files Used                          | Outputs / Side Effects                                      |
|-------|-----------|------------------------------------------------------------------------------|----------------------------------------------|-------------------------------------------------------------|
| 1     | **AR906P**    | Work With / List all Commission Charge entries with subfile navigation and options | ARCOMM (main), ARCOMMRD (read), BICONT, GSPROD, ARCUST, INLOC | Calls other programs, sends messages, refreshes subfile, calls print |
| 2     | **AR906**     | Maintain or Inquiry on a single Commission Charge record (detail screen)    | ARCOMM (add/update), APVEND, GSPROD, ARCUST, INLOC, BICONT | Creates or updates ARCOMM record, returns success flag     |
| 3     | **AR9064**    | Inactivate or Reactivate a Commission Charge record (confirmation window)   | ARCOMM                                       | Updates `acdel` flag ('A' ↔ 'I') in ARCOMM, returns status flag |
| 4     | **AR9065**    | Print full Commission Charge Listing report                                 | ARCOMM                                       | Generates spool file (QSYSPRT) with commission charge listing |

### Additional Notes on Flow:
- **AR906P** is the main entry point (called from OCL or menu).
- From AR906P, users can call **AR906** (options 1, 2, 5), **AR9064** (option 4), or **AR9065** (F15).
- All programs use file overrides based on parameter `p$fgrp` ('G' or 'Z') to switch between library sets.
- No hard deletes occur — only soft status changes via `acdel` field.


**Commission Charge Programs – Table Access Summary**

| Program   | ARCOMM                                      | ARCOMMRD                          | APVEND                          | GSPROD                          | ARCUST                          | INLOC                           | BICONT                          |
|-----------|---------------------------------------------|-----------------------------------|---------------------------------|---------------------------------|---------------------------------|---------------------------------|---------------------------------|
| **AR906P** | Read (subfile load, chain, readc)          | Read (sequential load via logical) | -                               | Read (validation)               | Read (validation)               | Read (validation)               | Read (validation)               |
| **AR906**  | **Write** (new record) / **Update** (existing) | -                                 | Read (validate vendor)          | Read (default values on create) | Read (display name)             | Read (display name)             | Read (company)                  |
| **AR9064** | **Update** (only `acdel` flag 'A' ↔ 'I')   | -                                 | -                               | -                               | -                               | -                               | -                               |
| **AR9065** | Read (full sequential scan for report)     | -                                 | -                               | -                               | -                               | -                               | -                               |

### Quick Explanation of Table Usage:
- **ARCOMM** — The central master file. Only **AR906** creates new records and **AR9064** performs status changes. **AR906P** and **AR9065** only read it.
- **ARCOMMRD** — Logical view used only by **AR906P** for efficient subfile loading.
- Other masters (**APVEND, GSPROD, ARCUST, INLOC, BICONT**) are used purely for validation and description lookup (display names, defaults, existence checks).

