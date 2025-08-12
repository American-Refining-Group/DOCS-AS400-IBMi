The provided document is an Operation Control Language (OCL) program, specifically `BB101.ocl36.txt`, which appears to be part of an IBM System/38 or AS/400 (now IBM i) environment for managing order entry processes. OCL is used to control job execution, define file operations, and invoke programs. Below is a detailed explanation of the process steps, followed by a list of external programs called and tables (files) used.

---

### Process Steps of the RPG .OCL Program

The OCL program `BB101.ocl36.txt` is structured to handle order entry, batch selection, and batch release processes. It includes file definitions, conditional logic, printer overrides, and calls to external programs. Here’s a step-by-step breakdown of the process:

1. **Initial Setup and Program Invocation**:
   - The program starts by calling `STRPCOCLP`, likely a utility or initialization program to set up the environment for the order entry process.
   - Comments indicate the program is related to "ORDER ENTRY" and includes temporary modifications by "JIMMY K."

2. **Order Batch Selection**:
   - **Local Variables Setup**:
     - Defines local variables at specific offsets (e.g., `OFFSET-470,DATA-'?13?'`, `OFFSET-494,DATA-'?USER?'`) to store runtime parameters such as batch type (`?13?`), user ID (`?USER?`), workstation (`?WS?`), and mode (`O` for Order Entry).
     - Conditionally sets a display message at `OFFSET-60` based on the value of `?13?`:
       - If `?13?` is blank, displays "* ORDER ENTRY *".
       - If `?13?` is `PP`, displays "* VISCOSITY ASN ENTRY *".
       - If `?13?` is `PM`, displays "* PRODUCT MOVES ENTRY *".
   - **Switch Logic**:
     - Defines a switch (`SWITCH 0XXXXXXX`) where `SWITCH1` is used to control batch deletion logic.
   - **File Loading and Execution**:
     - Loads program `BB001` and defines files `BBBTCH` and `BBBTCHX` (likely batch header files) with the label `?9?BBBTCH` and shared disposition (`DISP-SHR`).
     - Executes the program (`RUN`).
   - **Conditional Exit**:
     - If the condition `?L'121,6'?/CANCEL` is true (checking a specific field for the value "CANCEL"), the program returns, halting further execution.
   - **Batch Deletion Logic**:
     - Evaluates `P20` with the value from `?L'490,2'?` (a 2-character field at position 490).
     - If `SWITCH1` is set (i.e., `SWITCH1-1`):
       - Calls programs `BB215` and `BB003` with `*ALL` parameters.
       - Deletes records from files `?9?BBOR?20?` and `?9?BBOX?20?` if they exist (`DATAF1-?9?BBOR?20?` and `DATAF1-?9?BBOX?20?`).
       - Resets program `BB101` with `*ALL` parameters.

