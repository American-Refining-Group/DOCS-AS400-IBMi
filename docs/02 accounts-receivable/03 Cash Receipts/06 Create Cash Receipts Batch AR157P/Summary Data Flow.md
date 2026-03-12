Below is a table that lists the programs in the EFT cash receipts batch file creation suite as rows and the tables/files used as columns. At each intersection, a brief description indicates how the program interacts with the table (e.g., reading, modifying, creating, or writing). The programs and files are derived from the provided documents: `AR157P.ocl36`, `AR157P.rpgle`, `AR157.ocl36`, `AR157A.ocl36`, and `AR157A.rpg36`. Programs that do not explicitly use tables (`GSGENIEC`, `GSY2K`) are included for completeness but show no interactions.

<xaiArtifact artifact_id="ac7314ab-3c6b-4dda-b4f8-98ec3d6f2d82" artifact_version_id="de5eb158-7942-4d02-9c26-f36e06df7987" title="Program_Table_Interaction_Table.md" contentType="text/markdown">

# Program and Table Interaction Matrix

| **Program** | **ARCONT** | **GSCONT** | **AR157PFM** | **CRTRAN (?9?E?L'110,6'?)** | **?9?CRIEGG** |
|-------------|------------|------------|--------------|-----------------------------|---------------|
| **GSGENIEC** | - | - | - | - | - |
| **GSY2K** | - | - | - | - | - |
| **AR157P.ocl36** | Reads to validate company number (`?9?ARCONT`). | - | - | - | Checks existence and deletes if `KYDELT = 'Y'` (`DATAF1`). |
| **AR157P.rpgle** | Reads to validate `KYCO`. | Reads to retrieve default company number (`GXCONO`). | Reads user inputs (`KYCO`, `KYUPDT`); writes error messages. | - | - |
| **AR135TC** | - | - | - | Checks existence of file (`'GE' + KYUPDT`). | - |
| **AR157.ocl36** | - | - | - | Reads non-deleted records; deletes file after copy. | Creates or replaces with non-deleted records from `CRTRAN`. |
| **AR157A.ocl36** | - | - | - | Opens for shared access to allow updates. | - |
| **AR157A.rpg36** | - | - | - | Reads and updates fields (e.g., `KYUPDT` at position 37). | - |

## Notes
- **File Naming**: 
  - `?9?` is a placeholder for the company code (e.g., `CO123`).
  - `?L'110,6'?` represents the bank upload date (`KYUPDT`, e.g., `202311`).
  - Example: `CRTRAN` resolves to `QS36F/CO123E202311`, `?9?CRIEGG` to `QS36F/CO123CRIEGG`.
- **Interactions**:
  - **Reads**: Accessing file data for validation or retrieval.
  - **Writes**: Updating file content or displaying data (e.g., error messages on `AR157PFM`).
  - **Creates/Replaces**: Generating or overwriting a file (e.g., `?9?CRIEGG`).
  - **Deletes**: Removing a file (e.g., `CRTRAN` or `?9?CRIEGG`).
- **Missing Logic**: `AR157A.rpg36` lacks calculation specs, so updates are inferred (e.g., `KYUPDT`).
- **No Interaction**: Programs `GSGENIEC` and `GSY2K` have no explicit file interactions in the provided code.

</xaiArtifact>