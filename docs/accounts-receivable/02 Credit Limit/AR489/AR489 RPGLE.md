Process Steps of the RPGLE Program
The RPGLE program manages a workstation (interactive screen) application for maintaining credit limit groupings, allowing users to add, update, or delete customer records associated with credit limit groups. Here’s a step-by-step breakdown of the process:
1.	Program Initialization (*inzsr Subroutine): 
o	Code: 
rpgle
c     *inzsr        begsr
c                   z-add     0             cono
c                   z-add     0             cust
c                   endsr
o	Description: 
	The initialization subroutine (*inzsr) sets the company number (cono) and customer number (cust) to zero.
	This ensures a clean state for variables used in the program.
2.	Main Routine Execution: 
o	Code: 
rpgle
c                   if        qsctl = ' '
c                   move      '1'           *in09
c                   move      'R'           qsctl             1
c                   else
c                   move      '0'           *in09
c                   move      '0'           *in01
c                   move      '0'           *in02
c   lr              return
c                   end
o	Description: 
	Checks the control variable qsctl (likely a system or session control flag).
	If qsctl is blank, sets indicator *in09 to '1' (on) and qsctl to 'R', indicating the program is ready to process.
	If qsctl is not blank, clears indicators *in09, *in01, *in02, and sets *inlr (last record) to terminate the program.
	Indicators *in01 and *in02 control screen formats (AR489S1 and AR489S2).
3.	Read Workstation File: 
o	Code: 
rpgle
c   81              read      ar489s1                                lr
c   82              read      ar489s2                                lr
c   81              move      *on           *in01
c   82              move      *on           *in02
o	Description: 
	Reads the workstation file formats AR489S1 (first screen) and AR489S2 (second screen) based on indicators *in81 and *in82.
	Sets *in01 and *in02 to control which screen format is active for display or input.
	Clears message fields (msg1, msg2) and indicators (*in81, *in82, *in90) to prepare for user interaction.
4.	Handle Function Keys: 
o	Code: 
rpgle
c                   if        (*inka = *on)
c                   eval      *in01 = *off
c                   eval      *in02 = *off
c                   eval      *in81 = *on
c                   endif
c                   if        (*inkg = *on)
c                   eval      *inlr = *on
c                   eval      *in01 = *off
c                   eval      *in02 = *off
c                   endif
o	Description: 
	If function key F10 (*inka) is pressed, clears *in01 and *in02 (disables screen formats) and sets *in81 to display the first screen (AR489S1).
	If function key F3 or F12 (*inkg) is pressed, sets *inlr to terminate the program and clears *in01 and *in02.
5.	Validate Company Number: 
o	Code: 
rpgle
c   09              do
c     '00'          chain     gscont                             99
c  n99gxcono        ifne      *zeros
c  n99              z-add     gxcono        cono
c                   endif
c                   end
o	Description: 
	When indicator *in09 is on (initial program entry), the program reads the GSCONT file using key '00' to retrieve a company number (gxcono).
	If found (*in99 off) and gxcono is non-zero, it sets the cono variable to gxcono.
	This validates or sets the company number for subsequent processing.
6.	Screen Processing: 
o	Code: 
rpgle
c  n09              if        (*in01 = *on)
c                   exsr      s1
c                   endif
c                   if        (*in02 = *on)
c                   exsr      s2
c                   endif
c   81              write     ar489s1
c   82              write     ar489s2
o	Description: 
	If *inchelle01 is on, executes subroutine s1 (processes first screen for company/customer validation).
	If *in02 is on, executes subroutine s2 (processes second screen for group maintenance).
	Writes the appropriate screen formats (AR489S1 or AR489S2) to the workstation based on indicators *in81 and *in82.
7.	Subroutine s1 (First Screen Processing): 
o	Code: 
rpgle
csr   s1            begsr
c     cono          chain     arcont                             99
c                   if        (*in99 = *on)
c                   movel     msg(1)        msg2
c                   eval      *in81 = *on
c                   eval      *in90 = *on
c                   goto      ends1
c                   endif
c                   if        cust = *zero
c                   movel     msg(2)        msg2
c                   eval      *in81 = *on
c                   eval      *in90 = *on
c                   goto      ends1
c                   end
c     clgrky        chain     arclgr                             99
c                   move      cgdel         del
c                   movea     arc           ars
c                   eval      *in82 = *on
csr   ends1         endsr
o	Description: 
	Validate Company: Uses cono to chain (read) the ARCONT file. If not found (*in99 on), sets error message msg(1) ("INVALID COMPANY NUMBER ENTERED") in msg2, sets *in81 and *in90 (error indicator), and exits the subroutine.
	Validate Customer: Checks if cust is zero. If true, sets error message msg(2) ("INVALID CUSTOMER") in msg2, sets *in81 and *in90, and exits.
	Retrieve Group Data: Uses the key clgrky (concatenation of cono and cust) to chain the ARCLGR file. If found, moves the delete code (cgdel) to del and the customer array (arc) to ars. Sets *in82 to display the second screen.
	This subroutine validates input and retrieves existing group data for editing.