3. **Order Entry Processing**:
   - **Temporary Setup**:
     - Sets a local variable at `OFFSET-101` to `'10'`.
     - Defines `IN110` with parameters `INTZH` and `INTZXX` for files with labels `?9?`, `WK`, `HOLD`, `FUT`.
     - Calls program `IN805BC`, likely for additional processing or validation.
   - **File Creation**:
     - Conditionally creates files `?9?BBOR?20?` and `?9?BBOX?20?` using `BLDFILE` if they exist, with specifications for 999,000 records, 512-byte record length, and other attributes.
   - **File Definitions**:
     - Loads program `BB101` and defines multiple files with shared disposition (`DISP-SHR`), including:
       - Order-related files: `BBORTR`, `BBORTRX`, `BBORDR`, `BBORDRH`, `BBTRTX`, `BBORTX`.
       - Customer-related files: `BICONT`, `ARCUSTX`, `EDICUS`, `ARCOMMX`, `ARCUST`, `ARCUSP`, `SHIPTO`.
       - Inventory and pricing files: `BIPRTX`, `GSTABL`, `INFRMP`, `GLMAST`, `INTANK`, `INTAN2`.
       - Additional files for specific purposes: `BBOTHS1`, `BBOTDS1`, `BBOTA1`, `BBORHS1`, `BBORDS1`, `BBORA1`, `BBFRPR`, `GSMLCD`, `BBDATE`, `GSCNTRA`, `SA5SHS`, `BBORDHZ`, `SA5SHQ`, `BBORDHU`.
     - Defines files for called programs (e.g., `BB1011`, `BB1012`, `BB1013`, etc.), including `GSUMCV`, `PRCNTR`, `BBPRCE`, `BBBCPR`, `BICUAX`, `BICUAY`, `SA5FIZD`, `ARCUPR`, `BBORCL`, `ARCUST2`, `ARCLGR`, `BBORDHC`, `GSCTUM`, `BBCSR`, `BBSLSM`, `INLOC`, `BBSHSA1`, `BICUFR`, `GSCTWT`, `BBCFSH`, `BBCFSD1`, `BBNDFI2`, `BBTRANIN`, `BBORF`, `BBTRF`, `BBCAID`, `GSCNTR1`, `GSPRCT`, `FROF`, `FROFH`, `FROFCL1`, `FROFC`, `FROFCH`, `GSTABL6`, `GSCNTR6`, `ARCUP16`, `ARCUP36`, `GSPRD6`, `BICUF1`, `BBTRDS1I`, `BBPRXR`, `BBPRXY`, `BBPRXZ`, `SHPADR`, `CUADR`, `ARCUSX`, `BBORDD`, `SA5FIUD`, `SA5SHZ`, `BBOH2`, `BBOH3`, `BBOH`, `BBOHMS`, `BBORDB`, `BBORDH`, `BBORDI`, `BBORDM`, `BBORDO`, `BBPRCY`, `BICUA6`, `GSPROD`, `BBRCSC2`, `BBCNOR`, `BBORPX`, `GSCNTR`, `INCONT`, `INMTAJ`, `INMTRDY`, `INTKRV`, `BBSHSA`, `BBOTA`, `BBBLA`, `BBTRA`, `BBSHSARD`, `BBOTARD`, `BBBLARD`, `BBTRARD`.
   - **Printer Overrides**:
     - Overrides printer files `JBLIST` and `LIST198J` to output queue `QUSRSYS/CREDLIMIT`.
     - Conditionally overrides `CREMAL` and `SMEMAL` to output queues `CSROUTQ`, `SLMNOUTQ`, or `TESTOUTQ` if `?9?` equals `G`.
     - Defines a printer `BUGS` with device `PJ` and priority `0`.
   - **Execution**:
     - Runs the program (`RUN`) after defining files and overrides.

4. **Release Batch**:
   - Sets a local variable at `OFFSET-475` to `?F'A,?9?BBOR?20?'?`.
   - Loads program `BB005` and defines file `BBBTCH` with label `?9?BBBTCH` and shared disposition.
   - Executes the program (`RUN`).
   - Clears all local variables (`LOCAL BLANK-*ALL`).

---

### External Programs Called

The OCL program calls the following external programs:
1. **STRPCOCLP**: Initializes the environment for order entry.
2. **BB001**: Handles order batch selection.
3. **BB215**: Called during batch deletion if `SWITCH1` is set.
4. **BB003**: Called during batch deletion if `SWITCH1` is set.
5. **IN805BC**: Called for additional processing in the order entry section.
6. **BB005**: Handles batch release.
7. **BB1011**: Associated with file definitions for pricing, customer, and inventory data.
8. **BB1012**: Associated with customer pricing data (`ARCUPR`).
9. **BB1013**: Associated with order and customer data processing.
10. **BB1014**: Associated with location data (`INLOC`).
11. **BB1015**: Associated with shipment data (`BBSHSA1`).
12. **BB1016**: Associated with customer and pricing data (`ARCUP16`, `ARCUP36`).
13. **BB106**: Associated with freight and inventory data.
14. **BB1018**: Associated with pricing data (`BBPRXR`, `BBPRXY`, `BBPRXZ`).
15. **SHPAD1R**: Associated with shipping address data (`SHPADR`).
16. **CUADR1R**: Associated with customer address data (`CUADR`).
17. **MCSTSHP**: Associated with customer data (`ARCUSX`).
18. **BB115**: Associated with order and shipment data.
19. **BB117**: Associated with order header and master data.
20. **BB801**: Associated with order data processing.
21. **AR822R**: Associated with pricing and customer data.
22. **MBBQTY**: Associated with product and quantity data.
23. **BB104A**: Associated with contract order data (`BBCNOR`).
24. **IN805**: Associated with inventory and contract data.
25. **BI9005**: Associated with shipment and billing data.

---

### Tables (Files) Used

The program references numerous files (tables) for data processing, all defined with shared disposition (`DISP-SHR`) unless otherwise noted. Below is a comprehensive list of the files used:

