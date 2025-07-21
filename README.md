📊 Power BI – Building the Financial View (P&L)
🔧 Objective
To build a Profit & Loss (P&L) style financial matrix in Power BI, where key financial metrics like Gross Sales, Pre-Invoice Deduction, and Net Invoice Sales are arranged vertically like a financial statement — using a supporting table, DAX logic, and matrix visuals.

🧱 Step 1: Create the Core P&L Structure
This step involves preparing the base structure that will support the dynamic P&L view.

🧾 1.1 Create Supporting Table for P&L Rows
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

📸 Snapshot

![finance view](https://github.com/user-attachments/assets/6577eb22-de88-46aa-b362-8fba1ab61eca)


