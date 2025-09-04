The CLP (Control Language Program) `BI907C2.clp` is a minimal program within the Brandford Order Entry / Invoices system, designed solely to clear a temporary work file (`QTEMP/BI907W`). It is called from the main OCL program `BI890.ocl36.txt`, likely as a cleanup utility within the ship-to master maintenance workflow involving programs like `BI890`, `BI890P`, `BI8903`, `GB730P`, `BB800E`, `BI907AC`, `BI907`, and `BI9078`. Below, I’ll explain the process steps, business rules, tables used, and external programs called, based on the provided code and context from related programs.

---

### **Process Steps of the CLP Program**

The `BI907C2` program is a straightforward CLP with a single function: to clear a temporary work file. Here’s the step-by-step breakdown:

1. **Clear Work File**:
   - Executes the `CLRPFM` command to clear all records from the physical file `QTEMP/BI907W`. This ensures the work file is empty for subsequent use by other programs (e.g., `BI907` via `BI907AC`).

2. **Program Termination**:
   - Ends the program with `ENDPGM`.

---

### **Business Rules**

The program enforces the following business rules, based on the code and context:
1. **Work File Cleanup**:
   - Clears the temporary work file `QTEMP/BI907W` to ensure it contains no residual data from previous sessions, maintaining data isolation and preventing conflicts in subsequent processing by programs like `BI907`.
2. **No Error Handling**:
   - Does not include explicit error handling (e.g., `MONMSG` for `CPF3142` as in `BI907AC`). If the file does not exist, the `CLRPFM` command will fail, potentially causing the program to terminate abnormally.
3. **Session-Specific Data**:
   - Uses `QTEMP`, a job-specific library, to ensure the work file is isolated to the current user session and automatically deleted when the job ends.

---

### **Tables (Files) Used**

The program directly interacts with one file:
1. **BI907W** (Physical File, `QTEMP/BI907W`):
   - A temporary work file used by `BI907` (via `BI907AC`) for processing customer/ship-to product and freight terms data.
   - Cleared using `CLRPFM` to remove all records.

No other files are referenced in the CLP. The OCL (`BI890.ocl36.txt`) declares additional files (e.g., `arcupr`, `gsprod`, `gstabl`, `arcust`, `shipto`, `bicon`, `cuadr`, `bbshsa1`, `trrtcd`, `shipths`, `arcuphs`), but these are not used by `BI907C2`.

---

### **External Programs Called**

The program does not call any external programs. It only executes the `CLRPFM` command, which is a native IBM i system command.

---

### **Summary**

The `BI907C2` CLP program is a simple utility called from the OCL (`BI890.ocl36.txt`) to clear the temporary work file `QTEMP/BI907W`. It:
- Executes a single `CLRPFM` command to ensure the work file is empty for use by `BI907` (via `BI907AC`).
- Enforces rules for work file cleanup and session-specific data isolation.
- Lacks explicit error handling, assuming the file exists or relying on job-level error handling.

**Tables Used**:
- `BI907W` (`QTEMP/BI907W`, temporary work file)

**External Programs Called**:
- None

If you need further analysis of its integration with the system (e.g., why it’s called separately from `BI907AC`, which also clears `BI907W`), or if you have additional code or context, let me know!