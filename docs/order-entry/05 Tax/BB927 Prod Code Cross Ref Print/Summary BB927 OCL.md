### Use Cases Implemented

Based on the provided call stack (OCL procedure invoking #GSORT for sorting, followed by BB927 RPG program for processing and printing), the following use cases are implemented. These represent distinct functional purposes within the overall process of handling product code cross-references, though they are interconnected in a single batch workflow:

1. **Data Sorting for Hierarchical Reporting**: Sort the raw product cross-reference data (from file BBPRXR) by key fields (company number, cross-reference set, product cross-reference) to enable grouped, efficient reporting. This prepares data for business analysis or auditing, ensuring records are organized logically without duplication or manual intervention.

2. **Enriched Report Generation and Printing**: Produce a formatted printed report of product code cross-references, including lookups for company names, product descriptions, and cross-reference set descriptions. This supports business needs like inventory reconciliation, catalog verification, or customer reference checks, with hierarchical grouping (by company and set) and flagging of deleted records.

3. **Data Lookup and Enrichment**: Perform real-time lookups from supporting files (BICONT for company details, GSTABL for set descriptions, GSPROD for product details post-modification) to add descriptive context to raw codes. This use case enhances data usability for decision-making, such as identifying product-container relationships or validating cross-references.

4. **Audit and Deletion Flagging**: Identify and flag records marked for deletion (via 'D' codes in input or lookup files) in the output report, allowing business users to review soft-deleted items without permanent data loss. This supports compliance, error correction, or cleanup processes.

These use cases collectively form a batch reporting pipeline, with no interactive elements; they assume pre-existing data files as inputs.

### Function Requirement Document: GenerateProductCrossReferenceReport

#### Overview
This function represents the core use case of generating a product code cross-reference report as a large, non-interactive batch process. It takes input data sources (equivalent to files BBPRXR, BICONT, GSTABL, GSPROD) and produces a structured report output (e.g., a printable file or data array) without screen interactions. The function handles sorting, lookups, grouping, and formatting to provide a hierarchical, enriched view of cross-reference data for business reporting needs like inventory management, product catalog auditing, or customer reference validation.

#### Inputs
- **CrossReferenceData**: Array or dataset of records containing: DeleteFlag (1 char), CompanyNumber (2 chars), ProductXRef (20 chars), ProductCode (4 chars), XRefSet (6 chars), ContainerCode (3 chars). Assumed pre-validated; represents raw cross-references.
- **CompanyLookupData**: Dataset keyed by CompanyNumber, providing CompanyName (30 chars).
- **SetLookupData**: Dataset keyed by composite key (e.g., 'BBXSET' + CompanyNumber + XRefSet), providing SetDescription (30 chars) and DeleteFlag (1 char).
- **ProductLookupData**: Dataset keyed by ProductCode (6 chars), providing ProductDescription (30 chars), DeleteFlag (1 char), and optional FlagCode (1 char).
- **SystemDateTime**: Current date and time for report headers (auto-captured if not provided).

#### Outputs
- **ReportData**: Structured array or file with:
  - Headers: CompanyName, PageNumber, ReportDate, ReportTime, XRefSetDescription, XRefSetCode, Fixed title "* PRODUCT CODE CROSS REFERENCE FILE *".
  - Column Headings: "CUSTOMER PRODUCT CD", "ARG CODE", "ARG CNTR", "CUSTOMER NAME", "XREF CODE".
  - Detail Lines: Per record - DeleteFlag, ProductXRef, ProductCode, ContainerCode, ProductDescription, XRefSetDescription, XRefSetCode.
  - Paging: New pages on group breaks or overflow, with repeated headers.

#### Process Steps
1. **Sort Input Data**: Sort CrossReferenceData by CompanyNumber (ascending), XRefSet (ascending), ProductXRef (ascending) to enable hierarchical grouping.
2. **Initialize Report**: Capture system date/time; reset page counter to 0.
3. **Process Groups (Hierarchical Loop)**:
   - For each new CompanyNumber group:
     - Lookup CompanyName from CompanyLookupData; default to blank if not found.
     - Print company-level header with CompanyName, PageNumber, Date.
   - For each new XRefSet group within company:
     - Lookup SetDescription from SetLookupData using composite key; default to blank if not found.
     - Print set-level header with SetDescription, XRefSetCode, title, Time.
     - Print column headings.
   - For each detail record:
     - Lookup ProductDescription from ProductLookupData using ProductCode; default to blank if not found.
     - Output detail line with all fields, including enriched descriptions.
4. **Handle End-of-Data**: Close report after processing all records.

#### Business Rules
- **Grouping Hierarchy**: Report must group by CompanyNumber (major) and XRefSet (minor) for logical organization; new groups trigger header refreshes and page resets (for company).
- **Delete Flagging**: Include DeleteFlag in output if 'D' (from input or lookups); treat as soft delete—do not skip records, but flag for business review (e.g., cleanup or audit).
- **Lookup Defaults**: Use blanks for missing descriptions to avoid report errors; assume keys are valid—no additional validation.
- **Formatting**: Fixed-width output (e.g., ProductXRef at positions 2-23, etc.); dates in YYMMDD, times in HHMMSS; page numbering starts at 1 per company.
- **Modifications**: Product lookups use ProductLookupData (GSPROD equivalent) for descriptions; ignore legacy SetLookupData for products.
- **Calculations**: No arithmetic; only string moves, key composites (e.g., 'BBXSET' + CompanyNumber for set lookup), and system time/date extraction. Page increment on overflow (implicit, e.g., every ~60 lines assuming printer limits).
- **Assumptions/Constraints**: Input data pre-sorted or sortable; no updates to source data; batch-only (no real-time); supports up to 999,000 records (from OCL allocation).