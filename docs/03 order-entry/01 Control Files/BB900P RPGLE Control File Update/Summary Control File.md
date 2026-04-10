Here is a concise analysis and documentation based on the three programs provided (BB900P.rpgle.txt, BB900.rpgle.txt / BB915, and BB9004.rpgle.txt), which together form a coherent **billing control entry** maintenance module.

### Implemented Use Cases (as of the provided code)

Only **one primary end-to-end business use case** is fully implemented across the call stack:

**Use Case: Maintain Company-Level Billing Control Settings (including freight accounting rules)**

- **Create** a new billing control record for a company  
- **Update / Change** existing billing control record  
- **Display / Inquiry** of an existing record  
- **Logically Delete** (mark inactive) a billing control record  
- **Logically Restore / Undelete** (reactivate) a previously deleted record  

All other operations (positioning, filtering deleted records, subfile paging, F-keys, prompts) are supporting UI mechanics — not separate business use cases.

### Function Requirement Document  
**Business Function: MaintainBillingControlEntry**

**Purpose**  
Create, update, view, logically delete or restore a single **Billing Control Entry** record that defines company-level defaults for name, address, invoicing style, and freight-related general ledger account assignments.

**Inputs** (required to complete the process in a non-interactive / API-like call)

| Parameter       | Type    | Description                                      | Mandatory | Example / Format      |
|-----------------|---------|--------------------------------------------------|-----------|-----------------------|
| CompanyCode     | Char(2) | Unique company identifier (bcco)                 | Yes       | '01', 'E1'            |
| Mode            | Char(3) | Operation: 'MNT' = maintain, 'INQ' = view only   | Yes       | 'MNT' or 'INQ'        |
| FileGroup       | Char(1) | Environment/library group ('G' or 'Z')           | Yes       | 'G'                   |
| Name            | Char(?) | Company / billing entity name                    | Yes (create/update) | 'Acme Freight LLC'    |
| AddressLine1    | Char(?) | Primary address line                             | Yes (create/update) | '123 Main St'         |
| InvoicingStyle  | Char(1) | Freight invoicing method (1,2,3,5 allowed)       | Yes (create/update) | '2'                   |
| SalesFreightGL  | Char(?) | G/L account – sales freight                      | No        | '41000000'            |
| FreightExpGL    | Char(?) | G/L account – freight expense                    | No        | '52000000'            |
| MiscFreightGL   | Char(?) | G/L account – miscellaneous freight              | No        | —                     |
| InvFreightGL    | Char(?) | G/L account – inventory freight related          | No        | —                     |
| COGSFreightGL   | Char(?) | G/L account – cost of goods sold freight         | No        | —                     |
| TollingFreightGL| Char(?) | G/L account – tolling freight                    | No        | —                     |

**Outputs**

| Field           | Type    | Description                                      |
|-----------------|---------|--------------------------------------------------|
| ReturnFlag      | Char(1) | '1' = create/update successful<br>'A' = record restored<br>'D' = record deleted<br>'0' or blank = no change / cancelled / error |

**Process Steps & Business Rules** (non-interactive version)

1. **Locate Record**
   - Read billing control record by `CompanyCode`
   - If not found and Mode = 'MNT' → proceed to create
   - If not found and Mode = 'INQ' → return error / no data
   - If found → load current values

2. **Mode Handling**
   - If Mode = 'INQ' → return current values (read-only); no update allowed
   - If Mode = 'MNT' → continue to validation & update

3. **Validation Rules** (create or update)
   - `Name` must not be blank
   - `AddressLine1` must not be blank
   - `InvoicingStyle` must be one of: '1','2','3','5'
   - For every non-blank G/L account field (SalesFreightGL, FreightExpGL, etc.):
     - The account must exist in the General Ledger master
     - The account must **not** have delete flag = 'D'
     - The account must **not** have inactive flag = 'I' (per 2016 rule – JB01)
   - All validations must pass → otherwise return error

4. **Create / Update Logic** (only when Mode = 'MNT' and validations pass)
   - If record does **not** exist → create new record with supplied values
   - If record **exists** → update only if at least one field changed
   - Set ReturnFlag = '1' on successful create or change

