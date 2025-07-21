📊 Power BI – Building the Financial View (P&L)
🔧 Objective:
To build a Profit & Loss (P&L) style matrix in Power BI where financial KPIs such as Gross Sales, Pre-Invoice Deduction, and Net Invoice Sales are aligned vertically like a statement, using DAX and matrix visuals.

🧱 Step 1: Create a Supporting Table for P&L Rows
To design the layout, I created a new table manually named P&L rows that defines the label and order of each line item to appear in the matrix.

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
This table helps in vertically aligning our KPIs in a desired order using a matrix visual.

🧮 Step 2: Create the P&L Measure Using SWITCH
Then I created a dynamic measure that shows the correct value depending on the selected row using the SWITCH(TRUE()) logic. Here's the measure I used:

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
This measure pulls values for:

Gross Sales (GS ₹)

Pre-Invoice Deduction (₹)

Net Invoice Sales (₹)
…based on the selected row from the P&L rows table.

🎨 Step 3: Build the Matrix Visual
Rows: Drag the P&L rows[P&L Item]

Values: Use the [P&L values] measure

Sort Order: Sort by Order column from the P&L rows table to ensure correct sequence

Styling: Cleaned headers, adjusted formatting to show ₹ values neatly

📸 Snapshot:


![finance view](https://github.com/user-attachments/assets/f335a110-45e6-4c76-856c-700041dd81e8)


📝 Summary
This approach helped me build a structured and readable P&L-style dashboard, which can be easily extended to add more financial metrics or subtotals in future. This setup keeps my model modular and my DAX logic centralized.
