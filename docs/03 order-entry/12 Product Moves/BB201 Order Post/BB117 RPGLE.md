### Process Steps of the RPG Program (BB117.rpgle.txt)

This RPGLE program (BB117) updates the customer order history files for line items, handling additions, updates, deletions, and reactivations. It is designed to prevent duplicate orders by logging order details for checking (referenced in comments for BB115). The program is parameterized for flexibility and uses file overrides based on environment (p$fgrp = 'G' or 'Z' for library groups). It is called from BB201 (the main posting program) when deleting line items (per DC02/JB12 modifications: only on delete code 'D'). The process is driven by entry parameters (p$mode for action, p$flag for flags like duplicate override, p$dpok for message handling), with subroutines for each mode. Note: The provided code is truncated (e.g., in updoh and actoh subroutines), so some details are inferred from patterns in similar programs and visible logic.

High-level process steps, executed once per call:

1. **Initialization (*INZSR Subroutine)**:
   - Receives entry parameters via p$elist DS (company p$co, order# p$rdno, seq# p$rseq, customer p$cust, ship-to p$ship, PO# p$pord, order date p$ord8, product p$prod, container p$cntr, qty p$qty, batch p$btch, message flag p$dpok, mode p$mode, file group p$fgrp, flag p$flag).
   - Captures system time/date (PSDS for user, date, time; TIME opcode for t#time DS).
   - Converts dates to CYMD format (t#cymd).
   - Defines keys (kloh for company + order# + seq# on BBOH; klohms adds dates; klohhdr for company + order#).
   - Executes opntbl1 to apply OVRDBF (via QCMDEXC) and open BBOH.

2. **Main Processing (SELECT on p$mode)**:
   - 'UPD' (Update/Add Line Item):
     - Calls updoh: Chains to BBOH using kloh.
       - If not found (99 on): Clears record, moves parm values to fields (via movfldval: bhco=p$co, bhrdno=p$rdno, etc.), sets add user/date (bhusad=ps#usr, bhsdad=t#cymd), writes new record.
       - If found: Sets update user/date (bhusud=ps#usr, bhsdud=t#cymd), updates record.
     - If p$dpok = 'Y' (duplicate poke/override?): Calls updohms after opntbl2 (opens BBOHMS).
       - Chains to BBOHMS using klohms (includes start/add dates).
       - If found and p$flag='0': Updates message (bhmess = com(01): 'Possible Duplicates Accepted').
       - If not found: Adds new record with message.
   - 'DEL' (Delete):
     - If p$rseq = 0 (entire order): Calls delohall.
       - SETLL on klohhdr (company + order#) for BBOH, READE loop, deletes matching records.
       - Similar for BBOHMS after opntbl2.
     - If p$rseq <> 0 (single line): Calls delohrec.
       - Chains to BBOH using kloh, deletes if found.
       - SETLL on kloh for BBOHMS, READE loop with klohhdr, deletes matching messages.
   - 'ACT' (Reactivate):
     - Calls actoh: SETLL on klohhdr for BBOH, READE loop, sets bhstat='A' (active), updates.
     - Similar for BBOHMS, likely sets bhstad to current date (truncated, but inferred as reactivation timestamp).

3. **File Handling Subroutines (opntbl1, opntbl2)**:
   - If p$fgrp = 'G' or 'Z': Loops to apply OVRDBF SHARE(*NO) from arrays (ovg/ovz) via QCMDEXC (e.g., TOFILE(*LIBL/GBBOH) for G).
   - Opens BBOH (opntbl1) or BBOHMS (opntbl2).

4. **End of Processing**:
   - Closes all files (CLOSE *ALL).
   - Sets LR on and returns.
   - No output; updates are in-place.

### Business Rules

- **Modes (p$mode)**:
  - 'UPD': Adds new history if not exists, updates existing (timestamps/user). Adds 'Possible Duplicates Accepted' message to BBOHMS if p$dpok='Y' and p$flag='0' (override allowed duplicates).
  - 'DEL': Deletes entire order history (p$rseq=0) or specific line + messages (p$rseq<>0). Ensures no remnants for cancelled/deleted items.
  - 'ACT': Reactivates by setting status 'A' (and likely updating dates), for order revivals.
- **Duplicate Prevention**: Logs details (customer, ship-to, PO, date, product, container, qty, batch) for BB115 checks. Message added on overrides.
- **File Environments**: p$fgrp='G' (GBBOH/GBBOHMS) or 'Z' (ZBBOH/ZBBOHMS) for segregated libraries (e.g., prod/test). Overrides ensure exclusive access (SHARE(*NO)).
- **Timestamps/Users**: Always stamps add/update with current user (ps#usr) and CYMD date (t#cymd). No changes without timestamp.
- **Deletions**: Hard deletes (no soft flags). Matches by full key for lines, header key for orders.
- **No Validation**: Assumes valid parms; no business checks beyond existence (99). Errors via PSDS (ps#err), but not handled explicitly.
- **Testing**: Commented parms for standalone testing (e.g., p$co=10, p$rdno=999999).

### Tables Used (Files)

Files are disk-based, keyed, user-open (USROPN), with add/update capabilities.

- **BBOH**: Update/Add (UF A E), keyed disk. Customer order history (main details: company, order#, seq#, customer, ship-to, PO, date, product, container, qty, batch, status, users/dates).
- **BBOHMS**: Input/Add (IF A E), keyed disk. Order history messages/supplemental (e.g., duplicate notes, timestamps).

Overrides may alias to GBBOH/GBBOHMS or ZBBOH/ZBBOHMS based on p$fgrp.

### External Programs Called

- **QCMDEXC**: System program called to execute OVRDBF commands for file overrides. Parameters: command string (dbov##, 80 chars) and length (dbol##, 15.5). Used in opntbl1/opntbl2. No other calls.