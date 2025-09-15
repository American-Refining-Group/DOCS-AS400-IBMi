Below is a table that lists the provided programs (`AR135P`, `AR135`, `AR135B`, `AR135A`) as rows and the tables/files used (`ARCONT`, `GSCONT`, `AR135PD`, `AR135FM`, `CRTRAN`, `CRTRANR`, `ARCUST`, `ARCUSTX`, `GLMAST`, `ARDETL`, `LIST3`) as columns. At each intersection, a brief description indicates how the program modifies or interacts with the table/file, including creating, writing, or updating actions. If a program does not use a table, the cell is marked with "-".

<xaiArtifact artifact_id="2ae04183-476d-474f-89a5-26ded33430ea" artifact_version_id="06e25187-7112-49d0-b9d1-b0f4c380a9cb" title="Program_Table_Interaction.md" contentType="text/markdown">

# Program and Table Interaction Matrix

| Program | ARCONT | GSCONT | AR135PD | AR135FM | CRTRAN | CRTRANR | ARCUST | ARCUSTX | GLMAST | ARDETL | LIST3 |
|---------|--------|--------|---------|---------|--------|---------|--------|---------|--------|--------|-------|
| **AR135P** | Reads to validate company number (`KYCO`). | Reads to retrieve default company number (`GXCONO`). | Writes to display input screen and error messages. | - | - | - | - | - | - | - | - |
| **AR135** | Reads to validate company and retrieve G/L accounts. | - | - | Writes to display transaction entry/update screens and errors. | Creates/updates transaction records (e.g., `ATAMT`, `ATDISC`, `ATTYPE`). | Reads for reference/comparison of existing transactions. | Reads to validate customer and retrieve name (`ARNAME`), salesman (`ARSLS#`). | Reads for customer name abbreviation and address lookup. | Reads to validate G/L accounts (`ATGLCR`, `ATGLDR`, `ATGLDI`). | Updates payment details (`ADPAY`, `ADPART`) and terms (`ADTERM`). | - |
| **AR135B** | - | - | - | - | Updates date fields (`KYUPDT`, `KYSEDT`) in positions 228-239. | - | - | - | - | - | - |
| **AR135A** | Reads to retrieve company name (`ACNAME`) for report header. | - | - | - | Reads transaction data for report generation. | - | Reads to retrieve customer name (`ARNAME`) and EFT flag (`AREFT`). | - | - | - | Writes EFT pre-upload report with headers, details, and totals. |

</xaiArtifact>