I've completed the analysis with SQL test queries added. Below is the comprehensive list of complex business rules with TEST column:

---

## BB101 - Order Entry Main Program
**Business Function: Order Entry and Management**

| ID | Rule Name | Description | Type | TEST |
|----|-----------|-------------|------|------|
| 1 | Order Number Generation | When order number reaches 899999, reset to 100001 to maintain 6-digit length | Calculation | `SELECT CASE WHEN BCORD# >= 900000 THEN 100001 WHEN BCORD# = 899999 THEN 100001 ELSE BCORD# + 1 END AS NEXT_ORDER FROM BICONT WHERE BCCONO = 10` |
| 2 | Multi-Load Railcar Restriction | Multi-load orders not allowed for railcar carrier code (RC) | Validation | `SELECT * FROM BBORDR WHERE BOCACD = 'RC' AND BOBKCD = 'Y'` |
| 3 | Credit Limit Check | Checks customer credit limit against order total plus AR amount due | Validation | `SELECT A.ARCUST, A.ARCLMT, A.ARTOTD, B.BLTAMT, (A.ARTOTD + B.BLTAMT) AS TOTAL_EXPOSURE FROM ARCUST A JOIN BBORCL B ON A.ARCUST = B.BLCUST WHERE (A.ARTOTD + B.BLTAMT) > A.ARCLMT` |
| 4 | Customer Owned Product Bypass | When customer owned product only is 'Y', bypass credit check and force no charge | Validation | `SELECT * FROM SHIPTO WHERE CSCOON = 'Y'` |
| 5 | Freight Processor Restriction | 'SCO' freight processor cannot be entered on new order | Validation | `SELECT * FROM BBORDR WHERE BOFPCD = 'SCO' AND BOORD = 'Y'` |
| 6 | Carrier Code Validation | Carrier codes 'CC' and 'CT' are not accepted | Validation | `SELECT * FROM BBORDR WHERE BOCACD IN ('CC', 'CT')` |
| 7 | Inactive Carrier Rejection | Inactive carriers cannot be selected | Validation | `SELECT * FROM GSTABL WHERE TBTYPE = 'BBCAID' AND TBDEL IN ('D', 'I')` |
| 8 | Bill-To PO Required | Bill-to PO number is required field | Validation | `SELECT * FROM BBORDR WHERE TRIM(BOPORD) = '' OR BOPORD IS NULL` |
| 9 | Responsible Area-Major Location Required | Responsible area and major location required | Validation | `SELECT * FROM BBORDR WHERE TRIM(BORACD) = '' OR TRIM(BOMLCD) = ''` |
| 10 | EDI Order Access Control | EDI orders only accessible by matching plant when customer owned is 'Y' | Validation | `SELECT * FROM BBORDR B JOIN SHIPTO S ON B.BOCUST = S.CSCUST AND B.BOSHIP = S.CSSHIP WHERE B.BODEL = 'E' AND S.CSCOON = 'Y' AND B.BORACD <> ?LDA_RACD` |
| 11 | Product Move Type Validation | When param 13 is 'PM', only type 'M' orders allowed | Validation | `SELECT * FROM BBORDR WHERE BOTYPE <> 'M'` |
| 12 | Order Process Status Validation | Order process status validated against BBORST type in GSTABL | Validation | `SELECT B.* FROM BBORDR B LEFT JOIN GSTABL G ON G.TBTYPE = 'BBORST' AND G.TBCODE = B.BOPRST WHERE G.TBCODE IS NULL` |
| 13 | Holiday Date Warning | Warning if pickup or delivery date falls on holiday | Validation | `SELECT * FROM BBORDR B JOIN BBDATE D ON B.BORQDT = D.QQBBDA WHERE D.BBTYPE = 'H'` |
| 14 | Total Load Calculation Limit | Error if total load calculation exceeds 7 digits | Validation | `SELECT * FROM BBORDR WHERE BOTMLD > 9999999` |
| 15 | Hand Ticket Duplicate Check | Checks for duplicate hand ticket entry | Validation | `SELECT BOHTIC, COUNT(*) FROM BBORDR WHERE BODEL <> 'D' AND TRIM(BOHTIC) <> '' GROUP BY BOHTIC HAVING COUNT(*) > 1` |
| 16 | Freight Calculation Timing | Freight calculated at end of order before marks review | Calculation | `-- Executed as part of order completion workflow` |
| 17 | Auto-Populate Carrier ID | If carrier ID blank, auto-populate with first top 5 choice | Calculation | `UPDATE BBORDR SET BOCAID = (SELECT FTCA FROM FRGHT_TOP5 WHERE ROWNUM = 1) WHERE TRIM(BOCAID) = ''` |
| 18 | Carrier ID Default for Railcar | Default carrier ID to 'BPRR' when carrier code is 'RC' | Calculation | `UPDATE BBORDR SET BOCAID = 'BPRR' WHERE BOCACD = 'RC' AND TRIM(BOCAID) = ''` |

