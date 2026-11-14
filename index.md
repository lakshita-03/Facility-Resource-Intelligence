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

<iframe src="notebooks/analysis.html" width="100%" height="800px"></iframe> 

**Part 1: Explore the data**
- Energy intensity: the annualised EUI ranges from 126 to 192 kWh/m²/yr. The highest are E Chennai, B Mumbai and I Delhi (long operating hours), and the lowest is C Bengaluru. These figures are raw and not yet adjusted for weather.
- Weather: weekday energy correlates about 0.70 with cooling degree days. Eight of ten sites fit well (R² 0.86 to 0.96), but B Mumbai (0.47) and C Bengaluru (0.52) fit poorly, which hints that something other than weather is affecting them.
- Weekend use: general-shift offices use about 50% of their weekday energy on off-days, and 24x7 sites about 92%. J Mumbai uses 80% although only 7% of people are present.
- Night-time load: C Bengaluru jumps from about 259 to 391 kWh per night (+51%) in July, with no change in headcount.
- Generator: H Pune ran 121 hours against only 30 outage hours. Other sites are within 8 to 12 hours of their outage time.
- Solar: all three solar sites drop in the monsoon, and A Delhi is about 30% below the others in August.
- Water: F Kolkata rises from 50 to 54 to 78 to 82 litres per person per day from August.
- Waste: E Chennai recycles about 22% against about 44% elsewhere, has a wet-waste spike in August, and C Bengaluru has a one-off 1,250 kg of building waste.
- Emissions: 98.5% come from grid electricity and 1.5% from diesel.

**Part 2: Peer grouping and expected vs actual**
- Peer groups: sites are grouped by operating hours, giving General shift (6 sites) and Extended hours (4 sites), so each site is compared only with similar ones.
- Expected-consumption model: a regression per site on cooling degree days, headcount and day type. It is trained on April to June and tested on unseen July to September data.
- Accuracy: typical error is about 2% for eight sites. The larger errors at C Bengaluru (9.1%) and B Mumbai (5.5%) reflect real changes.
- Excess vs own baseline (July to September): C Bengaluru +9.2% (about ₹1.18 lakh), B Mumbai +4.4% (about ₹0.83 lakh), all other sites within about ±1%.
- Peer comparison: J Mumbai’s off-day intensity is 158% of its peers, and F Kolkata is about 10% above its peers.

**Part 3: Alerts and explanations**
- Nine checks: rolling z-score, CUSUM drift, night load, water, generator vs outages, solar vs peers, data quality, weekend vs peers, and waste.
- Calibration: the CUSUM alarm level was set from each site’s normal noise, not from the answer key.
- Detection: 19 alerts were raised, and all 10 planted problems were found. Alarms came the same day up to 4 days after onset. Waste took about 30 days because it is reported monthly. B Mumbai’s drift was first flagged on 2 July, since the baseline had already absorbed part of it.
- Explanations: each alert splits the change into weather, occupancy, calendar and unexplained effects. For C Bengaluru, weather should have lowered use by 138 kWh/day, but 167 kWh/day is unexplained, and 82% of the excess falls at night. That points to a schedule problem.
- B Mumbai: its excess is spread across the day, which points to plant efficiency.
- False alarms: a few small ones, costing between about ₹90 and ₹1,500.

**Part 4: Priority list**
- Merging: the 19 alerts were merged into 13 issues, 6 of them material. Adding the alerts naively gives ₹9.9 lakh, and the merged total is ₹6.9 lakh.
- Labels: each issue is marked ongoing or stopped, and given a confidence level.
- Score: money 50%, carbon 20%, ongoing 20%, confidence 10%, tested under four different weightings.
- Ranking: 1 J Mumbai (weekend setback), 2 C Bengaluru (energy), 3 H Pune (diesel), 4 B Mumbai (drift), 5 F Kolkata (water leak), 6 A Delhi (solar fault).
- Stability: J Mumbai ranks first under every weighting, and the middle ranks shift.
- Total impact: about ₹6.9 lakh and 43.8 tCO₂e over six months, roughly 2% of utility cost. The three ongoing issues would cost about ₹11.3 lakh a year if left unfixed.



