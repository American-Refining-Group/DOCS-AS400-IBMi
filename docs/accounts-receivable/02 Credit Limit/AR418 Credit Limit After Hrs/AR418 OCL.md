This is a classic **IBM System/36 OCL (Operations Control Language)** procedure named **AR418.ocl36**, used to launch an **Order Authorization / Credit Limit Update** program (interactive screen program AR418).

OCL was the job control/batch language on the System/36 (and early AS/400 in SSA compatibility mode). This particular OCL is quite sophisticated and contains many safety checks typical of financial/credit applications that must not be run concurrently.

### Step-by-Step Process Flow Explained

1. **Header & Initial Commands**
   ```
   DLTOVR FILE(*ALL)          Deletes any previous file overrides for this job
   // CALL PGM(GSGENIEC)      Calls a small utility (probably to initialize environment or check authority)
   // IFF ?L'506,3'?/YES   RETURN   If location 506 (usually security flag) says user is not authorized → exit immediately
   ```

2. **Concurrency Protection (Very Important)**
   ```
   // SCPROCP ,,,,,,,,?9?      Reserves a procedure lock (kind of semaphore)
   // LOCAL BLANK-*ALL         Clears all local data areas
   ```
   Then it checks if another job is already running the same critical programs:
   ```
   IF ACTIVE-AR880      → displays message "EITHER OPTION 18 OR 20 IS ALREADY IN PROGRESS"
                           asks user to wait or cancel
                           if user presses 0 → GOTO OUT (exits)
   IF ACTIVE-AR418      → same check for this exact program
   ```
   This prevents two people from updating the same customer's credit limit at the same time → avoids lost updates.

3. **Environment Setup**
   ```
   // GSY2K                     Probably a Year-2000 compliance switch or procedure
   // SWITCH 00000000          Clears all job switches
   // LOCAL OFFSET-103,DATA-'?USER?'   Puts the current user ID into local data area position 103 (passed to program)
   ```

4. **File Overrides** (crucial for System/36 environment)
   Forces the program to use uploaded/converted files in the QS36F library (typical when running S/36 programs under AS/400 SSA):
   ```
   OVRDBF FILE(BICONT) TOFILE(QS36F/GBICONT)
   OVRDBF FILE(BBCSR)  TOFILE(QS36F/GBBCSR)
   OVRDBF FILE(BBSLSM) TOFILE(QS36F/GBBSLSM)
   ```

5. **LOAD AR418**
   Prepares to load the actual interactive program AR418 with its required files and printer overrides.

6. **Additional File and Printer Overrides**
   ```
   FILE NAME-BICONT,... DISP-SHRRM    (share read multiple)
   FILE NAME-BBORCL,...
   FILE NAME-BBCSR,...
   FILE NAME-ARCUST,...
   FILE NAME-BBSLSM,...
   PRINTER NAME-JBLIST,...            Job log printer
   ```
   Conditional printer overrides for email/print routing depending on ?9? (usually a test/production flag):
   ```
   IF ?9?/G   → production outqueues (CSROUTQ, SLMNOUTQ)
   IF ?9?/G   → test outqueues (TESTOUTQ)
   ```

7. **RUN**
   Actually loads and starts the main interactive program **AR418**

8. **Post-Execution (rarely reached in interactive jobs)**
   ```
   // IF ?9?/G  CALL AR418TC     If in test mode, run a test cleanup/report program
   // TAG OUT                    Exit point
   ```

### External Programs Called

| Program     | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| GSGENIEC    | Initial security/authority check or environment initialization         |
| AR418       | Main interactive **Order Authorization / Credit Limit Update** screen program |
| AR418TC     | Test-mode cleanup or report program (only called in test environment)  |
| AR880       | Another related program (options 18/20) – checked for concurrency      |

### Physical Files / Tables Used (via overrides)

| Logical File | Redirected to Physical File | Description (typical in AR packages)                  |
|--------------|-----------------------------|-------------------------------------------------------|
| BICONT       | QS36F/GBICONT               | Probably **Branch/Item Control** or **Billing Control** |
| BBCSR        | QS36F/GBBCSR                | **Customer Sales Rep** or **Customer Service Rep** file |
| BBSLSM       | QS36F/GBBSLSM               | **Salesman Master** file                              |
| BBORCL       | (no override shown)         | **Open Order Credit Limit** or **Order Control** file |
| ARCUST       | (no override shown)         | **Accounts Receivable Customer Master** (key file)    |

The most critical files for credit limit update are almost certainly **ARCUST** (customer master with credit limit, balance, etc.) and **BBORCL** (open orders affecting available credit).

### Summary – What This Procedure Actually Does

1. Ensures only authorized users can run it
2. Prevents two users from running AR418 or the related AR880 at the same time (critical for data integrity)
3. Sets up the correct library/file environment (QS36F converted files)
4. Passes the user ID to the program
5. Launches the interactive **AR418** screen program that lets credit/order-entry staff:
   - Review customer credit status
   - Hold/release orders
   - Temporarily or permanently change credit limits
   - Authorize orders that are over credit limit

This was a very common and important procedure in wholesale/distribution companies running System/36 or SSA on AS/400 in the 1980s–2000s.