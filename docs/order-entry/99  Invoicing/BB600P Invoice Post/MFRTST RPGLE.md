The `MFRTST.rpgle.txt` RPGLE program is a module, not a standalone program, called by other programs (e.g., `BB603N`) within the invoice posting workflow on an IBM System/36 or AS/400 environment. Its primary function is to perform tolerance testing for the ARGLMS (Accounts Receivable and Load Management System) by comparing actual carrier invoice amounts against estimated and expected freight costs. It evaluates two sets of tolerance tests (Set 1 and Set 2) to ensure freight costs are within acceptable limits, setting pass/fail flags based on the results. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the MFRTST RPGLE Program

The `MFRTST` program performs tolerance testing by comparing actual carrier invoice amounts (`FCAMTA`) with estimated carrier costs (`FCAMTE`) and expected carrier costs from customer invoices. It evaluates two tolerance tests (originally four, but tests 2 and 4 were removed per a 2011 revision) and sets flags for pass/fail outcomes. The program uses a parameter (`PGMCL`) to determine which test set to perform and outputs results to be used by the calling program.

#### Process Steps:
1. **Program Initialization**:
   - The program receives input parameters:
     - `FCAMTA`: Actual carrier invoice amount (9 digits, 2 decimals).
     - `FCAMTE`: Estimated carrier cost from pick list (9 digits, 2 decimals).
     - `PGMCL`: Control parameter (1 character, `'1'` for Set 1, `'2'` for Set 2).
   - Output parameters:
     - `TEST1`: Tolerance Test 1 result (`P` for pass, `F` for fail, blank for not performed).
     - `TEST2`: Tolerance Test 2 result (`P` for pass, `F` for fail, blank for not performed, originally for Test 2 but now tied to Test 1 or 3).
     - `TTE1`: Tolerance Error Code 1 (Actual Carrier Invoice vs. Estimated Carrier Cost, `P` or `F`).
     - `TTE2`: Tolerance Error Code 2 (Actual Carrier Invoice vs. Estimated Billed Freight, `P` or `F`, removed per 2011 revision).
     - `TTE3`: Tolerance Error Code 3 (Actual Carrier Invoice vs. Expected Carrier Cost, `P` or `F`).
     - `TTE4`: Tolerance Error Code 4 (Actual Carrier Invoice vs. Actual Billed Freight, `P` or `F`, removed per 2011 revision).
     - `ALST`: Overall test status (`Y` for pass, blank or unchanged for fail).
   - No files are directly opened; the program operates on passed parameters.

2. **Calculate Difference for Set 1 (PGMCL = '1')**:
   - Computes the absolute difference between actual carrier invoice amount (`FCAMTA`) and estimated carrier cost (`FCAMTE`): `DIFF = |FCAMTA - FCAMTE|`.
   - If `DIFF > 75.00` (per `JB02` revision, updated from 50.00):
     - Sets `TEST1 = 'F'` (fail Tolerance Test 1).
     - Sets `TTE1 = 'F'` (fail Tolerance Error Code 1).
     - Sets `TTE2 = 'F'` (fail Tolerance Error Code 2, automatically fails as per 2011 revision).
   - If `DIFF ≤ 75.00`:
     - Sets `TTE1 = 'P'` (pass Tolerance Error Code 1).
     - Sets `TTE2 = 'P'` (pass Tolerance Error Code 2, automatically passes as per 2011 revision).
   - If both `TTE1` and `TTE2` are `'P'`, sets `ALST = 'Y'` (overall pass) and `TEST1 = 'P'`.

3. **Calculate Difference for Set 2 (PGMCL = '2')**:
   - Computes the absolute difference between actual carrier invoice amount (`FCAMTA`) and expected carrier cost from customer invoice (not explicitly named but implied as `FCAMTE` in Set 2 context): `DIFF = |FCAMTA - FCAMTE|`.
   - If `DIFF > 75.00` (per `JB02`):
     - Sets `TEST2 = 'F'` (fail Tolerance Test 2).
     - Sets `TTE3 = 'F'` (fail Tolerance Error Code 3).
     - Sets `TTE4 = 'F'` (fail Tolerance Error Code 4, automatically fails as per 2011 revision).
   - If `DIFF ≤ 75.00`:
     - Sets `TTE3 = 'P'` (pass Tolerance Error Code 3).
     - Sets `TTE4 = 'P'` (pass Tolerance Error Code 4, automatically passes as per 2011 revision).
   - If both `TTE3` and `TTE4` are `'P'`, sets `ALST = 'Y'` (overall pass) and `TEST2 = 'P'`.

