# 1. Unit at a glance
**Data cleaning** (also called **data cleansing**) = finding and fixing wrong, missing, duplicate or inconsistent values so that analysis and ML models can trust the data. Real data is almost never clean, so cleaning is a core step of every data pipeline, not an optional extra.

**One-line summary of the unit:** dirty data is normal, it silently breaks results, so learn to detect it, repair it, validate it, and always think critically about what could still be wrong.

## Learning ourcomes 

| S.no | After this unit should be able to...                                                  | Where in these notes |
| ---- | ------------------------------------------------------------------------------------- | -------------------- |
| 1    | Understand the importance of data quality                                             | Sections 2 and 4     |
| 2    | Understand how cleaning techniques work: outlier detection/correction, type inference | Sections 7 and 8     |
| 3    | Apply cleaning techniques and test your data for quality metrics                      | Sections 4, 6 and 10 |
| 4    | Think critically about your dataset and identify potential pitfalls                   | Section 14           |

## To-do list
- [ ] Read the provided resources on data cleaning and why it matters for ML pipeline (the six papers are summarized in Sections 9 to 13)
- [ ] Complete the quiz on ETL, ELT, ... Pipeline Naming (cheat sheet in Section 5) 
- [ ] Complete the assignment: set up a professional data cleaning pipeline (template code in Section 6)

# The resources at a glance

| Resource                                                                 | Main idea in one line                                                                           | Section |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- | ------- |
| How to Clean Your Data in Python                                         | Practical pandas steps: missing values, duplicates types, outlier, text                         | 8       |
| Data Cleaning Is a Machine Learning Problem That Needs Data Systems Help | Cleaning is too big for hand-written rules; use ML (e.g. HoloClean) plus scalable systems       | 9       |
| Data Validation for Machine Learning (Google, TFX)                       | Treat data like code: a schema plus automatic checks catch bad data before it reaches the model | 10      |
| Data Management Challenges in Production ML                              | Understanding, validating and cleaning data are ongoing jobs in production, not one-off tasks   | 11      |
| From Chaos to Clarity (LLMs for data cleaning)                           | LLMs can detect and fix messy text/tabular data, but need checks                                | 12      |
| What Should Data Science Education Do with LLMs?                         | LLMs shift the data scientist's role from doing every step to designing, checking and judging   | 13      |

---
# 2. Why data gets dirty, and why it matters
Errors enter data at every step between the real world and your database, so you should assume any dataset you receive is dirty until you have checked it.

## Where dirty data comes from
| Cause                                | What goes wrong                                        | Real-life example                                                                                  |
| ------------------------------------ | ------------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| App runs into an unanticipated state | It writes a value nobody planned for                   | A sign-up form stores age = -1 when the user skips the fields                                      |
| Bug in application logic             | Values are computed wrongly                            | Version 2 of an app stores prices in cents, version 1 in euros, so 19.99 and 1999 sit side by side |
| Bug in transmission                  | Records arrive cut off or garbled                      | "München" arrives as "MÃ¼nchen" because of a wrong text encoding                                   |
| Application crashes                  | Half-written rows, or the same row twice after a retry | A payment is logged twice after the app restarts mid-request                                       |
| Timing issues when writing           | Events land out of order                               | A delivery has a "delivered" time earlier than its "ordered" time                                  |
| Broken sensor                        | Stuck, zero or impossible readings                     | A room thermometer reports $300\degree C$ or 0.0 for whole day                                     |
| Humans typing into a form            | Typos, free text, mixed formats                        | "Comp Sci", "cs", and "Computer Science" for one study major; 03/04/2024 meaning March             |

## Why "just a few bad values" is not OK
A few faulty values can move a result a lot. The course calls this **brittle**: it may be fine, until it is not.

**Example 1, outlier skew the mean.** Think of taxi trip records with a passenger count column:

