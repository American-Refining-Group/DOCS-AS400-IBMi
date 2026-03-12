The provided document, `EDI404FTP.clp.txt`, is a CLP (Control Language Program) for IBM midrange systems (e.g., AS/400 or IBM i). It is called by the `EDI404SD.ocl36.txt` OCL program as part of the EDI (Electronic Data Interchange) outbound Bill of Lading (BOL) tracking maintenance workflow. The `EDI404FTP` program handles the FTP transmission of EDI 404-compliant records (likely stored in the `EDIOUT` file, labeled `?9?EDI404`) to an external trading partner, specifically RSI Logistics, and notifies the user or system operator of successful transmission. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called, integrating context from the related `EDI811P`, `EDI811`, `EDI404SD`, and `EDI404` programs.

---

### Process Steps of the EDI404FTP CLP Program

The `EDI404FTP` program is a CL (Control Language) program designed to transmit EDI data via FTP and provide feedback on the transmission’s success. Here’s a detailed breakdown of the process steps:

1. **Program Declaration and Variable Setup**:
   - `PGM`: Declares the start of the CL program.
   - `DCL VAR(&TYPE) TYPE(*CHAR) LEN(2)`: Declares a 2-character variable `&TYPE` to store the job type (batch or interactive).
   - `MONMSG MSGID(CPF0000)`: Monitors for any CPF (Control Program Facility) errors and ignores them, ensuring the program continues execution even if minor errors occur.

2. **Clear Error Output File**:
   - `CLRPFM FILE(GSSLIB/FTP_GIOUT) MBR(ERROR_GI13)`: Clears the member `ERROR_GI13` in the `FTP_GIOUT` file in the `GSSLIB` library. This file likely stores FTP error logs or output from the FTP session.
   - **Purpose**: Ensures the error log is empty before starting the FTP process, avoiding confusion with previous session errors.

3. **Retrieve Job Type**:
   - `RTVJOBA TYPE(&TYPE)`: Retrieves the job type into the `&TYPE` variable, where `0` indicates a batch job and `1` indicates an interactive job.
   - **Purpose**: Determines how to send the success message (to the user for interactive jobs or to the system operator for batch jobs).

4. **Set Up FTP File Overrides**:
   - `OVRDBF FILE(INPUT) TOFILE(GSSLIB/FTP_IN01) MBR(FTP_GIIN13)`: Overrides the logical file `INPUT` to point to the physical file `FTP_IN01` (member `FTP_GIIN13`) in the `GSSLIB` library. This file likely contains FTP commands or input data for the FTP session.
   - `OVRDBF FILE(OUTPUT) TOFILE(GSSLIB/FTP_GIOUT) MBR(ERROR_GI13)`: Overrides the logical file `OUTPUT` to point to `FTP_GIOUT` (member `ERROR_GI13`) in `GSSLIB`, where FTP session output (e.g., logs or errors) is written.
   - **Purpose**: Configures the FTP command to use specific input and output files for the session.

5. **Execute FTP Transmission**:
   - `FTP RMTSYS('B2B.RSILOGISTICS.COM')`: Initiates an FTP session to the remote system `B2B.RSILOGISTICS.com` (RSI Logistics). The FTP commands in `FTP_IN01(FTP_GIIN13)` are executed to transmit the EDI file (likely `EDIOUT`, labeled `?9?EDI404` in the OCL context).
   - **Purpose**: Sends the EDI 404-compliant BOL data to RSI Logistics, completing the outbound transmission.

6. **Remove File Overrides**:
   - `DLTOVR FILE(INPUT)`: Deletes the override for the `INPUT` file.
   - `DLTOVR FILE(OUTPUT)`: Deletes the override for the `OUTPUT` file.
   - **Purpose**: Cleans up the file overrides to prevent interference with other processes.

7. **Notify Success**:
   - `IF COND(&TYPE = '1') THEN(SNDPGMMSG MSG('FTP EDIRSI FILE TRANSMITTED SUCCESSFULLY.'))`: If the job is interactive (`&TYPE = '1'`), sends a program message to the user’s display, indicating successful transmission of the `EDIRSI` file.
   - `ELSE CMD(SNDMSG MSG('FTP EDIRSI FILE TRANSMITTED SUCCESSFULLY.') TOUSR(*SYSOPR))`: If the job is batch (`&TYPE = '0'`), sends a message to the system operator (`*SYSOPR`).
   - **Purpose**: Informs the user or operator that the FTP transmission was successful. Note that the message references `EDIRSI`, which may be a misnomer, as the OCL context suggests `EDIOUT` (labeled `?9?EDI404`) is the file being transmitted.

8. **Program Termination**:
   - `GOTO CMDLBL(ENDJOB)`: Jumps to the `ENDJOB` label.
   - `ENDJOB: ENDPGM`: Ends the program, closing any open resources.
   - **Purpose**: Ensures a clean exit after transmission and notification.

---

### Business Rules

The `EDI404FTP` CLP program enforces the following business rules:
1. **FTP Transmission**:
   - Transmits EDI 404-compliant BOL data (likely in `EDIOUT`, labeled `?9?EDI404`) to the remote system `B2B.RSILOGISTICS.com` (RSI Logistics) using FTP commands stored in `FTP_IN01(FTP_GIIN13)`.
2. **Error Handling**:
   - Ignores CPF errors (`MONMSG CPF0000`) to ensure the program continues even if minor issues occur, prioritizing transmission completion.
   - Clears the `FTP_GIOUT(ERROR_GI13)` file before the FTP session to ensure a clean error log.
