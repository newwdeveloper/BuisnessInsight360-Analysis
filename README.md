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




