# Olist E-Commerce Dashboard (Power BI)

An interactive 4-page Power BI dashboard built on the Brazilian E-Commerce Public Dataset by Olist. It covers sales performance, delivery, customers and payments, and sellers and products, with synced filters and page navigation.

## Table of Contents
- [Business Questions](#business-questions)
- [Dataset](#dataset)
- [Dashboard Pages](#dashboard-pages)
- [Data Model](#data-model)
- [DAX Measures](#dax-measures)
- [Key Insights](#key-insights)
- [Recommendations](#recommendations)
- [Limitations](#limitations)
- [Tools](#tools)
- [How to Use](#how-to-use)
- [Repository Structure](#repository-structure)

## Business Questions
1. What drives revenue, and how has it changed over time?
2. Where are the delivery problems, and how do they affect customer satisfaction?
3. Who are the customers, and how do they pay?
4. Which sellers and product categories perform best and worst?

## Dataset
- Source: [Olist Brazilian E-Commerce Public Dataset (Kaggle)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- 9 CSV files: orders, order items, payments, reviews, customers, sellers, products, geolocation, and the product category name translation table.
- Period: Sep 2016 to Oct 2018. Charts focus on Jan 2017 to Aug 2018, the months with complete data.

## Dashboard Pages

| Page | What it shows |
|---|---|
| **Overview** | KPI cards (Total Revenue, Orders With Items, Avg Review Score, Unique Customers, Total Freight, Avg Order Value), top 10 categories, top 10 states, revenue by month, revenue MoM %, order status donut |
| **Delivery** | Late Delivery %, Cancel Rate %, Total Orders, Avg Delivery Days, map of avg delivery days by state, review score by delivery status, late delivery % by month with an overall reference line |
| **Customers & Payments** | Total Payments, Repeat Customers %, Unique Customers, Avg Order Value, Avg Installments, top 10 cities, unique customers by month, orders by number of installments, payments by type |
| **Sellers & Products** | Total Revenue, Total Freight, Orders With Items, Active Sellers, Avg Review Score, top 10 sellers, top 10 states by sellers, top 10 categories (revenue vs review score combo), revenue vs review score scatter |

Interactivity:
- Year and State slicers are synced across all pages.
- Page navigator buttons on every page (Ctrl+Click in Power BI Desktop).

## Data Model
Star-schema style model:
- Fact-like tables: `Orders`, `Items`, `Payments`, `Reviews`
- Dimensions: `Customers`, `Sellers`, `Products`, `Geolocation`, `DimDate`, `product_category_name_translation`
- Measures live in a dedicated `Key Measures` table.

Important notes:
- `customer_unique_id` is used for customer counts, because `customer_id` changes with every order.
- `payment_installments` lives in `Payments`, so installment charts use a measure that counts orders from `Payments`.

## DAX Measures
Column names follow the original Olist files. Adjust them if your Power Query steps renamed anything.

```DAX
Avg Installments = AVERAGE ( Payments[payment_installments] )

Repeat Customers % =
VAR CustOrders =
    ADDCOLUMNS (
        VALUES ( Customers[customer_unique_id] ),
        "@Orders", CALCULATE ( DISTINCTCOUNT ( Orders[order_id] ) )
    )
RETURN
    DIVIDE (
        COUNTROWS ( FILTER ( CustOrders, [@Orders] > 1 ) ),
        COUNTROWS ( CustOrders )
    )

Active Sellers = DISTINCTCOUNT ( Items[seller_id] )

Orders by Payments = DISTINCTCOUNT ( Payments[order_id] )

Late Delivery % (Overall) =
CALCULATE (
    [Late Delivery %],
    REMOVEFILTERS ( DimDate[YearMonth] )
)
```

Other measures used: `Total Revenue`, `Total Freight`, `Avg Order Value`, `Total Orders`, `Orders With Items`, `Revenue MoM %`, `Late Delivery %`, `Cancel Rate %`, `Avg Delivery Days`, `Avg Review Score`, `Unique Customers`, `Total Payments`.

Why `Late Delivery % (Overall)` exists: the default average line on the monthly chart averages the monthly percentages (7.04%), which differs from the true rate across all orders (8.11%). This measure removes only the month filter, so the reference line matches the KPI card and still responds to the Year and State slicers.

## Key Insights

**Overview**
- Total revenue is **R$13.59M** from **98.7K** orders and **96K** customers; average order value is **R$137.75**.
- Revenue grows through 2017, peaks in **Nov 2017** (Black Friday season), then stays around R$0.85M to R$1.0M per month.
- Top categories: health_beauty (R$1.26M), watches_gifts (R$1.21M), bed_bath_table (R$1.04M).
- **SP** is the largest state by revenue (R$5.2M), followed by RJ and MG.

**Delivery**
- **8.11%** of orders were late; cancel rate is **0.63%**; average delivery time is **12.5 days**.
- Late delivery peaked at **21.36% in Mar 2018**, and at 14.31% in Nov 2017.
- Average review score is **4.3** for on-time orders, **2.6** for late orders, and **1.8** for undelivered orders.
- The North (RR, AP, AM) is slowest at 26 to 29 days; SP averages about 8.7 days with 5.89% late.

**Customers and Payments**
- **credit_card** is about 78% of payment value; boleto is about 18%.
- About half of the orders are paid in a single installment (49K); average installments is 2.9.
- Only **3.12%** of customers purchased more than once.
- sao paulo has about 15K customers, more than double rio de janeiro (6.6K).

**Sellers and Products**
- **3,095** active sellers; about 60% (1,849) are in **SP**.
- The top seller generated about R$229K.
- Most large categories score between 4.0 and 4.2; **office_furniture** is the lowest at about 3.6.

## Recommendations
1. Improve logistics to the North (closer fulfillment or local carriers), since late deliveries sharply reduce review scores.
2. Add shipping capacity ahead of the November peak.
3. Launch loyalty and re-engagement programs; repeat purchase is only about 3%.
4. Diversify the seller base beyond SP.
5. Review quality and listings in the lowest-rated categories (for example office_furniture).

## Limitations
- The dataset is a sample, not the full business.
- No cost or profit data, so the analysis covers revenue, not profitability.
- The State slicer filters by **customer state**, so "Top 10 States by Sellers" shows sellers who sold to customers in the selected state.
- Review scores are voluntary and represent only customers who chose to review.
- The link between late delivery and low reviews is a strong correlation, not proof of causation.
- Sep and Oct 2018 (and the earliest 2016 months) are excluded from monthly trend charts because they contain very few orders.

## Tools
- Power BI Desktop
- Power Query (data cleaning and shaping)
- DAX (measures)

## How to Use
1. Download or clone this repository.
2. Open `olist_dashboard.pbix` in Power BI Desktop.
3. If prompted, update the data source paths to where you saved the Kaggle CSV files, then click Refresh.
4. Use the Year and State slicers (synced across pages) and the page navigator to explore.

## Repository Structure
```
olist-ecommerce-powerbi-dashboard/
├── olist_dashboard.pbix
├── README.md
├── screenshots/
│   ├── overview.png
│   ├── delivery.png
│   ├── customers_payments.png
│   └── sellers_products.png
└── docs/
    └── Olist_Dashboard_QA.pdf
```

## Screenshots
![Overview](screenshots/overview.png)
![Delivery](screenshots/delivery.png)
![Customers and Payments](screenshots/customers_payments.png)
![Sellers and Products](screenshots/sellers_products.png)

## Acknowledgements
Dataset provided by Olist and published on Kaggle.
