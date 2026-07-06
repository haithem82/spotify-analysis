# Spotify Quarterly Analysis (2017–2022)

## Overview
This project analyzes trends in **user growth**, **premium user growth**, **company profitability**, and **revenue growth**. The goal is to provide data‑driven recommendations for future strategy – whether to focus on user base expansion, pricing, or feature differentiation.

## Tools
- Python (Pandas, Matplotlib, Seaborn)
- Data: `Spotify Quarterly.csv`
- Visualization: Matplotlib / Seaborn (can be adapted for Power BI)

## Objectives
1. Clean the CSV and prepare it for analysis.
2. Explore MAU, revenue, and ARPU trends.
3. Compare premium vs. ad‑supported user behaviour.
4. Calculate quarterly growth rates and profitability margins.
5. Identify patterns and provide actionable business insights.

```python
# Import required libraries
import os
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib.dates as mdates

# Set style for plots
sns.set_style("whitegrid")
plt.rcParams['figure.figsize'] = (12, 6)
```

## 1. Load and Clean the Data

```python
# Load the dataset (update the path if needed)
file_path = os.path.join(os.getcwd(), "Spotify Quarterly.csv")
df = pd.read_csv(file_path)

# The last row contains only nulls → drop it
df = df.dropna()

# Convert Date column to datetime and sort
df["Date"] = pd.to_datetime(df["Date"], dayfirst=True)
df = df.sort_values("Date").reset_index(drop=True)

# Convert numeric columns to appropriate types
# Columns 1..12 (index 1 to 12) are integers
for col in df.columns[1:13]:
    df[col] = df[col].astype(int)

# Column 13 (Premium ARPU) is float
df[df.columns[13]] = df[df.columns[13]].astype(float)

# Columns 14 onward are integers
for col in df.columns[14:]:
    df[col] = df[col].astype(int)

print("Data types after conversion:")
print(df.dtypes)
print("\nFirst 5 rows:")
df.head()
```

## 2. Feature Engineering

### ARPU (Average Revenue Per User)
- **Ad ARPU** = Ad Revenue / Ad MAUs  
- **Premium ARPU** = Premium Revenue / Premium MAUs  

### MAU Composition
- **AdRatio MAU** = Ad MAUs / Total MAUs  
- **PremiumRatio MAU** = Premium MAUs / Total MAUs

```python
# ARPU calculations
df["Ad ARPU"] = round(df["Ad Revenue"] / df["Ad MAUs"], 2)
df["Premium ARPU"] = round(df["Premium Revenue"] / df["Premium MAUs"], 2)

# MAU composition (as percentages)
df["AdRatio MAU"] = round((df["Ad MAUs"] / df["MAUs"]) * 100, 1)
df["PremiumRatio MAU"] = round((df["Premium MAUs"] / df["MAUs"]) * 100, 1)

print("New columns added:")
print(df[["Date", "Ad ARPU", "Premium ARPU", "AdRatio MAU", "PremiumRatio MAU"]].head())
```

## 3. Visualizations & Initial Observations

### 3.1 ARPU Trends

```python
plt.figure(figsize=(12, 5))
sns.lineplot(x="Date", y="Ad ARPU", data=df, label="Ad ARPU", marker='o')
sns.lineplot(x="Date", y="Premium ARPU", data=df, label="Premium ARPU", marker='s')
plt.title("Average Revenue Per User (ARPU): Ad-Supported vs. Premium")
plt.xlabel("Quarter")
plt.ylabel("ARPU (€)")
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

**Observation:** Both Ad ARPU and Premium ARPU show a clear decline starting around 2023. The drop is more pronounced for Premium ARPU, indicating that monetisation per user is weakening – possibly due to pricing pressure or a perceived narrowing of the gap between free and premium tiers.

### 3.2 MAU Composition (Premium vs. Ad-Supported)

```python
plt.figure(figsize=(12, 5))
sns.lineplot(x="Date", y="PremiumRatio MAU", data=df, label="Premium % of MAU", marker='o', color='green')
sns.lineplot(x="Date", y="AdRatio MAU", data=df, label="Ad‑Supported % of MAU", marker='^', color='orange')
plt.title("MAU Composition: Premium vs. Ad-Supported Users")
plt.xlabel("Quarter")
plt.ylabel("Percentage of Total MAU")
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

