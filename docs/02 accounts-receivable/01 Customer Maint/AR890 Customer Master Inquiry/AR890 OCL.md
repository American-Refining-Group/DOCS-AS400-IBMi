The provided document is an **OCL (Operation Control Language)** program, specifically `AR890.ocl36.txt`, used on IBM midrange systems (like AS/400 or IBM i) to control the execution of programs and manage file operations. This OCL script is for a "Customer Master Inquiry" process, likely part of an accounts receivable (AR) or customer management system. Below, I’ll explain the process steps, list the external programs called, and identify the tables (files) used.

---

### **Process Steps of the OCL Program**

OCL is a scripting language used to automate tasks, call programs, and manage files on IBM midrange systems. The `AR890` OCL script performs the following steps:

1. **Delete Overrides for All Files (`DLTOVR FILE(*ALL)`):**
   - This command removes any existing file overrides that may have been set in the current job. File overrides allow temporary redirection of file references (e.g., specifying a different library or file). Clearing overrides ensures a clean slate for file assignments in this job.

2. **Call Program `GSGENIEC` (`CALL PGM(GSGENIEC)`):**
   - The OCL script calls an external program named `GSGENIEC`. This program likely performs some initialization, validation, or setup tasks. The exact functionality depends on the program’s implementation, but it could be related to environment setup or user validation.

3. **Conditional Return (`IFF ?L'506,3'?/YES RETURN`):**
   - This line checks a condition based on the value at position 506, column 3 in a local data area or screen buffer (likely a flag or status code). If the condition evaluates to `YES`, the OCL script terminates (`RETURN`). This acts as an early exit mechanism, possibly for user cancellation or an error state.

4. **Call Procedure `SCPROCP` (`SCPROCP ,,,,,,,,?9?`):**
   - The script invokes a procedure named `SCPROCP` with parameters, where `?9?` is a placeholder (likely for a library or specific value passed at runtime). This procedure might handle additional setup or processing related to the customer inquiry.

5. **Run Program `GSY2K` (`GSY2K`):**
   - The script calls another program, `GSY2K`. This could be a utility program, possibly related to Year 2000 compliance or date handling, given the name. It may perform data validation or transformation before the main inquiry process.

6. **Set Local Data Areas:**
   - `LOCAL OFFSET-200,DATA-'        '`: Initializes a local data area at offset 200 with 8 blank spaces. This could be used to clear or set a specific field for subsequent processing.
   - `LOCAL OFFSET-480,DATA-'?9?'`: Sets a local data area at offset 480 with the value `?9?` (likely a library or parameter placeholder). This might be used to pass runtime-specific data to programs or files.

7. **Load Program `AR890` (`LOAD AR890`):**
   - The main program `AR890` is loaded into memory for execution. This is the core program for the Customer Master Inquiry process, likely an RPG (Report Program Generator) program that handles the inquiry logic.

8. **File Definitions:**
   - The script defines multiple files to be used by the `AR890` program (and possibly other called programs). Each file is opened in shared mode (`DISP-SHR`), meaning multiple jobs can access them concurrently. The files are:
     - `ARCUST` (Customer Master File)
     - `ARCUSP` (Customer Supplemental File)
     - `ARCUPR` (Customer Pricing File)
     - `ARCONT` (Contact File)
     - `BICONT` (Commented out; likely a Billing Contact File)
     - `GSPROD` (Product File)
     - `GSTABL` (Table File, possibly for codes or configurations)
     - `GSCONT` (General Contact File)
     - `ARCUFM` (Customer File Maintenance, for program `AR915P`)
     - `ARCUFMX` (Customer File Maintenance Extension, for program `AR915P`)
     - `ARCUP3` (Customer Pricing File, for program `BI907`)
     - `SHIPTO` (Ship-To Address File)
     - `SHIPTHS` (Ship-To History File)
     - `ARCUPHS` (Customer Pricing History File)
   - The `?9?` placeholder in the `LABEL` parameter likely represents a library or prefix defined at runtime, allowing flexibility in file access (e.g., different libraries for different environments).

