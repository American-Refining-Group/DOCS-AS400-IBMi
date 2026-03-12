### Process Steps of the OCL Program

This is an Operation Control Language (OCL) procedure for the IBM System/36 environment (not strictly an RPG program, but it loads and runs an RPG program as part of its execution). OCL is used to control job execution, allocate resources like files, and invoke programs or other procedures. The procedure appears to facilitate creating new orders by copying from a current order, likely in a business or inventory context, using parameter passing (via ?9?, which substitutes the 9th parameter from the caller) and file operations.

The steps are executed sequentially, with conditional branching. Here's a step-by-step breakdown:

1. **Invoke SCPROCP procedure**: The procedure starts by calling another OCL procedure named SCPROCP with 8 empty parameters (indicated by commas) and passes the 9th parameter from the caller (?9?). This likely performs initial setup, such as preparing data for copying orders, as SCPROCP may be a custom procedure for processing or validation.

2. **Blank all local variables**: Clears the entire local data area (a memory area for temporary data storage accessible by programs and procedures) by setting it to blanks. This ensures a clean state for subsequent operations, preventing carryover of old data.

3. **Invoke GSY2K procedure**: Calls a procedure named GSY2K with no parameters. This may handle global setup, possibly related to Year 2000 (Y2K) date compliance or system-wide configurations, as "GSY2K" suggests a Y2K-specific utility.

4. **Load the BB170P program**: Loads the RPG program BB170P into memory from the default library. This prepares the program for execution but does not run it yet.

5. **Define the BICONT file**: Allocates a disk file with the logical name BICONT, using a physical label constructed by prefixing the caller's 9th parameter (?9?) to "BICONT" (e.g., if ?9? is "TEST", the label becomes "TESTBICONT"). The file is opened in shared mode (DISP-SHR), allowing concurrent access by other jobs. This file likely serves as a control or index file for order data.

6. **Define the BBORDR file**: Allocates another disk file with the logical name BBORDR, using a physical label ?9?BBORDH (e.g., "TESTBBORDH"). Also opened in shared mode. This file appears related to order headers (BBORDH suggests "order header"), possibly used for reading or writing order records during the copy process.

7. **Run the loaded program**: Executes the previously loaded BB170P RPG program. This step likely performs the core logic, such as copying order data from current records to new ones using the defined files (BICONT for control and BBORDR for order data).

8. **Conditional check for cancellation**: Evaluates if the 6-character substring starting at position 129 in the local data area equals "/CANCEL". If true (e.g., set by BB170P to indicate user cancellation or error), branches to the END label, skipping the next step.

9. **Invoke BB170 procedure**: Calls another OCL procedure named BB170 with 8 empty parameters and the 9th parameter passed from the caller (?9?). This may handle post-processing, such as finalizing the new orders or cleanup, assuming no cancellation occurred.

10. **Reach END label**: Defines the "END" label as a branch target for the conditional GOTO.

11. **Blank all local variables again**: Clears the local data area once more, resetting it for any subsequent jobs or to release resources cleanly at the end of the procedure.

The procedure uses parameter substitution (?9?) to make it reusable (e.g., for different datasets via label prefixing) and includes basic error handling via the cancellation check.

### External Programs Called

- **SCPROCP**: An OCL procedure invoked at the start, likely for initial order copying setup.
- **GSY2K**: An OCL procedure called for system or date-related setup.
- **BB170P**: An RPG program loaded and executed to handle the main data processing (e.g., copying orders).
- **BB170**: An OCL procedure invoked near the end, possibly for additional processing or validation.

### Tables Used

In System/36 terms, "tables" refer to database files (disk files used for data storage). The procedure defines and uses the following:

- **BICONT**: A shared disk file, likely a control or configuration file (e.g., for constants or indexes related to orders). Label: ?9?BICONT.
- **BBORDR**: A shared disk file, likely an order-related file (e.g., for reading or writing order records; BBORDH suggests "order header"). Label: ?9?BBORDH.

These files are accessed by the BB170P RPG program during execution. No other files or tables are explicitly defined.