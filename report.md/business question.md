# Business Questions and Answers

36 business questions answered with SQL on the `HEALTHCARE` database (MySQL). Every query is in [`sql/HEALTHCARE_business_questions_MySQL.sql`](../sql/HEALTHCARE_business_questions_MySQL.sql); question numbers below match the query numbers.

**Conventions:** staffing gap = Required − Available (positive = shortage). Utilisation = occupied bed-hours ÷ available bed-hours (1 Jan – 28 Oct 2026). Overtime cost assumes 1.5× the hourly cost. Currency is INR.

---

## A. Patient flow and waiting times

| Q | Business question | Why it matters | Headline answer |
|---|---|---|---|
| 1 | Where do patients wait the longest, by stage and triage level? | Shows which step to fix first | Doctor step: Low-priority patients wait 110 min, Medium 67 min. Bed waits are 13–17 min and discharge waits about 48 min for every level |
| 2 | Which stage is the bottleneck (share of delayed steps)? | Targets process improvement | Triage 69% delayed, doctor 39%, registration 17%, discharge 9%, bed 5% |
| 3 | How many Critical patients waited more than 10 min for a doctor? | Patient-safety check | 88 of 927 (9.5%); the longest wait was 133 min |
| 4 | How long from arrival to seeing a doctor (door-to-doctor)? | The wait patients actually feel | Low 182 min, Medium 119, High 63, Critical 30 |
| 5 | Which hours are busiest and when does the doctor wait peak? | Roster planning | Arrivals peak at 11:00 (726) and 18:00 (687). Doctor waits are 64–78 min from 10:00 to 20:00 and 36–49 min from 02:00 to 07:00 |
| 6 | Is the ER getting better or worse month by month? | Trend monitoring | Flat: monthly average doctor wait stays between 69 and 75 min |
| 7 | Does a long doctor wait lead to a longer stay? | Links delay to bed-days | Yes: stay is 18.1 h when the wait is under 30 min and 21.1 h when it is over 120 min |
| 8 | Which 10 days had the worst ER doctor waits? | Finds exceptional days | Worst: Mon 10 Aug (126 min), then 1 Jan (106 min) and 25 Feb (102 min) |

## B. Bed capacity

| Q | Business question | Why it matters | Headline answer |
|---|---|---|---|
| 9 | How busy is each department's bed capacity? | Shows who is overloaded and who has spare beds | ER 74%, ICU 56%, Orthopedics 45%, Neurology 44%, Cardiology 42%, General Medicine 30%, Pediatrics 26%, General Surgery 19% |
| 10 | How often do ER patients wait more than 2 hours for a bed? | Measures ER overcrowding ("boarding") | Worst month July: 23.6%; then May 12.2%; March only 0.2%. Longest wait 613 min |
| 11 | Which department is slowest to ready a bed? | Bed turnaround drives capacity | ICU 82 min; every other department about 47 min |
| 12 | Where is capacity sitting idle (under 35% use)? | Candidates for moving beds or staff | General Medicine, Pediatrics and General Surgery |
| 13 | How many beds are offline and what do they cost? | Cost of unusable capacity | 36 of 300 beds (24 maintenance, 12 inactive), about Rs. 1.1 lakh a day; General Surgery has the most under maintenance (6) |
| 14 | Which days had the most beds in use at noon? | Hospital-wide pressure days | Peak was 141 beds (49% of open beds) on 21 Oct: the hospital as a whole never ran above half full, so the pressure is in the ER |

## C. Staffing and workforce

