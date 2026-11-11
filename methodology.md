# Methodology: "Who Comes Back?"


## 1. Data

- UCI Online Retail II: transaction lines from a UK online retailer that mostly sells giftware, with many wholesale customers.
- 1,067,371 rows, 1 Dec 2009 to 9 Dec 2011.
- **Analysis window: 1 Dec 2009 to 30 Nov 2011.** December 2011 is a partial month (the file stops on 9 Dec), and partial months make retention look artificially low, so it is dropped.

## 2. Cleaning (in order, with counts)

| Step | Rows removed | Why |
|---|---|---|
| Unparseable date, quantity or price | 0 | Nothing was broken |
| Cancellations (invoice starts with "C") | 19,494 (value -£1.53M) | Set aside, not netted off sales |
| Zero or negative quantity or price | 6,207 | Stock adjustments and bad-debt entries |
| Non-product stock codes (postage, fees, manual entries) | 4,793 | Kept only codes that start with 5 digits |
| Missing Customer ID | 234,245 (£2.67M, 12.8% of positive sales) | Cannot be tied to a customer |
| Dated after 30 Nov 2011 | 17,242 | Partial final month |

Final: 785,390 lines, £16.92M revenue, 5,824 customers.
Exact duplicate rows (34,335) were counted but **not removed**, because the same item can legitimately appear twice on an invoice.

**Returns:** £1.53M of returns were set aside and not subtracted, so all revenue figures are **gross of returns**. That is roughly 7% of gross sales value, so true net revenue is somewhat lower.

## 3. Definitions

- **Revenue** = quantity x unit price.
- **Order** = all of one customer's lines on one calendar day (so two invoices on the same day count as one order).
- **First order / cohort month** = the customer's earliest order day and its month.
- **Excluded group:** 951 customers first seen in December 2009. The file starts that month, so many of them had already been buying for years and their real first order is unknown. They are left out of every cohort, repeat-rate and LTV figure, but kept in the RFM segments. The remaining **4,873 "new" customers** are used everywhere else.
- **Repeat customer** = placed a second order on a different day.
- **180-day repeat** = second order within 180 days of the first. Only customers with at least 180 days of history (3,970) are counted, so recent customers are not unfairly treated as non-returners. **Baseline: 56.4%.**
- **Monthly retention** = share of a cohort that placed at least one order in calendar month N after their first-purchase month (month 0 is always 100% by definition). Headline numbers are weighted across the cohorts old enough to be observed at month N (22 cohorts at month 1, 11 at month 12). Because it counts calendar months, a customer who orders every two months shows as "not retained" in alternate months, so it understates how many customers are still active.
- **365-day LTV** = revenue within 365 days of the first order day, for customers with at least 365 days of history (3,288). It is revenue, not profit.
- **First-order value bands** = quartiles of first-order value among new customers: Q1 up to £170, Q2 £170 to £299, Q3 £299 to £457, Q4 £457 and above.
- **Gateway product** = a product that appeared in a customer's first order. For each product, the repeat rate is the 180-day repeat rate among eligible customers who bought it first. Lift = that rate minus the 56.4% baseline, in percentage points. Only products with at least 50 customers are ranked (607 products).

## 4. RFM segments

Calculated as at 1 Dec 2011 on **all** 5,824 customers.

- **Recency score:** 5 = last order within 30 days, 4 = within 90, 3 = within 180, 2 = within 365, 1 = older.
- **Frequency score:** 1 = one order day, 2 = two, 3 = three or four, 4 = five to nine, 5 = ten or more.
- **Monetary score:** quintile of total revenue (shown for information; the segment rules use only recency and frequency).

| Segment | Rule |
|---|---|
| Champions | Recency 4 or 5 and frequency 4 or 5 |
| Loyal | Recency 3 or more and frequency 3 or more (not Champions) |
| At risk | Recency 1 or 2 and frequency 3 or more |
| New or recent | Recency 4 or 5 and frequency 1 or 2 |
| Lost or low | Recency 1 or 2 and frequency 1 or 2 |
| Needs attention | Everything else (recency 3, frequency 1 or 2) |

I did not report lifetime value by RFM segment, because segments are built from total spend, so the comparison would be circular. First-order value bands are used instead.

## 5. Checks that passed

Month-0 retention is 100% for every cohort. Cohort sizes add up to the 4,873 new customers. No customer appears twice. Revenue reconciles (positive sales = analysed revenue + each removed amount). All analysed dates fall inside the window.

## 6. Limitations (say these in your post or README)

1. The data is from 2009 to 2011.
2. The retailer has many wholesale customers, so repeat rates and order values are higher than a typical consumer shop.
3. 12.8% of positive sales have no customer ID and are excluded.
4. Revenue is gross of returns.
5. Customers first seen in December 2009 are excluded from cohort analysis because their true first order is unknown.
6. Retention counts calendar months and so understates activity for customers who order every few months.
7. Gateway products are associations only. Big first orders repeat more anyway, so products that often sit in big baskets look better. Many products were tested, so the top of the list partly reflects chance.
8. The business is seasonal (gifts), and seasonality is not controlled for.

## 7. How to reproduce

Run `day_1_analysis_v2.py` with `online_retail_II.csv` in the same folder. It rebuilds every table and prints `summary.txt` with all checks.
