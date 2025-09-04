Based on the provided call stack (`PRICEGEN.clp`, `PRICES.ocl36.txt`, `SA505C.ocl36.txt`, `SA505X.rpg36.txt`, `GSDT12.ocl36.txt`, `GSDT12.rpg36.txt`, `SA505G.ocl36.txt`, `SA505G.rpg36.txt`, `SA505H.ocl36.txt`, `SA505H.rpg36.txt`, `SA505I.ocl36.txt`, `SA505I.rpg36.txt`), the workflow implements a Customer Shipping Analysis Report for blended lubes pricing in an IBM System/36 environment (or AS/400 in compatibility mode). Below, I will identify the use cases implemented in this call stack, followed by a function requirement document for a large function that encapsulates the primary use case, assuming it processes inputs programmatically rather than via screen interaction .

### Use Cases Implemented in the Call Stack

The call stack implements the following use cases for generating and refining a Customer Shipping Analysis Report, which supports pricing analysis and sales reporting for blended lubes:

1. **Date Range Setup for Reporting (GSDT12)**:
   - **Description**: Prompts for or calculates a date range (start and end dates) to filter sales data, ensuring reports cover a specific period.
   - **Components**: `GSDT12.ocl36.txt`, `GSDT12.rpg36.txt`.
   - **Purpose**: Initializes date parameters (`FRDT8`, `TODT8`) in the Local Data Area (LDA) for use by subsequent programs, ensuring consistent date filtering.
   - **Key Features**:
     - Uses system date or user input to set start date (`YY`, `MM`, `DD`).
     - Calculates end date as the last day of the same or next month.
     - Formats dates in CYMD (CCYYMMDD) with Y2K compliance (century hardcoded as 20).

2. **Sales Data Filtering by Freight and Carrier (SA505X)**:
   - **Description**: Filters sorted sales data based on freight code (`KYFRCD`) and carrier code (`KYCACD`), marking non-matching records for deletion.
   - **Components**: `SA505X.rpg36.txt`.
   - **Purpose**: Refines the sales dataset (`?9?S5505S`) to include only records matching specified freight and carrier criteria, preparing data for reporting.
   - **Key Features**:
     - Chains to `SA5SHX` for additional sales data.
     - Marks records with `'D'` in `SA5FILD` if they fail freight/carrier checks.
     - Supports flexible filtering (both codes blank, one specified, or both specified).

3. **Customer Shipping Analysis Report Generation (SA505C)**:
   - **Description**: Aggregates sales data by company, customer, ship-to, container, and product, producing a formatted report and summary data.
   - **Components**: `SA505C.ocl36.txt`, `SA505C.rpg36.txt`.
   - **Purpose**: Generates a detailed report (`LIST`) or summary file (`?9?SA505C`) with sales quantities, amounts, and invoice counts, supporting pricing analysis.
   - **Key Features**:
     - Aggregates totals at multiple levels (company, customer, ship-to, container, product).
     - Applies filters (date, location, product class, container, product).
     - Outputs to printer or file based on LDA settings.

4. **Sales and Pricing Data Integration (SA505G)**:
   - **Description**: Combines sales data (`SA505CX`, `SA505EX`) with pricing (`PRSABL`, `RKPRCEX`) to produce summarized output files.
   - **Components**: `SA505G.ocl36.txt`, `SA505G.rpg36.txt`.
   - **Purpose**: Integrates sales and pricing data to produce detailed (`?9?SA505G`) and summary (`?9?SA505G2`) outputs for further analysis.
   - **Key Features**:
     - Aggregates quantities, sales amounts, route types, and ship types.
     - Updates `PRSABL` with summarized data.
     - Processes all records regardless of sales matches.

