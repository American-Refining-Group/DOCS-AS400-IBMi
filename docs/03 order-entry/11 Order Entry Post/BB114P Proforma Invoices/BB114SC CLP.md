### Process Steps of the CLP Program BB114SC

This Control Language Program (CLP) acts as a wrapper to prepare and override specific database files based on a file group parameter, then calls a subroutine program to build work files for customer order confirmations. It is called from the main OCL (e.g., during subfile option processing in BB114 for option 6) and passes parameters for company and order. The program ensures the correct file versions (based on group, e.g., 'G' or 'Z') are used without hardcoding libraries. Below is a step-by-step breakdown of the execution:

1. **Parameter Reception and Variable Declaration**:
   - Receives three input parameters: &P$CO# (company number, decimal 2.0), &P$ORDR (order number, decimal 6.0), &P$FGRP (file group indicator, character 1).
   - Declares character variables (length 10) for file names: &BBORDR, &BBPROH, &BBPROD, &BBPROO, &BBPROI, &BBPROB, &BBPROM.

2. **File Name Construction**:
   - Constructs dynamic file names by concatenating the file group (&P$FGRP) as a prefix to base file names (e.g., &BBORDR = &P$FGRP + 'BBORDR'). This allows switching between file sets (e.g., GBBORDR vs. ZBBORDR) without explicit library references.

3. **File Overrides**:
   - Overrides (OVRDBF) the logical file names (BBORDR, BBPROH, etc.) to point to the constructed physical file names in the library list (*LIBL), using the first member (*FIRST). This redirects any subsequent file operations to the group-specific files.

4. **Call Subroutine Program**:
   - Calls the external program BB114S, passing &P$CO# (company) and &P$ORDR (order) as parameters. This program likely processes the order data and populates the work files (e.g., BBPROR, though not directly mentioned here; inferred from purpose).

5. **Cleanup and Termination**:
   - Deletes all file overrides (DLTOVR FILE(*ALL)) to restore default file mappings.
   - Ends the program (ENDPGM), returning control to the caller.

The program is concise and focuses on setup/cleanup around the BB114S call, ensuring isolation and correct file usage per invocation.

### Business Rules

- **File Group Flexibility**: Uses &P$FGRP to dynamically select file sets (e.g., 'G' for general, 'Z' for another environment), supporting multi-company or multi-dataset processing without code changes. This aligns with prior programs (e.g., BB114) for environment-specific data handling.
- **Order-Specific Processing**: Targets a single order (&P$ORDR) within a company (&P$CO#), building confirmation work files (e.g., for BB110 printing). Assumes valid inputs; no explicit validation here (likely handled in caller like BB114).
- **Work File Preparation**: Prepares temporary/work files (BBPRO* series) for order confirmation generation, used downstream in printing (e.g., BB110). These files store header, detail, and other order components.
- **Isolation and Cleanup**: Overrides are scoped to this call and removed post-execution, preventing side effects on other programs. No error handling visible; assumes success or caller-managed exceptions.
- **Integration Context**: Ties into the broader proforma/order confirmation flow (from OCL to BB114 to this to BB114S to BB110), enforcing data consistency for confirmations.

### Tables/Files Used

These are database files (physical/logical) overridden for use in the called program:

- BBORDR (order-related; overridden to prefixed version, e.g., GBBORDR).
- BBPROH (proforma/order headers).
- BBPROD (proforma/order details).
- BBPROO (proforma/order other?).
- BBPROI (proforma/order items?).
- BBPROB (proforma/order batches?).
- BBPROM (proforma/order marks?).

No display or printer files; focus is on data files.

### External Programs Called

- BB114S: Called with parameters &P$CO# (company) and &P$ORDR (order); this likely performs the core logic to build/populate the work files (e.g., BBPROR) for confirmations.