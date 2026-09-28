# Data Jobs Market Analysis

## Overview

This project analyzes job-market data using Python to explore patterns in job postings, Data Analyst demand, skill trends, and salary levels.

The project is divided into ****two separate parts****:

### Part 1 — Exploratory Data Analysis (EDA)

A separate exploratory analysis performed to understand the dataset and its overall structure.

### Part 2 — Data Analyst Job Market Analysis

A focused analysis consisting of:

* Job Demand
* Trend Skills
* Skills Pay

The project uses the `lukebarousse/data_jobs` dataset available through Hugging Face.

---

# Part 1 — Exploratory Data Analysis (EDA)

EDA was performed separately from the main Data Analyst job-market analysis.

The purpose of this stage was to explore the dataset, understand its structure, identify relevant columns, and prepare the data for further analysis.

### I) EDA Visualization

![EDA Visualization](/images/EDA1.png)

### II) EDA Visualization

![EDA Visualization](/images/EDA2.png)

### III) EDA Visualization

![EDA Visualization](/images/EDA3.png)

### IV) EDA Visualization

![EDA Visualization](/images/EDA4.png)

## Libraries

```python
import ast
import pandas as pd
from datasets import load_dataset
import matplotlib.pyplot as plt
```

## Loading the Dataset

```python
dataset = load_dataset('lukebarousse/data_jobs')
df = dataset['train'].to_pandas()
```

The dataset is loaded using the Hugging Face `datasets` library and converted into a Pandas DataFrame.

## Data Preparation

The `job_posted_date` column was converted to datetime format, while `job_skills` was converted from its stored string representation into Python lists.

```python
df['job_posted_date'] = pd.to_datetime(df['job_posted_date'])

df['job_skills'] = df['job_skills'].apply(
    lambda x: ast.literal_eval(x) if pd.notna(x) else x
)
```

---

# Part 2 — Data Analyst Job Market Analysis

After the separate EDA, the project focuses specifically on understanding the ****Data Analyst job market****.

The analysis is divided into three areas:

```text
Data Analyst Job Market
│
├── Job Demand
├── Trend Skills
└── Skills Pay
```

---

# 1. Job Demand

The Job Demand analysis examines job-posting activity in the ****United States**** and compares the monthly demand for different job categories.

### Data Preparation

```python
import pandas as pd
from datasets import load_dataset
import matplotlib.pyplot as plt

dataset = load_dataset('lukebarousse/data_jobs')
df = dataset['train'].to_pandas()

df['job_posted_date'] = pd.to_datetime(df['job_posted_date'])

df_US = df[df['job_country'] == 'United States'].copy()

df_US['job_posted_month'] = df_US['job_posted_date'].dt.strftime('%B')
```

The analysis first filters the dataset to US job postings and extracts the month from each posting date.

### Creating the Monthly Job-Demand Table

```python
df_US_pivot = df_US.pivot_table(
    index='job_posted_month',
    columns='job_title_short',
    aggfunc='size'
)

df_US_pivot.reset_index(inplace=True)

df_US_pivot['month_no'] = pd.to_datetime(
    df_US_pivot['job_posted_month'],
    format='%B'
).dt.month

df_US_pivot.sort_values('month_no', inplace=True)

df_US_pivot.set_index('job_posted_month', inplace=True)

df_US_pivot.drop(columns='month_no', inplace=True)
```

A pivot table is used to calculate job-posting counts by month and job category. The month number is temporarily added to ensure that the months are displayed in calendar order.

### Selecting the Top Job Categories

```python
top_3 = df_US['job_title_short'].value_counts().head(3).index.tolist()
```

The three most frequently occurring job categories are selected for the trend comparison.

### Visualization

```python
fig, ax = plt.subplots(figsize=(12, 6), dpi=100)

df_US_pivot[top_3].plot(
    kind='line',
    ax=ax,
    linewidth=2.5,
    colormap='tab10'
)

ax.set_title(
    'US Pivoted Trends',
    fontsize=15,
    fontweight='bold',
    loc='left',
    pad=15
)

ax.set_xlabel(
    'Months',
    fontsize=11,
    fontweight='semibold'
)

ax.set_ylabel(
    'Count of Jobs',
    fontsize=11,
    fontweight='semibold'
)

ax.grid(True, linestyle='--', alpha=0.5)

ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

ax.legend(
    title='Category',
    bbox_to_anchor=(1.02, 1),
    loc='upper left',
    frameon=False
)

plt.tight_layout()
plt.show()
```

### Job Demand Visualization

![Job Demand Visualization](/images/Job_Demand.png)

This visualization shows how job-posting volumes for the three most common job categories change across the months in the dataset.

---

# 2. Trend Skills

The Trend Skills analysis focuses specifically on ****Data Analyst**** job postings and examines how frequently different skills appear over time.

## Filtering Data Analyst Jobs

```python
df_DA = df[df['job_title_short'] == 'Data Analyst'].copy()

df_DA['job_posted_month_no'] = df_DA['job_posted_date'].dt.month
```

The dataset is filtered to Data Analyst positions, and the posting month is extracted as a numerical value.

## Creating the Skills Trend Table

Because each job posting can contain multiple skills, the `job_skills` column is exploded so that individual skills can be counted separately.