5. **Refined Sales and Pricing Summarization (SA505H)**:
   - **Description**: Refines `SA505G` outputs, integrating rack pricing and sales data to produce detailed and summary output files.
   - **Components**: `SA505H.ocl36.txt`, `SA505H.rpg36.txt`.
   - **Purpose**: Produces refined outputs (`?9?TMPFILX`, `?9?TMPFILY`) with detailed sales, pricing, and customer data for reporting.
   - **Key Features**:
     - Matches `SA505G2` with `SA505GX`, `RKPRCEX`, and `SA505EX`.
     - Handles unit of measure differences (notes need for gallon price conversion).

6. **Final Data Enrichment and Output (SA505I)**:
   - **Description**: Enriches `SA505H` output with customer, container, and product details, performing quantity conversions and updating the pricing file.
   - **Components**: `SA505I.ocl36.txt`, `SA505I.rpg36.txt`.
   - **Purpose**: Produces the final pricing data (`PRSABLW`) with enriched details and converted quantities for the Customer Shipping Analysis Report.
   - **Key Features**:
     - Retrieves descriptions from `ARCUST`, `GSCNTR`, `GSPROD`.
     - Converts quantities using `MBBQTY` (e.g., gallons to weights).
     - Updates `PRSABLW` with comprehensive data.

### Function Requirement Document

The primary use case is the generation of the Customer Shipping Analysis Report, encompassing filtering, aggregation, integration, and enrichment of sales and pricing data. Below is a function requirement document for a large function, `GenerateCustomerShippingAnalysis`, that encapsulates this use case, processing inputs programmatically without screen interaction.

<xaiArtifact artifact_id="14d65cfd-f455-44bd-9dde-3a5d56353da5" artifact_version_id="6cf77040-6773-462e-ac8c-e53deac4c4b9" title="CustomerShippingAnalysisFunction.md" contentType="text/markdown">

# Function Requirement Document: GenerateCustomerShippingAnalysis

## Purpose
Generate a Customer Shipping Analysis Report for blended lubes, filtering sales data by date range, freight, and carrier codes, aggregating by company, customer, ship-to, container, and product, integrating with pricing data, and enriching with customer, container, and product details, producing detailed and summary outputs.

## Inputs
- **Date Range**:
  - `startDate` (string, YYMMDD): Start date for sales data.
  - `endDate` (string, YYMMDD): End date for sales data.
- **Filters**:
  - `freightCode` (string, 2 chars, optional): Freight code (`KYFRCD`).
  - `carrierCode` (string, 3 chars, optional): Carrier code (`KYCACD`).
  - `division` (string, optional): Division code (`KYDIV`).
  - `locations` (array of strings, optional): Location codes (`KYLOC1`–`KYLOC5`).
  - `productClasses` (array of strings, optional): Product class codes (`KYPC01`–`KYPC10`).
  - `containers` (array of strings, optional): Container codes (`KYCT01`–`KYCT05`).
  - `products` (array of strings, optional): Product codes (`KYPD01`–`KYPD10`).
