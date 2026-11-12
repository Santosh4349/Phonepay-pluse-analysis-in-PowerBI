# Phonepay-pluse-analysis-in-PowerBI
# PhonePe Pulse Data Visualization and Exploration

An interactive two-page Power BI dashboard that explores digital payment activity across India, using the open [PhonePe Pulse](https://github.com/PhonePe/pulse) data (2018–2022). It combines a geo map, KPIs, growth analysis and top-performer rankings, with 10 dropdown selectors for slicing the data.

**Domain:** Fintech | **Tool:** Power BI | **Data period:** 2018 Q1 – 2022 Q4 (brand data to 2022 Q1)

---

## Features

### Page 1: Overview
- **Filled geo map** of India with state-to-district drill-down. A *Map Metric* selector switches the shading between transaction amount, transaction count and registered users.
- **KPI cards:** Total Transactions, Total Amount, Registered Users, Average Transaction Value, and Transactions per Registered User (engagement).
- **Pie chart:** share of transaction value by payment type.
- **Bar chart:** top states by transaction amount (controlled by *Top N*).

### Page 2: Trends & Top Performers
- **Metric trend line:** quarterly trend of the selected metric.
- **Growth line:** switch between year-over-year (YoY) and quarter-over-quarter (QoQ) growth.
- **KPI cards:** YoY growth and CAGR.
- **Top districts** by transaction amount and by registered users.
- **Top 10 device brands** by registered users.

### 10 dropdown selectors
Year, Quarter, State, District, Transaction Type, Device Brand, Metric, Map Metric, Top N, Growth View.

---

## Data model

Six cleaned CSV files are loaded into a star schema. The files sit at different levels of detail, so they are linked through small lookup tables instead of being merged into one table, which avoids double counting.

| Table | Grain |
|---|---|
| `aggregated_transaction` | state × quarter × transaction type |
| `aggregated_user` | state × quarter × device brand |
| `map_transaction` / `map_user` | state × quarter × district |
| `top_transaction` / `top_user` | top 10 districts per state × quarter |

**Lookup tables:** `Dim_State` (36 states and union territories), `Dim_Period` (20 quarters), `Dim_District` (732 districts).

Relationships are one-to-many with single-direction filtering:
- `Dim_State` → all `Agg_*` and `Top_*` tables, and → `Dim_District`
- `Dim_District` → `Map_Transaction` and `Map_User`
- `Dim_Period` → all six data tables

---

## Data preparation (Power Query)
- Removed the unnamed index column from each file.
- Cleaned state names (`andaman-&-nicobar-islands` → `Andaman and Nicobar Islands`) so the map can geocode them.
- Created `PeriodKey` (year × 10 + quarter) and `DistrictKey` (state + district) as join keys.
- Built one reusable load function applied to all six files.

---

## Key DAX measures
- `Total Amount`, `Total Transactions`, `Avg Transaction Value`
- `Registered Users`, `Brand Users`, `Map Registered Users`: use the **latest quarter** in the selection, because the user counts in the source are cumulative running totals
- `Transactions per Registered User`
- `Amount YoY %`, `Amount QoQ %`, `Amount CAGR %`
- Top N ranking measures built with `RANKX`
- Field parameters for *Metric*, *Map Metric* and *Growth View*

---

## Key insights
1. **Strong but slowing growth.** Transaction value grew from about ₹1.6 trillion in 2018 to about ₹64.3 trillion in 2022, a CAGR of roughly 151%. YoY growth slowed from about 287% in 2019 to 86% in 2022.
2. **Concentrated geography.** In 2022, Telangana (14.5% of value), Maharashtra (12.2%) and Karnataka (10.9%) lead, and the top 5 states account for about 54% of all transaction value. Hyderabad, Bengaluru Urban and Pune are the top districts.
3. **Payment mix.** Peer-to-peer transfers make up about 78% of value but 38% of transactions, while merchant payments make up about 53% of transactions but only 19% of value.
4. **Shift in usage.** Recharge and bill payments fell from about 37% of transactions in 2018 to 9% in 2022, while merchant payments rose from about 10% to 53%.
5. **Users versus value.** Maharashtra and Uttar Pradesh have the most registered users, but Telangana leads in value, which points to uneven value per user across states.
6. **Devices.** In 2022 Q1, Xiaomi (about 24%), Vivo (about 21%) and Samsung (about 18%) lead among registered users.

---

## Limitations
- The data covers 2018–2022 only. It is a static extract, not a live feed.
- Device brand data ends at 2022 Q1.
- Top-district tables contain only the top 10 districts per state and quarter, so national rankings built from them are approximate.
- User counts are cumulative, so they should never be summed across quarters.

---

## Repository contents
- `Phonepay-pluse.pbix`: Power BI report
- `data/`: The six cleaned CSV files
- `images/`:Two dashboard screenshots

## How to use
1. Download the `.pbix` file and open it in Power BI Desktop.
2. If prompted, update the CSV file path in the `fnLoad` query (Transform data → fnLoad) to point to your local `data/` folder.
3. Click **Refresh**, then explore using the slicers.

---

## Data source
[PhonePe Pulse](https://github.com/PhonePe/pulse), an open data repository by PhonePe.
