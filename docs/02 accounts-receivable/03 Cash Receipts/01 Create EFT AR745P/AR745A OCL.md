The provided document, `AR745A.ocl36.txt`, is an Operation Control Language (OCL) script used in an IBM midrange system (e.g., AS/400 or iSeries) to process and report credit invoices for Electronic Funds Transfer (EFT) customers. It is conditionally invoked by the `AR745.ocl36.txt` script when the substitution variable `?9?` equals 'G' (production mode). Below, I’ll explain the process steps, business rules, tables used, and external programs called, ensuring clarity and conciseness while adhering to the provided guidelines.

### Process Steps of the OCL Program

The `AR745A` OCL script checks for credit invoices, generates a report if they exist, and emails it. The steps are as follows:

1. **Run Query to Check for Credit Invoices**:
   - `RUNQRY QRY(GSSQRY/AREFTCREDT)`:
     - Executes the query `AREFTCREDT` from the query library `GSSQRY`.
     - This query likely checks the accounts receivable data (e.g., `ARDETL` or related files) for credit invoices (invoices with negative amounts) associated with EFT customers.
     - The query sets a condition code or output that determines whether credit invoices exist.

2. **Check for Credit Invoices**:
   - `// IF ?F'A,?9?ARINVCR'?/00000000 GOTO END`:
     - Checks the condition code or data area at file `A` with label `?9?ARINVCR` (substitution variable `?9?` for library).
     - If the value is `00000000` (indicating no credit invoices), the script jumps to the `END` tag, terminating execution.
     - If credit invoices are found, execution continues.

3. **Override Printer File for Report Output**:
   - `OVRPRTF FILE(QPQUPRFIL) OUTQ(QUSRSYS/AREFTCRINV)`:
     - Overrides the printer file `QPQUPRFIL` (default output file for queries) to direct output to the output queue `QUSRSYS/AREFTCRINV`.
     - This ensures the credit invoice report is sent to the designated EFT credit invoice queue.

4. **Run Credit Invoice Report Query**:
   - `RUNQRY QRY(GSSQRY/AREFTCRPRT)`:
     - Executes the query `AREFTCRPRT` from `GSSQRY` to generate a report of credit invoices.
     - The report is sent to the overridden output queue `AREFTCRINV`.

5. **Email the Report**:
   - `SFARDST RDSNM('AR CREDIT INV RPT')`:
     - Calls the `SFARDST` command or program to distribute the report named 'AR CREDIT INV RPT'.
     - This step likely emails the credit invoice report to relevant recipients (e.g., financial staff or customers), using a predefined distribution setup.

6. **End of Processing**:
   - `// TAG END`:
     - Marks the end of the script, used as the target for the `GOTO END` statement if no credit invoices are found.

### Business Rules

1. **Credit Invoice Check**:
   - The script only proceeds if credit invoices exist (non-zero value in `?9?ARINVCR`).
   - If no credit invoices are found, the script terminates early.

2. **Report Generation**:
   - The credit invoice report is generated using the `AREFTCRPRT` query.
   - Output is directed to the `AREFTCRINV` output queue for EFT-specific processing.

3. **Report Distribution**:
   - The report is emailed using `SFARDST`, ensuring stakeholders are notified of credit invoices.
   - The report name is fixed as 'AR CREDIT INV RPT'.

4. **Environment Dependency**:
   - The script is only called in production mode (`?9? = G`) by the `AR745.ocl36.txt` script, ensuring credit invoice processing occurs in the live environment.

### Tables (Files) Used

1. **?9?ARINVCR**:
   - Type: Data area or file (likely a data area)
   - Label: `?9?ARINVCR` (substitution variable `?9?` for library)
   - Purpose: Stores a value indicating the presence of credit invoices (e.g., non-zero for found, `00000000` for none).
   - Used by: `AREFTCREDT` query to set the condition.

2. **QPQUPRFIL**:
   - Type: Printer output file
   - Output Queue: `QUSRSYS/AREFTCRINV`
   - Purpose: Default output file for the `AREFTCRPRT` query, overridden to route the credit invoice report.

3. **(Implied) ARDETL or Related Files**:
   - Type: Input (disk, not explicitly declared)
   - Purpose: Likely accessed by the `AREFTCREDT` and `AREFTCRPRT` queries to retrieve credit invoice data.
   - Note: Not directly mentioned in the OCL but inferred from context (e.g., `ARDETL` used in `AR745` for invoice data).

### External Programs Called

1. **RUNQRY (AREFTCREDT)**:
   - Query: `GSSQRY/AREFTCREDT`
   - Purpose: Checks for credit invoices in accounts receivable data, setting a condition code or data area.

2. **RUNQRY (AREFTCRPRT)**:
   - Query: `GSSQRY/AREFTCRPRT`
   - Purpose: Generates the credit invoice report for output to `AREFTCRINV`.

3. **SFARDST**:
   - Command/Program: Distributes the report named 'AR CREDIT INV RPT' (likely via email).
   - Purpose: Sends the credit invoice report to predefined recipients.

### Summary

The `AR745A` OCL script, called by `AR745.ocl36.txt` in production mode, checks for credit invoices, generates a report if they exist, and emails it. It uses queries to identify and format credit invoice data, directs output to a specific queue, and distributes the report via email.

**Business Rules**:
- Processes only if credit invoices are found.
- Generates and emails a report using predefined queries and distribution settings.
- Operates in production mode only.

**Tables Used**:
- `?9?ARINVCR` (data area for condition check)
- `QPQUPRFIL` (printer output for report)
- Implied input files (e.g., `ARDETL`) accessed by queries

**External Programs**:
- `RUNQRY` (for `AREFTCREDT` and `AREFTCRPRT`)
- `SFARDST` (for report distribution)

If you need further details (e.g., specifics of the `AREFTCREDT` or `AREFTCRPRT` queries, or integration with other scripts), let me know!