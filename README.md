
# Healthcare Intelligence, Hospital Performance & Operational Optimization Platform
End-to-end healthcare analytics: messy hospital data cleaned with Python, modelled in MySQL and presented in a 5-page Power BI dashboard on patient flow, bed capacity, outcomes, workforce efficiency and cost.
**OmniCare ER: patient and staff flow analytics.** An end-to-end data analytics project on a hospital emergency department: messy raw data → cleaned and validated with **Python** → modelled and analysed with **SQL (MySQL)** → checked in **Excel** → visualised in a **5-page Power BI dashboard**.

> **Data note:** all data is synthetic (computer-generated) and contains no real patient information. It was built to look like a real hospital extract, including realistic data-quality problems.

---

## 1. Business problem

OmniCare's emergency department (ER) sees about 66 patients a day. Management wants to know:

1. Where do patients wait the longest, and why?
2. Do we have enough beds, and are they used well?
3. Do we have enough doctors and nurses on each shift?
4. Which patients are at risk of coming back (readmission)?
5. Can we trust the data behind these answers?

## 2. Tools and skills

| Area | Tools |
|---|---|
| Data cleaning | Python (pandas, numpy, difflib), Jupyter |
| Database and analysis | MySQL 8 (CTEs, window functions, joins, indexes) |
| Quick checks | Excel (pivot tables) |
| Dashboard | Power BI (star schema, DAX) |
| Skills shown | Data profiling, text/date/number cleaning, deduplication, referential integrity, KPI design, business storytelling |

## 3. Dataset overview

10 tables, 1 January – 28 October 2026 (shifts and staffing requirements cover September–October).

| Table | Rows (raw, messy) | Rows (clean) | One row is… |
|---|---:|---:|---|
| departments | 14 | 12 | one hospital department |
| calendar | 738 | 730 | one calendar day (2025–2026) |
| patients | 12,250 | 12,008 | one patient |
| staff | 472 | 450 | one employee |
| beds | 312 | 300 | one hospital bed |
| encounters | 20,170 | 19,870 | one hospital visit |
| patient_flow_events | 97,174 | 95,144 | one step in a visit (registration, triage, doctor, bed, discharge) |
| bed_occupancy | 16,425 | 15,753 | one bed assignment |
| staff_shifts | 12,536 | 12,266 | one employee's shift |
| staffing_requirements | 5,758 | 5,506 | one department / date / shift / acuity level |

Clean row counts are lower than raw because duplicates were removed and unusable rows were moved to a quarantine file (see Methodology).

## 4. Data model

```
departments ─┬─< staff ─┬─< staff_shifts
             │          └─< encounters (attending doctor)
             ├─< beds ───< bed_occupancy >─┐
             ├─< encounters ───────────────┤
             │       │                     │
             │       └─< patient_flow_events
patients ────┘ (encounters.Patient_ID)
calendar: joined to any table on the date
staffing_requirements >── departments
```

- **Facts:** encounters, patient_flow_events, bed_occupancy, staff_shifts, staffing_requirements
- **Dimensions:** patients, departments, staff, beds, calendar

## 5. Data dictionary (clean tables)

Dates are `YYYY-MM-DD`, date-times `YYYY-MM-DD HH:MM:SS`. Missing text categories are `Unknown`; missing numbers and dates are `NULL`. Currency is INR.

### departments_clean
| Column | Description |
|---|---|
| Department_ID | Primary key, D01–D12 |
| Department_Name | Emergency, ICU, Cardiology, Orthopedics, Neurology, General Medicine, Pediatrics, General Surgery, Radiology, Laboratory, Pharmacy, Outpatient Clinic |
| Department_Type | Emergency / Inpatient / Outpatient / Unknown |
| Hospital_Building | Main Block, Block A, Block B, Annex |
| Department_Capacity | Number of beds (ER and inpatient) or patients per day (outpatient) |
| Department_Manager | Staff_ID of the manager |
| Operating_Hours | `24x7` or `HH:MM-HH:MM` |

### calendar_clean
| Column | Description |
|---|---|
| Date_ID | Primary key, `YYYYMMDD` |
| Full_Date | Calendar date |
| Day_Name / Day_Of_Week | Monday…Sunday / 1 = Monday … 7 = Sunday |
| Month / Quarter | 1–12 / 1–4 |
| Is_Weekend, Is_Holiday | Yes / No |
| Holiday_Name | Name of the Indian public holiday; empty on normal days |
| Season | Winter, Summer, Monsoon, Post-Monsoon |

