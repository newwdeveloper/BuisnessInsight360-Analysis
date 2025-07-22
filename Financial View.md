📊 Power BI – Building the Financial View (P&L)
🔧 Objective
To build a Profit & Loss (P&L) style financial matrix in Power BI, where key financial metrics like Gross Sales, Pre-Invoice Deduction, and Net Invoice Sales are arranged vertically like a financial statement — using a supporting table, DAX logic, and matrix visuals.

🧱 Step 1: Create the Core P&L Structure
This step involves preparing the base structure that will support the dynamic P&L view.

🧾  1 Create Supporting Table for P&L Rows
Manually create a new table named P&L rows using DATATABLE to define each line item and its order:

DAX
Copy
Edit
P&L rows = DATATABLE(
    "P&L Item", STRING,
    "Order", INTEGER,
    {
        {"Gross Sales", 1},
        {"Pre-Invoice Deduction", 2},
        {"Net Invoice Sales", 3}
    }
)
This table acts as a layout blueprint for your P&L view, keeping the KPIs ordered and aligned vertically.

🧮 1.2 Create the Dynamic P&L Measure
Next, create a DAX measure named P&L values to return different financial values based on the selected row using SWITCH(TRUE()).

DAX
Copy
Edit
P&L values = 
SWITCH(
    TRUE(),
    MAX('P&L rows'[Order]) = 1, [GS ₹],
    MAX('P&L rows'[Order]) = 2, [Pre_Invoice_Deduction ₹],
    MAX('P&L rows'[Order]) = 3, [NIS ₹]
)
This measure dynamically pulls:

Gross Sales → [GS ₹]

Pre-Invoice Deduction → [Pre_Invoice_Deduction ₹]

Net Invoice Sales → [NIS ₹]

🎨 1.3 Build the Matrix Visual
Use Power BI’s matrix visual to create the final view:

Rows: P&L rows[P&L Item]

Values: P&L values measure

Sort: Sort by Order column to maintain line order

Styling:

Hide column headers

Format values as currency (₹)

Remove subtotals for cleaner layout  

snapshot:![finance view](https://github.com/user-attachments/assets/6577eb22-de88-46aa-b362-8fba1ab61eca)

✅ Step 2: Add Time Intelligence and Dynamic Slicer for Actuals
📌 2.1 Calculate Last Year (LY) Values
You want to compare the current period’s values with the same period from last year.

💡 DAX Formula:

dax
Copy
Edit
LY = 
CALCULATE(
    [P&L values], 
    SAMEPERIODLASTYEAR(dim_date[date])
)
🔍 Explanation:

[P&L values] is your base measure (e.g., Net Sales, Gross Sales).

SAMEPERIODLASTYEAR() shifts the current date context by one year.

Ensure dim_date[date] is used in your visual and has no missing dates.

📌 2.2 Create a Dynamic Slicer for Actual Fiscal Years Only
To show only those fiscal years where actual data is available.

💡 DAX Table:

dax
Copy
Edit
Actual_fiscal-year = 
DISTINCT (
    SELECTCOLUMNS (
        FILTER (
            fact_actual_estimate,
            NOT ISBLANK ( fact_actual_estimate[net_invoice_sales_amount] )
                && NOT ISBLANK ( fact_actual_estimate[fiscal_year] )
        ),
        "fiscal_year", fact_actual_estimate[fiscal_year]
    )
)

🔍 Explanation:

Filters out blank actual values.

Extracts distinct fiscal_year from your actuals.

Use this as a slicer to limit the visual to available years only.

🔗 Don’t forget to create a relationship between Actual_fiscal-year[fiscal_year] and dim_date[fiscal_year].

📌 2.3 Add “Est” to the Latest Fiscal Year Label
To show the most recent fiscal year as "2025 Est" or similar.

💡 Add Column to Actual_fiscal-year Table:

dax
Copy
Edit
fy_desc = 
VAR MaxFY = MAX('Actual_fiscal-year'[fiscal_year])
RETURN IF(
    'Actual_fiscal-year'[fiscal_year] = MaxFY,
    MaxFY & " Est",
    'Actual_fiscal-year'[fiscal_year]
)
🔍 Explanation:

Detects the most recent year.

Adds " Est" only to the latest year.

Keeps previous years unchanged.

Use this fy_desc column in your slicer instead of raw fiscal years.

✅ Result: Your slicer will look like:

yaml
Copy
Edit
2019  
2020  
2021  
2022  
2023  
2024  
2025 Est  



📸 Snapshot
![p1](https://github.com/user-attachments/assets/996af952-fe17-489f-9089-54e5117a9e46)


✅ Step 3 – Creating YoY Metrics and Dynamic Column Header Table for Final P&L Output
To enhance flexibility and visibility into year-over-year (YoY) performance, we extended our model by introducing YoY measures and a dynamic table for visual structuring. Here's the breakdown:

🔹 1. YoY and YoY% Change Measures
We first calculated the YoY change in profit and loss values using simple DAX measures:

DAX
Copy
Edit
P&L YoY Chg = [P&L values] - [P&L LY]
Then, we derived the YoY percentage change, with a safe divide to avoid divide-by-zero errors:

DAX
Copy
Edit
P&L YoY % chg = DIVIDE([P&L YoY Chg], [P&L LY], 0) * 100
These measures help compare current fiscal year values against the previous year in both absolute and percentage terms.

🔹 2. Dynamic Column Header Table (P&L Columns)
To organize these different perspectives (LY, YoY, YoY%) along with fiscal years, we created a new calculated table to serve as a column header source in matrix visuals:

DAX
Copy
Edit
P&L Columns =
VAR x = ALLNOBLANKROW('Actual_fiscal-year'[fiscal_year])
RETURN
UNION(
    ROW("Col Header", "LY"),
    ROW("Col Header", "YoY"),
    ROW("Col Header", "YoY %"),
    x
)
This allows the matrix visual to dynamically display LY, YoY, and YoY% columns alongside the actual fiscal years.

🔹 3. Unified Measure for Matrix Output
We then created a final measure to control which value appears under which column, based on the selected column header context:

DAX
Copy
Edit
P&L final value =
SWITCH(
    TRUE(),
    SELECTEDVALUE('Actual_fiscal-year'[fy_desc]) = MAX('P&L Columns'[Col Header]), [P&L values],
    MAX('P&L Columns'[Col Header]) = "LY", [P&L LY],
    MAX('P&L Columns'[Col Header]) = "YoY", [P&L YoY Chg],
    MAX('P&L Columns'[Col Header]) = "YoY %", [P&L YoY % chg]
)
💡 Why This Approach?
📊 More control: By combining actual fiscal years and derived metrics like LY and YoY into a single column header, this method enables more structured and readable visuals.

🔁 Dynamic behavior: It adapts automatically to the latest year using earlier logic (see Step 2), while keeping older years and comparison metrics aligned.

✅ Cleaner visuals: You no longer need separate visuals or columns for LY/YoY metrics — all logic is centralized in one measure.

📸 Snapshot
![p3](https://github.com/user-attachments/assets/ef0b7570-300a-4914-b23b-0bc74813163e)