---

## BB1011 - Pricing and Product Validation
**Business Function: Price Calculation and Product Validation**

| ID | Rule Name | Description | Type | TEST |
|----|-----------|-------------|------|------|
| 19 | Unit of Measure Conversion | For non-standard units, requires GSCTWT or GSUMCV conversion record | Validation | `SELECT * FROM BBORTR T WHERE T.TOUM NOT IN ('LBS','GAL','ECH','KG ','LI ','ML ','OZ ') AND NOT EXISTS (SELECT 1 FROM GSCTWT W WHERE W.WTCONO = T.TOCONO AND W.WTPROD = T.TOPROD AND W.WTCNTR = T.TOCNTR AND W.WTDEL <> 'D') AND NOT EXISTS (SELECT 1 FROM GSUMCV U WHERE U.UCCONO = T.TOCONO AND U.UCPROD = T.TOPROD AND U.UCTUNM = T.TOUM AND U.UCDEL <> 'D')` |
| 20 | Last Sold Date Check | Product/container cannot be ordered if >365 days since last sold | Validation | `SELECT * FROM PRCNTR WHERE PNSTAT NOT IN ('X','A') AND (CURRENT_DATE - DATE(DIGITS(PNLSD8))) > 365` |
| 21 | Container Type Determination | If container type not found, default to 'P' | Calculation | `SELECT COALESCE(TCCNTY, 'P') AS CNTR_TYPE FROM GSCNTR6 WHERE TCCNTR = ?CNTR` |
| 22 | Customer Owned Product Pricing | When customer owned is 'Y', use zero rack price | Calculation | `SELECT CASE WHEN S.CSCOON = 'Y' THEN 0 ELSE R.RKPC END AS RACK_PRICE FROM SHIPTO S LEFT JOIN BBPRCE R ON 1=1` |
| 23 | Export Pricing Location | For export customers, force location '999' for pricing | Calculation | `SELECT CASE WHEN S.CSEXPO = 'Y' THEN '999' ELSE ?LOC END AS PRICE_LOC FROM SHIPTO S` |
| 24 | Billed Customer Price List | If billed customer has price list, use BBBCPR instead of rack price | Calculation | `SELECT * FROM BBBCPR WHERE BPCONO = ?CO AND BPBCUS = ?BCUS AND BPGRUP = ?GRUP AND BPPROD = ?PROD` |
| 25 | Price List Quantity Minimum | Order quantity must meet minimum on price list | Validation | `SELECT * FROM BBPRCE WHERE RKMINQ > ?ORDER_QTY` |
| 26 | Rack Price Retrieval | Retrieves rack price by company, location, product, container, UM, date/time | Calculation | `SELECT RKPC FROM BBPRCE WHERE RKCONO = ?CO AND RKLOC = ?LOC AND RKPROD = ?PROD AND RKCNTR = ?CNTR AND RKUNMS = ?UM AND RKDATE <= ?DATE AND RKDEL <> 'D' ORDER BY RKDATE DESC, RKTIME DESC FETCH FIRST 1 ROW ONLY` |
| 27 | Inactive Product Handling | When inactive='I' reject; when 'B' warn product ships until stock gone | Validation | `SELECT RKINAC FROM BBPRCE WHERE RKINAC IN ('I','B')` |
| 28 | No Rack Price Required | Some products don't require rack price when RKRKRQ='N' | Validation | `SELECT * FROM BBPRCE WHERE RKRKRQ = 'N'` |
| 29 | Sales Agreement Pricing | Sales agreement uses company, customer, location, container, PO, start date | Calculation | `SELECT BAPRCE FROM BICUAY WHERE BACO = ?CO AND BACUST = ?CUST AND SUBSTR(BAKEY,3,3) = ?LOC AND BACNTR = ?CNTR AND BAPORD = ?PO AND BASTD8 <= ?DATE AND (BAEND8 = 0 OR BAEND8 >= ?DATE) AND BADEL <> 'D'` |
| 30 | Sales Agreement Unit Match | Sales agreement only used when UM matches detail line | Validation | `SELECT * FROM BICUAY WHERE BAUNMS = ?UM` |
| 31 | Sales Agreement Quantity Range | Order quantity must be within min/max range in sales agreement | Validation | `SELECT * FROM BICUAY WHERE ?QTY < BAMNQY OR ?QTY > BAMXQY` |
| 32 | Sales Agreement Order Number | Checks for sales agreement with order number before PO number | Calculation | `SELECT * FROM BICUAY WHERE BAPOOR = 'O' AND BAPORD = ?ORDER_NUM UNION SELECT * FROM BICUAY WHERE BAPOOR = 'P' AND BAPORD = ?PO_NUM` |
| 33 | Price Code Assignment | Assigns price code: O=override, R=rack, S=sales agreement | Calculation | `SELECT CASE WHEN ?OPRC > 0 THEN 'O' WHEN EXISTS(SELECT 1 FROM BICUAY) THEN 'S' ELSE 'R' END AS PRICE_CODE` |
| 34 | Hazmat Code Retrieval | Retrieves hazmat code (Y/N) from GSCTUM | Calculation | `SELECT CUHAZM FROM GSCTUM WHERE CUCONO = ?CO AND CUPROD = ?PROD AND CUCNTR = ?CNTR AND CUUNMS = ?UM` |

