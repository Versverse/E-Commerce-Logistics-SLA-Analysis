# E-Commerce Logistics SLA & CSAT Diagnostic

##  Business Context
In the e-commerce logistics network, missing the delivery Service Level Agreement (SLA) directly impacts customer retention and generates high CSAT (Customer Satisfaction) friction. Using the Olist Brazilian E-Commerce dataset (approx. 100k orders), this project executes a technical deep-dive to identify fulfillment bottlenecks, isolate root causes of delays, and quantify the exact business cost of late deliveries.
![Dashboard Preview](https://public.tableau.com/app/profile/qing.lu8336/viz/BrazilE-CommerceDeliveryAnalyticsDashboard/Dashboard1)
##  Tech Stack & Methodology
* **SQL (SQLite in-memory):** Fast macro-level aggregations, logistics funnel extraction, and hub-to-hub route performance tracking.
* **Python (Pandas):** Strict ETL execution, data governance validation, time-series segmentation, and multi-dimensional Root Cause Analysis (RCA).
* **Tableau**: Interactive dashboard design for operations managers to monitor the 4-day CSAT tipping point and regional delay hotspots.

##  Data Pipeline & Execution Steps

### 1. Macro-Level SQL Diagnostics
Simulated a lightweight data warehouse environment to query raw logistics tables.
* Computed top-level SLA KPIs (On-time rate, average delivery days).
* Sliced delay rates by destination states and monthly peak periods.
* Built a logistics funnel to separate **Seller Fulfillment Time** (payment to carrier handover) from **Carrier Transit Time** (handover to customer).
* Mapped out the worst-performing origin-to-destination transit lanes (e.g., SP -> RJ).

### 2. Data Governance & Pandas ETL
Raw operational databases contain noise that skews SLA metrics. 
* Executed strict data cleansing (dropping null operational timestamps, filtering out canceled/incomplete cycles).
* Engineered new temporal features: `total_delivery_days`, `delay_days`, and `seller_handling_days`.
* **Cross-Validation Insight:** Discovered that querying raw SQL inflated the delay rate to 8.11%, whereas strict Pandas filtering revealed the true clean delay rate was 6.77%. This highlights the necessity of filtering out lost-in-transit anomalies before reporting KPIs.

### 3. Root Cause Analysis (RCA)
Cut the dataset across three distinct dimensions to locate the bottleneck:
* **Seller Operations:** Grouped orders via `pd.cut()` to prove that seller handling speed (fast vs. slow) had negligible impact on final carrier delay days.
* **Volume Peaks:** Grouped by `to_period('M')` to identify capacity failure during mega-sales (Nov 2017 & Mar 2018).
* **Geographical Constraints:** Merged customer states to isolate infrastructure limitations in remote Northern regions (AL, MA).

### 4. Quantifying CSAT Cost
Merged logistics performance data with the customer reviews table to translate operational metrics into experiential cost.
* Segmented delay severity into distinct thresholds (On-time, 1-3 days late, 4-7 days late, etc.).
* Computed the 1-star review percentage for each specific segment.

##  Key Business Findings

1. **The Bottleneck is Carrier Transit, Not Sellers:** Seller dispatch times remained stable at 2-3 days year-round. Delays are almost entirely driven by the carrier's inability to scale capacity during peak months (delay rates spiked to 18.9% in Mar 2018).
2. **High-Risk Artery Identified:** The SP (São Paulo) to RJ (Rio de Janeiro) lane processes the highest volume (9,400+ orders) but suffers a severe 14.85% delay rate, marking it as the primary target for carrier reallocation.
3. **The 4-Day CSAT Tipping Point:** The cost of a delay is non-linear. Customers tolerate minor 1-3 day delays (25.1% 1-star rate). However, once a delay crosses the 4-day mark, the 1-star rate explodes to 58.5% (peaking at 70%+ for 8+ days).
4. **Recommendation:** Supply chain operations must implement an automated 4-day intervention protocol. Orders approaching the 3-day transit threshold without delivery status should trigger an automated escalation to carrier account managers to intercept the parcel before the CSAT friction escalates to irreparable 1-star reviews.