| Q | Business question | Why it matters | Headline answer |
|---|---|---|---|
| 15 | Which department and shift has the biggest staffing gap? | Where to hire or reassign | ER morning is short of 114 nurses, General Medicine evening 96, Cardiology evening 69, ER evening 68. Doctor gaps are negative (surplus) everywhere |
| 16 | For which acuity level is the hospital short-staffed most often? | Risk by patient type | Short of nurses in 45% of High, Low and Medium slots and 30% of Critical slots |
| 17 | What share of required ER staff do we actually have? | Coverage by shift | Nurses: 85% morning, 87% night, 91% evening. Doctors: 113%, 125%, 101% |
| 18 | How much overtime is worked and what does it cost? | Cost and fatigue | ER 921 hours, about Rs. 10.3 lakh; General Surgery Rs. 5.6 lakh; Outpatient Clinic Rs. 5.1 lakh |
| 19 | Which shifts and departments have the highest absence? | Reliability of cover | Laboratory nights 8.0%, ICU nights 6.8%, Laboratory mornings 6.5%, Pediatrics nights 6.5%, ER nights 6.2% |
| 20 | Who is at burn-out risk (overtime above 2× department average)? | Staff welfare and retention | Examples: a nurse in D12 with 30.6 h vs a 11.4 h department average; a nurse in D05 with 27.7 h vs 10.6 h |
| 21 | What is the staff mix and cost by department? | Workforce planning | The ER has 81 active staff (14 doctors, 51 nurses), 12.3% on contract, average Rs. 736 per hour |
| 22 | On which ER shifts did one nurse cover the most expected patients? | Shows unsafe ratios | Three shifts had one nurse for 14–24 expected patients (8 Oct morning: 24). Check whether these are data-entry gaps |

## D. Patient outcomes

| Q | Business question | Why it matters | Headline answer |
|---|---|---|---|
| 23 | Who is readmitted within 30 days (chronic condition and age)? | Targets follow-up care | Chronic 65+ 16.9%, chronic 18–64 13.7%, non-chronic 65+ 7.8%, non-chronic 18–64 4.2% |
| 24 | Which diagnoses have the most readmissions? | Clinical quality | Fairly flat at 7.6–8.5% (Oncology highest), so chronic status and age matter more than diagnosis |
| 25 | How often are patients transferred out, by triage level? | Capability and capacity signal | Critical 23.2% vs 7.1–7.7% for every other level |
| 26 | How long do patients stay in each department? | Bed planning | ICU 153 h, Orthopedics 109 h, Neurology 107 h, General Medicine 95 h, General Surgery 88 h |
| 27 | Which diagnoses drive ER volume (Pareto)? | Focus clinical pathways | Cardiovascular 16.1%, Trauma 15.6%, Respiratory 13.8%, Infectious Disease 12.0%, Gastrointestinal 12.0%: together 69.6% |
| 28 | Which doctors carry the heaviest ER caseload? | Workload balance | Well balanced: the top five each handle 6.8–6.9% of ER patients |

## E. Demand and patient mix

| Q | Business question | Why it matters | Headline answer |
|---|---|---|---|
| 29 | What are the busiest weekday and hour slots? | Roster design | Monday 11:00 (238 arrivals), Monday 17:00 (225), Friday 18:00 (220). Mondays take five of the top nine slots |
| 30 | How does monthly volume change? | Demand forecasting | 1,856–2,125 visits a month; peak in May; February down 11.1% on January; no sustained growth |
| 31 | First-time vs repeat visits by month? | Loyalty and follow-up load | Repeat share rises from 10% to 81%, but only because the data starts on 1 January |
| 32 | What is the mix by region and insurance? | Who the patients are | West dominates (Private 3,749, Self-pay 3,661, Public 3,283 visits); stays 46–48 h; readmission 7–8% |
| 33 | Are holidays busier? | Holiday rosters | 68.6 arrivals a day on holidays vs 65.9 on normal days (10 holiday days) |
| 34 | Does the season change demand? | Seasonal planning | Flat: 65.5–67.6 visits a day in every season |

## F. Data quality

| Q | Business question | Why it matters | Headline answer |
|---|---|---|---|
| 35 | How many key fields are still missing? | How far to trust the numbers | Attending doctor 478, discharge time 456, triage level 597, diagnosis 794, waiting time 2,294 rows |
| 36 | Which patient journeys are incomplete? | Missing steps distort waiting times | The ER has the most (149 encounters), then General Medicine (29) and Outpatient Clinic (25) |
