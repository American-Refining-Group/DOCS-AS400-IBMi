The provided document is an Operational Control Language (OCL) procedure, specifically `SA880.ocl36.txt`, used in IBM System/36 or System/36 Environment on IBM i for a Sales Analysis Summary Inquiry by Product. Below, I’ll explain the process steps of this OCL program, identify any external programs called, and list the tables (files) used, based on the provided code and relevant context from the search results.

### Process Steps of the SA880 OCL Program

OCL is a control language used to manage the execution of programs and allocate system resources on IBM System/3, System/32, System/34, and System/36 minicomputers, with backward compatibility on IBM i’s System/36 Environment. The OCL procedure in `SA880.ocl36.txt` defines the steps to execute a program (likely an RPG program, given the context) for a sales analysis inquiry. Here’s a breakdown of the process steps based on the provided OCL code:

1. **Procedure Initialization and Comments**:
   - The lines starting with `**` are comments, providing context that this is a "Sales Analysis Summary Inquiry by Product." Comments do not affect execution but document the purpose of the procedure.
   - The `// SCPROCP ,,,,,,,,?9?` statement likely specifies a procedure control expression (PCE) or a library/source file reference. The `?9?` is a substitution variable, typically replaced by a specific value (e.g., a library or user ID) at runtime, indicating dynamic configuration of the procedure’s environment or library.

2. **GSY2K Directive**:
   - The `// GSY2K` statement may be a custom or system-specific directive, possibly related to initializing a Year 2000-compliant environment or setting specific system parameters for the sales analysis program. Without specific System/36 documentation, its exact purpose is unclear, but it likely configures the runtime environment.

3. **Program Load**:
   - The `// LOAD SA880` statement instructs the system to load the program named `SA880` into memory for execution. This is likely an RPG (Report Program Generator) program, given the context of RPG being a common language for business applications on System/36 and the inquiry’s purpose.

4. **File Allocation**:
   - The OCL procedure allocates three files using `// FILE` statements, which define the data resources required by the `SA880` program:
     - `// FILE NAME-SASMY1,LABEL-?9?SASMY1,DISP-SHR`: Allocates a file named `SASMY1` with a label that includes the substitution variable `?9?` (e.g., a library prefix). The `DISP-SHR` (disposition shared) indicates that the file can be accessed by multiple programs simultaneously, ensuring concurrent access without locking.
     - `** FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHR`: This line is commented out (starts with `**`), so it is not executed. If uncommented, it would allocate a file named `GSTABL` with shared access, similar to `SASMY1`.
     - `// FILE NAME-GSPROD,LABEL-?9?GSPROD,DISP-SHR`: Allocates a file named `GSPROD` with a label including `?9?` and shared access, like `SASMY1`.
   - These files are likely database files containing sales data, product information, or related tables used for the inquiry.

5. **Program Execution**:
   - The `// RUN` statement transfers control to the loaded `SA880` program, initiating its execution. The `SA880` program processes the allocated files (`SASMY1` and `GSPROD`) to generate the sales analysis summary inquiry by product. The exact logic (e.g., calculations, filtering, or reporting) depends on the RPG code within `SA880`, which is not provided.

6. **Procedure Termination**:
   - The OCL procedure does not explicitly include a `// RETURN` or end-of-job statement, suggesting it either relies on the `SA880` program to terminate the job or assumes the calling procedure will handle termination. In System/36, if no `RETURN` is specified, the job typically ends after the program completes, unless additional OCL statements follow in a calling procedure.

### External Programs Called

- **SA880**: The only program explicitly called in the OCL procedure is `SA880`, loaded via the `// LOAD SA880` statement and executed with `// RUN`. This is likely an RPG program designed to perform the sales analysis summary inquiry by product, processing data from the specified files.
- No additional sub-procedures or external programs are referenced in the provided OCL code. If `SA880` itself calls other programs (e.g., via RPG’s `CALL` operation), this would require examining the RPG source code, which is not available.

### Tables Used

The OCL procedure references the following files (tables) used by the `SA880` program:
1. **SASMY1**: A file allocated with shared access (`DISP-SHR`). Likely contains sales summary data specific to the inquiry (e.g., aggregated sales by product).
2. **GSPROD**: A file allocated with shared access. Likely contains product master data (e.g., product codes, descriptions, or attributes) used to correlate with sales data.
3. **GSTABL** (commented out): This file is not currently used due to the comment (`**`), but if uncommented, it would be a shared-access file, possibly containing general system tables or additional reference data for the inquiry.

The `?9?` in the `LABEL` parameter suggests a dynamic prefix, possibly a library or environment-specific identifier resolved at runtime (e.g., a user or system library).

### Additional Notes

- **RPG Context**: RPG (Report Program Generator) is a high-level language for business applications, commonly used on IBM midrange systems like System/36. The `SA880` program likely uses RPG’s program cycle and column-oriented syntax to process the files and generate a report or inquiry output, possibly displaying or printing sales data by product.
- **OCL Functionality**: As described in the search results, OCL statements manage program execution and resource allocation. The `DISP-SHR` parameter ensures concurrent access, which is critical for multi-user environments typical in sales analysis applications.[](https://en.wikipedia.org/wiki/Operational_Control_Language)
- **Substitution Variables**: The `?9?` variable indicates dynamic substitution, allowing the procedure to be flexible across different environments or users by resolving file labels at runtime.
- **Limitations**: Without the RPG source code for `SA880`, I cannot detail its internal logic (e.g., calculations, data filtering, or output format). Similarly, the exact content of `SASMY1` and `GSPROD` (e.g., field definitions) is not specified, but their names suggest sales and product data, respectively.

### Summary

- **Process Steps**:
  1. Initialize the procedure with comments and possible environment setup (`// SCPROCP` and `// GSY2K`).
  2. Load the `SA880` program into memory.
  3. Allocate files `SASMY1` and `GSPROD` with shared access.
  4. Execute the `SA880` program to perform the sales analysis inquiry.
  5. Terminate (implicitly or via the calling procedure).
- **External Programs Called**: `SA880` (likely an RPG program).
- **Tables Used**: `SASMY1` (sales summary data), `GSPROD` (product data), and potentially `GSTABL` (if uncommented, for additional reference data).

If you have the RPG source code for `SA880` or additional details about the files, I can provide a more detailed analysis of the program’s logic or data processing. Let me know if you need further clarification or assistance![](https://en.wikipedia.org/wiki/Operational_Control_Language)[](https://en.wikipedia.org/wiki/IBM_RPG)