
| **Program** | **CRTRAN** | **AR137S** | **AREFTD** | **AREFTS** | **ARCONT** | **ARCUST** | **ARCUFMX** | **ARDTWSC** | **EFTEMALOTQ** | **EFTSPLITQ** | **EFTEMALQSV** | **EFTSPLTQSV** | **LIST1-LIST4** |
|-------------|------------|------------|------------|------------|------------|------------|-------------|-------------|----------------|---------------|----------------|----------------|-----------------|
| **AR137P.ocl36.txt** | - | - | - | - | Reads to validate company (`KYCO`). | Reads to validate customers (`KYCS01`-`KYCS10`). | - | - | - | - | - | - | - |
| **AR137P.rpgle.txt** | Reads to check file existence (‘GE’ + `KYUPDT`). | - | - | - | Reads to validate `KYCO`. | Reads to validate `KYCS01`-`KYCS10`. | - | - | - | - | - | - | - |
| **AR137.ocl36.txt** | Reads to filter transactions. | Creates/writes filtered records for ‘ALL’ or ‘SEL’. | Creates/writes filtered records for ‘ALL’ or ‘SEL’. | Clears before `AR137A`. | Reads for validation. | Reads for validation. | Reads for email data. | Defines as temporary (50 records). | Defines for report output. | Defines for split output. | Defines for saved splits. | Defines for saved splits. | - |
| **AR137X.rpg36.txt** | Updates by flagging deleted records (`ATDEL = 'D'`). | - | - | - | - | - | - | - | - | - | - | - | - |
| **AR137A.ocl36.txt** | - | - | Reads transactions for summary. | Clears to prepare for new summaries. | Reads for company validation. | Reads for customer validation. | - | - | Defines for output (via `AR137B`). | - | - | - | - |
| **AR137A.rpg36.txt** | - | - | Reads to aggregate amounts. | Creates/writes customer summary records (totals, dates). | Reads to validate `AFCO`. | Reads to validate `AFCUST`. | - | - | - | - | - | - | - |
| **AR137E.rpg36.txt** | - | - | - | Reads summaries to match customers. | - | - | Reads to count email addresses. | Creates/writes email counts (company + customer key). | - | - | - | - | - |
| **AR137B.ocl36.txt** | Reads transactions (as `?9?E?L'110,6'?` or `AREFTX`). | - | Reads transactions for reports. | - | Reads for company data. | Reads for customer data. | Reads for email data. | Reads for email counts. | Writes spool files for reports. | - | Writes saved splits. | Writes saved splits. | Defines printer files for output. |
| **AR137B.rpg36.txt** | - | - | Reads transactions for report details. | - | Reads to validate `AFCO`. | Reads for customer details (`ARNAME`, addresses). | Reads for email addresses (`EIFM`). | Reads for email counts. | Writes reports via `LIST1`-`LIST4`. | - | - | - | Creates/writes formatted EFT reports (headers, details, totals). |
| **AR137TC.clp.txt** | - | - | - | - | - | - | - | - | Reads/writes spool files; clears after processing. | Reads/writes split spool files; clears after processing. | Writes split spool files (`SFASPLIT`). | Writes moved split files (`SFAMOVE`). | - |

### Notes:
- **Columns (Files)**: Include database files (`CRTRAN`, `AR137S`, `AREFTD`, `AREFTS`, `ARCONT`, `ARCUST`, `ARCUFMX`, `ARDTWSC`) and output queues/printer files (`EFTEMALOTQ`, `EFTSPLITQ`, `EFTEMALQSV`, `EFTSPLTQSV`, `LIST1`-`LIST4`).
- **Intersections**: Describe actions like reading, writing, updating, creating, or clearing. A dash (-) indicates no interaction.
- **AR137P.ocl36.txt and AR137P.rpgle.txt**: Validate inputs using `ARCONT` and `ARCUST`; check `CRTRAN` existence.
- **AR137B.ocl36.txt**: References `AR136B` (likely a typo for `AR137B`), defining files and queues.
- **AR137TC.clp.txt**: Interacts with output queues, not database files, for PDF conversion and emailing.
- **Context**: Actions are derived from provided source code and inferred roles for `AR137P` components.
