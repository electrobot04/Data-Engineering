In Data Science and Machine Learning, **skewness** describes how much a dataset's distribution is tilted towards one side.

When analysing data, we usually look at the distribution of values to understand how the data is spread. 

1. Symmetrical Distribution
2. Positively Skewed Distribution (Righ-Skewed)
3. Negatively Skewed Distribution (Left-Skewed)
4. Other important distributions
	1. Uniform Distribution
	2. Bimodal Distribution
	3. Multimodal Distribution
	4. Exponential Distribution

# 1. Symmetrical Distribution (Normal Distribution)

![[Pasted image 20261115213148.png]]

A dataset is symmetrical when values are distributed almost equally on both sides of the centre.

**Characteristics:**
- Mean = Median = Mode (Approximately)
- Both sides of the distribution look similar
- Skewness is approximately 0.

**Example:**
Suppose student's marks are:
`60, 65, 70, 70, 70, 75, 80`
The values are distributed relatively evenly around 70.

# 2. Positively Skewed Distribution (Right-Skewed)

![[Pasted image 20261115213450.png]]

A dataset is positively skewed when its tail extends towards the **right side**.

In other words, most values are concentrated on the left, with a few unusually large values on the right.

**Example: Salaries**
`20k ,22k, 23k, 24k, 25k, 27k, 200k`
Most people earn between 20k and 27k, but one person earns 200k.

That unusually high salary pulls the mean towards the right.

**Characteristics:**
- Mean > Median > Mode.
- Tail extends towards the right.
- Skewness > 0

# 3. Negatively Skewed Distribution (Left-Skewed)

![[Pasted image 20261115214444.png]]

A dataset is negatively skewed when its tail extends towards the **left-side**.

Most values are concentrated on the right, with a few unusually small values on the left.

**Example: Exam marks**
`10, 75, 80, 85, 88, 90, 92`
Most students scored high marks, but one student scored very low.

That unusually low mark pulls the mean towards the left.

**Characteristics:**
- Mean < Median < Mode.
- Skewness < 0

# 4. Other important distributions you should know
Apart from skewness, there are other distribution patterns worth understanding.

## A. Uniform Distribution

![[Pasted image 20261115215201.png]]

All values have approximately equal frequency or probability within a given range.

**Example:** Rolling a fair six-sided dice
Each outcome (1,2,3,4,5,6) has an equal probability of 1/6.

# B. Bimodal Distribution

![[Pasted image 20261115215349.png|541]]

A dataset has two distinct peaks (modes).

Example:
	Imagine combining the heights of adult men and women into one dataset.
	You might observe two peaks because the two groups have different average heights.

# C. Multimodal Distribution

![[Pasted image 20261115215602.png|556]]

A dataset has more than two distinct peaks.

**Example:** Combining the heights of children, teenagers and adults.

## D. Exponential Distribution

![[Pasted image 20261115215705.png]]

A distribution where smaller values are much more frequent, and larger values become progressively less frequent.

**Example:** Waiting time between customer arrivals at a store (under suitable assumptions).

# 5. Important Comparison Table

| Distribution | Characteristics                | Skewness                           |
| ------------ | ------------------------------ | ---------------------------------- |
| Symmetrical  | Both sides approximately equal | 0                                  |
| Right-Skewed | Tail extends right             | Positive                           |
| Left-Skewed  | Tail extends Left              | Negative                           |
| Uniform      | Approximately equal frequency  | 0                                  |
| Bimodal      | Two peaks                      | Can be positive, negative, or zero |
| Multimodal   | Multiple peaks                 | Can be positive, negative, or zero |
| Exponential  | Long right tail                | Positive                           |

# 6. Why is this important in Data Cleaning and Machine Learning?

Understanding distribution helps us identify potential data-quality problems.

For example:

| Problem           | What you can do                                                |
| ----------------- | -------------------------------------------------------------- |
| Right-skewed data | Consider log transformation                                    |
| Left-skewed data  | Consider an appropriate transformatioon                        |
| Extreme Outliers  | Investigate and decide whether to cap or remove them           |
| Missing values    | Consider median or mode imputation                             |
| Multiple peaks    | Investigate whether the dataset contains different populations |

**Important:** Skewness doesn't automatically mean your data is incorrect. A salary dataset being right-skewed is perfectly natural.

# 7. Remember these three concepts
When studying data distributions, always understand these three:
1. **Central Tendency:** Where is the centre of your data? (Mean, Median, Mode)
2. **Dispersion:** How spread out is your data? (Range, Variance, Standard Deviation, IQR)
3. **Shape:** How is your data distributred? (Skewness, Symmetry, Kurtosis)

One additional concept worth learning is **Kurtosis**, which measures the heaviness of a distribution's tails and its tendency to produce extreme values.

These concepts are fundamental to statistics, exploratory data analysis (EDA), and machine learning.