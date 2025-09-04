The provided document is an Operational Control Language (OCL) procedure file named `BB943INQ.ocl36.txt`, used in the IBM System/36 environment to manage the execution of programs, particularly for a "Customer Sales Agreement Inquiry." Below, I’ll explain the process steps of this OCL program, identify the external programs called, and discuss the tables used, based on the provided content and general knowledge of OCL and RPG programming in the System/36 environment.

### Process Steps of the OCL Program

The OCL procedure `BB943INQ.ocl36.txt` contains a sequence of statements that control the execution of a program for a customer sales agreement inquiry. Here’s a step-by-step breakdown of the process:

1. **Procedure Initiation (`// SCPROCP ,,,,,,,,?9?`)**
   - The `SCPROCP` statement indicates the start of a System/36 procedure, likely a stored procedure member within a library. The `?9?` is a parameter placeholder, commonly used in System/36 OCL to pass a value (e.g., a user ID, job parameter, or mode) to the procedure. This parameter is likely passed to the program being called to customize its behavior.
   - This step sets up the environment for the procedure, ensuring system resources are allocated and control is ready to be transferred to the specified program.

2. **Comment (`** CUSTOMER SALES AGREEMENT INQUIRY`)**
   - This is a comment line, providing a human-readable description of the procedure’s purpose: to handle inquiries related to customer sales agreements. It does not affect execution but aids in documentation.

3. **GSY2K Inclusion (`// GSY2K`)**
   - The `GSY2K` statement likely invokes a global or system-level OCL procedure or library member named `GSY2K`. This could be a standard setup routine that initializes system parameters, such as date formats (potentially related to Year 2000 compliance, given the "Y2K" naming convention), file allocations, or environment settings required for the inquiry program.
   - This step ensures that any necessary system-wide configurations or resources are loaded before executing the main program.

4. **Program Call (`// CALL BB943P PARM('INQ' '?9?')`)**
   - The `CALL` statement invokes an external program named `BB943P`, passing two parameters:
     - `'INQ'`: A literal string indicating the program should run in "inquiry" mode, likely directing `BB943P` to display or retrieve customer sales agreement data without modifying it.
     - `'?9?'`: The same parameter passed to the procedure, forwarded to `BB943P`. This could represent a dynamic value, such as a user ID, customer number, or session identifier, used by the program to filter or process specific data.
   - This step transfers control to the `BB943P` program, which is likely written in RPG II (or possibly RPG III in a System/36 environment), to perform the core logic of the customer sales agreement inquiry.

### External Programs Called

Based on the OCL procedure, the only external program explicitly called is:
- **BB943P**: An RPG (or potentially COBOL) program responsible for executing the customer sales agreement inquiry logic. It accepts parameters to determine its mode of operation (`INQ`) and additional data (`?9?`).

The `GSY2K` reference may also invoke a system program or procedure, but it is not explicitly a user-called program. Without further details, it’s assumed to be a system-level setup routine rather than a user-defined program.

### Tables Used

The provided OCL procedure does not explicitly list any files or tables (e.g., via `// FILE` statements). However, based on the context of a customer sales agreement inquiry and typical RPG/System/36 applications, we can infer the following:

- **No Tables Explicitly Defined**: The OCL code does not include `// FILE` statements, which are typically used to specify files or tables (e.g., `// FILE NAME-PAYMAST,DISP-SHR` for a payroll master file). This suggests that file definitions are either:
  - Defined within the `GSY2K` procedure, which may allocate necessary files.
  - Hardcoded or defined within the `BB943P` program itself, a common practice in RPG II where file specifications are embedded in the program’s F-specs (File Specifications).
- **Likely Tables Involved**: Given the purpose of a "Customer Sales Agreement Inquiry," the `BB943P` program likely accesses tables such as:
  - **Customer Master File**: Contains customer details (e.g., customer ID, name, contact information).
  - **Sales Agreement File**: Stores details of sales agreements (e.g., agreement number, terms, dates, amounts).
  - **Transaction or Order File**: May include related sales order data for the inquiry.
  - **Product Catalog File**: If the inquiry involves product-related details.
  - These files would typically be database files (e.g., physical files in System/36 or IBM i terms) accessed via RPG’s file operations (e.g., `CHAIN`, `READ`, or `SETLL` for inquiries).

Without access to the `BB943P` program’s source code or additional OCL statements, the exact table names cannot be confirmed. If you have the RPG source code for `BB943P` or additional OCL files (e.g., `GSY2K`), I can provide a more precise list of tables.

### Additional Notes
- **System/36 Environment**: The `.ocl36` extension and references to OCL indicate this is running in the IBM System/36 environment (or System/36 compatibility mode on an IBM i/AS400). OCL is used to control program execution, file allocation, and resource management, while RPG II (or RPG III) handles the business logic.[](https://en.wikipedia.org/wiki/Operational_Control_Language)[](https://en.m.wikipedia.org/wiki/Operational_Control_Language)
- **Parameter Usage**: The `?9?` placeholder suggests dynamic parameterization, allowing the procedure to be reusable for different users or contexts. The `'INQ'` parameter ensures the program operates in a read-only inquiry mode, typical for displaying data on a screen (e.g., a subfile-based inquiry screen).[](https://programmers.io/ibmi-ebooks/data-structure-2/)
- **File Access**: In System/36, files are often defined in the RPG program’s File Specification (F-spec) section or in a separate OCL procedure like `GSY2K`. If `GSY2K` includes `// FILE` statements, it may define the tables used by `BB943P`.[](https://en.wikipedia.org/wiki/Operational_Control_Language)

### Summary
- **Process Steps**:
  1. Initiate the procedure with `SCPROCP`, passing a parameter (`?9?`).
  2. Include system setup via `GSY2K`.
  3. Call the `BB943P` program with parameters `'INQ'` and `?9?` to perform the inquiry.
- **External Programs Called**:
  - `BB943P` (RPG program for the inquiry logic).
  - Potentially a system routine via `GSY2K` (not a user-defined program).
- **Tables Used**:
  - Not explicitly listed in the OCL but likely include customer master, sales agreement, and related files, defined either in `BB943P` or `GSY2K`.

If you can provide the RPG source code for `BB943P`, the contents of `GSY2K`, or additional OCL files, I can refine the list of tables and provide more detailed process steps. Let me know if you’d like me to analyze related files or search for additional context!