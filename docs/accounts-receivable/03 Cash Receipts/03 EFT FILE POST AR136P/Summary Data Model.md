Below is a table that lists the provided programs (`AR136P.ocl36.txt`, `AR136P.rpgle.txt`, `AR136.ocl36.txt`, `AR136.rpg36.txt`) as rows and the tables/files used (`ARCONT`, `GSCONT`, `ar136pd`, `CRTRAN`, `ARCUST`, `LIST`) as columns. At each intersection, I describe how the program interacts with the respective table/file (e.g., reading, writing, or no interaction). For programs called but not provided (`GSGENIEC`, `GSY2K`, `AR135TC`), they are excluded from the table since their specific interactions with files are unknown, but I note their absence. The table focuses on the direct interactions of the provided programs with the specified files.

<xaiArtifact artifact_id="96b7c029-6d48-4f4b-a4da-d516f6ddba9f" artifact_version_id="097641f3-30c8-4ccc-b9d5-91d1befd66ed" title="Program_File_Interaction_Table.md" contentType="text/markdown">

# Program and File Interaction Table

| Program Name | ARCONT | GSCONT | ar136pd | CRTRAN | ARCUST | LIST |
|--------------|--------|--------|---------|--------|--------|------|
| **AR136P.ocl36.txt** | Reads (specifies file in shared mode for AR136P.rpgle.txt). | No direct interaction (used by AR136P.rpgle.txt). | No interaction. | No interaction. | No interaction. | No interaction. |
| **AR136P.rpgle.txt** | Reads to validate company number (`KYCO`). | Reads to retrieve default company number (`gxcono`). | Reads/Writes to prompt user input and display error messages. | No direct interaction (file name constructed for AR135TC). | No interaction. | No interaction. |
| **AR136.ocl36.txt** | Reads (specifies file in shared mode for AR136.rpg36.txt). | No interaction. | No interaction. | Reads (specifies file in shared mode for AR136.rpg36.txt). | Reads (specifies file in shared mode for AR136.rpg36.txt). | No interaction. |
| **AR136.rpg36.txt** | Reads to retrieve company name (`ACNAME`) for report header. | No interaction. | No interaction. | Reads transaction data (`ATAMT`, `ATDISC`, etc.) for report. | Reads customer details (`ARNAME`, `AREFT`) for report. | Writes EFT report (headers, detail lines, totals). |

## Notes
- **Files/Tables**:
  - `ARCONT`: Accounts Receivable control file.
  - `GSCONT`: General system control file.
  - `ar136pd`: Workstation display file for user interaction.
  - `CRTRAN`: EFT transaction file (dynamically named, e.g., `GE` + `KYUPDT`).
  - `ARCUST`: Customer master file.
  - `LIST`: Printer file for report output.
- **Interactions**:
  - No programs directly modify or create records in `ARCONT`, `GSCONT`, `CRTRAN`, or `ARCUST`; all are read-only (`DISP-SHR` in OCL or input files in RPG).
  - `ar136pd` is read/written by `AR136P.rpgle.txt` for user prompts and error messages.
  - `LIST` is written to by `AR136.rpg36.txt` to produce the EFT report.
- **Unprovided Programs**:
  - **GSGENIEC** (called in `AR136P.ocl36.txt`): Unknown file interactions; please upload source for details.
  - **GSY2K** (called in `AR136P.ocl36.txt`): Likely no file modifications (date utility); please upload source for details.
  - **AR135TC** (called in `AR136P.rpgle.txt`): Likely reads a file to check existence (e.g., `GE` + `KYUPDT`); please upload source for details.
- **Data Area**: Programs update the `STATUS` field (position 109) in a data area, but this is not a table/file and is excluded from the table.

</xaiArtifact>