# 🛡️ Shield Insurance - Power BI Analysis
[🔗 **Live Power BI Dashboard**](https://app.powerbi.com/view?r=eyJrIjoiODc5MTI0YWYtMDc4NC00MTFhLWE1MDUtZGVjY2JkOGMyZjRkIiwidCI6ImM2ZTU0OWIzLTVmNDUtNDAzMi1hYWU5LWQ0MjQ0ZGM1YjJjNCJ9)  
*(Click to view the interactive dashboard)*

## 🏢 Company Overview
**Shield Insurance Company** provides reliable and comprehensive insurance plans for individuals and businesses, ensuring protection from various risks.  
Known for its commitment to customer care and security, Shield stands out in the market for its focus on **coverage reliability**, helping customers feel safe and secure.

---

## 🎯 Problem Statement
**Shield Insurance** aims to improve its data-driven decision-making by implementing a **Power BI dashboard** for actionable insights into key performance metrics.  
To evaluate this, they are considering a collaboration with **AtliQ Technologies**.  
As a **Proof of Concept (POC)**, Shield Insurance requires a **Pilot Project** in Power BI to demonstrate AtliQ’s capability to meet their analytical needs before committing to a full-scale project.

---

## 🧩 1. ASK – Preliminary Analysis
**Key Requirements:**
- Show **Total Customers**, **Total Revenue**, **Daily Revenue Growth (DRG)**, and **Daily Customer Growth (DCG)** as key metrics.  
- Display **Month-over-Month % Change** on key metrics.  
- Segment customers by **Age Groups:** 18–24, 25–30, 31–40, 41–50, 51–65, 65+.  
- Show **Total Revenue & Customers** split by **Age Group** and **City**.  
- Plot **Customer and Revenue Growth Trends** by Month.  
- Add a **switch (toggle)** between *Revenue Trend* and *Customer Trend* graphs.  
- Include filters for **Sales Mode**, **Age Group**, **City**, **Month**, and **Policy ID**.  
- **Separate Pages**:
  - **Sales Mode Analysis:** Split percentage and monthly trend.
  - **Age Group Analysis:** Age Group vs. Expected Settlement, Sales Mode, and Policy Preference.

---

## 💾 2. PREPARE – Data Storage
The dataset is available on the **Code Basis** platform, consolidating multiple CSV files for analysis.

**Files Provided:**
- `dim_customer.csv`
- `dim_date.csv`
- `dim_policies.csv`
- `fact_premiums.csv`
- `fact_settlements.csv`

---

## ⚙️ 3. PROCESS – Tools & Data Preparation

### 🧰 Tools Used
- **Microsoft Excel**
- **Power BI**
- **DAX Studio**
- **DAX**

### 📂 Data Description

#### `dim_customer`
| Column | Description |
|---------|--------------|
| customer_code | Unique code for each customer |
| dob | Customer's date of birth |
| city | City of the customer |

#### `dim_date`
| Column | Description |
|---------|--------------|
| date | Date at the daily level |
| mmm_yy | Month-Year (e.g., Jan-23) |
| day_type | Weekday name |
| week_no | Week number of the year |

#### `dim_policies`
| Column | Description |
|---------|--------------|
| policy_id | Unique policy ID |
| base_cover | Base coverage amount |
| base_premium_amt(INR) | Base premium amount |

#### `fact_premiums`
| Column | Description |
|---------|--------------|
| date | Policy sale date |
| customer_code | Customer unique code |
| policy_id | Policy identifier |
| sales_mode | Mode of sales (e.g., Offline-Agent, Online-App) |
| final_premium_amt(INR) | Final premium paid |

#### `fact_settlements`
| Column | Description |
|---------|--------------|
| age | Policyholder age |
| settlement% | Settlement percentage by age |

---

### 🧹 Data Cleaning & Transformation
- Removed empty columns from `fact_settlements`.  
- Split `sales_mode` into **Mode: Online/Offline** and **Mode: Through Medium**.  
- Changed data types as needed (e.g., Settlement %).  
- Used **Power Query** for cleaning and transformation.

---

## 📊 4. ANALYZE – Power BI Data Modeling & Measures

### 🧠 Data Modeling
<img width="1012" height="515" alt="Screenshot 2025-11-05 233013" src="https://github.com/user-attachments/assets/82f811df-a121-496c-80d0-aef717b3b537" />

### **New Columns Created:**

-- dim_customer table

1. Age = 2023 - YEAR(dim_customer[dob].[Date])
2. AgeGroup = SWITCH(
    TRUE(),
    dim_customer[Age] >= 18 && dim_customer[Age] <= 24, "18-24",
    dim_customer[Age] >= 25 && dim_customer[Age] <= 30, "25-30",
    dim_customer[Age] >= 31 && dim_customer[Age] <= 40, "31-40",
    dim_customer[Age] >= 41 && dim_customer[Age] <= 50, "41-50",
    dim_customer[Age] >= 51 && dim_customer[Age] <= 65, "51-65",
    "65+"
)
3. SettlementINDecimal = RELATED(fact_settlements[settlement %])

-- dim_date table

1. Month = FORMAT(dim_date[date].[Date], "MMM")
2. SortedBy = SWITCH(TRUE(), dim_date[Month]="Nov", 1, dim_date[Month]="Dec", 2, dim_date[Month]="Jan", 3, dim_date[Month]="Feb", 4, dim_date[Month]="Mar", 5, 6)

---

## 📏 DAX Measures

1. Total Customers = COUNT(dim_customer[customer_code])
2. Total Revenue = SUM(fact_premiums[final_premium_amt(INR)])
3. DCG = VAR _tot_cus = [Total Customers]
       VAR _date = DISTINCTCOUNT(dim_date[date])
       RETURN DIVIDE(_tot_cus, _date, 0)
4. DRG = VAR _tot_rev = [Total Revenue]
       VAR _date = DISTINCTCOUNT(dim_date[date])
       RETURN DIVIDE(_tot_rev, _date, 0)
5. %CustomerChange = DIVIDE([Total Customers] - [LM Customers], [LM Customers], 0)
6. %RevenueChange = DIVIDE([Total Revenue] - [LM Revenue], [LM Revenue], 0)
7. %DCGChange = DIVIDE([DCG] - [LM DCG], [LM DCG], 0)
8. %DRGChange = DIVIDE([DRG] - [LM DRG], [LM DRG], 0)
9. LM Customers = CALCULATE([Total Customers], PREVIOUSMONTH(dim_date[date]))
10. LM Revenue = CALCULATE([Total Revenue], PREVIOUSMONTH(dim_date[date]))
11. LM DCG = CALCULATE([DCG], PREVIOUSMONTH(dim_date[date]))
12. Cus% = VAR _tot = CALCULATE([Total Customers], REMOVEFILTERS(fact_premiums))
       RETURN DIVIDE([Total Customers], _tot)
13. Rev% = VAR _tot = CALCULATE([Total Revenue], REMOVEFILTERS(fact_premiums))
       RETURN DIVIDE([Total Revenue], _tot)
14. Expected Settlement = ROUND(
    SUMX(fact_premiums, fact_premiums[final_premium_amt(INR)] * (1 + RELATED(dim_customer[SettlementINDecimal]))),
    0
)

---

## 📈 5. SHARE – Dashboard Snapshots

**Home Page**

<img width="1400" height="790" alt="image" src="https://github.com/user-attachments/assets/944c0ce6-2cfe-4800-8844-37f5cd9a02fb" />

**Overview Page**

<img width="1400" height="790" alt="Screenshot 2025-11-05 232154" src="https://github.com/user-attachments/assets/125212f7-15b7-4652-b534-f72132bd1de4" />

**Sales Mode Analysis**

<img width="1400" height="790" alt="Screenshot 2025-11-05 232214" src="https://github.com/user-attachments/assets/f58ad796-f897-4d8b-909e-81c2826a5edc" />

**Age Group Analysis**

<img width="1400" height="790" alt="Screenshot 2025-11-05 232231" src="https://github.com/user-attachments/assets/a01c6d1d-9f7a-42e1-b16f-9a1a6e7fa8c7" />


## 🔍 6. ACT – Insights & Recommendations
📊 Key Insights

1. Total Customers: 26,841
2. Total Revenue: ₹989.25 Million
3. Top City: Delhi with 11,007 customers and ₹401.57M revenue.
4. Top Age Group: 31–40 years — 11,171 customers contributing ₹343.76M.
5. Best Month: March 2023 — +85% revenue, +82% customers.
6. Decline Month: April 2023 — −41.73% revenue, −41.41% customers.
7. Top Sales Mode: Offline Agent – 55.41% customers, 55.67% revenue.
8. Top Policy (by Customers): POL4321HEL – 4,434 customers.
9. Top Policy (by Revenue): POL2005HEL – ₹324.26M.

---

## 💡 Recommendations

1. Focus on Digital Channels: Strengthen online platforms (app/website) to diversify revenue sources.
2. Target Younger Segments: Continue engaging the 31–40 group through digital marketing and tailored offers.
3. Analyze April Drop: Investigate sales or marketing gaps that caused the decline.
4. Promote High-Performing Policies: Highlight top policies like POL4321HEL and POL2005HEL for marketing.