```
import pandas as pd

passengers = pd.Series([1,1,2,1,3,1,2,1,1])
print(passengers.mean()) # 1.44 -> sensible

passengers_dirty = pd.Series([1,1,2,1,3,1,2,1,1,208]) # one bad record
print(passengers_dirty.mean()) # 22.1 -> nonsense: a taxi with 22 people on average
print(passengers_dirty.median()) # 1.0 -> the median is robust to the outlier
```

One wrong row out of ten made the average 15 times too big.

**Example 2, missing values hidden as 0 or -1.** If "missing" is stored as a number instead of NULL, the computer treats it as a real value:

```
import numpy as np

ages = pd.Series([25,30,-1,40,0]) # -1 and 0 mean "unknown"
print(ages.mean()) # 18.8 -> wrong

ages_clean = ages.replace({-1: np.nan, 0: np.nan})
print(ages_clean.mean()) # 31.67 -> right, NaN values are skipped
```

Garbage in, garbage out
- **"Garbage in, garbage out":** a model can only be as good as the data it learns from.
- **"Data quality beats fancy algorithms":** a simple model on clean data usually beats a complex model on dirty data.
- Real-life example: a spam filter trained on emails where many spam messages where labelled "not spam" learns to let spam through, however advanced the algorithm is.
- Data scientists spend about 80% of their time cleaning data.

## The hidden problem
You usually do not know what is wrong with a dataset you recieve. Even a dataset that was "already cleaned" can contain errors the cleaner never thought of. So: always think critically and ask what could go wrong.

---
# 3. Types of dirty data
Most real dataset have several error types at once, and they interact (Section 9), but it helps to learn them one by one first.

## A small dirty table to practise on
|  id | name         | age | country | signup_date | salary_eur |
| --: | ------------ | --: | ------- | ----------- | ---------: |
|   1 | Anna Mueller |  29 | Germany | 2024-03-01  |      52000 |
|   2 | anna mueller |  29 | germany | 01/03/2024  |      52000 |
|   3 | Ben Okafor   |  -1 | Nigeria | 2024-03-05  |            |
|   4 | Chen Wei     | 210 | China   | 2024-13-01  |      61000 |
|   5 | Dana Silva   |  34 | USA     | 2024-03-09  |  1,200,000 |
|   6 | Eli Cohen    |  41 | U.S.    | 2024-03-10  |      58000 |
What is wrong, row by row:
- Row 2 is a **near-duplicate** of row 1: different case and a different data format, same person.
- Row 3 has a **sentinel** (-1 means "unknown age") and a **missing** salary.
- Row 4 has an **impossible value** (age 210) and an **invalid date** (month 13).
- Row 5 stores salary as **text with commas**, so pandas reads the whole column as string, and the value itself is a suspicious **outlier**.
- Rows 5 and 6 spell one country two ways (**inconsistent representation:** USA vs U.S.)

