The provided OCL program, `AP251.ocl36.txt`, is a simple Operations Control Language (OCL) script for an IBM System/36 or AS/400 environment. It is called from the main OCL program (`AP250.ocl36.txt`) as part of an Accounts Payable (A/P) process to update a commission table with payment information. Below, I will explain the **process steps**, **business rules**, **tables (files) used**, and **external programs called** based on the provided OCL code.

---

### **Process Steps of the AP251 OCL Program**

The `AP251.ocl36.txt` program is straightforward and focuses on executing a single program to update a commission table. Here are the process steps:

1. **Load the AP251 Program**:
   - The OCL script uses the `LOAD` command to load the `AP251` program, which is likely an RPG or similar program designed to process payment data and update commission-related records.

2. **File Assignments**:
   - The program specifies two files to be used by `AP251`:
     - **APPAY** (labeled `?9?APPY?WS?`, `DISP-SHR`): A temporary payment file created by the main OCL program (`AP250.ocl36.txt`). It contains payment details such as vendor numbers, check numbers, payment amounts, and discounts.
     - **APTORCY** (labeled `?9?APTORCY`, `DISP-SHR`): A commission-related file, likely a master file containing commission data (e.g., sales orders, commission amounts, or agent details) that needs to be updated based on payments processed in `APPAY`.
   - Both files are opened with shared access (`DISP-SHR`), allowing concurrent access by other processes.

3. **Run the Program**:
   - The `RUN` command executes the `AP251` program, which processes the `APPAY` file and updates the `APTORCY` file based on the payment data.
   - The exact logic of the update (e.g., matching payments to commission records, updating commission statuses, or calculating amounts) is handled within the `AP251` program, which is not detailed in the OCL script.

4. **Completion**:
   - After the `AP251` program completes its execution, the OCL script ends, returning control to the main OCL program (`AP250.ocl36.txt`) for further processing (e.g., sorting data or generating reports).

---

### **Business Rules**

Since the OCL script itself is minimal and does not contain explicit logic, the business rules are inferred based on its role in the A/P process and the files involved:

1. **Commission Table Update**:
   - The primary purpose of `AP251` is to update the commission table (`APTORCY`) with payment information from the `APPAY` file.
   - This likely involves matching payment records (e.g., by vendor number, sales order number, or invoice) to commission records and updating fields such as paid amounts, payment dates, or commission statuses.

2. **Shared File Access**:
   - Both files (`APPAY` and `APTORCY`) are opened with shared access (`DISP-SHR`), ensuring that the program can read and update records without locking other processes that may need access to these files.
   - This supports a multi-user environment typical in A/P systems.

3. **Dependency on Main OCL**:
   - The `APPAY` file is a temporary file (`?9?APPY?WS?`) created by the main OCL program (`AP250.ocl36.txt`), indicating that `AP251` relies on prior processing (e.g., check register generation) to provide valid payment data.
   - The `APTORCY` file is a persistent master file, suggesting it maintains long-term commission data that is updated incrementally as payments are processed.

4. **No User Interaction**:
   - The OCL script does not include prompts or interactive elements, implying it runs automatically as part of the broader A/P process, likely triggered by the main OCL program in either interactive or auto mode (`?2?/AUTO`).

5. **Error Handling**:
   - The OCL script does not include explicit error handling or conditional logic, so any error handling (e.g., missing files, invalid data) is assumed to be managed within the `AP251` program itself.

---

### **Tables (Files) Used**

The OCL program references the following files:

1. **APPAY** (`?9?APPY?WS?`, `DISP-SHR`):
   - A temporary payment file created by the main OCL program (`AP250.ocl36.txt`).
   - Contains payment details such as company number, vendor number, check number, payment amount, discount, and sales order number (based on the `AP250.rpg36.txt` program, which uses `APPAY`).
   - Used as input to identify payments that need to be applied to the commission table.

2. **APTORCY** (`?9?APTORCY`, `DISP-SHR`):
   - A master file, likely containing commission-related data (e.g., sales orders, commission amounts, agent IDs, or vendor mappings).
   - Updated by `AP251` to reflect payment activity, such as marking commissions as paid or updating paid amounts.

---

### **External Programs Called**

The OCL program explicitly calls the following external program:

1. **AP251**:
   - Loaded and executed via the `LOAD` and `RUN` commands.
   - Responsible for processing the `APPAY` file and updating the `APTORCY` file.
   - Likely an RPG program (similar to `AP250.rpg36.txt`) that contains the detailed logic for commission updates, such as matching payment records to commission records and updating relevant fields.

No other external programs are called directly by this OCL script.

---

### **Summary**

The `AP251.ocl36.txt` OCL program is a concise script that serves as a bridge between the A/P check processing (handled by `AP250`) and commission table updates. Its key functions are:

- **Process Steps**: Loads the `AP251` program, assigns the `APPAY` (payment) and `APTORCY` (commission) files with shared access, and runs the program to update commission data based on payment records.
- **Business Rules**: Updates the commission table (`APTORCY`) using payment data from `APPAY`, operates in a shared access environment, and runs automatically as part of the A/P process without user interaction.
- **Tables Used**: `APPAY` (temporary payment file) and `APTORCY` (commission master file).
- **External Programs Called**: `AP251` (likely an RPG program for commission table updates).

The program is tightly integrated with the main OCL program (`AP250.ocl36.txt`) and relies on the `APPAY` file being properly populated by prior steps. If you have access to the `AP251` RPG source code or additional details about `APTORCY`, I can provide a more detailed explanation of the commission update logic. Please let me know if you need further clarification or analysis!