1. **BBBTCH**: Batch header file (`?9?BBBTCH`).
2. **BBBTCHX**: Alternate batch header file (`?9?BBBTCH`).
3. **BBORTR**: Order transaction file (`?9?BBOR?20?`, `EXTEND-100`).
4. **BBORTRX**: Alternate order transaction file (`?9?BBOR?20?`).
5. **BBORDR**: Order detail file (`?9?BBORDR`).
6. **BBORDRH**: Order header file (`?9?BBORDH`).
7. **BBTRTX**: Transaction file (`?9?BBOX?20?`, `EXTEND-50`).
8. **BBORTX**: Order transaction file (`?9?BBORTX`).
9. **BICONT**: Customer contract file (`?9?BICONT`).
10. **ARCUSTX**: Customer extension file (`?9?ARCUSX`).
11. **EDICUS**: EDI customer file (`?9?EDICUS`).
12. **ARCOMMX**: Customer communication file (`?9?ARCOMMX`).
13. **ARCUST**: Customer master file (`?9?ARCUST`).
14. **ARCUSP**: Customer pricing file (`?9?ARCUSP`).
15. **SHIPTO**: Ship-to address file (`?9?SHIPTO`).
16. **BIPRTX**: Product pricing file (`?9?BIPRTX`).
17. **GSTABL**: General system table (`?9?GSTABL`).
18. **INFRMP**: Inventory formula file (`?9?INFRMP`).
19. **GLMAST**: General ledger master file (`?9?GLMAST`).
20. **INTANK**: Inventory tank file (`?9?INTANK`).
21. **INTAN2**: Secondary inventory tank file (`?9?INTAN2`).
22. **BBOTHS1**: Order transaction history file (`?9?BBOTHS1`).
23. **BBOTDS1**: Order detail history file (`?9?BBOTDS1`).
24. **BBOTA1**: Order transaction file (`?9?BBOTA1`).
25. **BBORHS1**: Order header history file (`?9?BBORHS1`).
26. **BBORDS1**: Order detail file (`?9?BBORDS1`).
27. **BBORA1**: Order alternate file (`?9?BBORA1`).
28. **BBFRPR**: Freight pricing file (`?9?BBFRPR`).
29. **GSMLCD**: System calendar file (`?9?GSMLCD`).
30. **BBDATE**: Date file (`?9?BBDATE`).
31. **GSCNTRA**: Contract file (`?9?GSCNTR1`).
32. **SA5SHS**: Shipment history file (`?9?SA5SHS`).
33. **BBORDHZ**: Order header file (`?9?BBORDHZ`).
34. **SA5SHQ**: Shipment queue file (`?9?SA5SHQ`).
35. **BBORDHU**: Order header unit file (`?9?BBORDHU`).
36. **GSUMCV**: Customer summary file (`?9?GSUMCV`).
37. **PRCNTR**: Pricing control file (`?9?PRCNTR`).
38. **BBPRCE**: Pricing file (`?9?BBPRCE`).
39. **BBBCPR**: Contract pricing file (`?9?BBBCPR`).
40. **BICUAX**: Customer auxiliary file (`?9?BICUAX`).
41. **BICUAY**: Customer auxiliary file (`?9?BICUAY`).
42. **SA5FIZD**: Shipment detail file (`?9?SA5FIZD`).
43. **ARCUPR**: Customer pricing file (`?9?ARCUPR`).
44. **BBORCL**: Order close file (`?9?BBORCL`).
45. **BBORTRC**: Order transaction file (`?9?BBOR?20?`).
46. **ARCUST2**: Customer master file (`?9?ARCUST`).
47. **ARCLGR**: Customer ledger file (`?9?ARCLGR`).
48. **BBORDHC**: Order header file (`?9?BBORDH`).
49. **GSCTUM**: Customer master file (`?9?GSCTUM`).
50. **BBCSR**: Customer service file (`?9?BBCSR`).
51. **BBSLSM**: Sales master file (`?9?BBSLSM`).
52. **INLOC**: Inventory location file (`?9?INLOC`).
53. **BBSHSA1**: Shipment file (`?9?BBSHSA1`).
54. **BICUFR**: Freight customer file (`?9?BICUFR`).
55. **GSCTWT**: Contract weight file (`?9?GSCTWT`).
56. **BBCFSH**: Freight shipment file (`?9?BBCFSH`).
57. **BBCFSD1**: Freight shipment detail file (`?9?BBCFSD1`).
58. **BBNDFI2**: Non-delivery file (`?9?BBNDFI2`).
59. **BBTRANIN**: Transaction input file (`?9?BBTRAN`, `EXTEND-100`).
60. **BBORF**: Order freight file (`?9?BBORF`).
61. **BBTRF**: Transaction freight file (`?9?BBTRF`).
62. **BBCAID**: Carrier ID file (`?9?BBCAID`).
62. **GSCNTR1**: Contract file (`?9?GSCNTR1`).
64. **GSPRCT**: Product contract file (`?9?GSPRCT`).
65. **FROF**: Freight order file (`?9?FROF`).
66. **FROFH**: Freight order header file (`?9?FROFH`).
67. **FROFCL1**: Freight order close file (`?9?FROFCL1`).
68. **FROFC**: Freight order file (`?9?FROFC`).
69. **FROFCH**: Freight order header file (`?9?FROFCH`).
70. **GSTABL6**: System table (`?9?GSTABL`).
71. **GSCNTR6**: Contract file (`?9?GSCNTR1`).
72. **ARCUP16**: Customer pricing file (`?9?ARCUP1`).
73. **ARCUP36**: Customer pricing file (`?9?ARCUP3`).
74. **GSPRD6**: Product file (`?9?GSPRD6`).
75. **BICUF1**: Freight customer file (`?9?BICUF1`).
76. **BBTRDS1I**: Transaction detail file (`?9?BBTRDS1`).
77. **BBPRXR**: Pricing file (`?9?BBPRXR`).
78. **BBPRXY**: Pricing file (`?9?BBPRXY`).
79. **BBPRXZ**: Pricing file (`?9?BBPRXZ`).
80. **SHPADR**: Shipping address file (`?9?SHPADR`).
81. **CUADR**: Customer address file (`?9?CUADR`).
82. **ARCUSX**: Customer extension file (`?9?ARCUSX`).
83. **BBORDD**: Order detail file (`?9?BBORDD`).
84. **SA5FIUD**: Shipment detail file (`?9?SA5FIUD`).
85. **SA5SHZ**: Shipment file (`?9?SA5SHZ`).
86. **BBOH2**: Order header file (`?9?BBOH2`).
87. **BBOH3**: Order header file (`?9?BBOH3`).
88. **BBOH**: Order header file (`?9?BBOH`).
89. **BBOHMS**: Order header master file (`?9?BBOHMS`).
90. **BBORDB**: Order detail file (`?9?BBORDB`).
91. **BBORDH**: Order header file (`?9?BBORDH`).
92. **BBORDI**: Order item file (`?9?BBORDI`).
93. **BBORDM**: Order master file (`?9?BBORDM`).
94. **BBORDO**: Order file (`?9?BBORDO`).
95. **BBPRCY**: Pricing file (`?9?BBPRCY`).
96. **BICUA6**: Customer auxiliary file (`?9?BICUA6`).
97. **GSPROD**: Product file (`?9?GSPROD`).
98. **BBRCSC2**: Contract file (`?9?BBRCSC2`).
99. **BBCNOR**: Contract order file (`?9?BBCNOR`).
100. **BBORPX**: Order pricing file (`?9?BBORPX`).
101. **GSCNTR**: Contract file (`?9?GSCNTR`).
102. **INCONT**: Inventory contract file (`?9?INCONT`).
103. **INMTAJ**: Inventory adjustment file (`?9?INMTAJ`).
104. **INMTRDY**: Inventory ready file (`?9?INMTRDY`).
105. **INTKRV**: Inventory tank file (`?9?INTKRV`).
106. **BBSHSA**: Shipment file (`?9?BBSHSA`).
107. **BBOTA**: Order transaction file (`?9?BBOTA`).
108. **BBBLA**: Billing file (`?9?BBBLA`).
109. **BBTRA**: Transaction file (`?9?BBTRA`).
110. **BBSHSARD**: Shipment file (`?9?BBSHSA`).
111. **BBOTARD**: Order transaction file (`?9?BBOTA`).
112. **BBBLARD**: Billing file (`?9?BBBLA`).
113. **BBTRARD**: Transaction file (`?9?BBTRA`).
114. **SA5MOVD3**: Movement detail file (`?9?SA5MOVD3`, overridden via `OVRDBF`).

---

### Summary

The `BB101.ocl36.txt` OCL program manages order entry, batch selection, and batch release processes in an IBM midrange system. It initializes the environment, handles batch selection with conditional logic for display messages and batch deletion, processes order entry with extensive file operations, and releases batches. The program interacts with numerous files for orders, customers, inventory, pricing, and shipments, and calls multiple external programs to perform specific tasks. The use of dynamic file labels (e.g., `?9?`, `?20?`) and conditional logic (`IF`, `IFF`) allows flexibility in handling different environments and batch types.