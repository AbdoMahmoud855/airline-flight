# ✈️ Airline Performance Dashboard + Late-Departure ML Model (Power BI & Python)

An end-to-end airline analytics project: a Python cleaning pipeline that builds a star schema, a 5-page interactive Power BI dashboard, and a machine-learning model that predicts whether a flight will depart late. The dashboard covers **14.26M flights, 15 carriers and 334 airports between 2014 and 2017**, with a **79.45% arrival on-time rate** and a **19.47% late-departure rate**. Instead of staring at tables, it answers real questions: When do delays happen? Why do they happen? Which airports, routes and carriers are the weakest? And can we predict a late flight before it departs?

> 📁 **Full project (Power BI file, data, notebook and images):** [Open on Google Drive](https://drive.google.com/drive/folders/1eSSRvK-F9P9VJyIcIju40BEG64t2JRHt?usp=sharing)

> ⏱️ **Data range:** 2014 → 2017 (one raw file per year). Some monthly views only show January–August, so seasonal comparisons should be read with that in mind. See [Reading the Data Correctly](#-reading-the-data-correctly).

<img width="1417" height="670" alt="image" src="https://github.com/user-attachments/assets/b5701cdd-7efc-4abb-b877-bf56ef079512" />


---

## 🗂️ Project Contents

| File | Description |
|---|---|
| `[FileName].pbix` | The complete Power BI report (available in the [Google Drive folder](https://drive.google.com/drive/folders/1eSSRvK-F9P9VJyIcIju40BEG64t2JRHt?usp=sharing)) |
| ↳ **Overview** | Page 1: KPIs, on-time performance (OTP), delay status, ML accuracy |
| ↳ **Time Patterns (When)** | Page 2: late rate by month, time of day and weekday |
| ↳ **Time Patterns (Why)** | Page 3: delay causes, recovery in the air, severity |
| ↳ **Airports & Routes** | Page 4: airports, routes, distance and taxi-out |
| ↳ **Carrier** | Page 5: carrier ranking and primary delay cause per carrier |
| `cleainingdata.py` | Cleaning + star-schema pipeline (pandas, airportsdata) |
| `AiModel.ipynb` | Feature engineering and ML models (classification + regression) |
| `images/` | Dashboard screenshots and model charts |
| `data/` | Star-schema CSV files |

---

## 🏗️ Data Pipeline

Raw yearly CSV files → Python cleaning → star schema → Power BI.

**Cleaning steps (`cleainingdata.py`)**

1. Load one CSV per year (2014–2017) and append them into a single table, tagging each row with `SOURCE_YEAR`.
2. Remove empty `Unnamed` columns and convert `FL_DATE` to a real date.
3. Standardize carrier codes that were absorbed by mergers (US → AA, NW → DL, CO → UA, FL → WN).
4. Enrich airports with name, city and country using the `airportsdata` library (unknown codes are labeled `Unknown`).
5. Split the delay-cause columns out of the flight table into their own fact table (one row per flight per cause).
6. Replace text values in the fact table with surrogate keys, and check for unmapped rows.
7. Export every table in 500K-row chunks so large files save safely.

**Star schema**

| Table | Type | Content |
|---|---|---|
| `FactFlight` | Fact | One row per flight: times, delays, distance, status and dimension keys |
| `FactDelay` | Fact | Delay minutes per flight per cause (only causes above 0) |
| `DimDate` | Dimension | Date key, year, quarter, month, month name |
| `DimCarrier` | Dimension | Carrier code and full name |
| `DimAirport` | Dimension | Airport code, name, city, country |
| `DimCancellation` | Dimension | Cancellation code and reason (Carrier, Weather, NAS, Security) |
| `DimDelayReason` | Dimension | Carrier, Weather, NAS, Security, Late Aircraft |

---

## ❓ Questions This Dashboard Answers

### Page 1: Overview

**KPI cards: "Where do we stand right now?"**

- **14.263M** total flights.
- **79.45%** arrival on-time performance (OTP).
- **39K** diverted flights (0.27%).
- **10.90** average departure delay (minutes).
- **1.88%** cancellation rate.

| Chart | Question | Short Answer |
|---|---|---|
| Arrival & Departure OTP by Year (line) | Is punctuality improving? | Yes until 2016 (≈ 77% → ≈ 82%), then it falls back to ≈ 79.5% in the last year. |
| Arrival & Departure OTP (columns) | How are arrivals spread by status? | Early 7.8M, Major Delay 2.7M, Minor Delay 2.7M, On Time 1.0M. |
| Median, Avg and P90 Departure Delay | Is the average hiding the tail? | Yes. The typical flight is close to on time, but the P90 line is far longer than the average. |
| Total Flight by DEP_Delay_Status | How many flights completed? | 13.96M completed, 0.27M cancelled, 0.04M diverted. |
| Accuracy of ML (gauge) | Does the model reach its target? | 92.18% accuracy against an 85% target. |

### Page 2: Time Patterns (When do delays happen?)

| View | Question | Short Answer |
|---|---|---|
| KPI: Late Departure Rate | How often do flights leave late? | 19.47% (a delay of 15 minutes or more). |
| KPI: Evening vs Morning | How much worse is the evening? | 2.49× (28.59% vs 11.47%). |
| KPI: Weekend Gap | Are weekends worse? | No. Weekends are about 2 points better (−1.99%). |
| KPI: Seasonal Gap | How big is the seasonal swing? | 5.60 points (June 22.25% vs April 16.65%). |
| Late Rate by Month | Which months are worst? | June (22.25%), August (21.70%), July (21.40%). April and March are the best. |
| Late Rate by Time of Day | Which part of the day is worst? | Evening 28.59%, Night 25.96%, Afternoon 22.12%, Morning 11.47%. |
| Weekday × Time of Day (matrix) | Where is the single worst slot? | Thursday evening (32.14%), then Friday evening (30.91%). |

### Page 3: Time Patterns (Why do delays happen?)

| View | Question | Short Answer |
|---|---|---|
| KPI: Top Delay Cause | What causes the most delay? | **Late Aircraft**, with a 40.65% share of delay minutes. |
| KPI: Avg Recovery When Late | Can late flights catch up? | On average 3.95 minutes are recovered in the air. |
| Delay Minutes by Cause | How do causes rank? | Late Aircraft (~71M min), Carrier (~54M), NAS (~41M), Weather (~8M), Security (≈ 0). |
| Departure Delay to Arrival Delay (waterfall) | Does the air phase help? | Yes. About 10.9 min of departure delay becomes about 6.3 min at arrival. |
| Arrival Delay Severity by Primary Cause | Which causes create the worst delays? | Severity differs by cause, so each needs its own action. |

### Page 4: Airports & Routes

| View | Question | Short Answer |
|---|---|---|
| KPI cards | How big is the network? | 334 airports served, 16.19 min average taxi-out. |
| KPI: Airport Gap | How far apart are the best and worst airports? | 23.28 points of late rate. |
| KPI: Worst Origin Airport | Where is the problem biggest? | Rafael Hernandez Airport. |
| Late Rate by Flight Distance | Does distance matter? | Short flights are late more often, and the late rate falls as distance grows. |
| Top 10 Routes by Late Departure Rate | Which routes are the weakest? | SLC → HOU, BTV → PHL, PDX → LMT lead the list. |
| Flight Volume vs Late Rate (scatter) | Do busy airports run later? | A slight upward trend: busier airports are late a little more often. |
| Late Rate by Origin Airport (treemap) | Which origins stand out? | Scottsbluff / Western Nebraska is the largest block. |
| Total Flight & Primary Delay per Airport | What drives delay at each airport? | The mix of causes changes by airport (cancellation, carrier, late aircraft, NAS). |

### Page 5: Carrier

| View | Question | Short Answer |
|---|---|---|
| KPI cards | How many carriers and what is the spread? | 15 carriers; best is **Hawaiian Airlines**, worst is **Spirit Airlines**; gap 17.82 points. |
| Late Arrival Rate by Carrier | Who is late most often? | Spirit (27.63%), JetBlue (26.28%), Envoy Air (26.01%), Frontier (25.57%). |
| Primary Delay Cause by Carrier | Do carriers fail for different reasons? | Yes. The share of Carrier, Late Aircraft and NAS delay differs from carrier to carrier. |
| Carrier table | Who is best at recovering time? | United (8.17 min) and Delta (7.10 min) recover the most; Hawaiian loses 1.02 min in the air. |

### Questions That Emerge From Combining Charts

- **Is delay a time problem or a carrier problem?** Both. Evenings run 2.49× worse than mornings, and the best and worst carriers are 17.82 points apart.
- **Do delays start on the ground?** Mostly yes. Late Aircraft is the top cause (40.65%), which means one late flight creates the next one.
- **Is the network fragile?** Late Aircraft + Carrier + NAS delays make up almost all delay minutes. Weather and Security are small.
- **Is bigger worse?** Not clearly. Southwest has the most flights (3.10M) and sits near the middle at 77.39% OTP.
- **Do long flights suffer more?** No. Late rate falls with distance, so short hops are the fragile part of the network.

> 💡 The slicers let you ask any of these for a single slice, for example: *"What does the evening delay look like for one carrier in one year, excluding weather cancellations?"*

---

## 📅 Reading the Data Correctly

| Point | What to know |
|---|---|
| **Total flights vs carrier total** | Overview shows 14.263M flights (including cancelled). The carrier table shows 13,995,442 because cancelled flights have no arrival. The difference (~0.27M) is the cancelled flights. |
| **Late definition** | A flight is *late* when departure delay is **15 minutes or more**. |
| **Months** | The month chart shows only January to August. Seasonal conclusions apply to those months only. |
| **Small airports** | Airports with very few flights can show extreme late rates. Check flight volume before judging an airport (see the scatter on page 4). |

---

## 🎨 Dashboard Design

A dark green and gold theme keeps the numbers in focus and gives all five pages one identity. *(Hex values are approximate. Replace them with the exact codes from your theme.)*

| Role | Color | Where it's used |
|---|---|---|
| Header banner, side menu, KPI cards, chart cards | Dark green `#263E3A` | Title bar, navigation, cards |
| Primary data color | Gold `#D9B100` | Bars, lines, borders, buttons |
| Secondary / support color | Green `#3B7D5C` | Delay-cause segments, P90 line |
| Highlight vs normal | Gold vs light grey | Worst months highlighted in gold, best in dark green |
| Negative change | Red | Declining KPI arrows |
| Page background | Light grey `#EEF0EF` | Canvas behind the cards |

### Navigation
- The left menu switches between **Overview, Time Patterns, Airports & Routes and Carrier**.
- The two icon buttons in the header of the Time Patterns pages switch between **When** (page 2) and **Why** (page 3).

### 🎛️ Filters (Slicers)

| Slicer | Purpose |
|---|---|
| Year | Compare 2014, 2015, 2016 and 2017 |
| CarrierName | Focus on one airline |
| CancellationReason | Isolate Carrier, Weather, NAS or Security cancellations |

---

## 📄 Page 1: Overview

<img width="1417" height="670" alt="image" src="https://github.com/user-attachments/assets/a0ea2c7d-69f6-46b5-87cb-660ee1c38e59" />


### 🔢 KPI Cards
The story: in five seconds the cards say "where we stand".

| Card | Value | The Story |
|---|---|---|
| ✈️ Total Flights | 14.263M | The scale of the dataset, with a 33.1% growth indicator *(document how it is calculated)*. |
| ⏱️ Arrival OTP % | 79.45% | About 4 of every 5 flights arrive on time. The red arrow shows a 3.08% decline. |
| 🔀 Total Diverted Flights | 39K (0.27%) | Diversions are rare. |
| 🕒 Avg Departure Delay | 10.90 min | The average is pulled up by a small number of long delays. |
| ❌ Cancellation Rate | 1.88% | About 1 flight in 53 is cancelled. |

### 📈 Chart: Arrival & Departure OTP by Year
**Type:** Line chart · **Question:** Is punctuality improving?

The story: OTP rose from about 77% in 2014 to about 82% in 2016, then fell back to about 79.5%. Arrival and departure OTP move almost together, so late departures are the main source of late arrivals.

**Takeaway:** The improvement was not permanent. The latest year gave back most of the gains.

### 📊 Chart: Median, Avg and P90 Departure Delay
**Type:** Bar chart · **Question:** Is the average hiding the problem?

The story: the median delay is near zero, the average is 10.90 minutes, and the P90 delay is far higher. Most passengers have a normal trip while a small group suffers very long delays.

**Takeaway:** Judge punctuality with the P90 as well as the average, because the average alone looks acceptable.

### 🤖 Visual: Accuracy of ML
**Type:** Gauge · **Question:** Is the prediction model good enough?

The story: the model reaches **92.18%** accuracy against an **85%** target. See [the ML section](#-machine-learning-model-predicting-late-departures).

---

## 📄 Page 2: Time Patterns (When)

![Time Patterns](images/time-patterns-when.png)

### 🔢 KPI Cards

| Card | Value | The Story |
|---|---|---|
| ⏰ Late Departure Rate | 19.47% | About 1 in 5 flights leaves 15 minutes late or more. |
| 🌙 Evening vs Morning | 2.49 | An evening flight is about 2.5 times more likely to be late. |
| 📅 Weekend Gap | −1.99% | Weekends are slightly better than weekdays. |
| 🌦️ Seasonal Gap | 5.60% | Summer months run about 5.6 points worse than spring. |

### 🕒 Chart: Late Departure Rate by Time of Day
The story: delay builds up through the day, from **11.47% in the morning** to **28.59% in the evening**. Early flights start on a clean schedule, and every later delay is passed along to the next flight.

**Takeaway:** This is the strongest pattern in the whole dashboard. Evening is where punctuality is lost.

### 📅 Chart: Late Departure Rate by Month
The story: June (22.25%) and August (21.70%) are the worst months and April (16.65%) is the best, which points to summer congestion and storms.

**Takeaway:** Summer needs extra buffer time, and spring is the safest period in this data.

### 🧮 Matrix: Weekday × Time of Day

| Slot | Late Rate |
|---|---|
| Thursday evening | **32.14%** (worst) |
| Friday evening | 30.91% |
| Monday evening | 29.18% |
| Sunday morning | **10.28%** (best) |

**Takeaway:** The worst slot is almost **3×** worse than the best. Thursday and Friday evenings are the pressure points of the week, and Saturday is the calmest day (17.38%).

---

## 📄 Page 3: Time Patterns (Why)

![Time Patterns Why](images/time-patterns-why.png)

### 🔢 KPI Cards

| Card | Value | The Story |
|---|---|---|
| 📊 Top Cause Share % | 40.65% | Four out of ten delay minutes come from one cause. |
| 🔍 Top Delay Cause | Late Aircraft | A delay on a previous flight is the biggest single cause. |
| 🔄 Avg Recovery When Late | 3.95 min | Late flights catch up a little in the air. |

### 📊 Chart: Delay Minutes by Cause

| Cause | Delay Minutes (approx.) |
|---|---|
| Late Aircraft | ~71M |
| Carrier | ~54M |
| National Aviation System | ~41M |
| Weather | ~8M |
| Security | ≈ 0 |

The story: the top three causes make up almost everything. Weather gets the blame in the news, but it is a small slice.

**Takeaway:** Most delay minutes come from the airline's own operation (Late Aircraft + Carrier), so most of them are controllable.

### 🔄 Chart: Departure Delay to Arrival Delay (Waterfall)
The story: an average departure delay of about **10.9 minutes** becomes about **6.3 minutes** at arrival, because crews make up about **4.5 minutes** in the air.

**Takeaway:** Schedules already include some buffer. Recovery helps, but it does not erase the delay.

---

## 📄 Page 4: Airports & Routes

![Airports and Routes](images/airports-routes.png)

### 🔢 KPI Cards

| Card | Value | The Story |
|---|---|---|
| 🛫 Airports Served | 334 | A wide network. |
| 🚨 Worst Origin Airport | Rafael Hernandez Airport | The weakest origin in the filtered view. |
| 📏 Airport Gap | 23.28% | Large spread between the best and worst airports. |
| 🚖 Avg Taxi Out | 16.19 min | Time spent on the ground before takeoff. |

### 📉 Chart: Late Rate by Flight Distance
The story: short flights (under 500 miles) have the highest late rate, and long flights (1,000+ miles) the lowest. Short routes fly many times a day, so one delay quickly spreads to the next flights.

**Takeaway:** Short-haul flying is the fragile part of the network.

### 🛣️ Chart: Late Departure Rate by Route (Top 10)
The story: the weakest routes include SLC → HOU, BTV → PHL, PDX → LMT, OMA → PDX and VEL → SLC. Several involve Salt Lake City (SLC) and Portland (PDX).

**Takeaway:** Check each route's number of flights before acting, since low-volume routes can look extreme. *(The route chart shows rates of 1.4%–2.8%, far below the 19.47% overall rate. Document how this measure is defined.)*

### 🔵 Chart: Flight Volume vs Late Rate by Airport
The story: most airports sit between 10% and 30% late. A slight upward trend line suggests busy airports are late a bit more often.

**Takeaway:** Volume explains only a small part of lateness. Airport conditions matter too.

---

## 📄 Page 5: Carrier

![Carrier](images/carrier.png)

### 🔢 KPI Cards

| Card | Value | The Story |
|---|---|---|
| ✈️ Carriers Count | 15 | The airlines compared. |
| 🥇 Best Carrier | Hawaiian Airlines | 90.19% OTP, 9.81% late arrivals. |
| 🥉 Worst Carrier | Spirit Airlines | 72.37% OTP, 27.63% late arrivals. |
| 📏 Carrier Gap | 17.82% | Passengers' chance of an on-time arrival depends heavily on the airline. |

### 📋 Carrier Table (selected rows)

| Carrier | Total Flights | Arrival OTP % | Late Arrival % | Avg Arrival Delay | Avg Delay Recovery |
|---|---|---|---|---|---|
| Hawaiian Airlines | 190,016 | 90.19% | 9.81% | 1.06 | −1.02 |
| Alaska Airlines | 428,472 | 86.36% | 13.64% | −0.55 | 2.09 |
| Delta Air Lines | 2,153,902 | 84.57% | 15.43% | 2.15 | 7.10 |
| Southwest Airlines | 3,100,591 | 77.39% | 22.61% | 7.43 | 4.95 |
| American Airlines | 1,824,938 | 79.03% | 20.97% | 6.42 | 3.86 |
| United Airlines | 1,285,262 | 79.31% | 20.69% | 5.33 | 8.17 |
| JetBlue Airways | 665,564 | 73.72% | 26.28% | 10.85 | 4.46 |
| Spirit Airlines | 247,106 | 72.37% | 27.63% | 12.72 | 2.58 |
| **Total** | **13,995,442** | **79.45%** | **20.55%** | **6.34** | **4.48** |

The story: **Delta** is the standout among the large carriers, with 84.57% OTP on 2.15M flights and the second-best recovery time. **Southwest** is the largest carrier but sits below the network average. **Spirit, JetBlue, Envoy and Frontier** are all above 25% late arrivals.

**Takeaway:** Carrier choice matters more than most operational factors, and a small group of low-cost and regional carriers drags the average down.

---

## 🤖 Machine Learning Model: Predicting Late Departures

**Goal:** predict whether a flight will depart late (`Late = 1` when `DEP_DELAY ≥ 15` minutes), and estimate how many minutes the delay will be.

### Data Preparation
- Source: `CleanedFlights.csv` (output of the cleaning pipeline).
- Removed cancelled flights, diverted flights and rows with no departure delay → **275,965 rows** kept.
- A random sample of **200,000 rows** was used, split **80% train / 20% test** (stratified on the target, 40,000 test rows).
- Features were scaled with `StandardScaler` for Logistic Regression and SVM.

### Features

| Feature | Description |
|---|---|
| `DepHour` | Scheduled departure hour |
| `Month`, `DayOfWeek` | Date features |
| `DISTANCE`, `CRS_ELAPSED_TIME` | Flight distance and scheduled duration |
| `OP_CARRIER_ENC` | Label-encoded carrier |
| `CARRIER_DELAY`, `WEATHER_DELAY`, `NAS_DELAY`, `SECURITY_DELAY`, `LATE_AIRCRAFT_DELAY` | Delay minutes by cause *(see the limitation below)* |

Season, weekend and US federal-holiday flags were also created during exploration.

### Classification Results

| Model | Accuracy |
|---|---|
| 🥇 Random Forest (100 trees, depth 12) | **94.36%** |
| Decision Tree (depth 10) | 94.22% |
| SVM (RBF, trained on a 20K subsample) | 93.81% |
| Logistic Regression | 93.24% |

**Random Forest detail:** precision 0.99 / recall 0.81 for the *Late* class, and precision 0.93 / recall 1.00 for *On-time*. The model rarely raises a false alarm, but it misses about 19% of late flights.

### Regression Result (minutes of delay)

| Model | RMSE | R² |
|---|---|---|
| Linear Regression | 58.05 min | 0.035 |

The story: predicting **whether** a flight is late works well, but predicting **how long** the delay will be does not. A linear model explains only 3.5% of the variation, because delay length has a long tail of extreme cases.

### Feature Importance (Random Forest)
Carrier (≈ 0.32) and departure hour (≈ 0.31) lead, followed by distance (≈ 0.14), day of week (≈ 0.12) and scheduled duration (≈ 0.11). This matches the dashboard: **time of day** and **carrier** are the two biggest drivers of lateness.

### ⚠️ Model Limitations
- **Data leakage:** the delay-cause columns are only filled in *after* a flight is already late, so including them lets the model "see the answer". The accuracy above is therefore optimistic, and a model trained this way cannot be used to predict a real, upcoming flight. For a genuine pre-departure predictor, train on `DepHour`, `Month`, `DayOfWeek`, `DISTANCE`, `CRS_ELAPSED_TIME`, `OP_CARRIER` and the airports only, then report that accuracy.
- **Class imbalance:** about 30% of the flights in the sample are late, so accuracy alone is not enough. Watch recall for the *Late* class.
- **Regression is weak** (R² = 0.035). Use tree-based or gradient-boosting models, or predict delay buckets instead of exact minutes.
- The dashboard gauge shows 92.18%, while the notebook's best run shows 94.36%. *(Add the exact model and run used for the gauge.)*

---

## 🧩 The Whole Story in 6 Sentences

1. The network carried **14.26M flights** over 2014–2017 with a **79.45% arrival OTP** and a **19.47% late-departure rate**.
2. Punctuality improved from about 77% to about 82% by 2016, then slipped back to about 79.5%.
3. **Time of day is the strongest pattern**: evening flights are late 2.49× more often than morning flights, and Thursday evening is the worst slot (32.14%).
4. **Late Aircraft is the top cause (40.65%)**, followed by Carrier and NAS, so most delay minutes start inside the airline's own operation and spread from flight to flight.
5. Carrier matters a lot: Hawaiian leads with 90.19% OTP and Spirit trails with 72.37%, a **17.82-point gap**.
6. A Random Forest predicts late departures with about 94% accuracy, but it needs retraining without the delay-cause columns before it can be used to predict real flights.

---

## 🧭 Solution Strategy & Tactics

**Purpose of this section:** when the dashboard reveals a problem, the data owner needs a solution they can accept. Each proposal follows one pattern: **Problem (with evidence) → Proposed solution → Safe tactic (small pilot) → How we prove it worked.**

### 🏢 Strategy Assumed From the Data
*These are inferred from the numbers, not confirmed. Check them with the data owner.*

- Improve on-time performance without cutting the schedule.
- Keep cancellations and diversions low.
- Protect the carrier's reputation with passengers.

### 🧩 Problem → Solution → Tactic

**1. Evening delays build up through the day 🌙**

| | |
|---|---|
| Problem & evidence | Late rate rises from 11.47% (morning) to 28.59% (evening). Thursday evening hits 32.14%. |
| Proposed solution | Add buffer where the delay builds up, instead of changing the whole schedule. |
| Tactic | Pilot longer turnaround times for Thursday and Friday evening flights on a few busy routes for 4 to 8 weeks. |
| Proof it worked | The evening late rate on pilot routes falls compared with similar routes. |

**2. Late Aircraft is the top cause ✈️**

| | |
|---|---|
| Problem & evidence | 40.65% of delay minutes come from Late Aircraft. |
| Proposed solution | Break the chain so one late flight does not delay the next ones. |
| Tactic | Use the model to flag aircraft with a high risk of delay, and add a spare aircraft or crew on those rotations. |
| Proof it worked | Late Aircraft minutes fall on the piloted rotations. |

**3. Weak carriers drag the average down 🏷️**

| | |
|---|---|
| Problem & evidence | Spirit, JetBlue, Envoy and Frontier have 25%+ late arrivals, against 9.81% for Hawaiian. |
| Proposed solution | Learn from the best performers instead of penalizing the weakest. |
| Tactic | Compare turnaround and recovery practices with Hawaiian, Alaska and Delta, then test the best ones with one weak carrier. |
| Proof it worked | The late arrival rate of the tested carrier moves closer to the network average. |

**4. Short routes are fragile 📏**

| | |
|---|---|
| Problem & evidence | The late rate is highest for flights under 500 miles. |
| Proposed solution | Give short routes extra time between flights. |
| Tactic | Test a few minutes of extra buffer on the five weakest routes (SLC → HOU, BTV → PHL, PDX → LMT, OMA → PDX, VEL → SLC). |
| Proof it worked | Route late rate falls without a rise in scheduled flight time complaints. |

**5. Metrics that are unclear or mislabeled 📋**

| | |
|---|---|
| Problem & evidence | Total Flights growth (33.1%), the route late-rate scale (1–3%) and the OTP chart titles are not documented. |
| Proposed solution | Document every measure and rename visuals to match what they count. |
| Tactic | A one-page definitions sheet and corrected visual titles. |
| Proof it worked | Anyone reading the dashboard can say what each number means. |

### 🗣️ How to Present a Solution to the Data Owner

Use this sentence pattern:

> *"We noticed [problem] ([number from the dashboard]). We suggest testing [solution] on [small group] for [time period], without changing [what the owner cares about]. If [metric] improves, we expand. If not, we go back to how things were."*

Four rules that reduce pushback:

1. **Lead with the number, not the opinion.** "Evening flights are late 28.59% of the time" is stronger than "evenings feel bad."
2. **Offer a pilot, not a final decision.** Limited time and limited scope.
3. **Say what you will not change.** Published schedules, core routes, safety margins.
4. **Ask for approval on one step, not the whole plan.**

### 🗓️ Implementation Roadmap

| Phase | Timeframe | Focus | Problems addressed |
|---|---|---|---|
| **Phase 1: Clarify** | 0 – 30 days | Define metrics, retrain the model without leakage | 5 |
| **Phase 2: Pilot** | 30 – 90 days | Small, reversible schedule tests | 1, 3, 4 |
| **Phase 3: Scale** | 90 – 180 days | Expand only what the pilots proved, use the model for rotation planning | 2 and the winning pilots |

### 📊 Success KPIs to Track on the Dashboard

| KPI | Today | Direction |
|---|---|---|
| Arrival OTP % | 79.45% | ⬆️ Increase |
| Late Departure Rate | 19.47% | ⬇️ Decrease |
| Evening vs Morning ratio | 2.49 | ⬇️ Decrease |
| Late Aircraft share of delay | 40.65% | ⬇️ Decrease |
| Carrier gap | 17.82% | ⬇️ Decrease |
| Cancellation rate | 1.88% | ⬇️ Decrease |
| ML accuracy (no leakage) | *to be measured* | ⬆️ Above 85% target |

> 📝 Targets should be set by the business after the Phase 1 baseline. The figures above are current values from the dashboard, and the directions are the recommended goals.
---

## 🧰 Tools Used

- **Python** (pandas, NumPy, airportsdata): data cleaning and star-schema creation
- **scikit-learn**: Logistic Regression, Decision Tree, Random Forest, SVM, Linear Regression
- **matplotlib / seaborn**: model charts and confusion matrices
- **Google Colab**: notebook environment
- **Power BI Desktop**
  - Power Query: loading the star-schema tables *[add your steps]*
  - DAX measures: Arrival OTP %, Late Departure Rate %, Cancellation Rate %, Evening vs Morning, Weekend Gap, Seasonal Gap, Avg Delay Recovery *[add your measures]*
  - Data modeling: `FactFlight` and `FactDelay` connected to Date, Carrier, Airport (origin and destination), Cancellation and Delay Reason *[add your model]*
  - Dashboard design: custom dark green and gold theme, icons, slicers, page navigation

---

## 🚀 How to Use

**Dashboard**
1. Open `[FileName].pbix` in Power BI Desktop.
2. Use the slicers at the top (Year, CarrierName, CancellationReason) to filter every page.
3. Use the left menu to move between Overview, Time Patterns, Airports & Routes and Carrier.
4. On the Time Patterns pages, use the header icons to switch between *When* and *Why*.
-----
⭐ If you like this project, leave a star on the repository!