## The main error types
| Error type                     | What it looks like                                             | Real-life example                                               | How to detect                                  | Typical fix                                                      |
| ------------------------------ | -------------------------------------------------------------- | --------------------------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| Missing values                 | NULL, NaN, empty string, or a sentinel like 0, -1, 9999, "N/A" | 30% of Netflix titles have no director (Python guide)           | `df.isna().mean()` count sentinels             | Drop, fill (mean, median, mode, "Unknown"), or predict the value |
| Outliers and impossible values | Values far from the rest, or outside physical limits           | Age 210, a taxi with 208 passengers                             | Z-score, IQR rule, domain limits               | Fix if the true value is known else set to NaN, cap, or drop     |
| Wrong data type                | Same entity stored twice, exactly or almost                    | "Anna Meuller" and "anna mueller"                               | `df.duplicate()`, fuzzy matching               | Normalize, then drop or merge                                    |
| Inconsistent representation    | One meaning, several spellings                                 | USA/U.S/United States; Netflix rating UR vs NR                  | `value_counts()`,`unique()`                    | Map to one canonical value                                       |
| Value in the wrong column      | Data shifted into a neighbour field                            | Netflix "rating" contains "74 min" (a duration)                 | Unique value per column                        | Move it to the right column, restore the right value             |
| Outdated or merged categories  | Real-world change not reflected                                | "Soviet Union" and "Russia", "West Germany" and "Germany"       | Unique values plus domain knowledge            | Decide a rule and map consistently                               |
| Rule (constraint) violation    | Two fields that must agree do not                              | ZIP 10115 with city "Hamburg" (10115 is in Berlin)              | Check rules such as functional dependencies    | Repair the value that breaks the rule                            |
| Multi-valued cells             | Several values packed into one cell                            | country = "United States, India"                                | `str.contains(',')`                            | Split into lists or rows (`str.split.explode`)                   |
| Free-text categories           | Everyone types their own label                                 | Survey "Major": "cs", "Chem E","IMB"                            | `value_counts()` shows hundreds of tiny groups | Map to a fixed list by rules, or with an LLM (Section 12)        |
| Training-serving skew (ML)     | Data at prediction time differs from training data             | Booleans coded 0/1 in training but "true"/"false" in production | Compare statistics of both datasets            | Fix the pipeline code (Sections 10 and 11)                       |

---
# 4. Data Quality: dimensions and metrics you can test
Data quality is measured along a few standard dimensions; each one turns into a number or a pass/fail test you can run in code, exactly like unit tests for software.

| Dimension    | Question it answers                                             | Metric                                                           | Example check                                    |
| ------------ | --------------------------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------ |
| Completeness | Is anything missing?                                            | % of cells that are not missing                                  | salary present in at least 95% of rows           |
| Uniqueness   | Is anything stored twice?                                       | % of rows with a unique key; number of duplicate rows            | every `id` appears once                          |
| Validity     | Do values follow the rules (type, range, format, allowed list)? | % of rows that pass each rule                                    | age between 0 and 120; date is YYYY-MM-DD        |
| Consisitency | Do related values agree, inside one table and across tables?    | % of rows where linked fields match                              | ZIP code matches city; end date after start date |
| Accuracy     | Does the value match reality?                                   | % correct against a trusted source (often a hand-checked sample) | Sample 100 addresses and verify them             |
| Timeliness   | Is the data recent enough?                                      | Age of the newest record, delay since the event                  | Yesterdays's sales are loaded by 8:00            |
The basic formulas:
$$
\text{completeness} = \dfrac{\text{non-missing cells}}{\text{all cells}}\times 100\%
$$
$$
\text{validity}_{rule}=\dfrac{\text{rows that pass the rule}}{\text{all rows}}\times 100\%
$$
**Code: a quality report for the dirty table in Section 3**
```
import pandas as pd

df = pd.read_csv("customers.csv)    # the dirty table from section 3

def quality_report(df,key):
	return pd.Series({
		"rows": len(df),
		"completeness_%": round(100 * df.notna().to_numpy().mean(),1),
		"duplicate_rows": int(df.duplicated().sum()),
		"key_unique_%": round(100 * df[key].nunique() / len(df), 1)
	})
	
def validity_report(df):
	rules = {
		"age in 0..120": df["age"].between(0,120),
		"date is YYYY-MM-DD": pd.to_datetime(df["signup_date"], format="%Y-%m-%d", errors="coerc").notna(),
		"salary present": df["salary_eur"].notna(),
		"country known": df["country"].isin(["Germany", "Nigeria", "China", "USA"]),
	}
	return pd.Series({name: round(100 * ok.mean(), 1) for name, ok in rules.item()})
	
print(quality_report(df, "id"))
print(validity_report(df))
```

Output (tested):

|Metric|Value|
|---|---|
|rows|6|
|completeness_%|97.2 (35 of 36 cells filled)|
|duplicate_rows|0|
|key_unique_%|100|
|age in 0..120|66.7% valid|
|date is YYYY-MM-DD|66.7% valid|
|salary present|83.3% valid|
|country known|66.7% valid|

