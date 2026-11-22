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