---

## BB1012 - Customer Product Information
**Business Function: Customer-Specific Product Data Retrieval**

| ID | Rule Name | Description | Type | TEST |
|----|-----------|-------------|------|------|
| 35 | Customer Product Description | Retrieves customer-specific product description from ARCUPR | Calculation | `SELECT CPCPDS FROM ARCUPR WHERE CPCONO = ?CO AND CPCUST = ?CUST AND CPSHIP = ?SHIP AND CPPROD = ?PROD AND CPCNTY = ?CNTY AND CPDEL <> 'D'` |
| 36 | Freight Code Hierarchy | Checks customer/ship-to/product, then customer/000/product, then with blank container | Calculation | `SELECT CPFRCD, CPSFRT, CPCAFR FROM ARCUPR WHERE CPCONO = ?CO AND CPCUST = ?CUST AND CPSHIP IN (?SHIP, 0) AND CPPROD = ?PROD AND CPCNTY IN (?CNTY, ' ') AND CPDEL <> 'D' ORDER BY CPSHIP DESC, CPCNTY DESC FETCH FIRST 1 ROW ONLY` |
| 37 | Container Type Fallback | If not found with container type, retry with blank | Calculation | `SELECT * FROM ARCUPR WHERE CPCNTY = ' ' AND CPDEL <> 'D'` |

---

## BB1013 - Credit Limit Processing
**Business Function: Credit Limit Validation and Tracking**

| ID | Rule Name | Description | Type | TEST |
|----|-----------|-------------|------|------|
| 38 | Credit Limit Calculation | Total credit exposure = AR total due + current order value vs limit | Calculation | `SELECT ARCUST, ARCLMT, ARTOTD, (ARTOTD + ?ORDER_VALUE) AS TOTAL_EXPOSURE, CASE WHEN (ARTOTD + ?ORDER_VALUE) > ARCLMT THEN 'OVER' ELSE 'OK' END AS STATUS FROM ARCUST WHERE ARCUST = ?CUST` |
| 39 | Past Due Invoice Check | Checks past due invoices in aging buckets | Validation | `SELECT ARCUST, AR0110, AR1120, AR2130, AROV30, (AR0110 + AR1120 + AR2130 + AROV30) AS PAST_DUE FROM ARCUST WHERE (AR0110 + AR1120 + AR2130 + AROV30) > 0` |
| 40 | Credit Limit Cross Reference | Maintains BBORCL tracking orders against credit limit | Calculation | `SELECT * FROM BBORCL WHERE BLCOCU = ?COCU AND BLORDR = ?ORDER` |
| 41 | Order Value Calculation | Order value = product total + misc total + freight total | Calculation | `SELECT SUM(TDPRCE * TDQTY) AS PROD_TOT, (SELECT SUM(TMAMT) FROM BBORTR WHERE TSEQ >= 900) AS MISC_TOT, (SELECT BFFAMT FROM BBORF) AS FRT_TOT FROM BBORTR WHERE TSEQ < 900` |
| 42 | Delinquent Customer Handling | Identifies delinquent customers with specific messages | Validation | `SELECT ARCUST, CASE WHEN AROV30 > 0 THEN 'DELINQ' WHEN (ARTOTD + ?ORDER) > ARCLMT THEN 'LIMIT' ELSE 'OK' END AS STATUS FROM ARCUST` |
| 43 | Credit Authorization Tracking | Tracks overrides with user ID and authorization initials | Calculation | `SELECT BLORDR, BLOVCL, BLAUIN, BLUSID FROM BBORCL WHERE BLOVCL = 'Y'` |
| 44 | Customer Group Credit | Checks credit for customer groups with up to 25 customers | Validation | `SELECT C.CGCUST, SUM(A.ARTOTD) AS GROUP_TOTAL FROM ARCLGR C JOIN ARCUST A ON A.ARCUST = C.CGCUST WHERE C.CGGRP = ?GROUP GROUP BY C.CGCUST` |