### patients_clean
| Column | Description |
|---|---|
| Patient_ID | Primary key |
| Age | 0–110; NULL if unknown |
| Gender | Male / Female / Other / Unknown |
| City, Region | Patient city; Region (West, North, South, East, Central) is derived from City |
| Insurance_Type | Private / Public / Self-pay / Unknown |
| Patient_Category | New (registered in 2026) / Returning / Unknown |
| Chronic_Condition_Flag | Yes / No / Unknown |
| Registration_Date | First registration date |

### staff_clean
| Column | Description |
|---|---|
| Staff_ID | Primary key |
| Staff_Name | Synthetic name, standard "First Last" format |
| Department_ID | Foreign key → departments |
| Staff_Role | Doctor / Nurse / Technician / Unknown |
| Employment_Type | Full-time / Part-time / Contract / Unknown |
| Experience_Years | Years of experience, 0–60 |
| Joining_Date | Employment start |
| Employment_Status | Active / Inactive / Unknown |
| Hourly_Cost | Cost per working hour (INR) |

### beds_clean
| Column | Description |
|---|---|
| Bed_ID | Primary key |
| Department_ID | Foreign key → departments |
| Ward_Name | Ward the bed is in |
| Bed_Type | General / ICU / Emergency / Private / Unknown |
| Bed_Status | Available / Maintenance / Inactive / Unknown |
| Operational_Flag | Yes only when the bed is Available |
| Daily_Bed_Cost | Daily operating cost (INR) |

### encounters_clean
| Column | Description |
|---|---|
| Encounter_ID | Primary key (one hospital visit) |
| Patient_ID, Department_ID | Foreign keys → patients, departments |
| Admission_DateTime | Arrival time |
| Admission_Type | Emergency / Scheduled / Referral / Unknown |
| Triage_Level | Critical / High / Medium / Low / Unknown |
| Diagnosis_Category | Cardiovascular, Respiratory, Trauma, Infectious Disease, Gastrointestinal, Neurological, Orthopedic, Renal, Oncology, Metabolic, Obstetrics, Unknown |
| Admission_Status | Admitted (still in hospital) / Discharged / Transferred |
| Discharge_DateTime | Empty while the patient is still admitted |
| Discharge_Disposition | Home / Transfer / Other / Unknown |
| Attending_Doctor_ID | Foreign key → staff (a doctor) |
| Length_of_Stay_Hours | **Recalculated** from the two timestamps |
| Readmission_Flag | Back within 30 days: Yes / No / Unknown |

### patient_flow_events_clean
| Column | Description |
|---|---|
| Flow_Event_ID | Primary key |
| Encounter_ID | Foreign key → encounters |
| Event_Type | Registration / Triage / Doctor / Bed / Discharge / Unknown |
| Event_Start_Time | When the patient joined the queue for that step |
| Event_End_Time | When the step finished; empty if cancelled |
| Department_ID, Assigned_Staff_ID | Where it happened / who handled it |
| Waiting_Time_Minutes | Minutes waited before the step started |
| Event_Status | Completed / Delayed / Cancelled / Unknown |

`Delayed` is set when the wait passes the step's limit: registration and triage > 15 min, doctor > 10 / 30 / 60 / 90 min for Critical / High / Medium / Low, bed > 120 min, discharge > 90 min.

### bed_occupancy_clean
| Column | Description |
|---|---|
| Occupancy_ID | Primary key |
| Bed_ID, Encounter_ID | Foreign keys → beds, encounters (one bed stay per encounter) |
| Admission_Time, Discharge_Time | Bed in / bed out; Discharge_Time empty while occupied |
| Occupancy_Status | Occupied / Released |
| Bed_Turnaround_Minutes | Minutes to clean and reset the bed |
| Cleaning_Completed_Flag | Yes (Released) / No (Occupied) |