4. **Program Termination**:
   - Sets the last record indicator (`LR`) to end the program.
   - Returns output parameters (`TEST1`, `TEST2`, `TTE1`, `TTE2`, `TTE3`, `TTE4`, `ALST`) to the calling program (e.g., `BB603N`).
   - No files are closed as none are opened.

---

### Business Rules

1. **Tolerance Testing Scope**:
   - **Set 1 (PGMCL = '1')**:
     - Test 1: Compares actual carrier invoice (`FCAMTA`) to estimated carrier cost from pick list (`FCAMTE`).
     - Test 2: Originally compared actual carrier invoice to estimated billed freight but was removed (per 2/9/2011 revision). Now, `TTE2` mirrors `TTE1`.
   - **Set 2 (PGMCL = '2')**:
     - Test 3: Compares actual carrier invoice (`FCAMTA`) to expected carrier cost from customer invoice.
     - Test 4: Originally compared actual carrier invoice to actual billed freight but was removed (per 2/9/2011 revision). Now, `TTE4` mirrors `TTE3`.
   - Tests 2 and 4 are no longer performed; their results (`TTE2`, `TTE4`) are set to match `TTE1` and `TTE3`, respectively, to focus on carrier data validation.

2. **Tolerance Threshold**:
   - The tolerance threshold is 75.00 (updated from 50.00 per `JB02`).
   - If the absolute difference (`DIFF`) between amounts exceeds 75.00, the test fails (`F`); if ≤ 75.00, the test passes (`P`).

3. **Overall Status**:
   - For `PGMCL = '1'`, if both `TTE1` and `TTE2` are `'P'`, sets `ALST = 'Y'` and `TEST1 = 'P'`.
   - For `PGMCL = '2'`, if both `TTE3` and `TTE4` are `'P'`, sets `ALST = 'Y'` and `TEST2 = 'P'`.
   - `ALST` indicates whether the entire test set passed.

4. **Error Handling**:
   - The program assumes valid input parameters (`FCAMTA`, `FCAMTE`, `PGMCL`).
   - No explicit error handling for invalid parameters or calculations.

5. **Integration with ARGLMS**:
   - Designed to be called by `BB603N` or other programs needing tolerance testing for freight costs in the ARGLMS system.
   - Results (`TEST1`, `TEST2`, `TTE1`, `TTE2`, `TTE3`, `TTE4`, `ALST`) are used by the calling program to update freight records (e.g., `FRORST` in `BB603N`).

6. **Compilation Requirement**:
   - Must be compiled to the `GSSLIBTEST` library to ensure proper testing environment integration.

---

### Tables (Files) Used

The `MFRTST` program does not directly interact with any database files or tables. It operates solely on input parameters (`FCAMTA`, `FCAMTE`, `PGMCL`) and returns output parameters (`TEST1`, `TEST2`, `TTE1`, `TTE2`, `TTE3`, `TTE4`, `ALST`). No files are opened or accessed.

---

### External Programs Called

The `MFRTST` program does not call any external programs. It is a self-contained module that performs calculations and sets flags based on input parameters.

---

### Summary

The `MFRTST` RPGLE program, called by programs like `BB603N`, performs tolerance testing for the ARGLMS system by:
- Comparing actual carrier invoice amounts (`FCAMTA`) with estimated (`FCAMTE`) or expected carrier costs for two test sets:
  - Set 1 (PGMCL = '1'): Tests actual carrier invoice vs. estimated carrier cost (Test 1).
  - Set 2 (PGMCL = '2'): Tests actual carrier invoice vs. expected carrier cost from customer invoice (Test 3).
- Setting pass (`P`) or fail (`F`) flags for `TTE1`, `TTE3`, and mirroring `TTE2`, `TTE4` (Tests 2 and 4 removed per 2/9/2011 revision).
- Using a tolerance threshold of 75.00 (per `JB02`).
- Setting overall status (`ALST = 'Y'`) if all tests in a set pass.
- Returning results to the calling program for freight record updates.

**Tables Used**: None (operates on parameters only).
**External Programs Called**: None.

This program ensures freight cost accuracy within the invoice posting workflow, supporting ARGLMS by validating carrier costs against tolerances and providing pass/fail indicators for further processing.