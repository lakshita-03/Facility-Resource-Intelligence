# A Data-Driven Approach to Sustainability and Operational Efficiency
**Facility Resource Intelligence** is an end-to-end sustainability operations analytics project designed to identify resource inefficiencies across commercial facilities, quantify their financial and environmental impact and prioritise corrective actions.

The project analyses energy, water, diesel generator usage, solar generation and waste across 10 facilities over a six-month period (April–September 2026). It combines exploratory data analysis, SQL-based data preparation, statistical modelling, anomaly detection and interactive reporting to translate operational data into actionable insights. The solution goes beyond conventional consumption monitoring by establishing expected consumption baselines, identifying deviations from normal operating behaviour, estimating their impact and ranking issues based on financial loss, carbon emissions, urgency and confidence.

## 1. Project overview
The problem: Facility teams collect energy, water, diesel generator, solar and waste figures every month. The reporting mostly records and organises these numbers. It rarely answers the questions that matter:
a. Is a site using more than it should, once weather and the number of people present are taken into account?
b. If so, why, and what is it costing in rupees and carbon?
c. Of all the problems found, which should be investigated first?

### **Objectives**
- Prepare and check the data, and calculate standard KPIs (energy intensity, cost, emissions).
- Compare sites fairly, by grouping similar sites and removing the effect of weather and occupancy.
- Detect abnormal consumption, water use, generator use and solar output, and explain each finding.
- Convert findings into ₹ and tCO₂e and rank them into a priority list.

## 2. Data Sources
Why synthetic data
The project is based on the kind of data collected in facility reporting but real client data cannot be published. So the data was generated to follow the same structure, with realistic ranges and a few planted problems. This also lets the results be scored, because the correct answers are known.

**Table	Rows	Content**
### Database Schema

| Table Name | Rows | Description |
|---|---:|---|
| `dim_site` | 10 | Site profile including city, type, floor area, cooling type, operating hours, seating capacity, generator and solar capacity, and electricity tariff. |
| `dim_date` | 183 | Calendar dimension covering April–September 2026, including weekend and holiday flags. |
| `fact_daily_utilities` | 1,830 | Daily utility records for 10 sites across 183 days (10 sites × 183 days). |
| `fact_weather_daily` | 1,464 | Daily weather records covering 8 cities. |
| `fact_waste_monthly` | 30 | Monthly waste records (in kg) for 5 sites across 6 months. |
| `fact_maintenance_tickets` | 66 | Maintenance work orders containing site, asset, dates, ticket type, cost, and status. |
| `dim_reference_factors` | 4 | Reference factors for emission calculations and cost conversions, including emission factors and prices. |