### staff_shifts_clean
| Column | Description |
|---|---|
| Shift_ID | Primary key |
| Staff_ID, Department_ID | Foreign keys; department comes from the staff table |
| Shift_Date | Date the shift starts |
| Shift_Type | Morning 07:00–15:00, Evening 15:00–23:00, Night 23:00–07:00 (next day) |
| Scheduled_Start / Scheduled_End | Planned times (8 hours) |
| Actual_Start / Actual_End | Real times; empty when absent |
| Scheduled_Hours, Actual_Hours, Overtime_Hours | Overtime = actual hours above 8 |
| Absence_Flag | Yes / No / Unknown |

### staffing_requirements_clean
| Column | Description |
|---|---|
| Requirement_ID | Primary key |
| Department_ID, Requirement_Date, Shift_Type, Acuity_Level | Unique together (Acuity: Critical / High / Medium / Low) |
| Expected_Patients | Expected patient count |
| Required_Doctors, Required_Nurses | Staff needed |
| Available_Doctors, Available_Nurses | Staff actually available |

**Staffing gap = Required − Available** (positive = shortage, negative = surplus).

## 6. Methodology

### Step 1 – Profile the raw data
Every file was read as text (so nothing was guessed) and profiled for nulls, placeholder values (`NA`, `N/A`, `null`, `-`, `?`), duplicates, label variants and value ranges. About **114,000 data-quality problems** were found across the 10 files.

### Step 2 – Clean with Python (one notebook per table)
The same ordered recipe was applied to every table:

| # | Problem | Method |
|---|---|---|
| 1 | Hidden missing values | Strip spaces; convert `NA`, `N/A`, `null`, `None`, `-`, `?` to real nulls |
| 2 | Spelling mistakes and label variants | `standardise()` helper: exact match → alias dictionary → fuzzy match (`difflib`); an "unmatched" report shows anything missed |
| 3 | Mixed date formats | Try each exact format in turn; impossible and future dates become null |
| 4 | Numbers stored as text / impossible numbers | Extract numbers (`Rs. 4,500`, `45 yrs`, `five`); null negatives and outliers by rule |
| 5 | Derived columns that disagree with their source | Recalculate: length of stay, hours, overtime, day name, quarter, region, department |
| 6 | Logic errors | Swap reversed start/end, add a day to night-shift ends, fix status conflicts |
| 7 | Duplicates | Exact copies → same-ID copies (keep most complete) → same-record-new-ID copies (business key) |
| 8 | Broken links | Anti-join every foreign key; unusable rows go to `quarantine_*.csv`, conflicts to `review_*.csv` |
| 9 | Recoverable gaps | Fill from related data: a missing start/end time from the neighbouring event, a missing bed discharge from the discharge event, a department from the ward, capacity from the bed count |
| 10 | Remaining blanks | Text categories → `Unknown`; numbers and dates stay `NULL` |

### Step 3 – Validate
Cleaned data was checked against the original (pre-error) data: **99.96% of filled-in cells match**. The remaining mismatches are judgement calls on conflicting duplicates and shift end times that are a few minutes off.

### Step 4 – Load into MySQL
Tables are loaded parents-first (departments → calendar → patients → staff → beds → encounters → events/occupancy/shifts/requirements) with primary keys, foreign keys and CHECK rules, so the database itself rejects bad values. The final foreign-key check returns **0 problems**.

### Step 5 – Analyse with SQL
36 business questions (Section 9) answered with joins, CTEs, window functions (`RANK`, `LAG`, `ROW_NUMBER`, running totals) and conditional aggregation.

### Step 6 – Visualise in Power BI
Star schema with `calendar_clean` as the date table and DAX measures for waiting time, delayed %, bed utilisation, staffing gap, overtime and cost. The result is a 5-page dashboard (Section 8).

## 7. Key business insights

### Major problems
| # | Insight | Evidence |
|---|---|---|
| 1 | **Doctor waiting is the biggest bottleneck** | Average wait 72 min; Low-priority patients wait 110 min, Medium 67 min; 39% of doctor steps are delayed |
| 2 | **Triage is the most delayed step** | 69% of triage steps are delayed (registration 17%, discharge 9%, bed 5%) |
| 3 | **Some critical patients are left waiting** | 88 of 927 critical patients (9.5%) waited more than 10 min; the longest wait was 133 min |
| 4 | **Nurses are short, doctors are not** | ER nurse coverage: 85% morning, 87% night, 91% evening (morning shortfall 114 nurses). ER doctor coverage 101–125%. 42% of all department / shift / acuity slots are short of nurses |
| 5 | **ER beds are under pressure** | ER bed use 74% vs 19–56% in other departments. In July, 23.6% of ER bed requests waited over 2 hours; the longest wait was 613 min |
| 6 | **Chronic patients keep coming back** | Chronic patients aged 65+ are readmitted 16.9% of the time vs 4–8% for non-chronic patients |
| 7 | **Critical patients are transferred out far more** | 23.2% transfer rate for critical vs about 7% for every other level |
| 8 | **Overtime is costly and concentrated in the ER** | ER overtime ≈ ₹10.3 lakh (assumes 1.5× pay), the highest of any department; some staff work more than 2× their department's average overtime |

