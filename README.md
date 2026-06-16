# Exam-behaviour-analysis



Analysis of anonymized event-level logs from a large-scale online examination to uncover candidate behaviour patterns, engagement trends, and difficulty signals.





\##  Project Structure

exam-behaviour-analysis/



├── data\_extraction.ipynb     # Main analysis notebook



├── screenshots/              # Chart outputs



├── .env.example              # DB config template



└── README.md



\---



\##  Dataset Overview



| Property | Value |

|---|---|

| Source | PostgreSQL dump (`exam\_event\_logs`) |

| Table | `candidate\_log` |

| Total Rows | 8,246,901 |

| Unique Candidates | 87,999 |

| Exam Date | 2025-09-23 |

| Time Range | 09:00 — 19:12 |



\---



\## Setup \& Requirements



\### 1. Install dependencies

```bash

pip install psycopg2-binary pandas sqlalchemy plotly python-dotenv

```



\### 2. Configure database

```bash

cp .env.example .env

\# Edit .env and add your PostgreSQL credentials

```



\### 3. Restore the database dump

```bash

psql -U postgres -c "CREATE DATABASE exam\_event\_logs;"

pg\_restore -U postgres -d exam\_event\_logs --no-owner exam\_event\_logs.dump

```



\### 4. Run the notebook

Open `data\_extraction.ipynb` in Jupyter and run all cells top to bottom.



> A live PostgreSQL connection is required. The dataset is too large to be uploaded.



\---



\##  Data Processing Summary



| Step | Action | Reason |

|---|---|---|

| Composite Key | `log\_id + candidate\_id` | `log\_id` alone is not unique |

| Type Casting | `question\_section`, `question\_display\_id` → Int64 | Stored as strings in DB |

| Null Handling | `question\_response` NULL → `"No Response"` | Explicit over implicit nulls |

| Feature Engineering | Extracted `hour`, `minute`, `time\_slot` from `logged\_at` | Enable time-based analysis |

| Split Datasets | `df\_filtered` (behavioural) / `df\_raw` (Auto Save) | Different analysis needs |



\### Key Data Quality Finding

> All 7,482,437 Auto Save rows initially had NULL timestamps — traced to a kernel 

> state issue during processing. Resolved by applying all transformations on `df` 

> before splitting into `df\_filtered` and `df\_raw`.



\---



\## 📊 Analysis \& Key Findings



\### 1. Activity Distribution by Section

Section 1 had the highest candidate interactions (\~158k Mark for Review events),

nearly 25% more than other sections — suggesting highest difficulty or most time spent.



\### 2. Hourly Activity Pattern

Sharp drops at 10:00 and 14:00 reveal multiple exam batches running in parallel.

Peak activity at 16:00 (258k interactions) represents the largest batch mid-exam.



\### 3. Language Preference by Section

Consistent 69% English / 31% Hindi split across Sections 1-3.

Section 4 was 100% English — only section with no bilingual support.



\### 4. Response Distribution (A/B/C/D)

Section 1 shows a massive Option A bias (\~804k selections vs \~440k for B) —

a strong signal of guessing behaviour when candidates were uncertain.



\### 5. Review Resolution Rate

Section 1 had the highest resolution rate at 53.12% — candidates actively

returned to flagged questions. Sections 2 \& 3 dropped to \~35-36%,

indicating time pressure in later sections.



\### 6. Candidate Engagement Funnel

95.34% of candidates (83,899) attempted all 4 sections —

strong exam completion rate with minimal dropout.



\### 7. Most Reviewed Questions

Section 1-Q25 was flagged \~65k times — nearly double any other question.

Q25 appears in top 3 across multiple sections, consistently the last question

in each section flagged under time pressure.



\### 8. Average Time Spent per Section

Section 1 averaged 12.38 mins — almost double Section 2 (6.89 mins).

Candidates front-loaded effort on Section 1 and accelerated through the rest.



\---



\## Tech Stack



\- \*\*Python\*\* — Pandas, SQLAlchemy, Plotly

\- \*\*PostgreSQL\*\* — Data storage and querying

\- \*\*Jupyter Notebook\*\* — Analysis and documentation

\- \*\*Git/GitHub\*\* — Version control



\---



\##  Author

\*\*Ashutosh Pratap Singh\*\*  

\[GitHub Profile](https://github.com/akaFoul)

