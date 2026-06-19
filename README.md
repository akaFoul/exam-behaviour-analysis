# Exam Behaviour Analysis

Analysis of anonymized event-level logs from a large-scale online examination to uncover candidate behaviour patterns, engagement trends, and difficulty signals.

## Project Structure

```text
exam-behaviour-analysis/
├── data_extraction.ipynb     # Main analysis notebook
├── screenshots/             # Chart outputs
├── .env.example             # DB config template
└── README.md
```

## Dataset Overview

| Property          | Value                               |
| ----------------- | ----------------------------------- |
| Source            | PostgreSQL dump (`exam_event_logs`) |
| Table             | `candidate_log`                     |
| Total Rows        | 8,246,901                           |
| Unique Candidates | 8,246,901                             |
| Exam Date         | 2025-09-23                          |
| Time Range        | 09:00 – 19:12                       |

---

## Setup & Requirements

### 1. Install Dependencies

```bash
pip install psycopg2-binary pandas sqlalchemy plotly
```

### 2. Configure Database

```bash
cp .env.example .env
# Edit .env and add your PostgreSQL credentials
```

### 3. Restore the Database Dump

```bash
psql -U postgres -c "CREATE DATABASE exam_event_logs;"

pg_restore -U postgres \
  -d exam_event_logs \
  --no-owner \
  exam_event_logs.dump
```

### 4. Run the Notebook

Open `data_extraction.ipynb` in Jupyter Notebook and run all cells from top to bottom.

> A live PostgreSQL connection is required. The original dataset is too large to be uploaded directly to GitHub.

---

## Data Processing Summary

| Step                | Action                                                   | Reason                                          |
| ------------------- | -------------------------------------------------------- | ----------------------------------------------- |
| Composite Key       | `log_id + candidate_id`                                  | `log_id` alone is not unique                    |
| Type Casting        | `question_section`, `question_display_id` → Int64        | Stored as strings in source DB                  |
| Null Handling       | `question_response` NULL → `"No Response"`               | Explicit representation of unanswered questions |
| Feature Engineering | Extracted `hour`, `minute`, `time_slot` from `logged_at` | Enable temporal analysis                        |
| Dataset Split       | `df_filtered` (behavioural) / `df_raw` (Auto Save)       | Different analytical objectives                 |

### Key Data Quality Finding

> All 7.48M Auto Save records initially appeared with NULL timestamps due to a notebook kernel state issue during processing. The issue was resolved by applying all transformations on the master dataframe (`df`) before splitting into `df_filtered` and `df_raw`.

---

## Analysis & Key Findings

### 1. Activity Distribution by Section

* **Section 1** recorded the highest number of both **Mark for Review** (~159k) and **UnMark for Review** (~84k) interactions.
* The **UnMark-to-Mark ratio** was also highest in Section 1, indicating candidates revisited flagged questions more frequently in this section.
* Sections 2, 3, and 4 showed very similar interaction volumes, suggesting comparable engagement levels across these sections.

### 2. Hourly Activity Pattern

* Sharp activity drops around **10:00 AM** and **2:00 PM** suggest multiple exam batches operating in parallel.
* Peak activity occurred around **4:00 PM** (~258k interactions), representing the largest batch at peak engagement.

### 3. Language Preference by Section

* Sections 1–3 maintained a consistent language split of approximately **69% English** and **31% Hindi**.
* Section 4 was conducted entirely in English, making it the only mono-language section.

### 4. Response Distribution (A/B/C/D)

* Section 1 exhibited a significant **Option A preference**, with approximately **763k selections** compared to **495k selections** for Option B.
* This pattern may indicate a strong response bias and warrants further investigation.

### 5. Review Resolution Rate

* Section 1 achieved the highest review resolution rate at **53.12%**.
* Sections 2 and 3 declined to approximately **35–36%**, suggesting fewer flagged questions were revisited later in the exam.

### 6. Candidate Engagement Funnel

* **95.34% of candidates (83,899)** attempted all four sections.
* The exam demonstrated a very high completion rate with minimal candidate drop-off.

### 7. Most Reviewed Questions

* **Section 1 – Question 25** was flagged approximately **64k times**, nearly double any other question.
* Question 25 appeared among the most reviewed questions across multiple sections, indicating a recurring review pattern.

### 8. Average Time Spent per Section

* Section 1 recorded the highest average time spent (**32.38 minutes**).
* Sections 3 & 4 sit in between (~21-22 mins) — moderate, fairly balanced pacing in the latter half of the exam

---

## Tech Stack

* **Python** — Pandas, SQLAlchemy, Plotly
* **PostgreSQL** — Data storage and querying
* **Jupyter Notebook** — Analysis and documentation
* **Git & GitHub** — Version control and collaboration

---

## Author

**Ashutosh Pratap Singh**

GitHub: https://github.com/akaFoul