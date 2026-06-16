\# Data Processing \& Transformation Log



\## 1. Source Data



\* \*\*Database:\*\* `exam\_event\_logs` (PostgreSQL)

\* \*\*Table:\*\* `candidate\_log`

\* \*\*Total Raw Rows:\*\* 8,246,901



\---



\## 2. Composite Key Validation



\### Issue



`log\_id` was not unique across the dataset.



\### Action



Created a composite identifier:



```python

composite\_key = log\_id + "\_" + candidate\_id

```



\### Result



Verified uniqueness across all \*\*8,246,901\*\* records.



\---



\## 3. Data Type Corrections



| Column                | Original Type | Corrected Type | Reason                        |

| --------------------- | ------------- | -------------- | ----------------------------- |

| `question\_section`    | object        | Int64          | Numeric grouping and sorting  |

| `question\_display\_id` | object        | Int64          | Numeric ordering of questions |

| `subject\_id`          | object        | Int64          | Aggregation and filtering     |

| `logged\_at`           | object        | datetime64     | Time-based analysis           |



\---



\## 4. Null Handling



| Column              | Issue                                | Action                        |

| ------------------- | ------------------------------------ | ----------------------------- |

| `question\_response` | NULL values for unanswered questions | Replaced with `"No Response"` |

| `logged\_at`         | No missing values after processing   | No action required            |



\---



\## 5. Auto Save Event Segregation



\### Observation



System-generated \*\*Auto Save\*\* events accounted for \*\*7,482,437 rows (90.73%)\*\* of the dataset.



\### Action



Split the data into separate analytical datasets:



| Dataset       | Rows      | Purpose                        |

| ------------- | --------- | ------------------------------ |

| `df\_filtered` | 764,464   | Candidate behavioural analysis |

| `df\_raw`      | 7,482,437 | Response pattern analysis      |



\### Reason



Auto Save events do not represent deliberate candidate actions and would significantly distort behavioural metrics if included.



\---



\## 6. Feature Engineering



| Feature     | Source Column | Logic                              |

| ----------- | ------------- | ---------------------------------- |

| `hour`      | `logged\_at`   | `dt.hour`                          |

| `minute`    | `logged\_at`   | `dt.minute`                        |

| `time\_slot` | `hour`        | Categorized into exam-time buckets |



\---



\## 7. Data Quality Investigation



During processing, all Auto Save records initially appeared with NULL timestamps.



\### Root Cause



Notebook kernel state inconsistency during dataframe transformations.



\### Resolution



Applied all transformations on the base dataframe (`df`) before creating derived datasets (`df\_filtered` and `df\_raw`).



\### Outcome



Timestamp integrity restored across all records.



\---



\## 8. Final Dataset Summary



| Dataset       | Rows      | Unique Candidates | Purpose              |

| ------------- | --------- | ----------------- | -------------------- |

| `df`          | 8,246,901 | 87,999            | Master dataset       |

| `df\_filtered` | 764,464   | 79,407            | Behavioural analysis |

| `df\_raw`      | 7,482,437 | 87,999            | Response analysis    |



