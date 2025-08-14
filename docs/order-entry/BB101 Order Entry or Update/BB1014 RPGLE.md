The `BB1014.rpgle.txt` document is an RPGLE (RPG IV) program called from the `BB101.ocl36.txt` OCL program in an IBM System/36 or AS/400 (now IBM i) environment. It is a shared component of the `BB101`, `BB495`, and `BB500` programs, designed to retrieve location information from the `INLOC` file based on company and location codes. The program was last noted with a reminder to synchronize changes across all three programs (TLN-GSS, 12/06/99). Below is a detailed explanation of the process steps, business rules, tables (files) used, and external programs called.

---

### Process Steps of the RPGLE Program

The `BB1014` program retrieves and formats location data (name, phone number, and inventory type) from the `INLOC` file and returns it to the calling program via a parameter data structure. The steps are as follows:

1. **Program Initialization**:
   - The program defines the `INLOC` file as input-only (512 bytes, indexed, key length 5, starting at position 2).
   - A parameter data structure `PARMS4` is defined to receive and return data:
     - `@4CONO` (positions 1-2, 2 digits): Company number.
     - `@4LOC` (positions 3-5, 3 characters): Location code.
     - `@4LINT` (position 6, 1 character): Inventory type (e.g., '', 'D', 'P').
     - `@4LOCN` (positions 7-26, 20 characters): Location name.
     - `@4LOCP` (positions 27-40, 14 characters): Location phone number.
   - A secondary data structure `PHONE` formats the phone number:
     - `PARL` (position 1, '('): Left parenthesis.
     - `ILAREA` (positions 2-4, 3 digits): Area code.
     - `PARR` (positions 5-6, ') '): Right parenthesis and space.
     - `ILTEL3` (positions 7-9, 3 digits): Phone number prefix.
     - `DASH` (position 10, '-'): Hyphen.
     - `ILTEL4` (positions 11-14, 4 digits): Phone number suffix.
   - Input specifications for `INLOC`:
     - `ILDEL` (position 1, 1 character): Delete code ('D' for deleted).
     - `ILCONO` (positions 2-3, 2 digits): Company number.
     - `ILLOC` (positions 4-6, 3 characters): Location code.
     - `ILNAME` (positions 7-26, 20 characters): Location name.
     - `ILTELE` (positions 363-372, 10 digits): Telephone number.
     - `ILAREA` (positions 363-365, 3 digits): Area code.
     - `ILTEL3` (positions 366-368, 3 digits): Phone number prefix.
     - `ILTEL4` (positions 369-372, 4 digits): Phone number suffix.
     - `ILINTY` (position 458, 1 character): Inventory type.

2. **Parameter Input and Key Construction**:
   - Receives input parameters via `PARMS4` (`@4CONO`, `@4LOC`) through the `*ENTRY PLIST` (line 0033).
   - Constructs a 5-character key (`LOCKEY`) by combining `@4CONO` (company number) and `@4LOC` (location code, lines 0039-0040).

3. **Retrieve Location Information**:
   - Chains to the `INLOC` file using `LOCKEY` (company/location, line 0045).
   - If no record is found (indicator 91):
     - Clears `@4LOCN` (location name), `@4LOCP` (phone number), and `@4LINT` (inventory type) to blanks (lines 0012, 0072, JB).
   - If a record is found (`N91`):
     - Moves `ILNAME` to `@4LOCN` (location name, line 0012).
     - Checks if `ILTELE` (telephone number) is zero:
       - If zero, clears `@4LOCP` to blanks.
       - If non-zero, moves the formatted `PHONE` structure to `@4LOCP` (line JB).
     - Moves `ILINTY` to `@4LINT` (inventory type, line JB).

4. **Phone Number Formatting**:
   - Initializes the `PHONE` structure with formatting characters:
     - `PARL = '('` (line 0034).
     - `PARR = ') '` (line 0034).
     - `DASH = '-'` (line 0034).
   - Populates `ILAREA`, `ILTEL3`, and `ILTEL4` from `INLOC` when a record is found, creating a formatted phone number (e.g., `(123) 456-7890`).

5. **Program Termination**:
   - Sets the last record indicator (`LR`, line 0061) to exit the program.
   - Returns the updated `PARMS4` data structure (`@4LOCN`, `@4LOCP`, `@4LINT`) to the calling program.

---

### Business Rules

The program enforces the following business rules:

1. **Location Record Existence**:
   - The program requires a valid company (`@4CONO`) and location (`@4LOC`) combination to exist in the `INLOC` file.
   - If no record is found (indicator 91), the output fields (`@4LOCN`, `@4LOCP`, `@4LINT`) are cleared to blanks.

2. **Data Retrieval**:
   - Retrieves the location name (`ILNAME`), inventory type (`ILINTY`), and telephone number (`ILTELE`) from `INLOC` for valid records.
   - Formats the telephone number into a standard format (`(area) prefix-suffix`) if non-zero; otherwise, returns blanks.

3. **Consistency Across Programs**:
   - Changes to `BB1014` must be synchronized with `BB495` and `BB500` to ensure consistent location data retrieval across all three programs (TLN-GSS, 12/06/99).

4. **No Validation of Delete Code**:
   - The program does not explicitly check the `ILDEL` field (delete code) in `INLOC`, assuming the calling program handles validation of active records.

5. **No Error Messaging**:
   - The program does not generate or display error messages, relying on the calling program (`BB101`, `BB495`, or `BB500`) to handle invalid or missing data.

---

### Tables (Files) Used

The program interacts with the following file:

1. **INLOC**: Input-only file for inventory location data (512 bytes, indexed, key length 5, starting at position 2).
   - Fields:
     - `ILDEL` (position 1, 1 character): Delete code.
     - `ILCONO` (positions 2-3, 2 digits): Company number.
     - `ILLOC` (positions 4-6, 3 characters): Location code.
     - `ILNAME` (positions 7-26, 20 characters): Location name.
     - `ILTELE` (positions 363-372, 10 digits): Telephone number (subfields: `ILAREA`, `ILTEL3`, `ILTEL4`).
     - `ILINTY` (position 458, 1 character): Inventory type (e.g., '', 'D', 'P').

---

### External Programs Called

The `BB1014` RPGLE program does not call any external programs. All processing is handled internally through file operations and data formatting.

---

### Summary

The `BB1014` RPGLE program, called from the `BB101.ocl36.txt` OCL program, retrieves location information (name, phone number, and inventory type) from the `INLOC` file for use in `BB101`, `BB495`, and `BB500`. It chains to `INLOC` using a company/location key, formats the phone number, and returns data via a parameter data structure. Business rules ensure valid records are retrieved and formatted, with changes requiring synchronization across related programs. The program interacts solely with the `INLOC` file and does not call external programs, relying on internal logic for all operations.