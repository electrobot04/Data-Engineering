Mean, median, and mode are **measures of central tendency**. In data cleaning, they help us identify missing values, detect unusual values (outliers), and replace incorrect or missing data.

# 1. Mean (Average)
**How it helps:** Used to fill missing numerical values in a dataset.

Example:
Suppose we have student marks:
`70, 80, 90, NULL, 100`

Mean:
$$
Mean = \dfrac{70+80+90+100}{4} = 85
$$
We can replace the missing value (`NULL`) with **85**.

**When to use:**
- When numerical data is relatively consistent.
- When there are no extreme outliers.
- When missing values need to be filled.

**Limitation:** Mean is highly affected by outliers.

# 2. Median (Middle Value)
**How it  helps:** Used to fill missing values when the dataset contains outliers.

Example:
Suppose we have salaries:
`2000,2200,2500,NULL,50000`

Sorted values (excluding NULL):
`2000,2200,2500,50000`

Median:
$$
Median = \dfrac{2200+2500}{2} = 2350
$$
We can replace the missing value with **2350**.

Why not mean?
	Because the unusual high salary (50000) significantly increases the mean, whereas the median is less affected by extreme values.

**When to use:**
- When the dataset contains outliers.
- When numerical data is skewed.
- When replacing missing numerical values.

# 3. Mode (Most Frequent Value)
**How it helps:** Used to fill missing categorical or numerical values with the most frequently occurring value.

Example:
Suppose we have a dataset of students' department:
`CSE, IT, CSE, ECE, NULL, CSE, IT`

Mode = CSE (Appears most frequently).

We can replace the missing value with **CSE**.

**When to use:**
- When dealing with categorical data.
- When filling missing values in columns such as gender, department, city, etc.
- When identifying the most frequently occurring value.

# 4. Quick Comparison

| Measure | Data type               | Mean Application in cleaning                          |
| ------- | ----------------------- | ----------------------------------------------------- |
| Mean    | Numerical               | Replacing missing numerical values                    |
| Median  | Numerical               | Replacing missing values when outliers exist          |
| Mode    | Categorical & Numerical | Replacing missing values with the most frequent value |

# 5. Important: Can they help detect outliers?
Yes!

For example:
`10,12,11,13,12,100`
- Mean = 26.33
- Median = 12
- Mode = 12

Here, 100 is unusual high compared with the other values.

The difference between mean and median can indicate that the dataset might contain outliers or be skewed.

However, mean, median and mode alone cannot confirm an outlier. We can use:
- IQR (Interquartile Range)
- Z-score
- Box plots