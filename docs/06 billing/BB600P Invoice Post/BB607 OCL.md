The `BB607.ocl36.txt` is an OCL (Operation Control Language) program for the IBM System/36 environment, executed as part of the invoice posting workflow, likely called by the main OCL program (e.g., `BB600.ocl36.txt`). Its primary function is to prepare files for processing by clearing and reorganizing specific physical files related to shipment and customer data. The program does not directly execute an RPG program but sets up the environment for subsequent processing. Below is a detailed explanation of the process steps, business rules, tables used, and external programs called.

---

### Process Steps of the BB607 OCL Program

The `BB607` OCL program performs file management tasks to ensure that shipment and customer files are ready for processing by clearing and reorganizing them.

1. **File Allocation**:
   - Allocates four files with shared disposition (`DISP-SHR`) for read access:
     - `VISSHCN`: Shipment container file.
     - `BBORDR`: Order master file.
     - `VISSHCH`: Shipment header file.
     - `ARCUFMX`: Customer master file.
   - The `?9?` placeholder in the `LABEL` parameter indicates that the actual library or file label is resolved at runtime, typically by the System/36 environment or job setup.

2. **Clear Physical File (CLRPFM)**:
   - Executes the `CLRPFM` command to clear the contents of the `VISSHSM` file:
     - File: `VISSHSM` (shipment summary file).
     - Action: Deletes all records in `VISSHSM`, resetting it to an empty state.
     - The `?9?` prefix indicates the library is resolved at runtime.

3. **Reorganize Physical File (RGZPFM)**:
   - Executes the `RGZPFM` command to reorganize the `VISSHCN` file:
     - File: `VISSHCN` (shipment container file).
     - Action: Reorganizes the file to optimize storage, remove deleted records, and rebuild indexes for efficient access.
     - The `?9?` prefix indicates the library is resolved at runtime.

4. **Program Termination**:
   - The OCL program ends after executing the `CLRPFM` and `RGZPFM` commands, with no further processing or program calls.

---

### Business Rules

1. **File Preparation**:
   - Ensures `VISSHSM` is cleared of all records to prepare for new shipment summary data.
   - Reorganizes `VISSHCN` to maintain efficient access and storage for shipment container data.

2. **Shared File Access**:
   - Allocates `VISSHCN`, `BBORDR`, `VISSHCH`, and `ARCUFMX` with shared disposition (`DISP-SHR`), allowing concurrent read access by other programs or jobs.

3. **No Data Processing**:
   - The program does not process or manipulate data within the files; it only performs file maintenance tasks (`CLRPFM`, `RGZPFM`).

4. **Runtime Library Resolution**:
   - Uses `?9?` placeholders for library names, indicating that the actual library is determined at runtime, likely based on job or system configuration.

5. **Integration with ARGLMS**:
   - Part of the invoice posting workflow, preparing shipment and customer files for subsequent processing by other programs in the ARGLMS system.

---

### Tables (Files) Used

1. **VISSHCN**:
   - **Description**: Shipment container file.
   - **Purpose**: Stores shipment container data (e.g., container details for orders).
   - **Usage**: Allocated with `DISP-SHR` for read access and reorganized with `RGZPFM` to optimize storage and access.

2. **BBORDR**:
   - **Description**: Order master file.
   - **Purpose**: Stores order master data (e.g., customer, ship-to, order details).
   - **Usage**: Allocated with `DISP-SHR` for read access, likely used by subsequent programs.

3. **VISSHCH**:
   - **Description**: Shipment header file.
   - **Purpose**: Stores shipment header data (e.g., shipment-level information).
   - **Usage**: Allocated with `DISP-SHR` for read access, likely used by subsequent programs.

4. **ARCUFMX**:
   - **Description**: Customer master file.
   - **Purpose**: Stores customer master data (e.g., customer details, billing information).
   - **Usage**: Allocated with `DISP-SHR` for read access, likely used by subsequent programs.

5. **VISSHSM**:
   - **Description**: Shipment summary file.
   - **Purpose**: Stores summary data for shipments.
   - **Usage**: Cleared with `CLRPFM` to remove all records, preparing it for new data.

---

### External Programs Called

The `BB607` OCL program does not explicitly call any external programs. It performs file maintenance tasks (`CLRPFM` and `RGZPFM`) and sets up the environment for other programs in the invoice posting workflow (e.g., RPG programs like `BB610` or `BB620`).

---

### Summary

The `BB607` OCL program, called by the main OCL (e.g., `BB600.ocl36.txt`), prepares shipment and customer files by:
- Allocating `VISSHCN`, `BBORDR`, `VISSHCH`, and `ARCUFMX` with shared read access (`DISP-SHR`).
- Clearing `VISSHSM` to remove all records (`CLRPFM`).
- Reorganizing `VISSHCN` to optimize storage and access (`RGZPFM`).
- Terminating after file maintenance.

**Tables Used**: `VISSHCN` (shipment container), `BBORDR` (order master), `VISSHCH` (shipment header), `ARCUFMX` (customer master), `VISSHSM` (shipment summary).
**External Programs Called**: None.

This program ensures that shipment and customer files are properly initialized for subsequent processing in the ARGLMS invoice posting workflow.