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

| Table Name | Rows | Description |
|---|---:|---|
| `dim_site` | 10 | Site profile including city, type, floor area, cooling type, operating hours, seating capacity, generator and solar capacity, and electricity tariff. |
| `dim_date` | 183 | Calendar dimension covering April–September 2026, including weekend and holiday flags. |
| `fact_daily_utilities` | 1,830 | Daily utility records for 10 sites across 183 days (10 sites × 183 days). |
| `fact_weather_daily` | 1,464 | Daily weather records covering 8 cities. |
| `fact_waste_monthly` | 30 | Monthly waste records (in kg) for 5 sites across 6 months. |
| `fact_maintenance_tickets` | 66 | Maintenance work orders containing site, asset, dates, ticket type, cost, and status. |
| `dim_reference_factors` | 4 | Reference factors for emission calculations and cost conversions, including emission factors and prices. |

### Planted problems (used only to score detection)

| Site | Problem Identified |
|---|---|
| C Bengaluru | Night-time load increased by approximately 55% from 10 July due to a schedule override. |
| B Mumbai | Gradual increase in consumption over several months due to a fouled chiller; normalized after cleaning on 19 September. |
| H Pune | Generator operated on weekdays during June despite a healthy grid supply. |
| A Delhi | Solar output dropped to 40% between 15 August and 5 September due to an inverter fault. |
| F Kolkata | Water leakage began on 1 August. |
| J Mumbai | No weekend or holiday energy setback observed. |
| D Hyderabad | Five consecutive days of missing grid readings (12–16 June). |
| D Hyderabad | An anomalous grid reading, approximately 10 times higher than expected, recorded on 22 July. |
| E Chennai | Wet-waste spike in August and consistently low recycling levels. |
| C Bengaluru | One-time building waste spike of 1,250 kg recorded in July. |

## 3. Data-quality checks
Seven queries cover row counts, duplicate site-days, orphan keys, missing grid readings, extreme readings (more than 3× the site average), solar blanks at solar sites and generator hours with no matching outage. The last query gave an early hint of a planted problem: H Pune logged about 82 generator hours with no outage.

**SQL Views**
- **vw_daily_base**	Joins the daily facts with site, date and weather, giving one wide row per site-day
- **vw_daily_flagged**	Adds the data-quality flag (OK, MISSING, OUTLIER)
- **vw_daily_clean**	Replaces bad readings, and adds total consumption, cost, and Scope 1 and 2 emissions
- **vw_site_month**	Monthly KPIs per site: kWh per m², kWh per person, shares, water, cost, emissions
- **vw_site_kpi**	Whole-period KPIs per site, including annualised EUI
- **vw_waste_kpi**	Waste totals, recycling rate and waste per person
- **vw_waste_long**	Waste unpivoted from wide to long, the shape BI tools prefer

### Data Model
The project follows a Star Schema structure:
- **Fact Tables:** fact_daily_utilities, fact_waste_monthly, fact_maintenance_tickets
- **Dimension Tables:** dim_site, dim_date, dim_reference_factors
- **Relationships:** Tables are connected through relevant keys, primarily site_id and date_id.

## 4. Exploratory Data Analysis (EDA)
<iframe src="notebooks/analysis.ipynb" width="100%" height="800px"></iframe> 
The analysis covers:
- Energy consumption: Energy Use Intensity (EUI), measured in kWh/m²/year, and the contribution of grid electricity, solar and diesel generation.
- Weather sensitivity: Relationship between Cooling Degree Days (CDD) and electricity consumption.
- Operating patterns: Comparison of weekday and weekend consumption against corresponding occupancy levels.
- Night-time baseload: Weekly electricity consumption between 12 AM and 5 AM to identify unusually high overnight loads.
- Diesel generator usage: Comparison of generator operating hours with recorded grid outages.
- Solar performance: Solar generation per installed kWp across facilities and weather conditions.
- Water consumption: Weekly water usage per person.
- Waste management: Monthly recycling rates and wet/food waste generation per person.
Initial observations:
- C Bengaluru experienced a noticeable increase in night-time electricity consumption in July.
- H Pune showed generator operating hours substantially higher than recorded grid-outage hours.
- J Mumbai maintained relatively high energy consumption during weekends despite significantly lower occupancy.
- A Delhi showed comparatively weak solar generation against peer facilities during August.
- F Kolkata experienced an increase in water consumption per person.
- E Chennai showed comparatively low recycling rates and an increase in wet waste per person.
These observations formed the basis for further investigation rather than being treated as confirmed operational problems.

