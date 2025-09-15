The provided document is an OCL (Operation Control Language) procedure named `AR2011.ocl36.txt`, executed in the IBM System/36 or AS/400 System/36 Environment. It is called by the `AR200.ocl36.txt` procedure as part of the cash receipts posting process, specifically to handle the posting of discount amounts broken out by product on invoices. This procedure is invoked after the main AR posting (`AR200`), journal generation (`AR210`), and GL posting (`AR211`). Below, I’ll explain the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the OCL Program

The `AR2011.ocl36.txt` procedure is a concise control script that checks for the existence of a transaction file and runs the `AR2011` program to process discount amounts by product. Here’s a sequential breakdown of the steps:

1. **Initialization with Y2K Procedure**:
   - `// GSY2K`: Invokes the `GSY2K` procedure to ensure Year 2000 date compliance, likely setting century values or formatting dates for consistency with prior programs (e.g., `AR200P`, `AR200`, `AR210`, `AR211`).

2. **Check for Transaction File Existence**:
   - `// IF ?F'A,?9?CRTRGG'?/00000000 GOTO END`: Tests if the file `?9?CRTRGG` (Cash Receipts Transaction Register) exists in the `DATAF1` library (implied by prior OCL context). The `?F'A` function checks file attributes, and `/00000000` likely ensures the file is present and non-empty.
   - If the file does not exist or is empty, the procedure jumps to the `END` tag, terminating without further processing.

3. **Load and Run AR2011 Program**:
   - `// LOAD AR2011`: Loads the `AR2011` RPG program, which processes discount amounts by product.
   - File assignments:
     - `// FILE NAME-ARTRAN,LABEL-?9?CRTRGG,DISP-SHR`: Assigns the sorted transaction register file (`?9?CRTRGG`) as `ARTRAN`, shared access (DISP-SHR).
     - `// FILE NAME-SA5FIND,LABEL-?9?SA5FIND,DISP-SHR`: Assigns a sales finder file (`?9?SA5FIND`), likely containing invoice-to-product mappings.
     - `// FILE NAME-GSTABL,LABEL-?9?GSTABL,DISP-SHR`: Assigns a general table file (`?9?GSTABL`), possibly for product or discount codes.
     - `// FILE NAME-ARCUST,LABEL-?9?ARCUST,DISP-SHR`: Assigns the customer master file (`?9?ARCUST`).
     - `// FILE NAME-SA5DSC,LABEL-?9?SA5DSC,DISP-SHR`: Assigns a sales discount file (`?9?SA5DSC`), likely for discount details by product.
   - `// RUN`: Executes `AR2011` to process discounts, likely generating reports or updating files with product-specific discount breakdowns.

4. **Termination**:
   - `// TAG END`: Marks the end of the procedure. If the file check fails, execution skips to this tag, and no further processing occurs.

---

### Business Rules

The OCL enforces the following business rules for processing discount amounts by product:

1. **Conditional Execution**:
   - The procedure only runs `AR2011` if the transaction register file (`?9?CRTRGG`) exists and is non-empty. This ensures processing only occurs when valid cash receipt transactions are available from prior steps (e.g., `AR200`).

2. **Environment-Specific Processing**:
   - The `?9?` parameter (e.g., 'GG' per JB01 comment) specifies the library or prefix for file labels, enabling execution in the Atrium environment (a specific subsystem or test bed).
   - The use of `DISP-SHR` for all files indicates read-only access, ensuring no updates to master files (`ARCUST`, `GSTABL`) or temporary files (`CRTRGG`, `SA5FIND`, `SA5DSC`).

3. **Discount Processing**:
   - The `AR2011` program processes discounts (`ADDISC` from `ARTRAN`, as seen in `AR210`) by linking transactions to products via `SA5FIND` and `SA5DSC`.
   - Likely aggregates discount amounts by product code, using `GSTABL` for product or discount mappings and `ARCUST` for customer details.
   - May produce a report or update a file with product-specific discount totals, though no output file is specified in the OCL.

4. **Y2K Compliance**:
   - The `GSY2K` call ensures date fields (e.g., `ATDATE`, `ATDUDT` from `ARTRAN`) are correctly interpreted with century values, consistent with prior programs (`AR200P`, `AR210`).