- **Input Data**:
  - `salesData` (array of objects): Sales records from `SA5FILD` (`?9?S5505S`), containing:
    - `company` (string, 2 chars), `customer` (string, 6 chars), `invoice` (string, 7 chars), `shipTo` (string, 3 chars), `product` (string, 4 chars), `container` (string, 3 chars), `unitOfMeasure` (string, 3 chars), `quantity` (number, 4 decimals), `price` (number, 4 decimals), `salesAmount` (number, 4 decimals), `date` (string, YYMMDD), `freightCode` (string, 2 chars), `carrierCode` (string, 3 chars).
  - `salesIndex` (array of objects): Sales history index from `SA5SHX` (`?9?SA5SHX`), with key fields.
  - `rackPrices` (array of objects): Rack price data from `RKPRCEX` (`?9?RKPRCEX`), containing:
    - `company` (string, 2 chars), `location` (string, 3 chars), `productClass` (string, 3 chars), `product` (string, 4 chars), `container` (string, 3 chars), `unitOfMeasure` (string, 3 chars), `price` (number, 4 decimals), `date` (string, CCYYMMDD).
  - `pricingData` (array of objects): Blended lubes pricing from `PRSABL` (`?9?PRSABLW`), containing:
    - `company` (string, 2 chars), `customer` (string, 6 chars), `shipTo` (string, 3 chars), `product` (string, 4 chars), `container` (string, 3 chars), `unitOfMeasure` (string, 3 chars), `price` (number, 4 decimals), `startDate` (string, CCYYMMDD).
  - `customerData` (array of objects): Customer details from `ARCUST` (`?9?ARCUST`), containing:
    - `company` (string, 2 chars), `customer` (string, 6 chars), `name` (string), `address` (object with lines 1–4, zip).
  - `contractData` (array of objects): Contract details from `BICONT` (`?9?BICONT`), containing:
    - `company` (string, 2 chars), `name` (string).
  - `productData` (array of objects): Product details from `GSPROD` (`?9?GSPROD`), containing:
    - `product` (string, 4 chars), `description` (string).
  - `containerData` (array of objects): Container details from `GSCNTR` (`?9?GSCNTR`), containing:
    - `container` (string, 3 chars), `description` (string).
  - `unitConversions` (array of objects): Unit of measure conversions from `GSCTUM`, `GSUMCV`, containing:
    - `unitOfMeasure` (string, 3 chars), `toGallonsFactor` (number).
  - `containerWeights` (array of objects): Container weights from `GSCTWT`, containing:
    - `container` (string, 3 chars), `weight` (number).

## Outputs
- **DetailedReport** (array of objects): Detailed records (`?9?TMPFILX`-like), containing:
  - `company` (string, 2 chars), `customer` (string, 6 chars), `shipTo` (string, 3 chars), `product` (string, 4 chars), `container` (string, 3 chars), `unitOfMeasure` (string, 3 chars), `quantity` (number, 4 decimals), `salesAmount` (number, 4 decimals), `price` (number, 4 decimals), `date` (string, CCYYMMDD), `time` (string, HHMMSS).
- **SummaryReport** (array of objects): Summary records (`?9?TMPFILY`-like), containing:
  - `company` (string, 2 chars), `customer` (string, 6 chars), `product` (string, 4 chars), `container` (string, 3 chars), `unitOfMeasure` (string, 3 chars).
- **PricingOutput** (array of objects): Updated pricing data (`?9?PRSABLW`), containing:
  - `company` (string, 2 chars), `customer` (string, 6 chars), `customerName` (string), `status` (string), `shipTo` (string, 3 chars), `product` (string, 4 chars), `productDescription` (string), `container` (string, 3 chars), `unitOfMeasure` (string, 3 chars), `price` (number, 4 decimals), `quantitySold` (number, 4 decimals), `salesAmount` (number, 4 decimals), `startDate` (string, CCYYMMDD), `endDate` (string, CCYYMMDD).

## Process Steps
1. **Date Range Conversion**:
   - Convert `startDate` and `endDate` (YYMMDD) to CYMD (CCYYMMDD, century = 20) for `KYFRDT` and `KYTODT`.
   - If `startDate.day = 1`, set `endDate` to first day of next month; otherwise, set to last day of current month (31 for Jan, Mar, May, Jul, Aug, Oct, Dec; 29 for Feb; 30 for Apr, Jun, Sep, Nov).

2. **Sales Data Filtering**:
   - Filter `salesData` by:
     - `date` within `KYFRDT` to `KYTODT`.
     - `freightCode` matches `KYFRCD` (if specified) or `carrierCode` matches `KYCACD` (if specified).
     - `division`, `locations`, `productClasses`, `containers`, `products` (if specified).
   - Exclude records marked with delete code (`'D'`).

3. **Data Aggregation**:
   - Aggregate `salesData` by company, customer, ship-to, container, and product:
     - Sum `quantity` for `QTSOLD`.
     - Calculate `salesAmount` as `price * quantity`.
     - Count invoices for invoice totals.
   - Store aggregates in intermediate structures (`SA505CX`, `SA505EX`).

4. **Pricing and Sales Integration**:
   - Match aggregated sales with `pricingData` and `rackPrices` using keys (company, customer, ship-to, product, container, unit of measure).
   - Sum quantities and sales amounts from matched records; set to zero if no match.