```python
df_DA_explode = df_DA.explode('job_skills')

df_DA_pivot = df_DA_explode.pivot_table(
    index='job_posted_month_no',
    columns='job_skills',
    aggfunc='size',
    fill_value=0
)
```

The total number of occurrences for each skill is then calculated and used to order the skill categories.

```python
df_DA_pivot.loc['Total'] = df_DA_pivot.sum()

df_DA_pivot = df_DA_pivot[
    df_DA_pivot.loc['Total'].sort_values(ascending=False).index
]

df_DA_pivot = df_DA_pivot.drop('Total')
```

## Visualization

The five most frequently occurring skills are plotted as monthly trend lines.

```python
plt.style.use('seaborn-v0_8-whitegrid')

ax = df_DA_pivot.iloc[:, :5].plot(
    kind='line',
    figsize=(11, 5),
    linewidth=2.5,
    colormap='Set2',
    alpha=0.9
)

ax.set_title(
    'Top 5 Categories Trend Analysis',
    fontsize=14,
    fontweight='bold',
    loc='left',
    pad=15
)

ax.set_xlabel(
    'Date / Time Period',
    fontsize=11,
    labelpad=10
)

ax.set_ylabel(
    'Value',
    fontsize=11,
    labelpad=10
)

ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

ax.grid(True, linestyle='--', alpha=0.4)

ax.legend(
    title='Category',
    bbox_to_anchor=(1.02, 1),
    loc='upper left',
    frameon=False
)

plt.tight_layout()
plt.show()
```

### Trend Skills Visualization

![Trend Skills Visualization](/images/Trending_Skills.png)

This visualization helps examine how the frequency of the most common Data Analyst skills changes across the analyzed months.

---

# 3. Skills Pay

The Skills Pay analysis examines the relationship between ****Data Analyst skills and median annual salary**** in the United States.

Unlike the previous analyses, this section only considers Data Analyst positions that contain an available `salary_year_avg` value.

## Filtering the Data

```python
df_DA_US = df[
    (df['job_title_short'] == 'Data Analyst') &
    (df['job_country'] == 'United States')
].copy()

df_DA_US = df_DA_US.dropna(
    subset=['salary_year_avg']
)
```

## Preparing Skill-Level Salary Data

Since a single job posting can contain multiple skills, the skills are exploded into individual rows.

```python
df_DA_US = df_DA_US.explode('job_skills')
```

The data is then grouped by skill and aggregated using both the number of postings and the median annual salary.

```python
df_DA_US_group = df_DA_US.groupby(
    'job_skills'
)['salary_year_avg'].agg(['count', 'median'])
```

## Top-Paid Skills

The skills are sorted by median salary and the top ten are selected.

```python
df_DA_US_pay = df_DA_US_group.sort_values(
    by='median',
    ascending=False
).head(10)
```

## Most Common Skills

The ten most frequently occurring skills are also identified. These are then sorted by their median salary for comparison.

```python
df_DA_US_skills = df_DA_US_group.sort_values(
    by='count',
    ascending=False
).head(10).sort_values(
    by='median',
    ascending=False
)
```

## Salary Visualization

```python
plt.style.use('seaborn-v0_8-whitegrid')

fig, ax = plt.subplots(
    2,
    1,
    figsize=(10, 8),
    sharex=True
)

df_DA_US_pay[::-1].plot(
    kind='barh',
    y='median',
    ax=ax[0],
    legend=False,
    width=0.7
)

df_DA_US_skills[::-1].plot(
    kind='barh',
    y='median',
    ax=ax[1],
    legend=False,
    width=0.7
)

ax[0].set_title(
    'Top Paid Data Analyst Roles in the US',
    fontsize=12,
    fontweight='bold',
    pad=10
)

ax[1].set_title(
    'Top Paid Skills for Data Analysts in the US',
    fontsize=12,
    fontweight='bold',
    pad=10
)

ax[0].set_ylabel('')
ax[1].set_ylabel('')

ax[1].set_xlabel(
    'Median Salary ($)',
    fontsize=11,
    fontweight='bold',
    labelpad=10
)
```

The salary axis was then formatted into thousands of dollars.

```python
plt.draw()

ticks = ax[1].get_xticks()

ax[1].set_xticks(ticks)

ax[1].set_xticklabels(
    [f'${int(x/1000):,}k' for x in ticks]
)
```

The final chart also removes unnecessary borders and adjusts the gridlines for readability.

```python
for a in ax:
    a.spines['top'].set_visible(False)
    a.spines['right'].set_visible(False)
    a.grid(axis='x', linestyle='--', alpha=0.7)
    a.grid(axis='y', visible=False)

plt.tight_layout()
plt.show()
```

### Skills Pay Visualization

![Skills Pay Visualization](/images/Skills_Pay_Analysis.png)

---

# Tools & Technologies

* ****Python****
* ****Pandas**** — Data cleaning, transformation, grouping, pivot tables, and aggregation
* ****Matplotlib**** — Data visualization
* ****Hugging Face Datasets**** — Dataset loading
* ****AST**** — Converting stored skill strings into Python lists

---

# Conclusion

This project uses Python to analyze job-market data from multiple perspectives.

The ****EDA**** component provides a separate exploratory view of the dataset, while the main ****Data Analyst Job Market Analysis**** examines job demand, skill trends, and salary patterns.

The project demonstrates practical use of ****Pandas for data preparation and analysis**** and ****Matplotlib for creating clear visualizations from job-market data****.
