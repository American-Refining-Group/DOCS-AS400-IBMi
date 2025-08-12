### Process Steps of the CLP Program BB112SC

This Control Language Program (CLP) for IBM i/AS/400 serves as a wrapper to prepare and call a subroutine for building a customer order confirmation work file (BBOCFR) based on a specific company and order number. It dynamically overrides file names using a file group parameter (e.g., 'G' for production or 'Z' for test/archive) to ensure the correct dataset is used. The program is designed to be called from an OCL script, passing in parameters for targeted order processing. Below is a step-by-step breakdown of the execution flow:

1. **Parameter Reception**:
   - Receives three input parameters: `&P$CO#` (company number, decimal 2.0), `&P$ORDR` (order number, decimal 6.0), and `&P$FGRP` (file group identifier, character 1).

2. **Variable Declaration and Initialization**:
   - Declares character variables (length 10) for file names: `&BBORDR`, `&BBOCFH`, `&BBOCFD`, `&BBOCFO`, `&BBOCFI`, `&BBOCFB`, and `&BBOCFM`.
   - Constructs these file names by concatenating `&P$FGRP` with base file names (e.g., if `&P$FGRP='G'`, `&BBORDR` becomes 'GBBORDR').

3. **File Overrides**:
   - Overrides database files (OVRDBF) to redirect logical file names (e.g., BBORDR) to the physical files in the library list (*LIBL) with the constructed names (e.g., GBBORDR), using the first member (*FIRST). This applies to all seven declared files.
   - These overrides ensure that the subsequent program call uses the appropriate group-specific files without hardcoding library paths.

4. **Call Subprogram**:
   - Calls program `BB112S` (likely an RPG or similar program), passing `&P$CO#` and `&P$ORDR` as parameters. This subprogram is responsible for the core logic of building the order confirmation work file (BBOCFR) for use in downstream processes like printing (e.g., in BB110).

5. **Cleanup**:
   - Deletes all file overrides (DLTOVR FILE(*ALL)) to restore the default file mappings.
   - Ends the program (ENDPGM).

The program is short and focused on setup/teardown, delegating the actual data processing to BB112S. It runs synchronously and does not handle errors explicitly (e.g., no MONMSG for monitoring messages).

### Business Rules

- **File Grouping**: The file group (`&P$FGRP`) determines the dataset (e.g., 'G' for live/production files like GBBORDR, 'Z' for test/archive like ZBBORDR). This allows segregation of data environments, preventing accidental mixing of production and test data during order confirmation generation.
- **Targeted Processing**: Processes a single order per call (specified by company and order number), ensuring efficient, on-demand building of confirmation work files rather than batching multiple orders.
- **Work File Preparation**: The purpose is to populate BBOCFR (customer order confirmation work file) for use in printing/processing (e.g., by BB110). This implies rules for order eligibility (e.g., open orders only, as per prior programs like BB112).
- **Overrides Scope**: Overrides are temporary (call-level) and removed post-call to avoid impacting other programs. Files use *LIBL for library resolution, assuming the correct library is in the job's library list.
- **Assumptions**: Relies on BB112S to handle data validation, population of BBOCFR, and any business logic like checking order status or details. No direct updates here; it's preparatory.

### Tables Used

The program overrides the following logical file names to point to physical files (prefixed by `&P$FGRP`, e.g., GBBORDR) in *LIBL. These are used indirectly via the call to BB112S; no direct read/write occurs in this CLP:

- `BBORDR` (overridden to e.g., GBBORDR): Likely order details or related records.
- `BBOCFH` (overridden to e.g., GBBOCFH): Order confirmation header.
- `BBOCFD` (overridden to e.g., GBBOCFD): Order confirmation details.
- `BBOCFO` (overridden to e.g., GBBOCFO): Order confirmation other (possibly options or attachments).
- `BBOCFI` (overridden to e.g., GBBOCFI): Order confirmation items.
- `BBOCFB` (overridden to e.g., GBBOCFB): Order confirmation batch or blocks.
- `BBOCFM` (overridden to e.g., GBBOCFM): Order confirmation miscellaneous or marks.

Note: The purpose mentions building BBOCFR (confirmation work file), but it's not overridden here—likely handled internally by BB112S.

### External Programs Called

- `BB112S`: Called with parameters for company and order number. This is the core program that builds the order confirmation work file (BBOCFR).