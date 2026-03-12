The `BB5003.rpgle.txt` is an RPGLE program (likely converted from RPG/36, given the System/36 context and structure) called from the `BB502.ocl36.txt` OCL program and referenced in the `BB502.rpg36.txt` program (in the `OODMOV` subroutine). It is a utility module designed to retrieve the selling tank number (`STTANK`) and rail car tank number (`STRCTK`) from the `INSLTZ` file based on a provided key (`SELKEY`). Below, I’ll explain the process steps, business rules, tables used, and external programs called based on the provided RPG source code.

---

### Process Steps of the RPG Program

The `BB5003` program retrieves tank information for order entry by chaining to the `INSLTZ` file using a 17-character key (`SELKEY`). If a valid record is not found, it attempts to find the next previous record. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - The program defines the `INSLTZ` file as an input file (64 bytes, indexed, read-only, key length 1 byte).
   - A data structure (`SELLTK`) is defined to handle input and output:
     - `SKEY9` (positions 1–9): Part of the input key.
     - `SELKEY` (positions 1–17): Full input key for chaining to `INSLTZ`.
     - `STTANK` (positions 18–21): Output selling tank number.
     - `STRCTK` (positions 22–25): Output rail car tank number.
   - Input specifications (`I`) map fields from the `INSLTZ` file:
     - `IKDEL` (position 1): Delete flag ('D' for deleted).
     - `IKKEY9` (positions 2–10): Key field for comparison.
     - `IKTANK` (positions 18–21): Selling tank number.
     - `IKRCTK` (positions 22–25): Rail car tank number.

2. **Parameter Input**:
   - The program accepts the `SELLTK` data structure as a parameter via the `*ENTRY PLIST`, containing the input key (`SELKEY`) and expecting to return `STTANK` and `STRCTK`.

3. **Chain to `INSLTZ` File**:
   - The program uses `SELKEY` to perform a `CHAIN` operation on the `INSLTZ` file to retrieve a matching record.
   - If a record is found (indicator `99` is off):
     - If the record is not deleted (`IKDEL ≠ 'D'`), it moves `IKTANK` to `STTANK` and `IKRCTK` to `STRCTK` and jumps to the `END` tag.
     - If the record is deleted (`IKDEL = 'D'`), it sets indicator `99` and proceeds to the next step.

4. **Find Next Previous Record**:
   - If no record is found or the record is deleted (indicator `99` is on):
     - The program uses `SETLL` with `SELKEY` to position the file pointer at or before the key.
     - It performs a `READP` (read previous) operation to retrieve the next previous record.
     - If a record is read (indicator `66` is off):
       - It compares `IKKEY9` (from the record) with `SKEY9` (from the input key).
       - If they match and the record is not deleted (`IKDEL ≠ 'D'`), it moves `IKTANK` to `STTANK` and `IKRCTK` to `STRCTK`.
       - If the record is deleted, it loops back to the `RKAGN` tag to read the next previous record.
     - If no record is read (indicator `66` is on, beginning of file reached), it sets `STTANK` and `STRCTK` to blanks.

5. **Program Termination**:
   - The program jumps to the `END` tag, sets the `LR` (Last Record) indicator, and returns the updated `SELLTK` data structure with `STTANK` and `STRCTK` to the calling program.

---

### Business Rules

The program enforces the following business rules, inferred from the code and context:

1. **Valid Tank Record Lookup**:
   - The program requires a valid 17-character key (`SELKEY`) to retrieve a record from the `INSLTZ` file.
   - Only non-deleted records (`IKDEL ≠ 'D'`) are considered valid for returning tank numbers.

2. **Fallback to Previous Record**:
   - If the exact key is not found or the record is deleted, the program attempts to find the next previous non-deleted record with a matching `IKKEY9` (first 9 characters of the key).
   - This ensures that a relevant tank number is returned even if the exact match is unavailable.

3. **Default Output for No Valid Record**:
   - If no valid record is found (either no match or all records are deleted, or the beginning of the file is reached), the program returns blank values for `STTANK` and `STRCTK`.

4. **Integration with Order Entry**:
   - The program is part of the `BB500` order entry process (likely related to `BB502` for shipment weights) and is used to determine the selling tank (`STTANK`) and rail car tank (`STRCTK`) for an order, critical for shipment processing.

5. **Error Handling**:
   - The program handles missing or deleted records gracefully by returning blanks rather than raising an error, allowing the calling program (e.g., `BB502`) to decide how to proceed.

---

### Tables (Files) Used

The program uses one file, as defined in the `F` (File) specification:

1. **INSLTZ**: Selling tank file (64 bytes, indexed, read-only, key length 1 byte). Contains fields:
   - `IKDEL` (delete flag, 'D' for deleted).
   - `IKKEY9` (key field for comparison).
   - `IKTANK` (selling tank number).
   - `IKRCTK` (rail car tank number).

---

### External Programs Called

The program does not call any external programs. It is a standalone utility that performs a simple file lookup and returns results via the `SELLTK` data structure.

---

### Summary

The `BB5003` RPGLE program is a utility called from the `BB502` OCL program (and part of the `BB500` order entry suite) to retrieve the selling tank number (`STTANK`) and rail car tank number (`STRCTK`) from the `INSLTZ` file using a 17-character key (`SELKEY`). It enforces rules to return only non-deleted records, falling back to the next previous valid record if necessary, and returns blanks if no valid record is found. The program uses one file (`INSLTZ`) and does not call external programs, making it a lightweight lookup module for order entry and shipment processing.

If you need further details on specific logic, integration with `BB502`, or additional analysis (e.g., X posts or web searches), let me know!