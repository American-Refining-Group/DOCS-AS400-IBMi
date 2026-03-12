The `BB110E.rpg36.txt` is a minimal **RPG II** program running on an IBM System/36, designed to display a "busy screen" for the order entry picking ticket process and handle cancellation. It is called by the `BB110P.ocl36` program when the `BB110` program is active, likely to inform the user that a process is ongoing and to allow cancellation. Below, I’ll explain the process steps, business rules, tables (files) used, and any external programs called.

---

### Process Steps of the BB110E RPG Program

The program is straightforward, focusing on displaying a screen and handling a cancel request. Here’s a step-by-step breakdown of the process:

1. **Initialization and Screen Display**:
   - **Line C (before 0047)**: If indicator 01 is on (screen format active), indicator 10 is on (initial display), and the `KG` key (cancel key) is not pressed, clears indicator 10 to prevent reprocessing the initial display.
   - **Line 0110**: If indicator 09 is off (program is active), sets indicators 10 and 09 to control screen display and program state.

2. **Handle Cancel Request**:
   - **Line 0047**: If the `KG` key is pressed, moves `'CANCEL'` to the `CANCEL` field (offset 300–305 in the Local Data Area, LDA).
   - **Line 0048**: Clears indicators 01 and 10, sets the Last Record (`LR`) indicator to terminate the program, and returns control to the calling OCL program.

3. **Output to SCREEN**:
   - **Lines 0111–0112**: Displays the `BB110EFM` format on the `SCREEN` workstation file when indicators 01 and 10 are on. This is likely a "busy" or "processing" screen to inform the user that the picking ticket process is active.

---

### Business Rules

1. **Busy Screen Display**:
   - The program displays a "busy" screen (`BB110EFM`) to indicate that the picking ticket process (likely `BB110`) is active, preventing user interaction until the process completes or is canceled.

2. **Cancellation**:
   - If the user presses the cancel key (`KG`), the program sets the `CANCEL` field to `'CANCEL'` in the LDA, which is checked by the calling OCL program (`BB110P.ocl36`) to terminate the process.
   - The program exits immediately after cancellation by setting the `LR` indicator.

3. **Indicator Management**:
   - Indicator 10 controls the initial display of the busy screen.
   - Indicator 09 tracks the program’s active state.
   - Indicator 01 indicates the screen format is active.

---

### Tables (Files) Used

1. **SCREEN**:
   - **Type**: Workstation file (WORKSTN).
   - **Purpose**: Displays the `BB110EFM` format, likely a "busy" or "processing" message to the user.
   - **Format**: `BB110EFM`.

---

### External Programs Called

- **None**: The `BB110E` program does not explicitly call any external RPG programs. It is a standalone program that displays a screen and updates the LDA for cancellation, relying on the calling OCL program (`BB110P.ocl36`) for further processing.

---

### Summary

The `BB110E` RPG program is a simple utility that:
- Displays a "busy" screen (`BB110EFM`) to indicate that the picking ticket process is active.
- Allows the user to cancel the process by pressing the `KG` key, updating the `CANCEL` field in the LDA.
- Manages program flow with indicators and terminates upon cancellation.

**Tables Used**: SCREEN.
**External Programs Called**: None.