5. **No File Creation or Deletion**:
   - Unlike `AR200.ocl36.txt`, this procedure does not create or delete files, relying on pre-existing inputs from earlier steps.
   - Assumes `CRTRGG` persists from `AR200` (not yet deleted by final cleanup).

6. **Integration with AR Process**:
   - Runs as a post-processing step after AR and GL updates, focusing on analytical reporting or allocation of discounts by product.
   - Does not affect AR balances (`ARCUST`, `ARDETL`) or GL postings (`GLMAST`), as it likely reads data for reporting or summarization.

---

### Tables Used

In OCL, "tables" refer to disk files (physical or logical) assigned via labels. The procedure uses the following files, all with shared access (DISP-SHR):

1. **Input Files**:
   - `ARTRAN` (`?9?CRTRGG`): Sorted cash receipts transaction register, containing transaction details (e.g., customer `ATCUST`, invoice `ATINV#`, discount `ATDISC`, date `ATDATE`).
   - `SA5FIND` (`?9?SA5FIND`): Sales finder file, likely mapping invoices to product codes or lines.
   - `GSTABL` (`?9?GSTABL`): General table file, possibly containing product codes, discount rates, or other reference data.
   - `ARCUST` (`?9?ARCUST`): Customer master file, providing customer details (e.g., name `ARNAME`).
   - `SA5DSC` (`?9?SA5DSC`): Sales discount file, likely containing discount details by product or invoice line.

2. **No Output Files**:
   - The OCL does not define output disk files, suggesting `AR2011` either produces a report (printer file) or updates an existing file not specified here (e.g., `ARDALY` or a discount summary).

3. **No Printer Files**:
   - No explicit printer file is defined, but `AR2011` may output reports internally (similar to `AR200`, `AR211` with `REPORT`/`REPORTP`).

---

### External Programs Called

The OCL calls the following external programs/procedures:

1. **GSY2K**:
   - Procedure for Y2K date initialization, ensuring proper century handling for transaction dates.

2. **AR2011**:
   - RPG program loaded and executed to process discount amounts by product. Its exact functionality (e.g., report generation, file updates) is not detailed without the RPG source, but it likely reads `ARTRAN`, `SA5FIND`, `SA5DSC`, and `GSTABL` to allocate discounts and uses `ARCUST` for customer context.

---

### Integration with OCL and Other Programs

- **OCL Context**: Called by `AR200.ocl36.txt` after `AR211` posts GL entries, as the final processing step before file cleanup. Receives the `?9?` parameter (e.g., 'GG') for file labels, consistent with the Atrium environment (JB01 comment).
- **Input Dependency**: Relies on `?9?CRTRGG` from `AR200` (sorted transactions) and assumes `SA5FIND`, `SA5DSC`, `GSTABL`, and `ARCUST` are populated from prior data entry or setup.
- **Output**: Likely produces a report or updates a file (e.g., `ARDALY` or a discount-specific file), but no output file is explicitly defined.
- **Switches**: No switch checks or settings, unlike `AR200P`. Cancellation in `AR200P` (U1 switch) would prevent `AR2011` from running if the job terminates early.
- **Flow**: Follows GL posting (`AR211`) and precedes final cleanup in `AR200.ocl36.txt` (deleting `CRTRGG`, etc.).

---

### Notes

- **Purpose**: The procedure focuses on breaking out discount amounts by product, likely for reporting or analysis (e.g., profitability by product line). Without the `AR2011` RPG source, the exact output (report or file) is inferred.
- **Atrium Environment**: The JB01 comment (11/28/23) indicates modification for the Atrium system, using 'GG' as the file prefix to isolate processing.
- **Minimal Logic**: The OCL is lightweight, serving as a gatekeeper (file check) and file assigner for `AR2011`.
- **Assumptions**: Assumes `CRTRGG` contains valid discount data (`ATDISC`) from `AR200`, and `SA5FIND`/`SA5DSC` link invoices to products. `GSTABL` may provide product or discount code mappings.

If you have the `AR2011` RPG source, file layouts (e.g., `SA5FIND`, `SA5DSC`), or additional context, I can refine the analysis of discount processing or output details. Let me know if you need further clarification or specific details!