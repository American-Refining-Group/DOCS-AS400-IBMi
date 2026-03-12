### Use Cases Implemented

Based on the call stack (OCL procedure SCPROCP loading and executing RPG program GS973), the program implements interactive maintenance for product-container master records. The core functionality focuses on validation, inquiry, and limited updates to status and last sold date. The following use cases are supported:

1. **Inquire on Product-Container Combination**: Validate and retrieve details (including product description, container description, current status, and last sold date) for a given company, product, and container combination. This ensures the combination exists, is active, and displays associated master data.

2. **Accept Product-Container Combination**: Update the status of a valid product-container combination to 'Accept' ('A'), which records the current system date as the last sold date. This enforces acceptance tracking without allowing overrides or other changes in this program.

3. **Reset or Exit Maintenance**: Clear input fields and reset to the initial input state, or exit the maintenance process entirely, without saving changes.

No additional use cases are implemented, such as adding new combinations, deleting records, overriding status ('O'), or bulk processing.

### Function Requirement Document: Maintain Product-Container Combination

#### Overview
This function encapsulates the core use cases as a non-interactive, batch-style process. It accepts inputs (company number, product code, container code, and optional status) and performs validation, data retrieval, and conditional update in a single call. Outputs include retrieved details or error messages. The function replaces screen interactions with parameter-based input, focusing on business logic for inventory additive master maintenance.

#### Inputs
- **Company Number (CO)**: Required integer (non-zero).
- **Product Code (PROD)**: Required string (non-blank).
- **Container Code (CNTR)**: Optional string (may be blank, but combination must exist if provided).
- **Status (STAT)**: Optional string ('A' for accept; ignored otherwise).

#### Outputs
- **Success Response**: Structure containing:
  - Product Description (from GSPROD).
  - Container Description (from GSCNTR).
  - Last Sold Date (PNLSD8, formatted as YYYYMMDD).
  - Current/Updated Status (PNSTAT).
- **Error Response**: Message string (e.g., "COMPANY NUMBER CANNOT BE BLANK", "INVALID COMBINATION ENTERED").

#### Process Steps
1. **Validate Inputs**:
   - Check CO is non-zero and exists in company control file (INCONT).
   - Check PROD is non-blank.
   - Form composite key (CO + PROD + CNTR) and check existence in PRCNTR.
   - If record found in PRCNTR, ensure not deleted (PNDEL ≠ 'D') or inactive (PNDEL ≠ 'I').

2. **Retrieve Data**:
   - Fetch product details from GSPROD using key (CO + PROD), extract description (TPDESC).
   - Fetch container details from GSCNTR using CNTR, extract description (TCDESC).
   - From PRCNTR, retrieve current last sold date (PNLSD8) and status (PNSTAT).

3. **Conditional Update**:
   - If STAT = 'A', update PRCNTR record:
     - Set PNSTAT to 'A'.
     - Set PNLSD8 to current system date (computed as YYYYMMDD).
   - No update if STAT ≠ 'A' or invalid.

4. **Return Results**:
   - On success: Return retrieved/updated details.
   - On error: Return specific error message and halt.

#### Business Rules
- **Validation Rules**: Company must exist and be active; product required; combination must be valid, active, and not deleted. Errors prevent further processing.
- **Update Restriction**: Only 'Accept' status triggers update; no support for 'Override' or other values. Update implies acceptance of the combination for sales/inventory purposes.
- **Date Calculation**: Current date derived from system time, converted to YYYYMMDD via: Extract YYMMDD from time, multiply by 10000.01 for YMD format, prefix '20' for century (Y2K handling).
- **Data Integrity**: Read-only access to GSPROD and GSCNTR for descriptions; updates limited to PRCNTR status and date fields. Shared file access assumed for concurrency.
- **No Creation/Deletion**: Function assumes existing records; does not add new combinations or delete/mark inactive.

#### Dependencies
- Database Tables: INCONT (company validation), PRCNTR (main record), GSPROD (product details), GSCNTR (container details).
- No external programs called; all logic self-contained.