8.	Subroutine s2 (Second Screen Processing): 
o	Code: 
rpgle
csr   s2            begsr
c                   z-add     0             y                 2 0
c                   dou       y = 25
c                   add       1             y
c     clgrky        chain     arclgr                             99
c                   if        (*in99 = *on)
c                   except    addclg
c                   else
c                   except    updclg
c                   endif
c                   end
c                   eval      *in81 = *on
csr   ends2         endsr
o	Description: 
	Iterates through the ars array (25 elements) to process customer group assignments.
	For each iteration, uses clgrky to chain the ARCLGR file: 
	If not found (*in99 on), executes the addclg exception to add a new record to ARCLGR with del = 'A', cono, cust, and ars values.
	If found, executes the updclg exception to update the existing ARCLGR record with del and ars values.
	Sets *in81 to return to the first screen after processing.
9.	Exception Outputs: 
o	Code: 
rpgle
oarclgr    e            updclg
o                       del                  1
o                       ars                159
oarclgr    eadd         addclg
o                                            1 'A'
o                       cono                 3
o                       cust                 9
o                       ars                159
o	Description: 
	updclg: Updates an existing ARCLGR record with the delete code (del) and customer array (ars).
	addclg: Adds a new ARCLGR record with del = 'A' (active), cono, cust, and ars.
________________________________________
Business Rules
The program enforces the following business rules for credit limit grouping maintenance:
1.	Company Number Validation: 
o	The company number (cono) must exist in the ARCONT file. If not, an "INVALID COMPANY NUMBER ENTERED" error is displayed, and the user is prompted to correct the input.
2.	Customer Number Validation: 
o	The customer number (cust) must be non-zero. If zero, an "INVALID CUSTOMER" error is displayed.
3.	Credit Limit Group Management: 
o	A group is identified by a composite key (clgrky, combining cono and cust).
o	The program allows adding new group records (if not found in ARCLGR) or updating existing ones.
o	The ars array stores up to 25 customer numbers associated with a group.
o	The del field indicates the record’s status (e.g., 'A' for active, or other codes for deletion).
4.	Interactive Processing: 
o	The first screen (AR489S1) accepts cono and cust for validation and retrieves group data.
o	The second screen (AR489S2) displays group details (arname, ars, del) for editing or adding customers.
o	Function keys (F10, F3/F12) control navigation and program termination.
5.	Error Handling: 
o	Errors (invalid company or customer) are displayed via msg1 and msg2 fields, with indicator *in90 enabling error highlighting.
6.	Data Persistence: 
o	Changes to group assignments are saved to the ARCLGR file via addclg (add) or updclg (update) operations.
________________________________________
Tables (Files) Used
The program uses the following files (equivalent to tables in modern database terminology):
1.	ARCLGR: 
o	Description: Stores credit limit grouping data.
o	Attributes: Update file (uf), 240 bytes, keyed on position 2 (likely clgrky, a composite of cgcono and cgcust).
o	Fields: 
	cgdel (1 byte): Delete code (e.g., 'A' for active).
	cgcono (2 bytes): Company number.
	cgcust (6 bytes): Customer number.
	arc (150 bytes): Array of customer numbers in the group.
2.	ARCONT: 
o	Description: Accounts receivable control file, storing company information.
o	Attributes: Input file (if), 256 bytes, keyed on position 2.
o	Fields: 
	arname (30 bytes): Company name.
3.	GSCONT: 
o	Description: General system control file, likely for system-wide settings or defaults.
o	Attributes: Input file (if), 512 bytes, keyed on position 2.
o	Fields: 
	gxdel (1 byte): Delete code.
	gxcono (2 bytes): Company number.
4.	AR489D (Workstation File): 
o	Description: Display file for the interactive screens (AR489S1 and AR489S2), handled by PROFOUNDUI(HANDLER).
o	Formats: 
	AR489S1: Accepts cono, cust, and displays messages (msg1, msg2).
	AR489S2: Displays cono, arname, cust, ars (group customers), del, and messages.
________________________________________
External Programs Called
The RPGLE program does not explicitly call any external programs via CALL operations. All processing is handled within the program, using subroutines (s1, s2, *inzsr) and file I/O operations. However, the program is invoked by the OCL program (AR489.ocl36.txt), which calls GSGENIEC, SCPROCP, and GSY2K before loading AR489.
________________________________________
Summary
The AR489 RPGLE program manages credit limit groupings through an interactive workstation interface:
•	Process Steps: Initializes variables, validates company and customer inputs, retrieves group data, and allows users to add or update group assignments via two screens.
•	Business Rules: Enforces valid company and customer numbers, manages up to 25 customers per group, and handles add/update/delete operations with error messaging.
•	Files Used: ARCLGR (group data), ARCONT (company data), GSCONT (system control), and AR489D (display file).
•	External Programs: None called directly within the RPGLE code, but the OCL program invokes GSGENIEC, SCPROCP, GSY2K, and AR489.

