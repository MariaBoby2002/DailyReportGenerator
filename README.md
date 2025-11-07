## Daily Customer & Order Report Notebook

---

## Objective
To process and analyze customer and order data from multiple sources (CSV and XML) using both **database table** and **data-frame (in-memory)** approaches.  

- Data is shared once a day from the source, with new records for the previous day only.  
- Add missing data if needed.  
- Goal: derive key business insights such as:
  - Repeat customers
  - Monthly order trends
  - Regional revenue
  - Top spenders in the last 30 days

---

## 1. Prerequisites and Setup

### 1.1 Dependencies and Environment Setup
This notebook relies on the following Python libraries:

- `pandas` – For data manipulation and analysis  
- `numpy` – For numerical operations  
- `os` – For interacting with the operating system (e.g., creating directories)  
- `io` – For handling in-memory byte streams  
- `re` – For regular expressions  
- `datetime` – For date and time operations  
- `xml.etree.ElementTree` – For parsing XML data  
- `getpass` – For secure password input  
- `IPython.display` – For displaying content like file links in Colab  

> **Note:** In Google Colab, most libraries are pre-installed.  
> If running locally, install the required packages with:  
> ```bash
> pip install pandas numpy openpyxl
> ```
> `openpyxl` is required for writing Excel files.

---

## 2. Data Understanding

### 2.1 Customer Data (CSV/XLSX)
Each row represents a customer  

**Key columns:**  
- `customer_id` – Unique identifier  
- `customer_name` – Customer name  
- `mobile_number` – Contact number (used to link with orders)  
- `region` – Geographical region  

### 2.2 Order Data (XML)
Each row represents an order  

**Key columns:**  
- `order_id` – Unique order identifier  
- `mobile_number` – Links to customer  
- `order_date_time` – Order timestamp  
- `sku_id` – Product identifier  
- `sku_count` – Quantity ordered  
- `total_amount` – Total price  

### 2.3 Relationships
- Customers and orders are linked via `mobile_number`.  
- Orders can be aggregated by date, region, or customer for KPI calculations.

---

## 3. Handling Credentials with Colab Secrets
The notebook uses Google Colab Secrets for Stage-1 and Stage-2 authentication:

- `STAGE1_USER`  
- `STAGE1_PWD`  
- `STAGE2_USER`  
- `STAGE2_PWD`  

**Setup Steps:**  
1. Click the "🔑" icon in Colab left sidebar  
2. Click **Manage Secrets → Add new Secret**  
3. Add each secret with its corresponding value  
4. Ensure Notebook access is enabled  

**Access secrets in the notebook using:**  
userdata.get("SECRET_NAME")

  
## 4. Notebook Workflow

### 4.1 Setup & Configuration (Cells 1 & 2)
- Import libraries  
- Define global constants (`TODAY`, `CURRENT_MONTH_START_DATE`)  
- Set required column names for input DataFrames  
- Create folder for historical data  

**Includes helper functions for:**  
- Column normalization  
- Schema checking  
- Saving history  

---

### 4.2 Stage-1 Authorization & Data Upload (Cell 3)
- Prompt user for Stage-1 credentials  
- Authenticate against stored secrets  

**Upload interface for:**  
- Customer data (CSV or XLSX)  
- Orders data (XML)  

**Reads uploaded files into pandas DataFrames:**  
- `loaded_customers_df`  
- `loaded_orders_df`  

- Normalize column names and perform schema checks  

---

### 4.3 Data Preparation & History Backup (Cell 4)
- Split `loaded_orders_df` into:  
  - `temp_previous_day_df` – Orders before current day  
  - `current_day_orders_df` – Orders from current day (or current month if no current day orders)  
- Save customer and order data to daily history folder  

---

### 4.4 Merge Customers & Orders (Cell 5)
- Merge `loaded_customers_df` with `current_day_orders_df` on `mobile_number`  
- Resulting DataFrame `merged_df` used for KPI calculations  

---

### 4.5 KPI Function Definitions (Cell 6)
Python functions defined for KPIs:  
- `kpi_repeat_customers` – Identifies repeat customers  
- `kpi_regional_revenue` – Calculates total revenue per region  
- `kpi_monthly_order_trends` – Calculates monthly order trends  
- `kpi_top_customers_last_30_days` – Top customers by spend in last 30 days  

---

### 4.6 KPI Calculation (Cell 7)
- Calls KPI functions on appropriate DataFrames (`merged_df`, `loaded_orders_df`)  
- Temporary merge of orders + customers for top spenders  

**Outputs KPI DataFrames:**  
- `kpi_repeat`  
- `kpi_region`  
- `kpi_monthly_trends`  
- `kpi_top_customers`  

---

### 4.7 Stage-2 Authorization & Report Export (Cell 8)
- Prompt Stage-2 credentials  
- Format datetime columns for export  

**Exports all DataFrames into a single Excel file with sheets:**  
- `Customers` – Raw customer data  
- `Current_Day_Orders` – Orders of current day/month  
- `Archived_Previous_Day` – Orders before current day  
- `KPI_Repeat_Customers`  
- `KPI_Regional_Revenue`  
- `KPI_Monthly_Order_Trends`  
- `KPI_Top_Customers_Last_30_Days`  

- Generate Colab FileLink for download  

---

## 5. Key Performance Indicators (KPIs)

### 5.1 Repeat Customers
- Customers with more than one order  
- Group by `customer_id` & `customer_name`, count `order_id`  
- Filter `order_count > 1`  

### 5.2 Regional Revenue
- Sum `total_amount` per region in `merged_df`  
- Sort descending by revenue  

### 5.3 Monthly Order Trends
- Extract month from `order_date_time` in `loaded_orders_df`  
- Group by month, count `order_id`  

### 5.4 Top Customers by Spend (Last 30 Days)
- Merge orders with customer info  
- Filter orders in last 30 days from `TODAY_UTC`  
- Sum `total_amount` per customer  
- Sort descending  

---

## 6. Output Report
- **File Name:** `Daily_Report_YYYYMMDD.xlsx`  
- **Location:** `/content/` (Colab environment)  

**Sheets:**  
- `Customers` – Raw customer data  
- `Current_Day_Orders` – Current day/month orders  
- `Archived_Previous_Day` – Orders before current day  
- `KPI_Repeat_Customers`  
- `KPI_Regional_Revenue`  
- `KPI_Monthly_Order_Trends`  
- `KPI_Top_Customers_Last_30_Days`  

---

## 7. Execution Instructions
- Open the notebook in Google Colab  
- Set up Colab Secrets: `STAGE1_USER`, `STAGE1_PWD`, `STAGE2_USER`, `STAGE2_PWD`  
- Run cells sequentially (Shift + Enter or play button)  
- Stage-1 Authentication & Upload: Enter credentials and upload input files  
- Stage-2 Authentication: Enter credentials for export  
- Monitor execution and check outputs/errors  
- Download report using generated FileLink  

---

## 8. Future Suggestions
- **Database Integration & Automation:** Replace XML uploads with live DB connection; automate daily ingestion  
- **Interactive Visualization Dashboard:** Use Plotly or Streamlit to create KPI charts  

---

## 9. Summary & Insights
- Processes customer & order data to calculate KPIs  
- Generates daily Excel report with historical backups  

**KPIs calculated:**  
- Repeat Customers  
- Regional Revenue  
- Monthly Order Trends  
- Top Customers by Spend (last 30 days)  

- Output is ready for download in Colab  
- Potential improvements: error handling, scalability, automation for hands-off reporting