> **Lesson from this output:** `duplicate_rows` says 0, yet rows 1 and 2 are the same person. `duplicated()` only copies, so you must normalize text (lower case, trim spaces, one date format) **before** looking for duplicates. A metric is only as good as the check behind it.

## Turn metrics into tests
Write the rules as tests that stop the pipeline when the data is bad, the same way a failing unit test stops a software release:

```
def test_clean_data(df):
	assert df["id"].is_unique, "duplivate ids"
	assert df["age"].between(0,120).all(), "age out of range"
	assert df["salary_eur"].notna().mean() >= 0.95, "more than 5% salaries missing"
	assert pd.api.types.is_numeric_dtype(df["salary_eur"]), "salary is not numeric"
```

Libraries that do this at scale: **Great Expectations** and **panders** (Python), and **Tensorflow Data Validation** (Google's tool from the paper in Section 10).

---
# 5. The data cleaning pipeline, and ETL vs ELT naming
Professional cleaning is a repeatable pipeline, not a one-time manual fix: profile, detect, repair, validate, then monitor, and loop back whenever validation fails.
![[Pasted image 20261121182839.png]]
The inner loop runs unit every quality test passes; the outer loop runs on every new batch of data.

The five stages
1. **Profile (understand):** look at shape, types, ranges, distributions, unique values. Tools: `df.info()`,`df.describe()`,`value_counts()`,plots.
2. **Detect:** find missing values, outliers, duplicates, type errors and rule violations (Section 3).
3. **Repair:** drop, fill, convert, map, merge or flag. Keep the raw data untouched and write the cleaned data to a new table.
4. **Validate:** run the quality tests from Section 4. If a test fails, go back to Detect.
5. **Document and monitor:** record every decision (what you changed and why), and re-run the checks on every new batch of data, because new data brings new errors (Section 11).

## ETL, ELT and friends (for the quiz)
The letters give the **order** of steps: **E**xtract (pull data from sources), **T**ransform (clean, join, reshape), **L**oad (write into the target system).

![[Pasted image 20261121183744.png]]
Blue marks where cleaning happens: before storage in ETL, inside the warehouse in ELT.

| Name          | Order of steps                     | Where cleaning happens                                                                       | Typical use                                         | Real-life example                                                                                              |
| ------------- | ---------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| ETL           | Extract, transform, Load           | On a separate processing server, before the warehouse                                        | Classic on-premise data warehouses, strict schemas  | A bank cleans and checks transactions every night, then loads only valid rows into its warehouse               |
| ELT           | Extract, Load, Transform           | Inside the warehouse or data lake, after loading (often SQL, e.g. dbt)                       | Cloud warehouses such as BigQuery or Snowflake      | An online shop dumps raw orders into BigQuery, then cleans them with SQL; the raw copy stays for re-processing |
| EtLT          | Extract, small t, Load, big T      | A light fix before loading (mask personal data, drop exact duplicates), the heavy work after | When privacy rules forbid loading raw personal data | Hash e-mail addresses before loading, build reports later                                                      |
| Reverse ETL   | Warehouse to business tools        | Data is already clean in the warehouse                                                       | Send results back to apps people use                | Push a "customer churn risk" score from the warehouse into the sales team's CRM                                |
| Streaming ETL | Transform each event as it arrives | In the stream, record by record                                                              | Real-time dashboards, fraud alerts                  | Flag a card payment as suspicious within seconds                                                               |
| Zero-ETL      | No hand-built pipeline             | Handled by the platform's built-in replication                                               | Same-vendor database to warehouse copies            | An operational database replicated automatically into an analytics warehouse                                   |

**ETL vs ELT at a glance**

|                  | ETL                                            | ELT                                           |
| ---------------- | ---------------------------------------------- | --------------------------------------------- |
| Raw data kept?   | Usually not                                    | Yes, so you can re-clean later with new rules |
| Speed of loading | Slower (transform first)                       | Faster (load first)                           |
| Transform power  | Limited by the ETL server                      | Uses the warehouse's large compute            |
| Privacy          | Sensitive fields can be removed before storage | Raw sensitive data lands in storage first     |
| Fits best        | Small, structured, stable data                 | Large, varied, changing data                  |

---
# 6. Hands-on: cleaning data in Python with pandas
The resource "How to Clean Your Data in Python" (Huong Ngo, 2022) cleans the Netflix titles dataset step by step; this section rebuilds those steps on a tiny made-up sample so you can run every line yourself.

**What the guide found in the Netflix data**
- `director` is missing in about 30% of rows; `cast` and `country` in about 9%; `date_added`, `rating`, `duration` in 0 to 0.1%.
- Every column except `release_year` has dtype `object` (pandas label for text or mixed type).
- `duration` mixes units: "90min" for movies, "2 Seasons" for TV shows, so the guide splits the data into movies and TV shows.
- The `rating` column contains "74min", "84min" and "66min" in 3 rows: durations typed into the wrong column.
- "UR" (Unrated) and "NR" (Not Rated) mean the same thing, so UR is changed to NR.
- `country` and `listed_in` hold several comma-separated values per cell; a Python `set` plus `split(",")` lists the unique values.
- Countries include both "Soviet Union" and "Russia", and "West/East Germany" and "Germany".
- The genres "TV Shows" and "Movies" turned out to mean "no genre given", found by inspecting the rows that use them.
- Last, the cleaned movies and TV shows can be joined back into one dataset (optional).

## Pandas Cheat Sheet
| Task                                      | Command                                             |
| ----------------------------------------- | --------------------------------------------------- |
| Size, types, non-null counts              | `df.shape`, `df.info()`, `df.dtypes`                |
| Summary statistics                        | `df.describe()`                                     |
| % missing per column                      | `df.isna().mean() * 100`                            |
| Drop rows with missing values in a column | `df.dropna(subset=["col"])`                         |
| Fill missing values                       | `df["col"].fillna(value)`                           |
| Count each category                       | `df["col"].value_counts()`                          |
| See all distinct values                   | `df["col"].unique()`                                |
| Find and drop exact duplicates            | `df.duplicated().sum()`, `df.drop_duplicates()`     |
| Text to number (bad values become NaN)    | `pd.to_numeric(df["col"], errors="coerce")`         |
| Text to date                              | `pd.to_datetime(df["col"], format="%B %d, %Y")`     |
| Remove extra spaces, unify case           | `df["col"].str.strip().str.lower()`                 |
| Pull a number out of text                 | `df["col"].str.extract(r"(\d+)")`                   |
| Map variants to one label                 | `df["col"].replace({"UR": "NR"})`                   |
| One row per value of a list cell          | `df.assign(c=df["c"].str.split(", ")).explode("c")` |
## Ways to handle missing values
| Strategy                            | When to use it                                          | Risk                                                |
| ----------------------------------- | ------------------------------------------------------- | --------------------------------------------------- |
| Drop the rows                       | Very few rows are missing (e.g. `date_added`, under 1%) | Lose data; bias if values are not missing at random |
| Drop the column                     | Most values are missing and the column is not needed    | Lose a possible useful feature                      |
| Fill with a placeholder ("Unknown") | Text columns such as `director` (30% missing)           | The model may treat "Unknown" as a real category    |
| Fill with mean or median            | Numeric columns; median when there are outliers         | Shrinks the spread of the data                      |
| Forward/backward fill               | Time series (a sensor reading continues)                | Wrong across long gaps                              |
| Predict the value (model, KNN)      | Valuable column, related columns exist                  | More work; can leak information                     |
| Add an "is_missing" flag column     | Whenever missingness itself may carry meaning           | One more column to manage                           |
## Step by Step (tested code)
```
import pandas as pd

# A tiny made-up sample with the same problems as the Netflix titles dataset
df = pd.DataFrame({
    "show_id":    ["s1", "s2", "s3", "s4", "s5", "s6", "s6"],
    "type":       ["Movie", "TV Show", "Movie", "Movie", "TV Show", "Movie", "Movie"],
    "title":      ["Movie A", "Series B", "Movie C", "Stand-up D", "Series E", "Movie F", "Movie F"],
    "director":   ["Dir 1", None, "Dir 3", "Dir 4", None, "Dir 6", "Dir 6"],
    "country":    ["United States", "South Africa", "United States, Ghana", None, "West Germany", "Soviet Union", "Soviet Union"],
    "date_added": ["September 25, 2021", "September 24, 2021", " September 24, 2021", "April 4, 2017", None, "July 1, 2019", "July 1, 2019"],
    "rating":     ["PG-13", "TV-MA", "UR", "74 min", "TV-14", "NR", "NR"],
    "duration":   ["90 min", "2 Seasons", "125 min", None, "1 Season", "95 min", "95 min"],
})

# Step 1: first look
print(df.shape)
df.info()

# Step 2: remove exact duplicates
print("duplicates:", df.duplicated().sum())
df = df.drop_duplcates()

# Step 3: missing values (% per column, largest first)
print(df.isna().mean().mul(100).round(1).sort_values(ascending=False))
df["director"] = df["director"].fillna("Unknown") # many missing -> placeholder
df["country"] = df["country"].fillna("Uknown")
df = df.dropna(subset=["date_added"]) # very few missing -> drop rows

# Step 4: unique values reveal a value in the wrong column
print(df["rating"].unique())    # ... 'UR', '74 min', 'NR'
wrong = df["rating"].str.contains("min", na=False)
df.loc[wrong, "duration"] = df.loc[wrong, "rating"]    # move it where it belongs
df.loc[wrong, "rating"] = "TV-MA"    # the real rating, looked up

# Step 5: one meaning = one label
df["rating"] = df["rating"].replace({"UR":"NR"})

# Step 6: correct data types
df["date_added"] = pd.to_datetime(df["date_added"].str.strip(), format="%B %d, %Y")
df["duration_num"] = df["duration"].str.extract(r"(\d+)",expand=False).astype(int)

# Step 7: multi-valued column -> unique values with a set
def unique_values(col):
	values = set()
	for cell in col.dropna():
		values.update(v.strip() for v in cell.split(","))
	return sorted(values)
	
print(unique_values(df["country"]))   # ['Ghana', 'South Africa', 'Soviet Union', ...]
df["country"] = df["country"].replace(
	{"Soviet Union": "Russia" , "West Germany": "Germany", "East Germany": "Germany},
	regex=True
)

# Step 8: split by type (minutes vs seasons), then optionally join back
movies = df[df["type"] == "Movie"].rename(columns={"duration_num": "minutes"})
tv_shows = df[df["type"] == "TV Show"].rename(columns={"duration_num": "seasons"})

combined = pd.concat([movies, tv_shows], ignore_index = True) 
``` 

## Template for the assignment: a professional cleaning pipeline
A professional pipeline is made of small, named, testable functions, it logs what each step changed, never edits the raw data in place, and ends with validation that **stops** the run if the data is still bad.
```
import logging
import pandas as pd

logging.basicConfig(level=logging.Info, format="%(message)s")
log = logging.getLogger("cleaning")

COUNTRY_MAP = {"Soviet Union": "Russia", "West Germany": "Germany", "East Germany": "Germany"}
VALID_RATINGS = {"G", "PG", "PG-13", "R", "NR", "TV-14", "TV-MA", "TV-PG"}

def drop_duplicates(df):
	log.info(f"drop_duplicates: removed {df.duplicated().sum()} rows")
	return df.drop_duplicates()
	
def handle_missing(df):
	df = df.assign(director=df["director"].fillna("Unknown"), country=df["country"].fillna("Unknown"))
	before = len(df)
	df = df.dropna(subset=["date_added"])
	log.info(f"handle_missing: dropped {before - len(df)} rows without date_added")
	return df
	
def fix_types(df):
	return df.assign(
		date_added=pd.to_datetime(df["date_added"].str.strip(), format="%B %d %Y", errors="coerce"),
		duration_num=pd.to_numeric(df["duration"].str.extract(r"(\d+)",expand=False), errors="coerce"),
	)
	
def standardize_categories(df):
	return df.assign(rating=df["rating"].replace({"UR":"NR"}), country=df["country"].replace(COUNTRY_MAP, regex=TRUE))

def validate(df):
    problems = []
    if df["show_id"].duplicated().any():
        problems.append("duplicate show_id")
    if df["date_added"].isna().any():
        problems.append("unparseable date_added")
    unknown = set(df["rating"]) - VALID_RATINGS
    if unknown:
        problems.append(f"unknown rating values: {sorted(unknown)}")
    if problems:
        raise ValueError("Validation failed: " + "; ".join(problems))
    log.info("validate: all checks passed")
    return df

def clean(raw):
    return (raw.copy()                 # never modify the raw data
               .pipe(drop_duplicates)
               .pipe(handle_missing)
               .pipe(fix_types)
               .pipe(standardize_categories)
               .pipe(validate))
```

When tested on a sample that still had "74 min" in `rating`, the pipeline stopped with `Validation failed: unknown rating values: ['74 min']`. That is the point: the final check catches what the cleaning steps forgot.

---
# 7. Outlier detection and correction
An **outlier** is a value far away from the rest. Statistics can tell you a value is unusual; only domain knowledge can tell you it is wrong. So detect with statistics and rules, then decide with judgment.

## Two kinds of outliers
| Kind                       | Meaning                                | Example                                                                        | What to do                                          |
| -------------------------- | -------------------------------------- | ------------------------------------------------------------------------------ | --------------------------------------------------- |
| Error outlier              | The value is impossible or a mistake   | Taxi trip with $208$ passengers; age $210$; temperature $300\degree C$ indoors | Correct it, or set it to missing                    |
| Real (true) outlier        | Rare but genuine                       | A billionaire's income; a real 6-person group in a large taxi                  | Keep it; maybe use robust methods                   |
| Outlier that is the signal | The rare case is what you want to find | A fraudulent credit card payment                                               | Never delete; this is the target of fraud detection |

## Detection methods
1. **Domain rules (always first):** Write down what is physically or logically possible: 1 to 6 passengers, age 0 to 120, price above 0, end date after start date.
2. **Z-score:** How many standard deviations a value is from the mean. Flag if $|z|>3$.
$$
z=\dfrac{x-\bar{x}}{s}
$$
Weakness: the outlier itself inflates the mean and standard deviation, which can hide other outliers (the **masking effect**). Assumes roughly bell-shaped data.
3. **IQR rule (box plot rule):** Q1 = 25th percentile, Q3  = 75th percentile, IQR = Q3-Q1. Flag values outside the fences:
$$
\text{lower fence}=Q_{1}-1.5\cdot IQR, \text{upper fence}=Q_{3}+1.5\cdot IQR
$$
Robust, because quartiles barely move when one extreme value is added.
4. **Modified z-score (MAD):** Uses the median and the median absolute deviation (MAD) instead of mean and standard deviation. Flag if |M|>3.5.
$$
M_{i}=\dfrac{0.6745(x_{i}-\bar{x})}{MAD}, \text{MAD}=\text{median}(|x_{i}-\bar{x}|)
$$
5. **Model-based (many columns at once):** Isolation Forest, Local Outlier Factor, DBSCAN. Use when a row is only strange as a combination, e.g. age 12 with 3 childer.
