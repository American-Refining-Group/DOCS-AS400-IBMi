The provided document appears to be an excerpt of an Operation Control Language (OCL) program, likely used on IBM midrange systems like the IBM System/36 or AS/400 (now IBM i). OCL is a control language used to define job steps and manage program execution, typically in conjunction with RPG (Report Program Generator) programs. The snippet shown is from a file named `SA870.ocl36.txt`, which suggests it is an OCL procedure related to a customer inquiry process. Below, I’ll explain the process steps, identify any external programs called, and list the tables (files) used based on the provided information.

### Process Steps of the OCL Program

OCL programs define the setup and execution of jobs, including file assignments, program invocation, and runtime parameters. The provided OCL snippet includes directives for loading and running a program, along with file definitions. Here’s a breakdown of the process steps inferred from the document:

1. **Procedure Initialization**:
   - The OCL starts with a header indicating it’s a “S/A SUMMARY FILE CUSTOMER INQUIRY” procedure. This suggests the program is designed to handle customer-related inquiries, possibly retrieving or displaying customer data.
   - The `// SCPROCP ,,,,,,,,?9?` line likely specifies procedure parameters or options. The `?9?` is a placeholder, possibly for a parameter passed at runtime (e.g., a specific customer ID or inquiry type). In OCL, placeholders like `?n?` are replaced by actual values when the procedure is invoked.
   - `// GSY2K` may indicate a system-specific directive or a reference to a configuration (possibly related to Y2K compliance or a system library). Without further context, its exact purpose is unclear, but it could set environment variables or system settings.

2. **Loading the Program**:
   - `// LOAD SA870` instructs the system to load the program named `SA870`. This is likely an RPG program (or another executable) that performs the core logic of the customer inquiry process. The `LOAD` statement prepares the program for execution.

3. **File Assignments**:
   - The OCL defines two files to be used by the program:
     - `// FILE NAME-SASMC1,LABEL-?9?SASMC1,DISP-SHR`: This assigns a file named `SASMC1` to the program. The `LABEL-?9?SASMC1` suggests the file’s label includes a dynamic component (e.g., a prefix or suffix specified at runtime via the `?9?` placeholder). `DISP-SHR` indicates the file is opened in shared mode, allowing multiple processes to access it concurrently.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Similarly, this assigns a file named `ARCUST` (likely a customer master file) with a dynamic label and shared access.
   - These files are likely database files (tables) containing customer data or related information required for the inquiry process.

4. **Program Execution**:
   - `// RUN` initiates the execution of the loaded `SA870` program. The program uses the files `SASMC1` and `ARCUST` to perform its tasks, such as retrieving customer records, generating reports, or displaying inquiry results.

5. **Program Logic (Inferred)**:
   - Since the OCL itself doesn’t contain the RPG program’s logic, we can infer that `SA870` (likely an RPG program) processes the data from `SASMC1` and `ARCUST`. Typical tasks for a customer inquiry program might include:
     - Reading customer records from `ARCUST` (e.g., customer details like name, account number, or balance).
     - Accessing summary data from `SASMC1` (e.g., transaction summaries or inquiry-specific data).
     - Displaying results on a screen (for interactive inquiries) or generating a report.
   - The `?9?` placeholders in file labels and the procedure suggest dynamic file naming or parameterization, allowing the program to work with different datasets or environments based on input parameters.

### External Programs Called

Based on the provided OCL snippet, the only program explicitly referenced is:
- **`SA870`**: Loaded and executed via the `// LOAD SA870` and `// RUN` statements. This is likely an RPG program responsible for the customer inquiry logic.

No additional external programs are explicitly called in the OCL snippet. However, if `SA870` is an RPG program, it could internally call other subprograms or utilities, but such details would be in the RPG source code, which isn’t provided.

### Tables (Files) Used

The OCL specifies two files (tables) used by the program:
1. **`SASMC1`**:
   - Description: Likely a summary file related to customer data, possibly containing aggregated or inquiry-specific records.
   - Label: `?9?SASMC1` (dynamic label with a placeholder, suggesting variability in file naming).
   - Disposition: `DISP-SHR` (shared access, allowing concurrent read access).
2. **`ARCUST`**:
   - Description: Likely a customer master file containing core customer information (e.g., account details, names, addresses).
   - Label: `?9?ARCUST` (dynamic label with a placeholder).
   - Disposition: `DISP-SHR` (shared access).

These files are likely physical or logical files in the IBM System/36 or AS/400 database, used by the RPG program to retrieve or update customer data.

### Additional Notes
- **Dynamic Parameters**: The `?9?` placeholders indicate the OCL procedure is designed to be flexible, possibly called with different parameters to access specific files or perform different types of inquiries. For example, `?9?` might be replaced with a library name, date, or other identifier.
- **Missing Context**: The snippet is brief, so some assumptions are made about typical OCL and RPG behavior. The full OCL or RPG source code would provide more details about the program’s logic, additional file usage, or subprogram calls.
- **System Context**: The OCL syntax and file structure suggest an IBM System/36 or early AS/400 environment, commonly used for business applications like customer management or accounting.

If you have additional details (e.g., the RPG source code, more OCL lines, or specific questions about the program’s functionality), I can provide a more detailed analysis. Would you like me to search for related information or clarify anything further?