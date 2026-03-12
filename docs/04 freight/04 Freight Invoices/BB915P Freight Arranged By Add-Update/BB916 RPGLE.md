### Process Steps of the RPG Program (BB916)

This RPG IV (RPGLE) program, named BB916, is a specialized confirmation and deletion program for "Freight Arranged By" entries in the Bradford Order Entry/Invoices system. It is invoked from a calling program (e.g., the "work with" list program BB915P) to handle soft deletion or reactivation of a specific record identified by company code and freight arranged by code. Unlike physical deletion, it toggles a delete flag ('D' for deleted, 'A' for active) in the record. The program uses a confirmation window (delwdw format) overlaid on the caller's screen, dynamically adjusting the prompt and function keys based on the record's current status. It does not support inquiry mode—it's strictly for delete/reactivate actions—and returns a flag to the caller indicating the outcome.

The high-level execution flow is as follows:

1. **Initialization (*INZSR Subroutine)**:
   - Receives entry parameters: `p$co` (company code), `p$fpcd` (freight arranged by code), `p$fgrp` (Z or G for file group selection), and `p$flag` (output flag to indicate action taken: 'D' for deleted, 'A' for reactivated, or blank for no action).
   - Moves parameters to display fields (e.g., `f$co`, `f$fpcd`).
   - Defines data structures for time/date conversions, message handling, display file feedback, and program status.
   - Sets up constants (e.g., function keys like F12 for cancel, ENTER).
   - Initializes work fields, message queues, current timestamp, and key lists (e.g., KLFRPR for chaining by company + freight code).
   - Prepares output parameters (initially blank) and sets flags for loop control.

2. **Open Database Tables (opntbl Subroutine)**:
   - Based on `p$fgrp` (G or Z), applies file overrides using `QCMDEXC` to point to the correct library (e.g., gbbfrpr for G, zbbfrpr for Z).
   - Opens input files: `bicont` (company data) and `bbfrpr` (freight entries, with update capability).

3. **Retrieve Data for Passed Parameters (rtvdta Subroutine)**:
   - Chains to `bbfrpr` by key (company + freight code) to fetch the record.
   - If the record is not found (indicator 99 ON), closes all files, sets *INLR ON, and returns immediately (exits without action).
   - Sets the window header (`f$hdr`) and function key label (`f$fvar`) based on the delete flag (`fpdel`) from compile-time arrays:
     - If not 'D' (active): Header = "Freight Arranged By Entry Delete", F-key = "F23=Delete", *IN72 OFF.
     - If 'D' (deleted): Header = "Freight Arranged By Entry Re-Activate", F-key = "F22=ReActivate", *IN72 ON.
   - Chains to `bicont` by company code to retrieve the company name (clears if not found).
   - If deleted (`fpdel` = 'D'), sets *IN72 ON to color the display red.

4. **Process Panel Formats (srfmt Subroutine - Main Program Loop)**:
   - Initializes the window loop flag (`winagn` ON).
   - Enters a main DO loop (while `winagn` is ON):
     - Displays messages if queued (wrtmsg) or clears the message area.
     - Writes a window overlay record (wdwovr) to position the confirmation window.
     - Performs EXFMT on the delete window format (delwdw) to display the confirmation and wait for user input.
     - Clears messages and error indicators (50-69).
     - Processes user input:
       - F12: Cancels the action, sets `winagn` OFF (exits loop with no change, `p$flag` remains blank).
       - F23 (if *IN72 OFF, i.e., record active): Chains to `bbfrpr`; if found and not already deleted, sets `fpdel` to 'D', updates the record, sets `p$flag` to 'D', and exits the loop.
       - F22 (if *IN72 ON, i.e., record deleted): Chains to `bbfrpr`; if found and deleted, sets `fpdel` to 'A', updates the record, sets `p$flag` to 'A', and exits the loop.
       - Other keys (e.g., ENTER): Loops again without action.
   - Clears the key after the loop.

5. **Message Handling**:
   - **addmsg**: Sends messages to program message queue using QMHSNDPM (though no custom errors are queued in this program; used for potential extensions).
   - **wrtmsg**: Writes to message subfile control.
   - **clrmsg**: Clears message subfile using QMHRMVPM.

6. **Program End**:
   - Closes all files.
   - Sets *INLR ON and returns (with `p$flag` indicating the result for the caller).

### Business Rules
- **Soft Deletion Only**: Records are not physically deleted from the database; instead, a flag (`fpdel`) is toggled:
  - 'D': Marks as deleted (logical delete).
  - 'A': Marks as active (reactivation).
- **Confirmation Required**: All actions use a pop-up window for user confirmation. The window dynamically changes based on current status:
  - Active record: Prompts to delete with F23.
  - Deleted record: Prompts to reactivate with F22.
- **Record Existence Check**: If the passed key (company + freight code) does not exist in `bbfrpr`, the program exits immediately without displaying anything.
- **No Validation/Editing**: No input fields are editable in the window; it's display-only for confirmation. No business validation beyond existence and current delete status.
- **Color Coding**: Deleted records are displayed in red (*IN72 ON) for visual distinction.
- **Cancel Behavior**: F12 cancels without changes; `p$flag` remains blank.
- **No Inquiry Mode**: Assumes maintenance context from caller; all operations are update-based.
- **Output Flag**: Returns 'D' if deleted, 'A' if reactivated, or blank if canceled or no action. This allows the caller to display confirmations (e.g., "has been deleted" or "has been reactivated").

### Tables (Files) Used
- **bb916d**: Display file (workstation file) for the user interface, including the confirmation window format (delwdw) and message subfile.
- **bicont**: Input-only keyed file (company master, used for chaining to retrieve company name).
- **bbfrpr**: Update keyed file (freight arranged by entries, primary data file for chain and update).

Files are opened with USROPN and overridden based on the file group parameter (to g* or z* prefixed files in *LIBL).

### External Programs Called
- **QCMDEXC**: System program for executing override commands (file redirection).
- **QMHSNDPM**: System program for sending program messages.
- **QMHRMVPM**: System program for removing program messages.

No other business programs are called; this program is self-contained for its delete/reactivate operations but is invoked by callers like BB915P or BB915 for confirmation.