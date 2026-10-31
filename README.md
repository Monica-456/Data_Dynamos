# FinTrust Bank: Credit Card Spending, Repayment and Churn Analysis

Hackathon project **DA-02**, **Team Data Dynamos**, Codegnan.
Tools used: **Python (pandas), SQL (MySQL), Excel and Power BI**.

> **About the data:** the 5,000-customer dataset in this repository is synthetic practice data, made to match the fields in the problem statement. It is not real customer data.

---

## Table of contents
1. [What is this project about?](#1-what-is-this-project-about)
2. [Words used in this project](#2-words-used-in-this-project)
3. [The data: what each table and column means](#3-the-data-what-each-table-and-column-means)
4. [Folder structure: what is in each folder](#4-folder-structure-what-is-in-each-folder)
5. [How to run the project](#5-how-to-run-the-project)
6. [Python scripts: what each one does](#6-python-scripts-what-each-one-does)
7. [SQL files: what each one does](#7-sql-files-what-each-one-does)
8. [Power BI: how the dashboard is built](#8-power-bi-how-the-dashboard-is-built)
9. [The 7 KPIs explained](#9-the-7-kpis-explained)
10. [The 7 charts explained](#10-the-7-charts-explained)
11. [Findings: what is happening, why, and what the bank should do](#11-findings-what-is-happening-why-and-what-the-bank-should-do)
12. [Three customer groups to target](#12-three-customer-groups-to-target)
13. [Limits and assumptions](#13-limits-and-assumptions)
14. [Common problems and fixes](#14-common-problems-and-fixes)
15. [Deliverables checklist](#15-deliverables-checklist)
16. [Team](#16-team)

---

## 1. What is this project about?

**FinTrust Bank** is a retail bank in South India with many credit card customers.

**The problem:**
- Card spending is growing.
- But many customers have **stopped using their cards** or **closed them**.
- The bank keeps spending more on **rewards and cashback**.
- The risk team sees more **late payments** in some customer groups.
- Leadership is not sure whether its offers are working.

**The questions we answer:**
1. Which customers are leaving?
2. Why are they leaving?
3. Where is the bank losing money?

**Who will use the answers:**

| Team | What they need from us |
|---|---|
| Card Business Head | Where to invest to grow |
| Product and Rewards Team | Whether cashback and rewards work |
| Risk Team | Which groups pay late or use too much of their limit |
| Customer Retention Team | Which at-risk customers to contact with offers |

**What we did, in short:** cleaned the data in Python, analysed it in SQL, summarised it in Excel, built a dashboard in Power BI, and wrote recommendations.

---

## 2. Words used in this project

| Word | Meaning |
|---|---|
| **Customer** | One person with one credit card. Each has a unique `customer_id` like `C000001`. |
| **Transaction** | One purchase made with the card. |
| **Statement** | The monthly bill sent to a customer. |
| **Active** | A customer whose account is open and in use. |
| **Inactive** | A customer whose account is still open but who stopped using the card. |
| **Closed** | A customer who closed the card. |
| **Churn** | A customer leaving. In the dashboard: **Inactive plus Closed**. |
| **Churn rate** | Churned customers divided by all customers. |
| **Credit limit** | The most a customer is allowed to spend on the card. |
| **Credit utilisation** | How much of the limit is used. Here: spend divided by credit limit. |
| **Late payment** | A statement paid after its due date (days late above 0). |
| **Reward points** | Points earned for spending. |
| **Redemption** | Using points to get cashback. |
| **Redemption rate** | Points redeemed divided by points earned. |
| **Cashback** | Money given back to the customer when points are redeemed. |
| **KPI** | Key performance indicator: one important number. |
| **Segment** | A group of similar customers, for example "Basic card, Low income". |
| **Cr** | Crore. 1 Cr = 10,000,000 (1 crore). |
| **L** | Lakh. 1 L = 100,000. |

---

## 3. The data: what each table and column means

There are **5 tables**. All of them link through **`customer_id`**.

### 3.1 customers (5,000 rows, one row per customer)
| Column | Meaning |
|---|---|
| `customer_id` | Unique ID of the customer |
| `age_group` | 18-25, 26-35, 36-45, 46-55 or 56+ |
| `city` | One of 12 South Indian cities (Hyderabad, Chennai, Bengaluru, Kochi and others) |
| `income_band` | Low, Middle, Upper-Middle or High |
| `card_type` | Basic, Silver, Gold or Platinum |
| `card_open_date` | Date the card was opened |
| `credit_limit` | Spending limit in rupees |

### 3.2 transactions (one row per purchase)
| Column | Meaning |
|---|---|
| `txn_id` | Unique ID of the transaction |
| `customer_id` | Who made it |
| `txn_date` | Date of the purchase |
| `amount` | Amount in rupees |
| `merchant_category` | Groceries, Fuel, Dining, Shopping, Travel, Electronics, Healthcare, Utilities, Entertainment, Education, Jewellery (or Unknown) |
| `channel` | Online or POS (in a shop) |
| `txn_city` | City where it was made |
| `transaction_month` | Added by us. Month like `2025-03` |
| `amount_category` | Added by us. Low (under 1,000), Medium (1,000 to 5,000), High (above 5,000) |

### 3.3 repayments (one row per customer per statement month)
| Column | Meaning |
|---|---|
| `statement_month` | Month of the bill |
| `bill_amount` | Amount billed |
| `amount_paid` | Amount the customer paid |
| `payment_date` | Date of payment |
| `days_late` | Days after the due date (0 means on time) |
| `payment_status` | Added by us. Fully Paid or Partially Paid |
| `late_payment_status` | Added by us. On Time or Late |

### 3.4 rewards (one row per customer per statement month)
| Column | Meaning |
|---|---|
| `reward_points_earned` | Points earned that month |
| `points_redeemed` | Points used that month |
| `cashback_amount` | Cashback paid for the redeemed points |
| `remaining_points` | Added by us. Earned minus redeemed |
| `redemption_status` | Added by us. Redeemed or Not Redeemed |

### 3.5 account_status (one row per customer)
| Column | Meaning |
|---|---|
| `account_status` | Active, Inactive or Closed |
| `closure_date` | Date the card was closed (only for Closed) |
| `closure_reason` | Why it was closed (only for Closed): better offer from a competitor, low rewards, high interest rate, not using the card, high annual fee, poor customer service |

### How the tables connect
```
customers  ──< transactions     (one customer, many purchases)
    │      ──< repayments       (one customer, many statements)
    │      ──< rewards          (one customer, many reward records)
    └──── account_status        (one customer, one status)
```

---

## 4. Folder structure: what is in each folder

```
Credit_Card_Analytics_Project/
├── data/
│   ├── raw/
│   ├── cleaned/
│   └── powerbi/
├── scripts/
├── SQl/
│   ├── SQL_Queries/
│   └── Cleaned_Datasets/
├── powerbi/
├── docs/
├── run_all.py
├── requirements.txt
├── .gitignore
└── README.md
```

| Path | What it holds |
|---|---|
| `data/raw/` | The original CSV files. **Never edit these.** If something goes wrong you can always start again from here. `account_status_extra.csv` is an extra file that the scripts do not use. |
| `data/cleaned/` | The cleaned CSV files created by scripts 01 to 05. |
| `data/powerbi/` | `FinTrust_PowerBI_Ready.csv`, one flat table (60,000 rows) made for Power BI. |
| `scripts/` | The Python scripts, numbered in the order they run. |
| `SQl/SQL_Queries/` | The MySQL files: create tables, import, validate, analyse, views. |
| `SQl/Cleaned_Datasets/` | A copy of the cleaned CSVs for the SQL import. |
| `powerbi/` | Dashboard preview (HTML), the 7 KPI formulas, colour theme, build guide. |
| `docs/` | `index.html`, the same dashboard, so GitHub Pages can show it online. |
| `run_all.py` | Runs all the scripts in the right order with one command. |
| `requirements.txt` | The Python libraries needed (`pandas`, `numpy`). |
| `.gitignore` | Tells Git which temporary files not to upload. |

---

## 5. How to run the project

### What you need first
- **Python 3.9 or newer** (check with `python --version`)
- **MySQL 8.0** (only for the SQL part)
- **Power BI Desktop** (free, Windows only; only for the `.pbix` part)

### Steps
1. Download or clone this repository and open a terminal in the `Credit_Card_Analytics_Project` folder.
2. Install the libraries:
   ```bash
   python -m pip install -r requirements.txt
   ```
3. Run everything:
   ```bash
   python run_all.py
   ```
   This runs 7 scripts in order. It stops and tells you if one of them fails.
4. To run only one script, for example the transactions cleaning:
   ```bash
   python scripts/05_clean_transactions.py
   ```

On Windows, if `python` does not work, use `py` instead.
All scripts find their folders on their own, so you can run them from any folder.

---

## 6. Python scripts: what each one does

Every script follows the same pattern: **load, inspect, remove duplicates, fix text, fix data types, fix bad values, add new columns, save**.

### 01_clean_account_status.py
Reads `data/raw/account_status_5000.csv` and writes `data/cleaned/cleaned_account_status.csv`.
1. Prints the first rows, size and missing values.
2. Removes duplicate rows, then duplicate `customer_id` (keeps the first).
3. Removes extra spaces and makes `account_status` consistent (for example `active` becomes `Active`).
4. Converts `closure_date` to a real date.
5. Checks that every Closed customer has a closure date and reason.
6. Prints how many customers are Active, Inactive and Closed.

### 02_clean_customers.py
Reads `customers_5000.csv`, writes `cleaned_customers.csv`.
1. Removes duplicate rows and duplicate `customer_id`.
2. Makes column names lowercase and trims spaces.
3. Makes text consistent: city, income band and card type in Title Case (`HYDERABAD ` becomes `Hyderabad`).
4. Turns empty text into proper missing values.
5. **Fills missing `income_band` with the most common value.** This is a simple baseline, not the real band.
6. Converts `card_open_date` to a date and `credit_limit` to a number.
7. Sets negative credit limits to missing.
8. Checks age groups, card types and future opening dates (it reports problems but does not change them).

### 03_clean_repayments.py
Reads `repayments_5000.csv`, writes `cleaned_repayments.csv`.
1. Removes duplicate rows and duplicate customer plus month records (keeps the first).
2. Converts `statement_month` and `payment_date` to dates, and the amounts to numbers.
3. Sets negative bills, payments and days late to missing.
4. Checks overpayments (paid more than billed) and keeps them, because they can be advance payments.
5. Checks for payment dates before the statement month.
6. **Adds `payment_status`** (Fully Paid or Partially Paid) and **`late_payment_status`** (On Time or Late).

### 04_clean_rewards.py
Reads `rewards_5000.csv`, writes `cleaned_rewards.csv`.
1. Removes duplicates (same customer and month).
2. Converts points and cashback to numbers.
3. Sets negative points and cashback to missing.
4. Checks for redeemed points greater than earned points.
5. **Adds `remaining_points`** (earned minus redeemed) and **`redemption_status`**.

### 05_clean_transactions.py
Reads `transactions_5000.csv`, writes `cleaned_transactions.csv`.
1. Removes duplicate rows and duplicate `txn_id` (this removed 192 duplicates).
2. Trims and standardises text columns.
3. Converts `txn_date` to a date and `amount` to a number.
4. **Missing `merchant_category` becomes `Unknown`** instead of deleting the transaction.
5. Checks negative amounts (kept, because they may be refunds), zero amounts and future dates.
6. **Adds `transaction_month`** and **`amount_category`** (Low, Medium, High).

### 06_build_powerbi_table.py
Reads the five cleaned files and writes `data/powerbi/FinTrust_PowerBI_Ready.csv`.
- It creates **one row for every customer and every month** (5,000 customers times 12 months = 60,000 rows).
- Each row has the customer details plus that month's spend, transaction count, statements, late statements, reward points and cashback.
- Months with no activity get 0.
- It adds helper columns: `is_active_month` (1 if the customer spent that month), `credit_limit_active` (credit limit if active, else 0), `month_no`, `month_name`, `quarter`.
- **Why one table?** In Power BI you then need no relationships, and the formulas stay short.

### 07_build_dashboard.py
Reads the Power BI table and cleaned files, and creates the interactive dashboard:
- `powerbi/FinTrust_Dashboard.html`
- `docs/index.html` (same file, for GitHub Pages)

It uses `powerbi/dashboard_template.html` as the page design and puts the data inside it.

### run_all.py
Runs scripts 01 to 07 in order and stops if any fails.

---

## 7. SQL files: what each one does

All SQL files are in `SQl/SQL_Queries/` and are written for **MySQL 8.0**. Run them in this order.

### 01_create_database_tables.sql
Creates the database `fintrust_bank` and five tables: `customers_clean`, `transactions_clean`, `repayments_clean`, `rewards_clean`, `account_status_clean`. It does not delete any existing data.

### 02_csv_import_templates.sql
Commands (`LOAD DATA LOCAL INFILE`) that load the cleaned CSV files into the tables.
- **Replace `C:/REPLACE_PATH/` with the real path** of your files. Use forward slashes.
- `LOCAL INFILE` must be enabled on both the MySQL server and client.
- Load each file **only once** into an empty table, or you will get duplicates.

### 03_validation_queries.sql
Checks that the import worked:
- Row counts should be: customers 5,000, account status 5,000, repayments 36,684, rewards 36,684, transactions 63,954.
- Shows the first 10 rows of each table.
- Checks for "orphan" rows, meaning transactions, repayments or rewards that point to a customer who does not exist. The result should be 0.

### 04_analysis_queries.sql (13 queries)
| # | Question it answers |
|---|---|
| 1 | Total spending and number of transactions |
| 2 | Monthly spending trend |
| 3 | Top merchant categories for each quarter |
| 4 | Spend by card type |
| 5 | Spend by city |
| 6 | Late payment rate |
| 7 | Late payment rate by income band and card type |
| 8 | Churn rate (assumes the values Active and Closed) |
| 9 | Churn by card type |
| 10 | Closure reasons |
| 11 | Rewards and redemption |
| 12 | Customers above 80% utilisation (uses spend divided by limit as a stand-in) |
| 13 | Customers whose monthly spend fell three months in a row |

### 05_powerbi_views.sql
Creates 4 views that Power BI can read: `vw_customer_profile`, `vw_monthly_spending`, `vw_monthly_repayments`, `vw_monthly_rewards`. Each view gives one row per customer per month, so totals are not counted twice when tables are joined.

### 06_business_queries.sql (20 queries)
The same analysis written as business questions (Q01 to Q20), for example: how many customers are active or closed, which cities spend the most, which customers have the most late payments, what is the closure percentage, which segments close the most, and a final dashboard KPI summary (Q20).

### Which SQL queries cover the 4 required tasks?
| Requirement | Query |
|---|---|
| Top merchant categories by spend each quarter | 04 query 3, 06 Q04 |
| Credit utilisation and flag above 80% | 04 query 12, 06 Q09 |
| Spend fell three months in a row | 04 query 13, 06 Q13 |
| Late payment rate by income band and card type | 04 query 7, 06 Q10 |

---

## 8. Power BI: how the dashboard is built

Power BI Desktop has to create and save the `.pbix` file. Follow these steps (about 20 minutes). The same steps are in `powerbi/POWERBI_BUILD_GUIDE.md`.

1. **Load the data.** Home > Get data > Text/CSV > choose `data/powerbi/FinTrust_PowerBI_Ready.csv` > Load. The table is called `FinTrust`.
2. **Add the colour theme.** View > Themes > Browse for themes > choose `powerbi/FinTrust_Colourful_Theme.json`.
3. **Add the 7 KPI measures.** Home > New measure. Open `powerbi/FinTrust_PowerBI_Simple_DAX.txt` and paste **one** measure at a time. Format Total Spend as Currency and every rate as Percentage.
4. **Fix the month order.** Select `month_name` > Column tools > Sort by column > `month_no`.
5. **Add the slicers (filters)** at the top: `month_name`, `card_type`, `city`, `age_group`.
6. **Add 7 card visuals**, one for each KPI.
7. **Add the charts** (see section 10).
8. **Save** as `FinTrust_Dashboard.pbix` inside the `powerbi/` folder.

**Layout tip:** slicers on the first row, KPI cards on the second row, charts below in rows of 2 or 3. Colour the Late Payment Rate card amber and the Churn Rate card red so risk stands out.

**Check your numbers.** With the 5,000-customer data and no filter selected, your cards should show the values in the next section. If they do not, check the data types of the columns and make sure no slicer is selected.

**Preview without Power BI:** open `powerbi/FinTrust_Dashboard.html` in a browser. It looks like the final dashboard and works with the same filters.

**Publish the preview online (free):** on GitHub go to Settings > Pages > Branch `main` > Folder `/docs` > Save. Your link will be `https://<your-username>.github.io/<repo-name>/`.

---

## 9. The 7 KPIs explained

| KPI | How it is calculated | Value (5,000 customers) | What it tells you |
|---|---|---|---|
| **Total Spend** | Sum of all transaction amounts | about 273.8 Cr | How much customers spent in 2025 |
| **Active Cards** | Customers with at least one transaction | 4,989 | How many cards are really in use |
| **Average Spend per Card** | Total Spend divided by Active Cards | about 5.5 L | How much a typical active card spends |
| **Credit Utilisation %** | Spend divided by the credit limit of cards used that month | 59.1% | How much of their limit customers use |
| **Late Payment Rate %** | Late statements divided by all statements | 17.6% | How often bills are paid late |
| **Churn Rate %** | (Inactive + Closed) divided by all customers | 18.5% | The share of customers who left |
| **Reward Redemption Rate** | Points redeemed divided by points earned | 43.0% | How much of the rewards customers actually use |

---

## 10. The 7 charts explained

| Chart | What it shows | How to read it |
|---|---|---|
| **Monthly spend trend** (line) | Total spend for each month | Rising line means growing spend. Look for dips and the year-end rise. |
| **Account status** (donut) | Share of Active, Inactive and Closed customers | The red and yellow parts are the customers lost or at risk. |
| **Spend by card type** (column) | Total spend for Basic, Silver, Gold, Platinum | Shows which card tiers bring the most spend. |
| **Top merchant categories** (bar) | The 5 categories with the highest spend | Shows where customers spend. Use it to plan cashback offers. |
| **Late payment rate by income band** (column) | Late rate for each income group | A tall bar means a risky group. |
| **Churn rate by card type** (column) | Churn rate for each card tier | A tall bar means that card type loses more customers. |
| **Why customers closed their cards** (bar) | Number of closed accounts for each closure reason | Shows what to fix first. |

Notes:
- The **Month filter does not change** the churn rate, account status or closure reasons, because those are about customers, not months.
- The other three filters (Card type, City, Age group) change everything.

---

## 11. Findings: what is happening, why, and what the bank should do

These numbers come from the 5,000-customer dataset. Re-check them if you use different data.

### Finding 1: Cheaper cards lose more customers
- **What is happening:** Churn is 18.5% overall (528 inactive and 395 closed). By card type: Basic 21.7%, Silver 18.8%, Gold 16.1%, Platinum 13.4%.
- **Why:** Basic card holders get the least value from rewards and have the lowest limits.
- **What to do:** Offer Basic card holders an upgrade or a first-year fee waiver.

### Finding 2: Late payments are concentrated in low-income and Basic customers
- **What is happening:** Late payment rate by income: Low 30.4%, Middle 18.3%, Upper-Middle 10.3%, High 6.7%. By card: Basic 23.6%, Silver 18.4%, Gold 14.0%, Platinum 10.0%.
- **Why:** Lower-income customers have tighter budgets, so bills are paid later.
- **What to do:** Send payment reminders before the due date and offer auto-pay to this group.

### Finding 3: Unused rewards go with higher churn
- **What is happening:** Customers who redeem under 20% of their points churn at 28.8%. Those who redeem over 60% churn at 10.9% and spend more on average.
- **Why:** If customers do not use rewards, the rewards do not give them a reason to stay.
- **What to do:** Remind customers about their points and make redemption easier. This shows a link, not proof that rewards cause the difference, so test it first.

### Finding 4: Customers leave for value reasons
- **What is happening:** The top closure reasons are a better offer from a competitor (28.4%) and low rewards (24.3%).
- **Why:** Both are about what the card gives the customer.
- **What to do:** Compare the rewards rate and fees with competitors.

### Finding 5: Everyday spending drives revenue
- **What is happening:** The top 5 categories by spend are Groceries, Shopping, Dining, Fuel and Utilities.
- **Why:** These are regular purchases that customers make all the time.
- **What to do:** Focus cashback offers on these categories.

---

## 12. Three customer groups to target

| # | Group | Why this group | Offer |
|---|---|---|---|
| 1 | **Low redeemers** (under 20% redemption) | Highest churn at 28.8% | Points reminders and easy-redeem offers |
| 2 | **Basic card holders with Low income** | Highest late payment and churn | Fee waiver, upgrade offer, payment reminders |
| 3 | **Customers aged 18 to 25** | Highest churn of any age group at 23.4% | Starter rewards and category cashback |

---

## 13. Limits and assumptions

- **The data is synthetic.** Findings describe this practice dataset, not a real bank.
- **Churn has two meanings.** The dashboard counts Inactive plus Closed (18.5%). Some SQL queries (for example 04 query 8 and 06 Q14) count only Closed customers, which is about 7.9%. Always say which one you mean.
- **Credit utilisation is an approximation.** It uses spending, not the outstanding balance, so it is not true utilisation.
- **Missing income bands** were filled with the most common value. This does not recover the real band.
- **Negative transaction amounts** were kept because they may be refunds.
- **A few transactions are above the credit limit** (36 in the cleaned data) and were kept. Review them before using them for decisions.
- **Spending decline (3 months in a row)** can happen by chance for customers with few transactions. Compare flagged and unflagged customers before drawing a conclusion.
- **The age check in script 02** does not include the `56+` group in its valid list, so it will report `56+` as invalid. The data is not changed by this check.

---

## 14. Common problems and fixes

| Problem | Fix |
|---|---|
| `python` is not recognised | Use `py` instead, or install Python and tick "Add Python to PATH" |
| `ModuleNotFoundError: pandas` | Run `python -m pip install -r requirements.txt` |
| SQL import says `LOCAL INFILE` is disabled | Enable `local_infile` on the MySQL server and the client |
| SQL import gives duplicate rows | You loaded a file twice. Empty the table and load it once. |
| Power BI month names are in the wrong order | Sort `month_name` by `month_no` |
| A Power BI KPI is different from the table above | Check column data types and clear all slicers |
| The dashboard HTML shows empty charts | You need an internet connection, because the chart library loads online |

---

## 15. Deliverables checklist
- [x] Cleaning scripts and cleaned datasets
- [x] SQL files (tables, import, validation, analysis, views)
- [x] Power BI table, DAX measures, theme and build guide
- [x] Interactive dashboard preview
- [ ] Python EDA notebook (`.ipynb`)
- [ ] Excel pivot-table workbook
- [ ] Power BI file (`.pbix`), built from the guide
- [ ] Short report and presentation (5 to 10 minutes)

---

## 16. Team
**Team Data Dynamos**

| Name | Role |
|---|---|
| _Yogesh Varma | SQL analysis |
| _Harika_ | Python data cleaning |
| _Mouni and Sai _ | Power BI, report, presentation |

**Contact:** add your GitHub profile or email here.