### Minor problems
| # | Insight | Evidence |
|---|---|---|
| 1 | Idle beds elsewhere | General Surgery 19%, Pediatrics 26%, General Medicine 30% bed use |
| 2 | Beds out of service | 36 of 300 beds offline (24 maintenance, 12 inactive) |
| 3 | Slow ICU bed turnaround | 82 min vs about 48 min elsewhere |
| 4 | Discharge waiting | 48 min average in every department |
| 5 | Night-shift absence | 5.7% vs 4.3–4.5% on other shifts |
| 6 | Monday overload | 3,306 visits vs 2,443 on Sundays (about 35% more) |
| 7 | Peak hours | ER doctor waits run 64–77 min from 10:00 to 21:00, against 36–49 min between 02:00 and 08:00 |
| 8 | Long waits lengthen stays | ER patients who waited over 2 hours stayed 21.1 h vs 18.8 h for shorter waits |

Weekends and seasons made no difference to doctor waits (about 72 min in every case); holidays were only slightly worse (77 vs 72 min).

### Financial and operational insights (estimates)
There is no revenue or billing data, so the financial page measures **cost and efficiency**, not profit.

| # | Insight | Evidence |
|---|---|---|
| 1 | **Staff cost is concentrated in the ER** | ER labour cost ₹1.43 crore of ₹6.91 crore total (15 Sep – 28 Oct), about ₹9,900 per ER visit |
| 2 | **Overtime adds about 7% to labour cost** | ₹48.5 lakh of overtime pay (at 1.5×); the ER is highest at ₹10.3 lakh |
| 3 | **Absence has a cost** | About ₹33 lakh of paid scheduled hours were lost to absence |
| 4 | **Unused bed capacity is the largest hidden cost** | Idle bed-days cost about ₹25 crore over 301 days; General Medicine (₹6.2 crore), ICU (₹5.9 crore) and General Surgery (₹3.9 crore) lead |
| 5 | **Offline beds still cost money** | 36 offline beds carry ₹1.10 lakh a day of running cost (₹13.8 lakh a day for all beds) |

Cost figures use `Hourly_Cost` and `Daily_Bed_Cost` from the data; 26 beds have no cost and are excluded. Idle bed cost = (bed-days available − bed-days occupied) × average daily bed cost, which assumes a bed costs the same whether or not it is used.

### Recommendations
1. **Add triage and nurse capacity at peak hours (10:00–21:00) and on Mondays**, since doctors are not the shortage.
2. **Set a hard "see a doctor within 10 minutes" rule for Critical patients** and monitor breaches daily.
3. **Move capacity:** open inpatient beds (General Medicine, Surgery, Pediatrics) to relieve ER boarding, and bring maintenance beds back faster.
4. **Start a follow-up programme for chronic patients aged 65+** (call-backs, discharge plans).
5. **Rebalance rosters and overtime:** cap overtime for burn-out-risk staff and review night-shift coverage.
6. **Fix data capture:** missing attending doctor, triage level and waiting times limit what can be measured.

## 8. Power BI dashboard

**Healthcare Intelligence, Hospital Performance & Operational Optimization Platform** has five pages. Each page answers one management question.

| Page | Name | Question it answers | Key KPIs and visuals |
|---|---|---|---|
| 1 | **Executive Hospital Performance** | How is the hospital doing overall? | Total visits, unique patients, average length of stay, readmission %, transfer %; visits by month and by department |
| 2 | **Patient Flow & Capacity Management** | Where do patients wait and are beds available? | Average doctor wait, delayed % by stage, boarding %, bed utilisation %, bed turnaround; wait by hour and by triage level |
| 3 | **Patient Outcomes & Quality of Care** | Are patients getting good care? | Length of stay by department and diagnosis, readmission by chronic condition and age, transfer % by triage, discharge delays |
| 4 | **Department & Workforce Efficiency** | Do we have the right staff at the right time? | Nurse and doctor gap, coverage %, overtime hours, absence %, patients per nurse; gap by department and shift |
| 5 | **Financial & Operational Performance** | Where is money being spent or wasted? | Labour cost, overtime cost, absence cost, idle and offline bed cost, cost per ER visit |



