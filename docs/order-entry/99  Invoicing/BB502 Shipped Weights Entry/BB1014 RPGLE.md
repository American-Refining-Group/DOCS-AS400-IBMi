The `BB1014.rpgle.txt` is an RPG program (likely RPG III or RPG/36, given the System/36 context) called from the `BB502.ocl36.txt` OCL program. It is designed to retrieve location information from the `INLOC` file, supporting programs like `BB101`, `BB495`, and `BB500`. Below, I’ll explain the process steps, business rules, tables used, and external programs called based on the provided RPG source code.

---

### Process Steps of the RPG Program

The `BB1014` program is a simple utility that retrieves location-specific data (e.g., location name, phone number, inventory type) based on input parameters (company number and location code). It formats the output and returns it to the calling program. Here’s a step-by-step breakdown of the process:

1. **Program Initialization**:
   - The program defines the `INLOC` file as an input file (512 bytes, indexed, read-only) with a key length of 2 bytes (`KEYLOC(2)`).
   - Two data structures are defined:
     - `PARMS4`: An output parameter structure containing:
       - `@4CONO` (2 digits, company number).
       - `@4LOC` (3 characters, location code).
       - `@4LINT` (1 character, inventory type).
       - `@4LOCN` (20 characters, location name).
       - `@4LOCP` (14 characters, location phone number).
     - `PHONE`: A substructure to format the phone number, consisting of:
       - `PARL` (1 character, left parenthesis).
       - `ILAREA` (3 digits, area code).
       - `PARR` (2 characters, ") ").
       - `ILTEL3` (3 digits, phone number prefix).
       - `DASH` (1 character, hyphen).
       - `ILTEL4` (4 digits, phone number suffix).
   - Input specifications (`I`) map fields from the `INLOC` file to program variables:
     - `ILDEL` (delete flag, position 1).
     - `ILCONO` (company number, positions 2–3).
     - `ILLOC` (location code, positions 4–6).
     - `ILNAME` (location name, positions 7–26).
     - `ILTELE`, `ILAREA`, `ILTEL3`, `ILTEL4` (telephone number components, positions 363–372).
     - `ILINTY` (inventory type, position 458).

2. **Parameter Input**:
   - The program accepts input parameters via the `*ENTRY PLIST` using the `PARMS4` data structure, which contains the company number (`@4CONO`) and location code (`@4LOC`) provided by the calling program.

3. **Phone Number Formatting**:
   - The program initializes the phone number formatting characters:
     - `PARL` is set to `'('`.
     - `PARR` is set to `') '`.
     - `DASH` is set to `'-'`.

4. **Retrieve Location Record**:
   - The program constructs a 5-character key (`LOCKEY`) by combining:
     - `@4CONO` (company number, moved to the first 2 positions).
     - `@4LOC` (location code, moved to the last 3 positions).
   - It uses `LOCKEY` to perform a `CHAIN` operation on the `INLOC` file to retrieve the corresponding record.
   - If the record is not found (indicator `91` is on):
     - `@4LOCN` (location name) is set to blanks.
     - `@4LOCP` (phone number) is set to blanks.
     - `@4LINT` (inventory type) is set to blanks.
   - If the record is found (indicator `91` is off):
     - `ILNAME` is moved to `@4LOCN` (location name).
     - If `ILTELE` (telephone number) is zero, `@4LOCP` is set to blanks; otherwise, the formatted phone number (`PHONE`) is moved to `@4LOCP`.
     - `ILINTY` is moved to `@4LINT` (inventory type).

5. **Program Termination**:
   - The program sets the `LR` (Last Record) indicator to terminate and return the `PARMS4` data structure to the calling program with the retrieved or blank values.

---

### Business Rules

The `BB1014` program enforces the following business rules, inferred from the code and comments:

1. **Valid Location Lookup**:
   - The program requires a valid company number (`@4CONO`) and location code (`@4LOC`) to retrieve a record from the `INLOC` file.
   - If no matching record is found (based on the composite key `LOCKEY`), the program returns blank values for location name, phone number, and inventory type, indicating an invalid or non-existent location.

2. **Phone Number Formatting**:
   - The phone number is formatted as `(XXX) XXX-XXXX` using the `PHONE` data structure, combining `ILAREA`, `ILTEL3`, and `ILTEL4` with parentheses and a hyphen.
   - If the telephone number (`ILTELE`) is zero, the phone number field (`@4LOCP`) is returned as blank, indicating no valid phone number exists for the location.

3. **Inventory Type Handling**:
   - The inventory type (`ILINTY`) is retrieved and returned as-is (valid values are blank, 'D', or 'P', as per the comment in the `I` specification).
   - This field is critical for programs like `BB502` to determine how inventory is managed at the specified location.

4. **Consistency Across Programs**:
   - The comment at the top (TLN-GSS-12/06/99) emphasizes that changes to `BB1014` must be coordinated with `BB101`, `BB495`, and `BB500` to ensure consistency, as these programs share similar logic for accessing `INLOC` data.

5. **Error Handling**:
   - The program handles missing records gracefully by returning blank values rather than raising an error, allowing the calling program to decide how to proceed.
   - No explicit validation errors are raised within `BB1014`, as it is a utility program focused on data retrieval.

---

### Tables (Files) Used

The program uses only one file, as defined in the `F` (File) specification:

1. **INLOC**: Location master file (512 bytes, indexed, read-only, key length of 2 bytes). Contains fields like:
   - `ILDEL` (delete flag).
   - `ILCONO` (company number).
   - `ILLOC` (location code).
   - `ILNAME` (location name).
   - `ILTELE`, `ILAREA`, `ILTEL3`, `ILTEL4` (telephone number components).
   - `ILINTY` (inventory type).

---

### External Programs Called

The `BB1014` program does not call any external programs. It is a standalone utility that performs a single `CHAIN` operation on the `INLOC` file and returns data to the calling program via the `PARMS4` data structure.

---

### Summary

The `BB1014` RPG program is a utility called from the `BB502` OCL program (and potentially `BB101`, `BB495`, and `BB500`) to retrieve location information from the `INLOC` file. It accepts a company number and location code, retrieves the corresponding location name, phone number, and inventory type, and formats the phone number for output. If no record is found, it returns blank values. The program enforces simple business rules around valid lookups and phone number formatting, using a single file (`INLOC`) and no external program calls. Its design ensures consistency with related programs, as noted in the change history.

If you need further analysis (e.g., cross-referencing with `BB502` logic, specific validations, or related content from X or web searches), let me know!