9. **Execute the Program (`RUN`):**
   - The `RUN` command executes the loaded `AR890` program with the defined files. This program likely performs the customer inquiry, retrieving and displaying customer-related data from the specified files.

10. **Set Program Switch (`SWITCH 00000000`):**
    - This sets the program switch (a set of 8 binary flags) to all zeros (`00000000`). Switches are used to control program behavior or pass status information to the called program (`AR890` or others).

11. **Clear Local Data Area (`LOCAL BLANK-*ALL`):**
    - This clears all local data areas used by the job, resetting them to blanks. This ensures no residual data affects subsequent processes.

12. **End of Script (`TAG END`):**
    - Marks the end of the OCL script. Execution terminates here unless an earlier `RETURN` was triggered.

---

### **External Programs Called**

The OCL script explicitly calls or references the following external programs:
1. **`GSGENIEC`**: Called early in the script, likely for initialization or validation.
2. **`GSY2K`**: Called after the conditional check, possibly for date-related processing or data validation.
3. **`AR890`**: The main program loaded and executed for the Customer Master Inquiry.
4. **`AR915P`**: Referenced in comments, indicating it uses the `ARCUFM` and `ARCUFMX` files. This program is likely called by `AR890` for customer file maintenance.
5. **`BI907`**: Referenced in comments, indicating it uses the `ARCUP3`, `SHIPTO`, `SHIPTHS`, and `ARCUPHS` files. This program is likely called by `AR890` for billing or shipping-related inquiries.

---

### **Tables (Files) Used**

The OCL script defines the following files (tables) used by the `AR890` program and potentially by the referenced programs (`AR915P` and `BI907`):
1. **`ARCUST`**: Customer Master File (core customer data).
2. **`ARCUSP`**: Customer Supplemental File (additional customer details).
3. **`ARCUPR`**: Customer Pricing File (pricing information for customers).
4. **`ARCONT`**: Contact File (customer contact information).
5. **`BICONT`**: Billing Contact File (commented out, not currently used).
6. **`GSPROD`**: Product File (product-related data).
7. **`GSTABL`**: Table File (likely for lookup tables or configuration codes).
8. **`GSCONT`**: General Contact File (possibly for non-customer contacts).
9. **`ARCUFM`**: Customer File Maintenance (used by `AR915P`).
10. **`ARCUFMX`**: Customer File Maintenance Extension (used by `AR915P`).
11. **`ARCUP3`**: Customer Pricing File (used by `BI907`).
12. **`SHIPTO`**: Ship-To Address File (customer shipping addresses).
13. **`SHIPTHS`**: Ship-To History File (historical shipping data).
14. **`ARCUPHS`**: Customer Pricing History File (historical pricing data).

---

### **Summary**

The `AR890.ocl36` script automates the execution of the `AR890` RPG program for a Customer Master Inquiry process. It:
- Initializes the environment by clearing file overrides and calling setup programs (`GSGENIEC`, `GSY2K`).
- Checks for a condition to exit early.
- Defines multiple customer-related files for shared access.
- Loads and runs the `AR890` program, which likely queries and displays customer data.
- References additional programs (`AR915P`, `BI907`) for specific functions like file maintenance and billing/shipping inquiries.

**External Programs Called**: `GSGENIEC`, `GSY2K`, `AR890`, `AR915P` (referenced), `BI907` (referenced).  
**Files Used**: `ARCUST`, `ARCUSP`, `ARCUPR`, `ARCONT`, `BICONT` (commented out), `GSPROD`, `GSTABL`, `GSCONT`, `ARCUFM`, `ARCUFMX`, `ARCUP3`, `SHIPTO`, `SHIPTHS`, `ARCUPHS`.

If you need further details about the RPG program `AR890` or the referenced programs, please provide their source code or additional context, as their exact functionality depends on the RPG logic.