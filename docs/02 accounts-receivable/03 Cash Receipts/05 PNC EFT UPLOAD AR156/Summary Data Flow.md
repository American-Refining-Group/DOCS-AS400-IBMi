Below is a table that lists the provided programs (`AR156.ocl36`, `AR156P.rpgle`, `AR135TC.clp`, `AR137A.ocl36`, `AR137A.rpg36`, `AR156.rpg36`) in the rows and the tables (files) used in the columns. At the intersection of each program and table, a brief description explains how the program modifies, creates, or writes to that table. If a program does not interact with a table, the cell is marked with "N/A". The table is presented as a Markdown artifact, as per the requirements.

<xaiArtifact artifact_id="7afb1714-ff52-424e-9d26-3a8e2d68bfa1" artifact_version_id="22880c67-5320-45de-bb4d-2d9cd0912f9f" title="Program_Table_Interaction.md" contentType="text/markdown">

# Program and Table Interaction Matrix

| Program File    | ARCONT                      | ARCUST                      | AREFTD                      | AREFTS                      | ARCUSP                      | EFTFILE                     | REPORT                      | GSCONT                      | ARDTGGC                     |
|-----------------|-----------------------------|-----------------------------|-----------------------------|-----------------------------|-----------------------------|-----------------------------|-----------------------------|-----------------------------|-----------------------------|
| AR156.ocl36     | Reads for configuration data | Reads for customer data     | Copies data to from source file | Clears and writes staging data | Reads for customer EFT data | Clears and writes NACHA file | N/A                         | N/A                         | Creates if needed            |
| AR156P.rpgle    | Chains to validate company number | N/A                         | N/A                         | N/A                         | N/A                         | N/A                         | N/A                         | Chains to retrieve default company number | N/A                         |
| AR135TC.clp     | N/A                         | N/A                         | Checks existence of file     | N/A                         | N/A                         | N/A                         | N/A                         | N/A                         | N/A                         |
| AR137A.ocl36    | Reads for configuration data | Reads for customer data     | Reads EFT data              | Clears before processing     | N/A                         | N/A                         | N/A                         | N/A                         | N/A                         |
| AR137A.rpg36    | Chains for company settings | Chains for customer details | Reads EFT transaction data  | Writes summarized EFT data  | N/A                         | N/A                         | Generates EFT report        | N/A                         | N/A                         |
| AR156.rpg36     | Chains for company settings | Chains for customer details | N/A                         | Reads staging EFT data      | Chains for EFT banking details | Writes NACHA-compliant file | Generates EFT audit report  | N/A                         | N/A                         |

## Notes
- **ARCONT**: Company control file, read for configuration or validation.
- **ARCUST**: Customer master file, read for customer details and EFT participation status.
- **AREFTD**: EFT transaction data file, read or copied for processing.
- **AREFTS**: Staging file, cleared and written with summarized EFT data.
- **ARCUSP**: Customer-specific file, read for EFT banking details.
- **EFTFILE**: NACHA-compliant output file, cleared and written with EFT records.
- **REPORT**: Printer file for generating EFT audit reports.
- **GSCONT**: Control file, read for default company number.
- **ARDTGGC**: Dynamically created file for additional EFT data if needed.

</xaiArtifact>