**Observation:** There is a sharp decline in the proportion of Premium MAUs from 2023 onward, while the ad‑supported share increases. This suggests that users are either downgrading or new users prefer the free tier. Possible causes:
- Premium price is too high relative to competitors.
- Feature gap between free and premium has narrowed.
- Competitors (Apple Music, Amazon Music, YouTube Music) offer better value.

**Recommended actions:** 1. Survey users to understand why they churn from premium or never upgrade.
2. Experiment with free trial conversions and targeted discounts.
3. Introduce innovative, exclusive features for premium subscribers (e.g., concert ticket priority, early access to new releases).

## 4. Quarterly Growth Analysis (Year-over-Year)

We calculate the **year‑over‑year quarterly growth** (same quarter of previous year) for MAUs, Premium MAUs, Total Revenue, and Premium Revenue.

```python
# Year-over-year quarterly growth (periods=4 because quarterly data)
df["quarterly MAU growth"] = round(df["MAUs"].pct_change(periods=4) * 100, 1)
df["quarterly PremMAU growth"] = round(df["Premium MAUs"].pct_change(periods=4) * 100, 1)
df["quarterly REV growth"] = round(df["Total Revenue"].pct_change(periods=4) * 100, 1)
df["quarterly PremREV growth"] = round(df["Premium Revenue"].pct_change(periods=4) * 100, 1)

# Display the growth columns
df[["Date", "quarterly MAU growth", "quarterly PremMAU growth", 
    "quarterly REV growth", "quarterly PremREV growth"]].tail(10)
```

## 5. Profitability Analysis

We compute **Gross Profit Margin** (Total) and **Premium Gross Profit Margin**.

```python
df["Profitability"] = round((df["Gross Profit"] / df["Total Revenue"]) * 100, 1)
df["Prem Profitability"] = round((df["Premium Gross Profit"] / df["Premium Revenue"]) * 100, 1)

plt.figure(figsize=(12, 5))
sns.lineplot(x="Date", y="Profitability", data=df, label="Total Gross Margin", marker='o')
sns.lineplot(x="Date", y="Prem Profitability", data=df, label="Premium Gross Margin", marker='s')
plt.title("Gross Profit Margins (Total vs. Premium)")
plt.xlabel("Quarter")
plt.ylabel("Margin (%)")
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

**Insight:** Since 2023, both total and premium profitability have stagnated at around **27%**. This suggests a **margin ceiling** – revenue increases are matched by proportional increases in cost of revenue.

**Suggested courses of action:**
- **Reduce cost of revenue** – negotiate more favourable licensing deals (especially with major labels).
- **Increase price optimisation** – use data‑driven pricing experiments to raise ARPU without losing subscribers.
- **Introduce high‑margin offerings** – e.g., bundled services (Hulu, audiobooks), concert ticket access for premium users, or exclusive podcast content.

## 6. Export Cleaned Dataset (Optional)

If you want to use the cleaned data with Power BI or another tool, uncomment the lines below.

```python
#export_path = os.path.join(os.getcwd(), "spotify_clean.csv")
# df.to_csv(export_path, index=False)
# print(f"Cleaned dataset exported to {export_path}")
```

## 7. Conclusion & Recommendations

The data indicates a strategic shift around 2020 from **user‑growth centric** to **revenue‑optimising centric**. This led to:
- Slower MAU growth but stabilised / slightly growing revenue.
- Profitability plateau at ~27%, indicating limited user base expansion compared to earlier years.

### To optimise both user engagement and profitability, we recommend:

1. **Reignite Premium growth** - Run A/B tests on pricing and feature bundles.  
    - Launch a “Premium Lite” tier in price‑sensitive markets.  

2. **Increase ARPU** - Improve ad‑targeting technology to boost Ad ARPU.  
    - Introduce annual prepaid plans with a discount to lock in users.  

3. **Break the margin ceiling** - Invest in original content (podcasts, audiobooks) that has better margin than licensed music.  
    - Automate royalty reporting and reduce operational overhead.  

4. **Monitor competitive moves closely** – the decline in premium share aligns with aggressive offers from Apple Music and YouTube Music. Consider partnering with telecom carriers to subsidise premium subscriptions.
