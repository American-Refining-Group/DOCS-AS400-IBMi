The `BB630.ocl36.txt` document is an Operation Control Language (OCL) program for an IBM System/36 or AS/400 environment. It is a simple program that serves as a wrapper to execute the `BB630` program, likely part of the invoice posting process initiated by `BB600P.ocl36.txt`. Below is a detailed explanation of the process steps, external programs called, and tables used.

---

### Process Steps of the BB630 OCL Program

The `BB630.ocl36.txt` OCL program is straightforward and performs the following steps:

1. **Set System Date Handling**:
   - The `// GSY2K` statement indicates that the program uses Year 2000-compliant date handling, ensuring proper processing of dates in the system (likely for handling two-digit year formats with a century indicator).

2. **Load and Execute Program**:
   - The `// LOAD BB630` statement loads the `BB630` program into memory for execution.
   - The `// FILE NAME-ARCONT,LABEL-?9?ARCONT,DISP-SHR` statement specifies that the `ARCONT` file (Accounts Receivable control file) is opened with the label `?9?ARCONT` (where `?9?` is a system prefix, typically the library or environment identifier) and disposition `SHR` (shared access, allowing multiple processes to read the file concurrently).
   - The `// RUN` statement executes the `BB630` program.

3. **Program Termination**:
   - After `BB630` completes, the OCL program terminates, as there are no further instructions or branching logic.

The OCL program itself does not contain complex logic; its primary role is to set up the environment (file access and date handling) and invoke the `BB630` program, which presumably handles specific A/R-related processing, such as updating journal entries or control records after invoice posting.

---

### External Programs Called

1. **BB630**:
   - This is the primary program loaded and executed by the OCL script.
   - While the exact functionality of `BB630` is not specified in the OCL code, it is likely responsible for processing or updating A/R data, given the context of the `ARCONT` file and its invocation within the invoice posting workflow (related to `BB600P.ocl36.txt`).

---

### Tables (Files) Used

1. **ARCONT**:
   - **Description**: Accounts Receivable control file.
   - **Label**: `?9?ARCONT` (system-generated label with a prefix).
   - **Disposition**: `SHR` (shared access, allowing read-only access by multiple processes).
   - **Purpose**: Likely contains control information for A/R processing, such as the next journal number, A/R general ledger number, or last age date, as seen in the `BB600P.rpg36.txt` program (e.g., `ACARJ#`, `ACSLJ#`, `ACDATE`, `ACDTCL`).
   - **Usage**: Opened for use by the `BB630` program, possibly to read or update A/R control data during invoice posting.

---

### Summary

The `BB630.ocl36.txt` OCL program is a minimal script that:
- Configures Year 2000-compliant date handling.
- Opens the `ARCONT` file with shared access.
- Loads and runs the `BB630` program, which likely performs A/R-related processing as part of the invoice posting workflow.

**External Programs Called**: `BB630`.
**Tables Used**: `ARCONT`.

Since the OCL program is a simple wrapper, the specific functionality depends on the `BB630` program, which is not provided but is assumed to interact with the `ARCONT` file to manage A/R data.