Function Requirement: Generate and Process A/P Check Register
Overview
This function processes Accounts Payable (A/P) transactions to generate a Check Register, update master files (vendor, open invoices, history, reconciliation, commission), produce a Cash Disbursements Journal, and generate vendor payment detail reports for ACH transactions. It handles checks, ACH, wire transfers, and employee expenses, ensuring accurate financial tracking and reporting.
Inputs

Payment File (APPAY): Contains payment details (company number, vendor number, invoice number, gross amount, discount, payment amount, sales order number, check number, payment date).
Check File (APPYCK): Contains check details (check number, amount, date, vendor, payment type: check, ACH, wire, expense).
Transaction File (APPYTR): Contains check date and hold code (e.g., ACH, wire).
Vendor Files (APVEND, APVEND2, APVNFMX): Vendor details (name, address, balance, email addresses, ACH form type).
Control File (APCONT): Company details (name, next check/journal numbers).
Open Invoice Files (APOPEN, APOPENH, APOPEND, APOPENV): Invoice details (gross amount, partial paid, open amount).
Freight Files (FRCFBH, FRCINH): Freight invoice details (carrier ID, invoice number).
Commission File (APTORCY): Commission details (order number, invoice number, payment status).
System Date/Time: For report headers and date calculations.

Outputs

Check Register (APPRINT): Report listing checks with totals by payment type.
Positive Pay File (APPNCF): Bank reconciliation file with check details.
Cash Disbursements Journal (APCDJR, TEMGEN): Journal entries for cash, discounts, and A/P, including summarized A/P entries.
Updated Files: APVEND, APOPEN*, APHIST*, APCHKR, FRCFBH, FRCINH, APPYDS, APTORCY with updated payment data.
ACH Vendor Reports (REPORT1 to REPORT4): Payment detail reports emailed to up to four vendor email addresses, with vendor-specific messages.

Process Steps

Initialize Data:

Retrieve system date/time for reporting.
Fetch company details from APCONT (e.g., name, next check/journal numbers).


Process Payments and Checks:

Read APPYCK, APPAY, and APPYTR to process payments by company and vendor.
Identify payment type (check, ACH, wire, expense, void) using AXRECD and PTHOLD.
Format check dates (e.g., 20YYMMDD) using century-aware logic (Y2KCEN).


Update Master Files:

Vendor (APVEND): Update last payment amount/date, discounts, balance, year-to-date paid.
Open Invoices (APOPEN*): Update partial payments; mark fully paid invoices for deletion.
History (APHISTH, APHISTD, APHISTV): Record payment history.
Reconciliation (APCHKR): Update check status (open/void), amount, and clear date.
Freight (FRCFBH, FRCINH): Update freight invoices if carrier ID exists.
Discounts (APPYDS): Record missed discounts.
Commission (APTORCY): Update with payment amount, status ('P'), and check number.


Generate Check Register:

Produce APPRINT with company name, payment method, check details (number, date, amount, vendor), and totals by payment type (check, ACH, wire, expense).


Generate Positive Pay File:

Output APPNCF with check number, date, amount, vendor name, and status (issued/void) for bank reconciliation.


Generate Cash Disbursements Journal:

Process APCDJR to produce TEMGEN with detailed and summarized A/P journal entries (company, G/L number, amount, debit/credit).
Output APPRINT with journal headers, detail lines, and totals (debits/credits).


Generate ACH Vendor Reports:

Process APDTWS and APDTWSC to count ACH transactions per vendor.
Retrieve vendor details (APVEND) and up to four email addresses (APVNFMX, AMFMTY = 'ACHE').
Produce REPORT1 to REPORT4 with vendor details, payment details (invoice number, amount, discount, payment), totals, and messages (standard or crude-specific).
Direct reports to spool files for emailing.


Clean Up:

Delete temporary files (APDTWS, APDTWSC) via main OCL program.



Business Rules

Payment Types:

Handle checks ('C'), ACH ('A'), wire transfers ('W'), employee expenses ('E'), and void checks ('F', 'V').
Skip credit/no-pay records ('C' in AXRECD).
ACH, wire, and expenses skip APCHKR updates.


Vendor Updates:

Update APVEND with last payment amount/date, month/year-to-date discounts/payments, and current balance (VNCBAL -= payment + discount).


Invoice Management:

Calculate open amount (OPOPEN = OPGRAM - OPPPTD - payment).
Mark fully paid invoices (OPOPEN = 0) for deletion in APOPEN*.


Freight Processing:

Update FRCFBH or FRCINH for freight invoices with valid carrier IDs, setting status to 'P' (posted).


Commission Updates:

Match payments to APTORCY using company, vendor, and invoice description.
Update with payment amount, status ('P'), and check number.


Journal Entries:

Generate detailed entries for non-A/P transactions and summarized entries for A/P (CDTYPE = 'AP      ') in TEMGEN.
Adjust negative amounts by reversing sign and switching debit/credit code.


ACH Reporting:

Count ACH transactions (AMFMTY = 'ACHE') per vendor in APDTWSC.
Generate up to four reports per vendor, emailed to AMEMLA addresses.
Use standard messages for non-crude vendors, crude-specific messages for VNCAID = 'CRUDE'.


Date Handling:

Use century-aware formatting (Y2KCEN) for all dates (e.g., check date, payment date).


File Access:

Support shared access (DISP-SHR) for concurrent processing.
Delete temporary files after processing.



Calculations

Vendor Balance: VNCBAL = VNCBAL - (OPLPAM + OPDISC) (payment + discount).
Open Invoice Amount: OPOPEN = OPGRAM - OPPPTD - (OPLPAM + OPDISC).
Journal Amount: For negative amounts, reverse sign and switch debit ('D') to credit ('C') or vice versa.
ACH Count: Increment COUNT for each APVNFMX record where AMFMTY = 'ACHE' and AMCOVN matches ADCOVN.
Report Totals: Sum invoice (ADINV$), discount (ADDISC), and payment (ADLPAM) amounts at vendor level (L1INV$M, L1DIS$M, L1PYM$M).

Assumptions

Input files (APPAY, APPYCK, APPYTR, etc.) are populated by prior steps.
Vendor and control files (APVEND, APCONT, APVNFMX) contain valid data.
Reports are emailed via spool files using an external system (not handled by RPG).
No user interaction; process runs automatically.