### Data model
`calendar_clean` is the date table. Relationships (one-to-many, single direction from the one side):

- `departments_clean[Department_ID]` → `encounters_clean`, `beds_clean`, `staff_clean`, `staff_shifts_clean`, `staffing_requirements_clean`, `patient_flow_events_clean`
- `patients_clean[Patient_ID]` → `encounters_clean`
- `encounters_clean[Encounter_ID]` → `patient_flow_events_clean`, `bed_occupancy_clean`
- `beds_clean[Bed_ID]` → `bed_occupancy_clean`
- `staff_clean[Staff_ID]` → `staff_shifts_clean`
- `calendar_clean[Full_Date]` → a date-only column in each fact table (for example `Admission_Date = DATE(YEAR([Admission_DateTime]), MONTH([Admission_DateTime]), DAY([Admission_DateTime]))`)

### Main DAX measures
```DAX
-- Page 1: Executive
Total Visits       = COUNTROWS(encounters_clean)
Unique Patients    = DISTINCTCOUNT(encounters_clean[Patient_ID])
Avg Length of Stay = AVERAGE(encounters_clean[Length_of_Stay_Hours])
Readmission % =
DIVIDE(
    CALCULATE(COUNTROWS(encounters_clean), encounters_clean[Readmission_Flag] = "Yes"),
    CALCULATE(COUNTROWS(encounters_clean), encounters_clean[Readmission_Flag] <> "Unknown"))
Transfer % =
DIVIDE(CALCULATE(COUNTROWS(encounters_clean), encounters_clean[Admission_Status] = "Transferred"), [Total Visits])

-- Page 2: Patient flow and capacity
Avg Doctor Wait (min) =
CALCULATE(AVERAGE(patient_flow_events_clean[Waiting_Time_Minutes]), patient_flow_events_clean[Event_Type] = "Doctor")
Delayed % =
DIVIDE(
    CALCULATE(COUNTROWS(patient_flow_events_clean), patient_flow_events_clean[Event_Status] = "Delayed"),
    CALCULATE(COUNTROWS(patient_flow_events_clean), patient_flow_events_clean[Event_Status] IN {"Completed", "Delayed"}))
Boarding % (bed wait > 2h) =
DIVIDE(
    CALCULATE(COUNTROWS(patient_flow_events_clean), patient_flow_events_clean[Event_Type] = "Bed", patient_flow_events_clean[Waiting_Time_Minutes] > 120),
    CALCULATE(COUNTROWS(patient_flow_events_clean), patient_flow_events_clean[Event_Type] = "Bed"))
Occupied Bed Hours =
SUMX(bed_occupancy_clean,
     DATEDIFF(bed_occupancy_clean[Admission_Time],
              IF(ISBLANK(bed_occupancy_clean[Discharge_Time]), DATE(2026, 10, 29), bed_occupancy_clean[Discharge_Time]),
              MINUTE) / 60)
Available Bed Hours   = DISTINCTCOUNT(beds_clean[Bed_ID]) * 301 * 24
Bed Utilisation %     = DIVIDE([Occupied Bed Hours], [Available Bed Hours])

-- Page 4: Workforce
Nurse Gap          = SUM(staffing_requirements_clean[Required_Nurses]) - SUM(staffing_requirements_clean[Available_Nurses])
Nurse Coverage %   = DIVIDE(SUM(staffing_requirements_clean[Available_Nurses]), SUM(staffing_requirements_clean[Required_Nurses]))
Overtime Hours     = SUM(staff_shifts_clean[Overtime_Hours])
Absence % =
DIVIDE(CALCULATE(COUNTROWS(staff_shifts_clean), staff_shifts_clean[Absence_Flag] = "Yes"), COUNTROWS(staff_shifts_clean))

-- Page 5: Financial and operational (Overtime Rate = a What-if parameter, default 1.5)
Labour Cost    = SUMX(staff_shifts_clean, staff_shifts_clean[Actual_Hours] * RELATED(staff_clean[Hourly_Cost]))
Overtime Cost  = SUMX(staff_shifts_clean, staff_shifts_clean[Overtime_Hours] * RELATED(staff_clean[Hourly_Cost]) * [Overtime Rate Value])
Absence Cost =
SUMX(FILTER(staff_shifts_clean, staff_shifts_clean[Absence_Flag] = "Yes"),
     staff_shifts_clean[Scheduled_Hours] * RELATED(staff_clean[Hourly_Cost]))
Offline Bed Cost per Day =
CALCULATE(SUM(beds_clean[Daily_Bed_Cost]), beds_clean[Bed_Status] IN {"Maintenance", "Inactive"})
Idle Bed Cost = ([Available Bed Hours] - [Occupied Bed Hours]) / 24 * AVERAGE(beds_clean[Daily_Bed_Cost])
```
Measure names and table names are suggestions: adjust them to match your own model.