---

## BB1014 - Location Information
**Business Function: Location Data Retrieval**

| ID | Rule Name | Description | Type | TEST |
|----|-----------|-------------|------|------|
| 45 | Location Validation | Validates location exists and retrieves name, phone, inventory type | Validation | `SELECT ILLOC, ILNAME, ILTELE, ILINTY FROM INLOC WHERE ILCONO = ?CO AND ILLOC = ?LOC AND ILDEL <> 'D'` |
| 46 | Phone Number Formatting | Formats phone as (area) prefix-suffix | Calculation | `SELECT '(' || SUBSTR(DIGITS(ILTELE),1,3) || ') ' || SUBSTR(DIGITS(ILTELE),4,3) || '-' || SUBSTR(DIGITS(ILTELE),7,4) AS FORMATTED_PHONE FROM INLOC` |

---

## BB1015 - Marks and Accessorials
**Business Function: Shipping Marks and Accessorial Charges**

| ID | Rule Name | Description | Type | TEST |
|----|-----------|-------------|------|------|
| 47 | Shipto Marks Retrieval | Retrieves order marks, invoice marks, dispatch info, BOL marks | Calculation | `SELECT CSOMK1, CSOMK2, CSOMK3, CSOMK4, CSIMK1, CSIMK2, CSDSP1, CSDSP2, CSBMK1, CSBMK2, CSBMK3, CSBMK4 FROM SHIPTO WHERE CSCONO = ?CO AND CSCUST = ?CUST AND CSSHIP = ?SHIP AND CSDEL <> 'D'` |
| 48 | Accessorial Inheritance | Copies accessorials from ship-to master if order doesn't have them | Calculation | `SELECT * FROM BBSHSA1 WHERE BACO = ?CO AND BACUST = ?CUST AND BASHIP = ?SHIP AND BADEL <> 'D' AND NOT EXISTS (SELECT 1 FROM BBOTA1 WHERE BACO = ?CO AND BAORDN = ?ORDER)` |
| 49 | Freight Bill Address | Retrieves freight bill name and address from ship-to | Calculation | `SELECT CSFRNM, CSFRA1, CSFRA2, CSFRA3 FROM SHIPTO WHERE CSCONO = ?CO AND CSCUST = ?CUST AND CSSHIP = ?SHIP` |

---

## BB106 - Freight Calculation
**Business Function: Freight Charge Calculation**

