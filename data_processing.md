# Data Processing & Transformation Log

## 1. Source Data

* **Database:** `exam_event_logs` (PostgreSQL)
* **Table:** `candidate_log`
* **Total Raw Rows:** 8,246,901

---

## 2. Composite Key Validation

### Issue

`log_id` was not unique across the dataset.

### Action

Created a composite identifier:

```python
composite_key = log_id + "_" + candidate_id
```

### Result

Verified uniqueness across all **8,246,901** records.

---

## 3. Data Type Corrections

| Column                | Original Type | Corrected Type | Reason                        |
| --------------------- | ------------- | -------------- | ----------------------------- |
| `question_section`    | object        | Int64          | Numeric grouping and sorting  |
| `question_display_id` | object        | Int64          | Numeric ordering of questions |
| `subject_id`          | object        | Int64          | Aggregation and filtering     |
| `logged_at`           | object        | datetime64     | Time-based analysis           |

---

## 4. Null Handling

| Column              | Issue                                | Action                        |
| ------------------- | ------------------------------------ | ----------------------------- |
| `question_response` | NULL values for unanswered questions | Replaced with `"No Response"` |
| `logged_at`         | No missing values after processing   | No action required            |

---

## 5. Auto Save Event Segregation

### Observation

System-generated **Auto Save** events accounted for **7,482,437 rows (90.73%)** of the dataset.

### Action

Split the data into separate analytical datasets:

| Dataset       | Rows      | Purpose                        |
| ------------- | --------- | ------------------------------ |
| `df_filtered` | 764,464   | Candidate behavioural analysis |
| `df_raw`      | 7,482,437 | Response pattern analysis      |

### Reason

Auto Save events do not represent deliberate candidate actions and would significantly distort behavioural metrics if included.

---

## 6. Feature Engineering

| Feature     | Source Column | Logic                              |
| ----------- | ------------- | ---------------------------------- |
| `hour`      | `logged_at`   | `dt.hour`                          |
| `minute`    | `logged_at`   | `dt.minute`                        |
| `time_slot` | `hour`        | Categorized into exam-time buckets |

---

## 7. Data Quality Investigation

During processing, all Auto Save records initially appeared with NULL timestamps.

### Root Cause

Notebook kernel state inconsistency during dataframe transformations.

### Resolution

Applied all transformations on the base dataframe (`df`) before creating derived datasets (`df_filtered` and `df_raw`).

### Outcome

Timestamp integrity restored across all records.

---

## 8. Final Dataset Summary

| Dataset       | Rows      | Unique Candidates | Purpose              |
| ------------- | --------- | ----------------- | -------------------- |
| `df`          | 8,246,901 | 87,999            | Master dataset       |
| `df_filtered` | 764,464   | 79,407            | Behavioural analysis |
| `df_raw`      | 7,482,437 | 87,999            | Response analysis    |
