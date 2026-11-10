# RFM Customer Segmentation Analysis (SQL)

## Summary

This project uses MySQL to segment a retailer's customers from a transactions table (`data`) and measure how much each group contributes to the business. It contains five analysis queries covering:

1. Customer segment distribution (count and % share)
2. Revenue share (by country, top 5)
3. Average Order Value (AOV) by value segment
4. Average recency and frequency by behavioural (RFM-style) segment
5. Top-spending customers (candidates for churn-risk review)

Customers are grouped in two ways:

- **Value segments** (by total spend): High Value (> 5,000), Medium Value (1,000 to 5,000), Low Value (< 1,000).
- **Behavioural segments** (by recency and frequency): Champions, Loyal Customers, Regular Customers, At Risk, One-Time Customers.

**Data fields used:** `InvoiceNo`, `InvoiceDate`, `CustomerID`, `Quantity`, `UnitPrice`, `Country`.
**Cleaning rules:** rows without a `CustomerID` are excluded; only positive `Quantity` and `UnitPrice` are kept; cancelled invoices (`InvoiceNo` starting with `C`) are removed from the recency/frequency query.

## Business Problem

The retailer does not know which customers drive revenue, which are slipping away, and which are worth marketing effort. Without this view, marketing spend is spread evenly across everyone, loyal buyers go unrewarded, and lapsing high spenders are noticed too late.

The analysis answers:

- How large is each customer group?
- Where does revenue come from?
- How much does a typical order bring in per segment?
- How recently and how often does each group buy?
- Which high-value customers should be prioritised for retention?

## Key Findings

**1. Segment distribution (1,481 customers)**

| Segment | Customers | Share |
|---|---|---|
| Low Value | 1,211 | 81.77% |
| Medium Value | 235 | 15.87% |
| High Value | 35 | 2.36% |

Most customers are low spenders, and only a very small group is high value.

**2. Revenue share (top 5 countries)**

| Country | Revenue | Share |
|---|---|---|
| United Kingdom | 1,121,137.17 | 82.12% |
| Germany | 37,933.06 | 2.78% |
| Netherlands | 36,916.37 | 2.70% |
| EIRE | 34,558.07 | 2.53% |
| France | 30,723.84 | 2.25% |

Revenue is heavily concentrated in the UK.

**3. Average Order Value**

| Segment | AOV |
|---|---|
| High Value | 1,675.79 |
| Medium Value | 512.54 |
| Low Value | 258.30 |

High-value customers place orders roughly 6.5x larger than low-value customers.

**4. Recency and frequency**

| Segment | Avg recency (days) | Avg orders |
|---|---|---|
| Champions | 6.07 | 17.43 |
| Loyal Customers | 10.48 | 6.04 |
| Regular Customers | 26.45 | 2.59 |
| One-Time Customers | 42.16 | 1.00 |

Frequent buyers are also the most recent buyers. One-time customers are the least recently active group.

**5. Top revenue drivers**

Customers with total spend above 5,000 are listed from highest to lowest. The top customer, **12346**, has a total spend of **77,183.60**.

## Recommendations

- **Protect the High Value group.** Just 35 customers (2.36%) account for a disproportionate share of spend. Offer dedicated account contact, early access, and loyalty rewards.
- **Reward Champions and Loyal Customers.** They buy most often and most recently; use perks and referral incentives to keep them active.
- **Convert One-Time Customers.** They are the clearest repeat-purchase opportunity; use post-purchase follow-ups, second-order discounts, and product recommendations.
- **Upgrade Medium Value customers.** With 235 customers and an AOV of about 513, bundles and cross-sell offers could move them toward the High Value tier.
- **Reduce UK dependence.** Over 82% of revenue comes from one market. Test targeted campaigns in Germany, the Netherlands, EIRE and France.
- **Run win-back campaigns** for any high-spend customer whose last purchase is becoming old (see Next Steps).

## Next Steps

1. **Fix Query 2.** It groups by `Country` but is labelled "segment". Join to the value segments so revenue share is reported per customer segment, as the question asks.
2. **Complete Query 5.** It currently lists only high spenders. Add recency (days since last purchase) so it truly flags high-value customers who have stopped buying.
3. **Align filters across queries.** Query 1 requires `UnitPrice > 0`, but Query 3 does not; Queries 3 and 5 do not exclude cancellations (`InvoiceNo LIKE 'C%'`) as Query 4 does. Using one cleaned base table or view would make results consistent.
4. **Standardise segment labels** (e.g. `'high value'` in Query 1 vs `'High Value'` elsewhere) and make the `BETWEEN 1000 AND 5000` boundary explicit (`>= 1000 AND <= 5000`).
5. **Review the At Risk rule.** The `recency_days > 90` condition returned no customers in the results (maximum average recency was about 42 days), so check the date range or adjust the threshold.
6. **Move to scored RFM.** Use `NTILE` to score Recency, Frequency and Monetary values from 1 to 5 and combine them into segments, instead of fixed thresholds.
7. **Visualise** the results in a dashboard (Power BI or Tableau) and track segments over time.

## How to Run

1. Load the transactions into a MySQL table named `data`.
2. Run the queries in `rfm_segmentation.sql` in order (1 to 5).
3. `InvoiceDate` is parsed with `STR_TO_DATE(InvoiceDate, '%m/%d/%Y %H:%i')`, so it is expected in `MM/DD/YYYY HH:MM` text format.