5. **Delete / Restore Logic** (separate invocation – typically via option 4)
   - **Delete** (F23 equivalent):
     - Record must currently **not** be deleted (`bcdel ≠ 'D'`)
     - Company must have **zero** assigned customers in customer master (arcust)
     - If customers exist → reject with error "This Company Has Assigned Customers, Cannot Delete"
     - Set `bcdel = 'D'`
     - Set ReturnFlag = 'D'
   - **Restore** (F22 equivalent):
     - Record must currently be deleted (`bcdel = 'D'`)
     - No dependency check required
     - Set `bcdel = 'A'`
     - Set ReturnFlag = 'A'

**Tables / Entities Used**

- **Billing Control Master** (`bicont` / gbicont or zbicont)  
  → Primary entity (one record per company)
- **Customer Master** (`arcust` / garcust or zarcust)  
  → Used only for delete protection check
- **General Ledger Master** (`glmast` / gglmast or zglmast)  
  → Used to validate referenced G/L accounts

**Non-Functional Notes**

- All delete operations are **logical** (flag-based)
- Environment ('G'/'Z') determines physical library prefix
- No audit trail, versioning, or effective dates implemented in provided code
- Referential integrity enforced only on delete (via customer check)



## Call Stack
| Call Order | Program Name | Main Purpose (one sentence)                                                                 | Tables / Files Used                          | Outputs / Side Effects                                                                 |
|:----------:|:-------------|:--------------------------------------------------------------------------------------------|:---------------------------------------------|:---------------------------------------------------------------------------------------|
| 1          | BB900P       | Displays a subfile list of billing control entries and allows users to create, change, display, delete or restore them | bicont (gbicont/zbicont), bicontrd (read-only view of same file) | Calls BB900 or BB9004; updates screen subfile; shows confirmation messages             |
| 2          | BB900 / BB915| Maintains or displays a single billing control entry (company name, address, invoicing style, freight-related G/L accounts) | bicont (gbicont/zbicont), glmast (gglmast/zglmast) | Creates or updates record in bicont; returns success flag ('1' = maintained)           |
| 3          | BB9004       | Confirmation window to logically delete (mark 'D') or restore (mark 'A') a billing control entry | bicont (gbicont/zbicont), arcust (garcust/zarcust) | Updates bcdel flag in bicont; returns 'D' (deleted) or 'A' (restored) flag             |






## Data Integrations
| Program   | bicont (gbicont / zbicont)                          | glmast (gglmast / zglmast)                  | arcust (garcust / zarcust)                     |
|-----------|------------------------------------------------------|---------------------------------------------|-------------------------------------------------|
| **BB900P** | **Read** (chain, setll, readc via subfile load)<br>**No direct write/update**<br>(delegates changes to BB900 / BB9004) | — (not used)                                | — (not used)                                    |
| **BB900 / BB915** | **Create** new record (WRITE bicontpf)<br>**Update** existing record (UPDATE bicontpf) when Mode = 'MNT'<br>**Read** (CHAIN) to check existence & load data | **Read only** (CHAIN) to validate and retrieve description of each G/L account field (Sales, Freight, Misc, Inv, COGS, Tolling) | — (not used)                                    |
| **BB9004** | **Update** only (UPDATE bicontpf)<br>Changes `bcdel` flag:<br>• 'D' → logical delete<br>• 'A' → logical restore/undelete | — (not used)                                | **Read only** (SETLL) to check whether any customers exist for this company (delete protection rule) |

### Quick summary of modification patterns

- **Only BB900 / BB915 creates or fully updates** billing control records (name, address, invoicing style, G/L accounts).
- **Only BB9004 modifies the delete flag** (`bcdel`) — it never touches other fields.
- **BB900P** is the user interface / coordinator and does **not directly write** to any table — it calls the other two programs to perform the actual database operations.
- **glmast** is used only for validation (existence + not deleted/inactive).
- **arcust** is used only for delete protection (business rule: cannot delete a company that still has assigned customers).

