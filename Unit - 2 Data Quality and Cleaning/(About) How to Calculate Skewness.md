Skewness can be calculated using a mathematical formula that measures the asymmetry of dataset.

There are two commonly used method's: **Pearson's coefficient of skewness** and **Moment coefficient of skewness**.

# 1. Pearson's Coefficient of Skewness
A simple way to calculate skewness using Mean, Median, and Standard Deviation.

**Formula:**
$$
\boxed{\text{Skewness} = \dfrac{3(\text{Mean}-\text{Median})}{\text{Standard Deviation}}}
$$
**Example:**
Consider this dataset:
$$X=[10,12,14,16,18,50]$$
**Step 1:** Calculate Mean
$$Mean = \dfrac{10+12+14+16+18+50}{4} = 20$$
**Step 2:** Calculate Median
Arrange the values in ascending order.
Since there are 6 values:
$$\text{Median}=\dfrac{14+16}{2}= 15$$
**Step 3:** Calculate Standard Deviation
Using population standard deviation:
$$
\sigma=\sqrt{ \dfrac{\sum(x_{i}-\mu)^2}{N}} = \sqrt{ \dfrac{(10-12)^2+\dots+(50-20)^2}{6} } = \sqrt{ 180 } = 13.416
$$
**Step 4:** Calculate Skewness
$$\text{Skewness}=\dfrac{3(20-15)}{13.416}=1.118$$
Since the result is positive, the dataset is **Positive skewed (right-skewed)**.

# 2. Moment Coefficient of Skewness
This is another commonly used method, particularly in statistics libraries.

**Formula:**
$$\boxed{\text{Skewness}=\dfrac{\dfrac{1}{N}\sum_{i=1}^{N}(x_{i}-\mu)^3}{\sigma^3}}$$
# 2. How to interpret the result?

| Skewness | Interpretation            |
| -------- | ------------------------- |
| 0        | Symmetrical distribution  |
| >0       | Positively skewed (right) |
| <0       | Negatively skewed (left)  |

A commonly used rough interpretation is:

| Skewness value | Interpretation               |
| -------------- | ---------------------------- |
| -0.5 to +0.5   | Approximately symmetrical    |
| -1 to -0.5     | Moderately negatively skewed |
| +0.5 to +1     | Moderately positevily skewed |
| Below -1       | Highly negatively skewed     |
| Above +1       | Highly positively skewed     |

# 4. Calculate Skewness using Python

```
import pandas as pd

data = [10,12,14,16,18,50]

df = pd.DataFrame(data, columns=["Values"])

print(df["Values"].skew())
```

Pandas uses **the adjusted sample skewness** formula by default.

```
from scipy.stats import skew

data = [10,12,14,16,18,50]

print(skew(data))
```

