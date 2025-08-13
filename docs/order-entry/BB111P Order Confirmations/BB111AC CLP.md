### Process Steps of the CLP Program BB111AC

This Control Language Program (CLP) for IBM i/AS/400 automates the handling of spool files generated from order confirmation printing (e.g., from BB110 via LIST3 printer file). It uses Spoolflex (a third-party spool file management tool) commands to split, convert to PDF, copy, distribute (e.g., for emailing), and archive the files to a network folder (G:\CUSTSRV\ORDER CONFIRMATIONS). The program runs in a loop to handle errors gracefully, such as when a PDF is already open on the network share, prompting the user to close it and retry. It retrieves the current user and workstation for messaging. Below is a step-by-step breakdown of the execution flow:

1. **Variable Declaration and Job Information Retrieval**:
   - Declares variables: `&WRKSTN` (workstation/job name, char 10), `&USER` (user ID, char 10), `&PICK` (extracted PDF name for error handling, char 15), `&MSGID` (message ID, char 7), `&ERRMSG` (error message data, char 80).
   - Retrieves the current job's user ID (`&USER`) and job name (`&WRKSTN`) using `RTVJOBA`.

2. **Main Processing Loop (Labeled AGAIN)**:
   - Executes `SFASPLIT` to split spool files from output queue `ORDCONOUTQ` (order confirmation outq), selecting files named 'ORDER CONF LIST3', renaming output to 'order confirmation', and moving to `ORDCONSVOQ` (save queue). Combines selections (*NO).
   - Monitors for message `SF00136` (no files selected); if triggered, jumps to `EMPTY` label.

3. **Add Document for Emailing/Archiving**:
   - Executes `FFAEDOC` to add the spool file as an electronic document named 'ARG ORDER CONFIRM', for printer 'ORDER CONFIRMATION', selecting from `QUSRSYS/ORDCONOUTQ` and naming output 'ORDER CONFIRMATION' (likely converts to PDF for emailing/saving).
   - Monitors `SF00136`; if triggered, jumps to `EMPTY`.

4. **Copy Spool File for Naming/Email**:
   - Executes `SFACOPY` to copy spool files from `QUSRSYS/ORDCONOUTQ` named 'ORDER CONF LIST3', renaming to 'ORD CONF NAMING EMAL', no combine.
   - Monitors `SF00136`; if triggered, jumps to `EMPTY`.

5. **Distribute to Multiple Delivery Points**:
   - Executes `SFARDST` three times for different recipients/distribution names: 'ORDCONF EL DELIVERY1', 'ORDCONF EL DELIVERY2', 'ORDCONF EL DELIVERY3', all from `ORDCONOUTQ`.
   - For each:
     - Monitors `CPFA0D4` (object not found or bad condition); jumps to `BAD` label.
     - Monitors `CPFA09E` (object in use, e.g., PDF open); jumps to `ERROR` label.
     - Monitors `SF00136`; jumps to `EMPTY`.

6. **Jump to Successful Completion**:
   - Jumps to `GOOD` label if no errors.

7. **Error Handling (ERROR Label)**:
   - Receives the last message data into `&ERRMSG` and ID into `&MSGID` using `RCVMSG`.
   - If `&MSGID` is not 'CPFA09E', loops back to `ERROR` (safety check, though unlikely).
   - Extracts the PDF name (`&PICK`) from positions 38-49 of `&ERRMSG`.
   - Sends a break message (`SNDBRKMSG`) to the user's workstation queue (`&WRKSTN`) informing them the PDF is open, to close it, and press Enter to continue.
   - Jumps back to `AGAIN` to retry.

8. **Bad Condition Handling (BAD Label)**:
   - Sends a break message to `&WRKSTN` indicating a PDF is open for a selected confirmation, to close it and press Enter to reset and retry.
   - Jumps back to `AGAIN`.

9. **Empty/No Files Handling (EMPTY Label)**:
   - Sends a break message to `&WRKSTN` indicating no order confirmations were selected, press Enter to return to the menu.

10. **Successful Completion (GOOD Label)**:
    - Executes `SFAMOVE` to move spool files named 'ORDER CONF list3' from `QUSRSYS/ORDCONOUTQ` to `QUSRSYS/ORDCONSVOQ` (archive/save queue).
    - Monitors `SF00136` (ignores if no files to move).

11. **Program Termination**:
    - Ends the program (`ENDPGM`).

The program is interactive via break messages, assuming a user at the workstation to respond (e.g., close PDFs). It retries indefinitely on file-open errors until resolved or no files.

### Business Rules

- **Spool File Selection and Processing**: Only processes spool files named 'ORDER CONF LIST3' from `ORDCONOUTQ`. If none exist, informs user and exits (no unnecessary processing).
- **Error Retry Mechanism**: If a PDF is open on the network share (G:\CUSTSRV\ORDER CONFIRMATIONS), prompts the user to close it and automatically retries the entire process. This ensures data integrity and avoids overwrites on locked files.
- **Distribution**: Distributes copies to three predefined delivery points ('ORDCONF EL DELIVERY1/2/3'), likely for emailing to different recipients or systems (e.g., customer service, archives). Fails gracefully if any distribution point is invalid.
- **Archiving**: Moves processed files to `ORDCONSVOQ` for saving/archiving after successful distribution.
- **User Interaction**: Uses break messages to the calling workstation for real-time feedback (e.g., close PDF, no files). Assumes an interactive session from OCL.
- **Environment**: Relies on Spoolflex for PDF conversion/emailing/saving to Windows share (G: drive). Ignores non-critical errors like no files to move.
- **Assumptions**: Spool files are generated prior (e.g., by BB110). No direct data validation; focuses on spool management.

### Tables Used

None. This is a spool file manipulation program using output queues (e.g., ORDCONOUTQ, ORDCONSVOQ), but no database files (physical or logical) are declared, opened, or accessed.

### External Programs Called

None. The program uses system commands (e.g., `RTVJOBA`, `RCVMSG`, `SNDBRKMSG`, `MONMSG`) and Spoolflex-specific commands (e.g., `SFASPLIT`, `FFAEDOC`, `SFACOPY`, `SFARDST`, `SFAMOVE`), but no explicit `CALL` to other programs. Spoolflex commands are likely custom or product-specific but not treated as external program calls in CLP context.