## 9. SQL business questions

36 tested queries in `HEALTHCARE_business_questions_MySQL.sql`:

| Section | Questions |
|---|---|
| A. Patient flow and waiting (Q1–Q8) | Wait by stage and triage, bottleneck stage, critical-patient breaches, door-to-doctor time, busiest hours, monthly trend, wait vs length of stay, 10 worst days |
| B. Bed capacity (Q9–Q14) | Utilisation by department, ER boarding by month, bed turnaround, idle capacity, offline beds, busiest days |
| C. Staffing (Q15–Q22) | Staffing gap by department and shift, shortage by acuity, ER coverage %, overtime cost, absence, burn-out risk, workforce mix, patients per nurse |
| D. Outcomes (Q23–Q28) | Readmission by chronic condition and age, readmission by diagnosis, transfers, length of stay, diagnosis Pareto, doctor caseload |
| E. Demand (Q29–Q34) | Busiest day-and-hour slots, monthly growth, first vs repeat visits, region and insurance mix, holidays, seasons |
| F. Data quality (Q35–Q36) | Remaining missing values, incomplete patient journeys |

Example – which stage is the bottleneck?
```sql
SELECT Event_Type AS stage, COUNT(*) AS events,
       ROUND(100.0 * SUM(CASE WHEN Event_Status = 'Delayed' THEN 1 ELSE 0 END) / COUNT(*), 1) AS delayed_pct,
       ROUND(AVG(Waiting_Time_Minutes), 1) AS avg_wait_min
FROM patient_flow_events_clean
WHERE Event_Status IN ('Completed', 'Delayed') AND Event_Type <> 'Unknown'
GROUP BY Event_Type
ORDER BY delayed_pct DESC;
```


## 12. How to run

**Python and SQLite (quick start)**
```bash
pip install pandas numpy
python run_all.py              # cleans all 10 tables, then builds HEALTHCARE.db
```

**MySQL**
1. `CREATE DATABASE HEALTHCARE; USE HEALTHCARE;`
2. Import each `clean/*_clean.csv` as a table (Workbench → Table Data Import Wizard); tables are named `departments_clean`, `calendar_clean`, `patients_clean`, `staff_clean`, `beds_clean`, `encounters_clean`, `patient_flow_events_clean`, `bed_occupancy_clean`, `staff_shifts_clean`, `staffing_requirements_clean`.
3. Run Step 0 at the top of `HEALTHCARE_business_questions_MySQL.sql` (null fixes and indexes), then the questions.

## 12. Assumptions and limitations
- The data is synthetic; the findings show the method, not a real hospital.
- Overtime cost assumes overtime is paid at 1.5× the hourly cost; no overtime rate exists in the data.
- Missing values are not guessed: they stay null (numbers, dates) or `Unknown` (categories). Some metrics use only rows where both inputs exist.
- Where duplicate rows disagreed (for example triage level), the most complete row was kept and the conflict saved in `review_*.csv`.
- There is no revenue or billing data, so the financial page shows cost and efficiency only.
- Idle bed cost assumes a bed costs the same whether or not it is used.
- Staffing requirements cover September–October only; encounters cover January–October.

## 13. Author
**[solayma molla ]** · [LinkedIn](www.linkedin.com/in/solayma-molla-1a7126356) · [Email](shifamolla24@gmail.com)