3. **Success Notification**:
   - Sends a success message to the user (for interactive jobs) or system operator (for batch jobs), indicating the transmission was successful.
   - The message references `EDIRSI`, which may be a contextual error, as `EDIOUT` is the primary EDI output file per the `EDI811` and `EDI404SD` programs.
4. **File Management**:
   - Uses file overrides to direct FTP input to `FTP_IN01(FTP_GIIN13)` and output to `FTP_GIOUT(ERROR_GI13)`, ensuring proper command execution and logging.
   - Deletes overrides after the FTP session to maintain a clean environment.
5. **Job Type Handling**:
   - Differentiates between batch (`&TYPE = '0'`) and interactive (`&TYPE = '1'`) jobs to send notifications appropriately.

---

### Tables (Files) Used

The CLP program uses the following files:
1. **FTP_IN01**:
   - Location: `GSSLIB/FTP_IN01`, member `FTP_GIIN13`.
   - Purpose: Input file containing FTP commands for the transmission session (e.g., login credentials, file transfer instructions).
   - Accessed via override to the logical file `INPUT`.
2. **FTP_GIOUT**:
   - Location: `GSSLIB/FTP_GIOUT`, member `ERROR_GI13`.
   - Purpose: Output file for storing FTP session logs or errors.
   - Accessed via override to the logical file `OUTPUT`.
3. **EDI404 (Implied)**:
   - Label: `?9?EDI404` (likely `EDIOUT` from `EDI811`, per `EDI404SD.ocl36`).
   - Purpose: The EDI 404-compliant file containing BOL data to be transmitted via FTP. Not directly referenced in the CLP but implied as the file being sent, based on the OCL context.
4. **EDIRSI (Implied)**:
   - Label: `?9?EDIRSI` (from `EDI811` and `EDI404SD`).
   - Purpose: Referenced in the success message, but likely a misnomer, as `EDIOUT` is the primary file transmitted. `EDIRSI` may be used indirectly if the FTP commands in `FTP_IN01` include it.

---

### External Programs Called

The `EDI404FTP` CLP program does not explicitly call other CL or RPG programs but invokes the system’s FTP command:
- **FTP Command**:
  - Executed via `FTP RMTSYS('B2B.RSILOGISTICS.COM')`.
  - Purpose: Runs the FTP client to transmit the EDI file (likely `EDIOUT`) to RSI Logistics, using commands from `FTP_IN01(FTP_GIIN13)` and logging output to `FTP_GIOUT(ERROR_GI13)`.

No other external programs are called directly in the CLP code.

---

### Integration with EDI811P, EDI811, EDI404SD, and EDI404 Programs

- **EDI811P (OCL and RPG)**:
  - `EDI811P.rpg36` validates company numbers (`KYCO`) and BOL numbers, writing them to `EDIBOL`.
  - `EDI811P.ocl36` sets up files (`BICONT`, `EDIBOL`, `EDIBOLTX`) and calls `EDI811`.
- **EDI811 (OCL and RPG)**:
  - `EDI811.rpg36` reads `EDIBOL` and `EDIBOLTX`, generating EDI 404 records in `EDIOUT` (labeled `?9?EDI404`) and routing/shipment data in `EDIRSI`.
  - `EDI811.ocl36` defines these files and calls `EDI404SD`.
- **EDI404 (RPGLE)**:
  - `EDI404.rpgle` displays a screen (`SCREEN1`) for user validation, setting the `CANCEL` field to blanks (proceed) or `'CANCEL'` (cancel).
- **EDI404SD (OCL)**:
  - Checks `EDIRSI` for records, calls `EDI404` for validation, calls `EDI404FTP` for transmission, archives `EDIRSI` to `EDIRSIH`, and clears `EDIRSI`.
  - Clears `EDI404` (likely `EDIOUT`) if `EDI404` sets `CANCEL`.
- **EDI404FTP (CLP)**:
  - Transmits the `EDIOUT` file (labeled `?9?EDI404`) to `B2B.RSILOGISTICS.com` via FTP, logs output, and sends a success message.
- **Workflow Continuity**:
  - `EDI811P` prepares validated BOL data (`EDIBOL`).
  - `EDI811` formats it into EDI 404 records (`EDIOUT`, `EDIRSI`).
  - `EDI404` allows user validation.
  - `EDI404SD` orchestrates validation and transmission, calling `EDI404FTP` to send the data and archive `EDIRSI`.

---

### Summary

- **Process Overview**: The `EDI404FTP` CLP program clears the FTP error log, retrieves the job type, sets up file overrides for FTP input (`FTP_IN01`) and output (`FTP_GIOUT`), transmits the EDI file (likely `EDIOUT`) to `B2B.RSILOGISTICS.com`, removes overrides, and sends a success message to the user (interactive) or system operator (batch).
- **Business Rules**: Transmits EDI 404 data via FTP, handles batch and interactive jobs differently for notifications, clears error logs, uses file overrides for FTP, and ignores minor errors to ensure transmission completion.
- **Tables/Files**: `FTP_IN01` (FTP commands), `FTP_GIOUT` (FTP output/errors), `EDI404` (implied, likely `EDIOUT`), `EDIRSI` (implied, referenced in message).
- **External Programs**: None called directly; uses the system `FTP` command.

If you need further details (e.g., the contents of `FTP_IN01` or the DDS for `EDI404SC`), please provide additional files or context.