5. **Data Enrichment**:
   - Join with `customerData` for customer name and address.
   - Join with `contractData` for company name.
   - Join with `productData` for product description.
   - Join with `containerData` for container description.

6. **Quantity Conversion**:
   - Convert `QTSOLD` to gallons or weights using `unitConversions` and `containerWeights`:
     - Gallons: `QTSOLD * toGallonsFactor`.
     - Weights: Use `containerWeights.weight` and `QTSOLD` (logic in `MBBQTY`).
   - Set temperature to 60°F, clear gravity and volume correction factors.

7. **Output Generation**:
   - Produce `DetailedReport` with individual transaction details (company, customer, ship-to, product, container, unit of measure, quantity, sales amount, price, date, time).
   - Produce `SummaryReport` with aggregated data (company, customer, product, container, unit of measure).
   - Produce `PricingOutput` with enriched data (company, customer, customer name, status, ship-to, product, product description, container, unit of measure, price, quantity sold, sales amount, start date, end date = '20791231', PPD flag = 'Y', record type = 'R').

## Business Rules
1. **Date Range**:
   - Start date is user-provided or system date; end date is calculated as last day of month or first day of next month if start day is 1.
   - Dates formatted as CCYYMMDD with century = 20 for Y2K compliance.
2. **Filtering**:
   - Include sales records within date range and matching optional filters (freight code, carrier code, division, locations, product classes, containers, products).
   - Exclude deleted records (`'D'`).
3. **Aggregation**:
   - Sum quantities and sales amounts at company, customer, ship-to, container, and product levels.
   - Track invoice counts for customer-level totals.
4. **Pricing Integration**:
   - Match sales with pricing and rack price data; use zero for quantities/sales if no match.
   - Prioritize `pricingData` for blended lubes, supplemented by `rackPrices`.
5. **Quantity Conversion**:
   - Convert non-gallon units to gallons or weights using conversion factors.
   - Use 60°F as standard temperature; ignore gravity and volume correction.
6. **Output**:
   - Detailed report includes all transaction details.
   - Summary report aggregates by key fields without quantities.
   - Pricing output includes enriched data with hardcoded end date ('20791231'), PPD flag ('Y'), and record type ('R').
7. **Context**: Supports pricing validation and sales analysis for blended lubes, complementing rack pricing.

## Calculations
- **Sales Amount**: `salesAmount = price * quantity`.
- **Date Conversion**: `CCYYMMDD = 20 * 1000000 + YY * 10000 + MM * 100 + DD`.
- **End Date**:
  - If `DD = 1`: Set `NWMM = MM + 1` (or `1` if `MM = 12`, with `NWYY = YY + 1`); `NWDD = 1`.
  - Else: Set `NWDD = 31` (Jan, Mar, May, Jul, Aug, Oct, Dec), `29` (Feb), or `30` (Apr, Jun, Sep, Nov).
- **Quantity Conversion**: `gallons = QTSOLD * toGallonsFactor`; weights via `MBBQTY` logic (not detailed).

</xaiArtifact>

### Additional Notes
- **Scope**: The function encapsulates the entire workflow (`GSDT12`, `SA505X`, `SA505C`, `SA505G`, `SA505H`, `SA505I`), assuming programmatic inputs replace screen prompts.
- **Missing Programs**: `SA505E`, `SA505J`, `SA505L` are not provided but likely perform additional reporting or cleanup. The function covers the core use case.
- **Assumptions**: `MBBQTY` logic for quantity conversion is abstracted as a simple multiplication; actual implementation may involve weights or additional factors.
- **Y2K Compliance**: Dates use century = 20, limiting applicability to 2000–2099.
- **Error Handling**: Assumes valid inputs; errors (e.g., file access, invalid dates) are handled by the System/36 environment.

If you have additional files (`SA505E`, `SA505J`, `SA505L`) or need further refinement, please provide them! Let me know if you have questions or require additional details.