| ID | Rule Name | Description | Type | TEST |
|----|-----------|-------------|------|------|
| 50 | Freight Calculation Options | Two modes: CALC (single table with updates) or TOP5 (five tables no updates) | Calculation | `-- Mode determined by FOPTN parameter: 'CALC' or 'TOP5'` |
| 51 | Freight Table Selection | Selects freight table by customer, ship-to, location, carrier, container, products, dates | Calculation | `SELECT * FROM BICUF1 WHERE BFCONO = ?CO AND BFCUST = ?CUST AND BFSHIP = ?SHIP AND BFLOC = ?LOC AND BFCAID = ?CAID AND BFCNTR = ?CNTR AND BFSTDT <= ?DATE AND (BFEXDT = 0 OR BFEXDT >= ?DATE) AND BFDEL <> 'D'` |
| 52 | Multi-Load Freight Adjustment | For multi-load, calculates freight for one load then converts to total | Calculation | `SELECT BFFLAT * 1 AS SINGLE_LOAD, BFFLAT * ?MLCNT AS TOTAL_LOADS FROM BICUF1` |
| 53 | Multi-Load Invoice Flat Rate | For multi-load invoices with flat rate, multiply by detail line count | Calculation | `SELECT BFFLAT * (SELECT COUNT(*) FROM BBTRAN WHERE TSEQ < 900) AS TOTAL_FREIGHT FROM BICUF1 WHERE BFFLAT > 0` |
| 54 | Override Freight Rate | When override rate present, skip freight calculation | Validation | `SELECT * FROM BBORTR WHERE TDOFRR > 0` |
| 55 | Freight Components | Includes rate per mile, per cwt, per gallon, minimum, flat, cleaning, pump, hose, pickup, tolls, detention, insulated tank, stop-off, service, scale | Calculation | `SELECT BFRPM, BFCWT, BFRPG, BFMIN, BFFLAT, BFCLEN, BFPUMP, BFHOSE, BFPUUP, BFTOLL, BFDENT, BFINTA, BFSTOP, BFSERV, BFSCAL FROM BICUF1` |
| 56 | Fuel Surcharge Calculation | Calculates fuel surcharge by percentage or amount; can show separately | Calculation | `SELECT CASE WHEN BFSCCA = 'L' THEN BFSCHG * ?LINEHAUL WHEN BFSCCA = 'T' THEN BFSCHG * ?TOTAL ELSE BFSCHG END AS SURCHARGE FROM BICUF1` |
| 57 | Carrier Freight Calculation | Calculates both customer freight and carrier freight separately | Calculation | `SELECT (BFRPM * ?MILES + BFCWT * ?CWT + BFRPG * ?GAL + BFFLAT) AS CUST_FRT, (CFRPM * ?MILES + CFCWT * ?CWT + CFRPG * ?GAL + CFFLAT) AS CARR_FRT FROM BICUF1` |
| 58 | Insurance Calculation | Calculates insurance as percentage of order value for customer and carrier | Calculation | `SELECT (BFINSP / 100) * ?ORDER_VALUE AS CUST_INS, (CFINSP / 100) * ?ORDER_VALUE AS CARR_INS FROM BICUF1` |
| 59 | Tender Sequence Priority | Prioritizes freight tables with tender sequence for top 5 | Calculation | `SELECT * FROM BICUF1 WHERE BFTSEQ > 0 ORDER BY BFTSEQ FETCH FIRST 5 ROWS ONLY` |
| 60 | Route Code Assignment | Route code retrieved from ship-to, not freight table | Calculation | `SELECT CSROUT FROM SHIPTO WHERE CSCONO = ?CO AND CSCUST = ?CUST AND CSSHIP = ?SHIP` |
| 61 | Freight Collect with Service | For CYY freight code, charges $100 service fee (misc seq 943) | Calculation | `SELECT BCSVFE AS SERVICE_FEE FROM BICONT WHERE BCCONO = ?CO AND ?FRCD = 'CYY'` |
| 62 | Freight Proration | Prorates miscellaneous freight to detail lines | Calculation | `SELECT TDSEQ, (TDPRCE * TDQTY) / ?TOTAL_PROD * ?MISC_FRT AS PRORATED_FRT FROM BBORTR WHERE TSEQ < 900` |
| 63 | Customer Price Components | Customer price = product price + freight price; varies by freight terms | Calculation | `SELECT CASE WHEN ?FRCD = 'C' THEN ?PROD_PRC + 0 WHEN ?FRCD = 'A' THEN ?PROD_PRC + (?FRT_PRC * 0.25) WHEN ?FRCD = 'P' AND ?PRCD = 'R' THEN ?PROD_PRC + (?FRT_PRC * 0.25) WHEN ?FRCD = 'P' AND ?PRCD = 'S' THEN (?PROD_PRC * 0.75) + (?FRT_PRC * 0.25) END AS CUST_PRICE` |

---

**Total Complex Business Rules: 63**
- Validations: 25
- Calculations: 38

**Note:** SQL queries use IBM i DB2 syntax. Replace `?VARIABLE` placeholders with actual values when testing. Some calculations are part of program logic and don't have direct SQL equivalents.