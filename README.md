# Hotel Booking & Revenue Analysis (OYO Case Study)

> **Where should an online hotel platform grow, and where is it losing revenue to cancellations?**
> A descriptive analysis of 2,889 bookings across 357 hotels in 10 Indian cities, built with SQL Server.

![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![Excel](https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white)

---

## Project at a Glance

| | |
|---|---|
| **Analysis type** | Descriptive analytics (exploratory data analysis and KPI performance analysis) |
| **Domain** | Hospitality and online travel |
| **Business goal** | Decide where to grow and how to reduce revenue lost to cancellations |
| **Data** | 2,889 bookings, 357 hotels, 10 cities (bookings made Jan–Mar 2022) |
| **Tools** | Microsoft SQL Server (SSMS), Excel / Power Query |
| **Techniques** | Joins, CTEs, `CASE` logic, `EXCEPT`, date functions, aggregation |
| **Deliverables** | SQL script, charts, findings, prioritized recommendations |

## Executive Summary

| # | What the data shows | So what |
|---|---|---|
| 1 | **33% of bookings are cancelled**, and cancelled bookings are worth more than average, so they wipe out **38% of gross booking value**. Delhi, Noida and Hyderabad are above 37%; Pune is lowest at 23.3%. | Cancellations are the biggest revenue leak. Pune is an internal benchmark to learn from. |
| 2 | **Gurgaon earns about 5.5x more net revenue per hotel than Kolkata** (31.5K vs 5.7K). Bangalore and Mumbai follow Gurgaon. | Rank expansion by revenue per hotel, not by room rate. Kolkata has a high rate but weak demand. |
| 3 | **70% of bookings are made on the check-in day or the day before** (48.5% on the day itself). | This is a last-minute market. Same-day offers and live inventory matter more than early-bird deals. |
| 4 | **Only 19% of February customers had booked in January** (133 of 699). Discounts are flat at 16.6–18.2% in every city. | Retention is weak, and discounting isn't tailored to city performance. |

## What I Did

1. **Framed the problem** as a stakeholder brief and turned it into eight business questions.
2. **Loaded and validated the data:** imported two Excel files into SQL Server and checked for duplicate booking IDs and unmatched hotel IDs (none found).
3. **Prepared the data:** derived price before discount, nights stayed and rate per room per night.
4. **Wrote SQL to build city-level KPIs:** cancellation rate, gross and net revenue, average room rate, discount %, monthly bookings, booking lead time, rooms and nights per booking, and new vs repeat customers.
5. **Extended the original analysis** with net revenue per hotel and a corrected new-vs-repeat customer count.
6. **Turned results into decisions:** charts, insights and a prioritized recommendation list with a KPI for each.

## Charts

![Cancellation rate by city]()

![Net revenue per hotel]()

![Booking lead time]()

## Business Questions

- How many hotels does each city have, and how do average room rates compare?
- What is the cancellation rate in each city, and how much revenue does it cost?
- How do gross and net revenue compare by city?
- How many bookings does each city receive per month?
- How far ahead of check-in do customers book?
- How many rooms and nights does a typical booking involve?
- How large are the discounts in each city?
- How many customers come back (new vs repeat)?

## Dataset

Two tables, loaded from Excel into SQL Server under the `OYO` schema.

| Table | Rows | Columns |
|---|---|---|
| `OYO.Hotel_Sales` | 2,889 | `booking_id`, `customer_id`, `status`, `check_in`, `check_out`, `no_of_rooms`, `hotel_id`, `amount`, `discount`, `date_of_booking` |
| `OYO.City` | 357 | `hotel_id`, `city` |

- **Cities (10):** Delhi, Bangalore, Gurgaon, Mumbai, Hyderabad, Jaipur, Noida, Pune, Chennai, Kolkata
- **Booking status:** Stayed (61.9%), Cancelled (33.0%), No Show (5.1%)
- **Quality checks:** no duplicate `booking_id`, every `hotel_id` in sales exists in the city table, no zero-night stays

## Data Preparation

```sql
-- Price before discount
UPDATE OYO.Hotel_Sales SET Price = amount + discount;

-- Length of stay
UPDATE OYO.Hotel_Sales SET no_of_nights = DATEDIFF(day, check_in, check_out);

-- Rate per room per night
UPDATE OYO.Hotel_Sales
SET rate = ROUND(
    CASE WHEN no_of_rooms = 1 THEN Price / no_of_nights
         ELSE Price / no_of_nights / no_of_rooms END, 2);
```

## Metric Definitions

| Metric | Definition |
|---|---|
| Cancellation rate | Cancelled bookings ÷ all bookings in the city |
| Gross revenue | Sum of `amount` across all bookings |
| Net revenue | Sum of `amount` for Stayed and No Show bookings (excludes Cancelled) |
| Net revenue per hotel | Net revenue ÷ hotels listed in the city |
| Average room rate | Average of `rate` (price per room per night, before discount) |
| Discount % | Average of `discount ÷ price` |
| Lead time | Days between `date_of_booking` and `check_in` |
| Repeat customer | February customer who also booked in January |

```sql
-- Cancellation rate and net revenue by city
SELECT c.city,
       FORMAT(100.0 * SUM(CASE WHEN h.status = 'Cancelled' THEN 1 ELSE 0 END)
              / COUNT(*), 'f1') AS cancellation_rate_pct,
       SUM(h.amount) AS gross_revenue,
       SUM(CASE WHEN h.status IN ('Stayed', 'No Show') THEN h.amount END) AS net_revenue
FROM OYO.Hotel_Sales h
JOIN OYO.City c ON c.hotel_id = h.hotel_id
GROUP BY c.city;
```

## Detailed Findings

| City | Hotels | Cancellation rate | Avg room rate | Net revenue per hotel |
|---|---|---|---|---|
| Delhi | 85 | 39.1% | 3,752.94 | 18.4K |
| Bangalore | 61 | 28.1% | 3,178.30 | 25.0K |
| Gurgaon | 51 | 32.1% | 2,632.99 | 31.5K |
| Mumbai | 36 | 32.4% | 5,128.27 | 24.5K |
| Hyderabad | 26 | 37.8% | 3,343.00 | 13.2K |
| Jaipur | 25 | 28.3% | 2,680.55 | 10.6K |
| Noida | 24 | 37.8% | 2,914.35 | 18.4K |
| Pune | 21 | 23.3% | 3,958.48 | 21.0K |
| Chennai | 17 | 29.6% | 3,003.58 | 14.6K |
| Kolkata | 11 | 31.8% | 3,473.89 | 5.7K |

- **Cancellations:** cancelled bookings average 4,760 vs 3,851 for stays, which is why 33% of bookings account for 38% of gross value.
- **Pune** combines the lowest cancellation rate with the fourth-highest net revenue per hotel.
- **Kolkata** has the fewest hotels, the fewest bookings and the lowest revenue per hotel, despite an above-average room rate.
- **Booking behavior:** 94.3% of bookings are for a single room and 79.8% are for one night.
- **Lead time:** about 85% of bookings are made within 4 days of check-in, and very few are made a month or more ahead.

## Recommendations

| Priority | Recommendation | Evidence | KPI to track |
|---|---|---|---|
| 1 | Investigate and reduce cancellations in Delhi, Noida and Hyderabad, using Pune as the benchmark | 39.1%, 37.8% and 37.8% vs 23.3% in Pune; 38% of gross value lost | Cancellation rate by city |
| 2 | Rank expansion cities by net revenue per hotel, and verify demand in Kolkata before adding hotels | Gurgaon 31.5K vs Kolkata 5.7K per hotel | Net revenue per hotel |
| 3 | Build same-day and next-day offers and live inventory visibility | 70% of bookings are made within one day | Share of same-day bookings |
| 4 | Test city-specific discount levels | Discounts are flat at 16.6–18.2% across very different cities | Discount % vs net revenue per hotel |
| 5 | Pilot a loyalty or re-engagement offer | Only 19% of February customers had booked in January | Repeat-customer rate |

These are hypotheses to test, not proven causes.

## Assumptions and Limitations

- The data covers one quarter, so seasonality cannot be assessed.
- Net revenue treats No Show bookings as earned revenue and excludes only Cancelled bookings.
- The dataset has no cancellation reasons, so it shows where cancellations happen but not why.
- This is descriptive analysis. It explains what happened and does not forecast future bookings.
- Amounts are shown in the dataset's own currency units.

## Skills Demonstrated

T-SQL (SQL Server) · Joins · CTEs · `CASE` expressions · Set operations (`EXCEPT`) · Date functions · Aggregation · Data validation and preparation · KPI definition · Data visualization · Business insight and recommendation writing

## How to Run

1. Install Microsoft SQL Server and SQL Server Management Studio (SSMS).
2. Create a database and an `OYO` schema:
   ```sql
   CREATE DATABASE OYO_Analysis;
   GO
   USE OYO_Analysis;
   GO
   CREATE SCHEMA OYO;
   ```
3. Import `data/Oyo_Sales.xlsx` into `OYO.Hotel_Sales` and `data/Oyo_City.xlsx` into `OYO.City` using the SSMS Import Wizard.
4. Run `sql/oyo_analysis.sql` from top to bottom. Run the three `ALTER`/`UPDATE` preparation steps once only.

## Project Structure

```
Hotel-Booking-Revenue-Analysis/
├── data/
│   ├── Oyo_Sales.xlsx          # Booking data (2,889 rows)
│   └── Oyo_City.xlsx           # Hotel-to-city mapping (357 rows)
├── sql/
│   └── oyo_analysis.sql        # All queries with comments
├── images/
│   ├── cancellation_rate_by_city.png
│   ├── net_revenue_per_hotel.png
│   └── booking_lead_time.png
└── README.md
```
