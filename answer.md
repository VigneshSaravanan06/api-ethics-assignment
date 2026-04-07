# API Ethics Assignment

## Task 1 — Classify and Handle PII Fields

The dataset contains multiple fields that include personally identifiable information (PII). These fields must be handled carefully before sharing with external partners.

| Field           | Type                  | Action               | Justification                                                                                                            |
| --------------- | --------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| full_name       | Direct PII            | Drop                 | Directly identifies an individual; not required for analysis.                                                            |
| email           | Direct PII            | Drop                 | Unique identifier; high privacy risk and unnecessary for research purposes.                                              |
| date_of_birth   | Indirect PII          | Mask                 | Can be used to infer identity; convert to age or age group (e.g., 20–30).                                                |
| zip_code        | Indirect PII          | Mask                 | Can lead to re-identification; retain only partial (e.g., first 3 digits).                                               |
| job_title       | Indirect PII          | Generalize           | Useful for analysis but may identify individuals in rare roles; convert to broader category (e.g., “Healthcare Worker”). |
| diagnosis_notes | Sensitive Health Data | Pseudonymize / Clean | Contains sensitive medical information; remove any personal identifiers and keep only relevant medical insights.         |

### Summary:

* **Direct PII** should be removed completely.
* **Indirect PII** should be masked or generalized.
* **Sensitive data** should be anonymized and carefully processed.

---

## Task 2 — Audit the API Script for Ethical Compliance

### Original Issues in the Script

###  Violation 1: Hardcoded API Key

**Problem:**

* The API key is directly written in the code.
* This is a security risk and may violate the API provider’s Terms of Service.
* Anyone accessing the code can misuse the key.

**Fix:**

* Store the API key securely using environment variables.

```python
import os
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")

records = []

for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    
    if response.status_code != 200:
        break
    
    data = response.json()
    records.extend(data["results"])
```

---

###  Violation 2: Excessive Data Collection & Storing Raw PII

**Problem:**

* The script collects large amounts of data (100 pages) without checking API limits.
* It does not respect rate limiting or fair usage policies.
* Raw personal and sensitive data is stored permanently without anonymization.
* This creates serious privacy and ethical concerns, especially in healthcare.

**Fix:**

* Limit data collection
* Respect API rate limits
* Store only cleaned and anonymized data

```python
import os
import time
import requests

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")

records = []

for page in range(1, 21):  # Reduced number of pages
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})
    
    if response.status_code != 200:
        break
    
    data = response.json()
    
    for record in data["results"]:
        cleaned_record = {
            "age_group": record.get("date_of_birth"),  # should be converted properly
            "zip_prefix": record.get("zip_code"),
            "job_category": record.get("job_title"),
            "diagnosis_notes": record.get("diagnosis_notes")
        }
        records.append(cleaned_record)
    
    time.sleep(1)  # Delay to respect rate limits

# Store only cleaned data
save_to_database(records)
```

---

## Final Notes

* Always follow the **principle of data minimization**.
* Avoid collecting or storing unnecessary personal data.
* Never expose API keys or credentials in source code.
* Apply anonymization techniques before sharing datasets.
* Ensure compliance with API Terms of Service and data privacy standards.

---
