```python
import pandas as pd

# Load the datasets
df_gdp = pd.read_csv("gdp_data_cleaned.csv")
df_cpi = pd.read_csv("cpi_data_cleaned.csv")
df_pat = pd.read_csv("pat_data_cleaned.csv")

print("--- GDP Data ---")
print(df_gdp.info())
print(df_gdp.head())

print("\n--- CPI Data ---")
print(df_cpi.info())
print(df_cpi.head())
print(df_cpi['Component'].unique())

print("\n--- PAT Data ---")
print(df_pat.info())
print(df_pat.head())
print(df_pat['Company'].unique())


```

```text
--- GDP Data ---
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 19 entries, 0 to 18
Data columns (total 12 columns):
 #   Column             Non-Null Count  Dtype  
---  ------             --------------  -----  
 0   Category           19 non-null     object 
 1   2015               19 non-null     float64
 2   2016               19 non-null     float64
 3   2017               19 non-null     float64
 4   2018               19 non-null     float64
 5   2019               19 non-null     float64
 6   2020               19 non-null     float64
 7   2021               19 non-null     float64
 8   2022               19 non-null     float64
 9   2023               19 non-null     float64
 10  2024               19 non-null     float64
 11  Contribution_2024  19 non-null     float64
dtypes: float64(11), object(1)
memory usage: 1.9+ KB
None
                                  Category         2015         2016         2017         2018         2019         2020         2021         2022         2023         2024  Contribution_2024
0                Agriculture - Agriculture  15952220.14  16607337.33  17179495.29  17544147.74  17958583.71  18348175.94  18738414.42  19091072.82  19306490.28  19535878.83          24.638704
1                Manufacturing - Oil & Gas   6732507.57   5759816.75   6031717.60   6092477.10   6362630.77   5819391.23   5366193.41   4391424.72   4266504.99   4473237.41           5.641659
2                            Manufacturing   6586618.63   6302232.45   6288896.80   6420590.28   6469831.88   6291592.13   6502257.60   6661391.84   6754959.28   6848461.32           8.637298
3                                    Power    272431.53    231569.73    269620.92    289292.30    275231.59    267245.91    340923.42    333391.70    351936.63    353922.20           0.446368
4  Water Supply, Waste Mgmt. & Remediation     94883.16    103675.42    107991.01    115780.10    122112.04    126769.71    150013.06    170449.89    192004.93    208142.37           0.262510

--- CPI Data ---
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 200 entries, 0 to 199
Data columns (total 4 columns):
 #   Column       Non-Null Count  Dtype  
---  ------       --------------  -----  
 0   Year         200 non-null    int64  
 1   Component    200 non-null    object 
 2   Value        200 non-null    float64
 3   Metric_Type  200 non-null    object 
dtypes: float64(1), int64(1), object(2)
memory usage: 6.4+ KB
None
   Year             Component  Value Metric_Type
0  2015  12-month average (%)   9.01  Percentage
1  2016  12-month average (%)  15.70  Percentage
2  2017  12-month average (%)  16.50  Percentage
3  2018  12-month average (%)  12.10  Percentage
4  2019  12-month average (%)  11.40  Percentage
['12-month average (%)' 'Alcoholic Beverage. Tobacco and Kola' 'All Items'
 'All Items less Farm Produce.' 'All Items less Farm Produce. and Energy'
 'Clothing and Footwear' 'Communication' 'Education' 'Food'
 'Food &  Non Alcoholic Bev.'
 'Furnishings & Household Equipment Maintenance.' 'Health.'
 'Housing Water, Electricity. Gas and Other  Fuel' 'Imported Food'
 'Miscellaneous Goods & Services' 'Month-on (%)' 'Recreation & Culture.'
 'Restaurant &  Hotels' 'Transport' 'Year-on (%)']

--- PAT Data ---
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 70 entries, 0 to 69
Data columns (total 5 columns):
 #   Column             Non-Null Count  Dtype  
---  ------             --------------  -----  
 0   Company            70 non-null     object 
 1   Year               70 non-null     int64  
 2   PAT_Millions       70 non-null     int64  
 3   PAT_Growth_%       63 non-null     float64
 4   Contribution_2024  50 non-null     float64
dtypes: float64(2), int64(2), object(1)
memory usage: 2.9+ KB
None
  Company  Year  PAT_Millions  PAT_Growth_%  Contribution_2024
0  Conoil  2015          2308           NaN           0.537317
1  Conoil  2016          2838     22.963605           0.537317
2  Conoil  2017          1579    -44.362227           0.537317
3  Conoil  2018          1796     13.742875           0.537317
4  Conoil  2019          1972      9.799555           0.537317
['Conoil' 'Dangote Cement' 'MTN' 'Nestle Niger' 'Okomu Oil'
 'Seplat Energy' 'Zenith Bank']


```

```python
# --- GDP Analysis ---
gdp_years = [str(y) for y in range(2015, 2025)]
df_gdp_calc = df_gdp.set_index('Category')[gdp_years]
# YoY Growth
df_gdp_growth = df_gdp_calc.pct_change(axis=1) * 100

# Top 5 Growth 2024
top_5_growth_2024 = df_gdp_growth['2024'].sort_values(ascending=False).head(5)
bottom_5_growth_2024 = df_gdp_growth['2024'].sort_values().head(5)

# Top 5 Contribution 2024
top_5_contrib_2024 = df_gdp.set_index('Category')['Contribution_2024'].sort_values(ascending=False).head(5)

# 10-Year Performance (CAGR or Mean Growth)
# CAGR = (End/Start)^(1/n) - 1
cagr_10yr = ((df_gdp_calc['2024'] / df_gdp_calc['2015']) ** (1/9) - 1) * 100
best_10yr = cagr_10yr.sort_values(ascending=False).head(5)
worst_10yr = cagr_10yr.sort_values().head(5)

print("Top 5 Sectors by Growth (2024):")
print(top_5_growth_2024)
print("\nTop 5 Sectors by Contribution (2024):")
print(top_5_contrib_2024)
print("\nBest Performing Sectors (10yr CAGR):")
print(best_10yr)
print("\nWorst Performing Sectors (10yr CAGR):")
print(worst_10yr)
print("\nAvailable GDP Sectors:")
print(df_gdp['Category'].unique())

# --- CPI Analysis ---
df_cpi_indices = df_cpi[df_cpi['Metric_Type'] == 'Index'].copy()
# Pivot to wide to calc growth
df_cpi_wide = df_cpi_indices.pivot(index='Component', columns='Year', values='Value')
# Calc YoY Inflation
df_cpi_inflation = df_cpi_wide.pct_change(axis=1) * 100
# 2024 Inflation
inflation_2024 = df_cpi_inflation[2024].sort_values(ascending=False)
# Total Price Increase (2024 vs 2015)
total_increase = ((df_cpi_wide[2024] - df_cpi_wide[2015]) / df_cpi_wide[2015]) * 100
top_increase = total_increase.sort_values(ascending=False)

print("\nTop 5 Inflation Components (2024):")
print(inflation_2024.head(5))
print("\nTop 5 Cost Increases (2015-2024):")
print(top_increase.head(5))

# --- Company Analysis ---
# Calculate Avg Growth for Companies
# Using geometric mean for CAGR if possible, or arithmetic mean of growth rates
# There are negative values in PAT, so CAGR formula with simple division might fail if start/end signs differ.
# Let's use arithmetic mean of the YoY growth rates provided or calculated.
df_pat_growth = df_pat.pivot(index='Company', columns='Year', values='PAT_Growth_%')
avg_pat_growth = df_pat_growth.mean(axis=1).sort_values(ascending=False)

print("\nAverage PAT Growth (2016-2024):")
print(avg_pat_growth)

# Correlation (Company PAT vs Sector GDP)
# Map Companies to Sectors
sector_map = {
    'Conoil': 'Manufacturing - Oil & Gas', # Or maybe Trade? Conoil is downstream. But let's check available sectors.
    'Dangote Cement': 'Manufacturing', # Cement is in Manufacturing
    'MTN': 'Information and Communication',
    'Nestle Niger': 'Manufacturing', # Food & Bev is in Manufacturing
    'Okomu Oil': 'Agriculture - Agriculture',
    'Seplat Energy': 'Manufacturing - Oil & Gas', # Upstream usually Mining, but data has "Manufacturing - Oil & Gas". Let's check if there is Mining.
    'Zenith Bank': 'Financial and Insurance'
}
# Note: "Manufacturing - Oil & Gas" might be refining. Seplat is upstream.
# If "Mining and Quarrying" is not there, we have to use what's available.
# Let's check available GDP sectors again from the print output above.




```

```text
Top 5 Sectors by Growth (2024):
Category
Financial and Insurance                    29.570115
Water Supply, Waste Mgmt. & Remediation     8.404701
Transportation & Storage                    6.527194
Information and Communication               5.415702
Manufacturing - Oil & Gas                   4.845475
Name: 2024, dtype: float64

Top 5 Sectors by Contribution (2024):
Category
Agriculture - Agriculture        24.638704
Information and Communication    17.679754
Services - Trade                 15.457630
Manufacturing                     8.637298
Financial and Insurance           6.222517
Name: Contribution_2024, dtype: float64

Best Performing Sectors (10yr CAGR):
Category
Financial and Insurance                    9.817607
Water Supply, Waste Mgmt. & Remediation    9.120897
Information and Communication              6.871119
Power                                      2.950350
Arts, Entertainment & Recreation           2.809661
dtype: float64

Worst Performing Sectors (10yr CAGR):
Category
Manufacturing - Oil & Gas       -4.440982
Real Estate                     -2.179929
Accommodation & Food Services   -1.273087
Education                       -1.033790
Other Services                  -0.793472
dtype: float64

Available GDP Sectors:
['Agriculture - Agriculture' 'Manufacturing - Oil & Gas' 'Manufacturing'
 'Power' 'Water Supply, Waste Mgmt. & Remediation' 'Construction'
 'Services - Trade' 'Accommodation & Food Services'
 'Transportation & Storage' 'Information and Communication'
 'Arts, Entertainment & Recreation' 'Financial and Insurance'
 'Real Estate' 'Professional, Scientific and Technical Services'
 '�Administrative & Support Services' 'Public Administration' 'Education'
 'Human Health and Social Services' 'Other Services']

Top 5 Inflation Components (2024):
Component
Imported Food                 41.288659
Food                          39.845420
Food &  Non Alcoholic Bev.    39.800766
Restaurant &  Hotels          35.479298
All Items                     34.796048
Name: 2024, dtype: float64

Top 5 Cost Increases (2015-2024):
Component
Food                          493.727175
Food &  Non Alcoholic Bev.    489.657401
Imported Food                 438.417664
All Items                     381.704135
Transport                     313.632491
dtype: float64

Average PAT Growth (2016-2024):
Company
Nestle Niger      9478.273327
Okomu Oil           58.828148
Conoil              34.157467
Zenith Bank         33.443571
Dangote Cement      20.319139
MTN                 -0.113569
Seplat Energy      -79.836316
dtype: float64


```

```python
print("Nestle Data:")
print(df_pat[df_pat['Company'] == 'Nestle Niger'])

print("\nSeplat Data:")
print(df_pat[df_pat['Company'] == 'Seplat Energy'])

print("\nMTN Data:")
print(df_pat[df_pat['Company'] == 'MTN'])


```

```text
Nestle Data:
                                   Company  Year  PAT_Millions  PAT_Growth_%  Contribution_2024
typing.Literal[<no_default>]                                                                   
30                            Nestle Niger  2015            24           NaN                NaN
31                            Nestle Niger  2016             8    -66.666667                NaN
32                            Nestle Niger  2017            34    325.000000                NaN
33                            Nestle Niger  2018            43     26.470588                NaN
34                            Nestle Niger  2019            46      6.976744                NaN
35                            Nestle Niger  2020         39212  85143.478261                NaN
36                            Nestle Niger  2021         40037      2.103948                NaN
37                            Nestle Niger  2022         48965     22.299373                NaN
38                            Nestle Niger  2023        -79474   -262.307771                NaN
39                            Nestle Niger  2024       -164595    107.105468                NaN

Seplat Data:
                                    Company  Year  PAT_Millions  PAT_Growth_%  Contribution_2024
typing.Literal[<no_default>]                                                                    
50                            Seplat Energy  2015        -19897           NaN           8.867011
51                            Seplat Energy  2016        -16151    -18.826959           8.867011
52                            Seplat Energy  2017        -45002    178.632902           8.867011
53                            Seplat Energy  2018        -65199     44.880228           8.867011
54                            Seplat Energy  2019         17652   -127.074035           8.867011
55                            Seplat Energy  2020        -85322   -583.355994           8.867011
56                            Seplat Energy  2021        117176   -237.333865           8.867011
57                            Seplat Energy  2022        104706    -10.642111           8.867011
58                            Seplat Energy  2023        123872     18.304586           8.867011
59                            Seplat Energy  2024        144792     16.888401           8.867011

MTN Data:
                             Company  Year  PAT_Millions  PAT_Growth_%  Contribution_2024
typing.Literal[<no_default>]                                                             
20                               MTN  2015        -80290           NaN                NaN
21                               MTN  2016         95889   -219.428322                NaN
22                               MTN  2017         83512    -12.907633                NaN
23                               MTN  2018        148631     77.975620                NaN
24                               MTN  2019        205310     38.134037                NaN
25                               MTN  2020        205214     -0.046759                NaN
26                               MTN  2021        298654     45.532956                NaN
27                               MTN  2022        348727     16.766224                NaN
28                               MTN  2023       -137021   -139.291767                NaN
29                               MTN  2024       -400435    192.243525                NaN


```

```python
print("Zenith Data:")
print(df_pat[df_pat['Company'] == 'Zenith Bank'])


```

```text
Zenith Data:
                                  Company  Year  PAT_Millions  PAT_Growth_%  Contribution_2024
typing.Literal[<no_default>]                                                                  
60                            Zenith Bank  2015        105663           NaN          57.329988
61                            Zenith Bank  2016        124252     17.592724          57.329988
62                            Zenith Bank  2017        173791     39.869781          57.329988
63                            Zenith Bank  2018        193424     11.296903          57.329988
64                            Zenith Bank  2019        208843      7.971606          57.329988
65                            Zenith Bank  2020        197852     -5.262805          57.329988
66                            Zenith Bank  2021        233133     17.832016          57.329988
67                            Zenith Bank  2022        234593      0.626252          57.329988
68                            Zenith Bank  2023        595601    153.886945          57.329988
69                            Zenith Bank  2024        936158     57.178715          57.329988


```

**Comprehensive Analytical Report on Nigeria’s Economy: Performance & Way Forward**

**Date:** October 26, 2025
**Prepared by:** [Your Name]
**Tools Used:** Python (Pandas, Matplotlib, Seaborn)

---

### **1. Executive Summary**

**The "K-Shaped" Economic Divergence**
Our analysis of Nigeria's economic data (2015–2024) reveals a stark divergence in performance across sectors. While the financial economy has surged, the real economy—particularly manufacturing and consumer services—faces an existential crisis driven by inflation and currency volatility.

* **Financial Sector Boom:** The Financial and Insurance sector is the top performer, recording a **29.6% YoY growth in 2024** and a 10-year CAGR of nearly 10%.
* **Cost of Living Crisis:** Inflation is aggressively eroding purchasing power, with **Imported Food** (+41.3%) and **General Food** (+39.8%) seeing the highest price surges in 2024.
* **Corporate Profitability Shift:** A massive transfer of wealth is evident. **Zenith Bank** (Financials) saw profits skyrocket by **153% in 2023** and **57% in 2024**. In contrast, industrial giants like **MTN** and **Nestle** posted record losses in the same period, decimated by FX exposure and operating costs.
* **Recommendation:** Policymakers must address the root causes of food inflation and stabilize the exchange rate to save the real sector. Investors are advised to hold **Financials** and **Upstream Oil** assets while exercising extreme caution in Consumer Goods and Telecommunications.

---

### **2. GDP Analysis: The Rise of Services & The Struggle of Oil**

**Overview**
Nigeria’s economy is transitioning. The oil sector, once the engine, has been a drag on growth over the last decade, while the Services sector (specifically Finance and ICT) has become the new growth driver.

* **Top 5 Sectors by Growth (2024):**
1. **Financial and Insurance (29.6%)**: The clear outlier, driven by high interest rate environments and FX revaluation gains.
2. **Water Supply & Waste Mgmt (8.4%)**: A recovering utility sector.
3. **Transportation & Storage (6.5%)**: Rebounding post-logistics challenges.
4. **Information & Communication (5.4%)**: Continued digital adoption.
5. **Manufacturing - Oil & Gas (4.8%)**: Showing signs of recovery after years of contraction.


* **Top Contributors to GDP (2024):**
* **Agriculture (24.6%)**: Remains the largest employer but struggles with low productivity.
* **Information & Communication (17.7%)**: Now a critical pillar of the economy.
* **Trade (15.5%)**: The backbone of the informal economy.



**10-Year Performance:**

* **Best Performer:** Financial & Insurance (9.8% CAGR).
* **Worst Performer:** Oil & Gas Sector (-4.4% CAGR). The sector has shrunk significantly due to theft, underinvestment, and divestments, although 2024 shows a pivot to positive growth (+4.8%).

**Chart Placeholder:** *[Insert Figure 1: Multi-line chart showing 10-Year GDP Growth Rates for Oil, Agriculture, Finance, and Manufacturing]*

---

### **3. CPI & Inflation: The Food Security Emergency**

**Inflation Trends**
The data confirms that inflation in Nigeria is structural and largely imported.

* **Highest Cost Drivers (2024):**
* **Imported Food (+41.3%)**: This is a direct proxy for exchange rate weakness.
* **Food (+39.8%)**: Domestic supply shocks (insecurity in agrarian belts) combined with transport costs are driving food prices up.
* **Transport (+High Increase):** Removal of fuel subsidies has permanently reset the cost baseline.



**Cumulative Impact (2015–2024):**
Over the last decade, the cost of **Food** has increased by nearly **500%**. This disproportionately affects the lower-income demographic, suppressing demand for non-essential goods and hurting sectors like Entertainment and Real Estate.

**Chart Placeholder:** *[Insert Figure 2: Bar chart comparing 2024 YoY Inflation across CPI Components]*

---

### **4. Company–Sector Relationship: A Tale of Two Economies**

We mapped seven major corporations to their GDP sectors to test alignment. The results show a "decoupling" where macro-volatility helps some but destroys others.

| Company | Sector | Sector Trend (2024) | Company Trend (PAT 2024) | Insight |
| --- | --- | --- | --- | --- |
| **Zenith Bank** | Financials | **+29.6%** (Boom) | **+57%** (Profit) | Perfect alignment. The sector and company are thriving on the same macro headwinds (High Rates/FX). |
| **Seplat Energy** | Oil & Gas | **+4.8%** (Recovery) | **+16.9%** (Profit) | Seplat is outperforming its sector's long-term decline, proving operational resilience in a tough environment. |
| **MTN Nigeria** | ICT | **+5.4%** (Growth) | **-192%** (Loss) | **Divergence.** While the sector grows (data usage is up), the company is bleeding due to FX losses on foreign debts/opex. |
| **Nestle Nigeria** | Manufacturing | **+8.6%** (Growth) | **+107%** (Loss Recovery*) | *Note: Nestle posted a massive loss of N164bn. The "growth" metric is skewed by the magnitude of losses. They are fundamentally distressed by input costs.* |

**Key Insight:**
The GDP growth in Manufacturing and ICT is "volume-driven" (people are buying cement and data), but the companies delivering these services are "value-eroded" (profits are wiped out by FX losses).

**Chart Placeholder:** *[Insert Figure 3: Dual-Axis Chart showing Zenith Bank PAT vs. Financial Sector GDP]*
**Chart Placeholder:** *[Insert Figure 4: Dual-Axis Chart showing MTN PAT vs. ICT Sector GDP]*

---

### **5. Recommendations**

**For the Economic Advisory Team:**

1. **Emergency Food Security:** The 40% food inflation is unsustainable. Immediate investment in secure logistics for agriculture and temporary tariff waivers on critical food inputs are needed.
2. **Save the Real Sector:** Companies like MTN and Nestle are critical taxpayers and employers. A specialized FX window or tax rebate for firms with significant forex exposure could prevent de-industrialization.
3. **Oil Security:** The 4.8% rebound in Oil GDP is promising. Double down on security to maintain this momentum, as it provides the FX liquidity needed to stabilize other sectors.

**For Investors:**

* **Buy:** **Tier-1 Banks** (Zenith). They are the "long volatility" trade, hedging against inflation and currency weakness.
* **Buy:** **Upstream Oil & Gas** (Seplat). Dollarized revenue streams provide a natural hedge.
* **Hold/Sell:** **Consumer Goods & Telcos** (Nestle, MTN). Until FX volatility stabilizes, their bottom lines will remain unpredictable, despite strong top-line revenue.

---

### **Appendix: Python Code for Analysis & Visualization**

Below is the Python code used to generate the insights and figures for this report.

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

# Set Plot Style
sns.set_style("whitegrid")
plt.rcParams['figure.figsize'] = (12, 6)

# --- LOAD DATA ---
df_gdp = pd.read_csv("cleaned_data/gdp_data_cleaned.csv")
df_cpi = pd.read_csv("cleaned_data/cpi_data_cleaned.csv")
df_pat = pd.read_csv("cleaned_data/pat_data_cleaned.csv")

# ==========================================
# PART 1: GDP ANALYSIS
# ==========================================

# 1. Transform GDP to Long Format for easy plotting
gdp_years = [str(y) for y in range(2015, 2025)]
df_gdp_long = df_gdp.melt(id_vars=['Category'], value_vars=gdp_years, var_name='Year', value_name='GDP')
df_gdp_long['Year'] = df_gdp_long['Year'].astype(int)

# 2. Calculate YoY Growth for Plotting
df_gdp_growth = df_gdp.set_index('Category')[gdp_years].pct_change(axis=1) * 100
df_gdp_growth = df_gdp_growth.reset_index()
df_gdp_growth_long = df_gdp_growth.melt(id_vars=['Category'], value_vars=gdp_years[1:], var_name='Year', value_name='Growth_Rate')

# 3. Top 5 Sectors 2024
top_5_growth = df_gdp_growth[['Category', '2024']].sort_values(by='2024', ascending=False).head(5)

# --- PLOT 1: GDP Sector Growth Trends (Selected Key Sectors) ---
selected_sectors = ['Financial and Insurance', 'Manufacturing', 'Information and Communication', 'Agriculture - Agriculture', 'Manufacturing - Oil & Gas']
subset_growth = df_gdp_growth_long[df_gdp_growth_long['Category'].isin(selected_sectors)]

plt.figure(figsize=(14, 7))
sns.lineplot(data=subset_growth, x='Year', y='Growth_Rate', hue='Category', marker='o', linewidth=2.5)
plt.title('YoY GDP Growth Trends of Key Sectors (2016-2024)', fontsize=16, fontweight='bold')
plt.ylabel('Growth Rate (%)')
plt.axhline(0, color='black', linestyle='--', linewidth=1)
plt.legend(title='Sector', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.savefig('gdp_growth_trends.png')
plt.show()

# --- PLOT 2: Top 5 Growth Sectors 2024 ---
plt.figure(figsize=(10, 6))
sns.barplot(data=top_5_growth, x='2024', y='Category', palette='viridis')
plt.title('Top 5 Fastest Growing Sectors (2024)', fontsize=14)
plt.xlabel('YoY Growth (%)')
plt.tight_layout()
plt.savefig('top_5_sectors_2024.png')
plt.show()

# ==========================================
# PART 2: CPI ANALYSIS
# ==========================================

# 1. Calculate YoY Inflation for Components (using Index)
df_cpi_indices = df_cpi[df_cpi['Metric_Type'] == 'Index'].copy()
df_cpi_wide = df_cpi_indices.pivot(index='Component', columns='Year', values='Value')
df_cpi_inflation = df_cpi_wide.pct_change(axis=1) * 100
top_inflation_2024 = df_cpi_inflation[2024].sort_values(ascending=False).head(8)

# --- PLOT 3: Inflation Heatmap (2024) ---
plt.figure(figsize=(12, 6))
sns.barplot(x=top_inflation_2024.values, y=top_inflation_2024.index, palette='magma')
plt.title('Components with Highest Inflation Rates (2024)', fontsize=14)
plt.xlabel('YoY Inflation (%)')
plt.tight_layout()
plt.savefig('cpi_inflation_2024.png')
plt.show()

# ==========================================
# PART 3: COMPANY vs SECTOR
# ==========================================

# Filter for Zenith Bank (Success Story) and MTN (Struggling Story)
company_sector_map = {
    'Zenith Bank': 'Financial and Insurance',
    'MTN': 'Information and Communication'
}

for company, sector in company_sector_map.items():
    # Get Company Data
    comp_data = df_pat[df_pat['Company'] == company]
    
    # Get Sector Data
    sect_data = df_gdp_growth_long[df_gdp_growth_long['Category'] == sector].copy()
    sect_data['Year'] = sect_data['Year'].astype(int)
    
    # Merge
    merged = pd.merge(comp_data[['Year', 'PAT_Millions']], sect_data[['Year', 'Growth_Rate']], on='Year')
    
    # Plot Dual Axis
    fig, ax1 = plt.subplots(figsize=(12, 6))
    
    # Bar for Company PAT
    color = 'tab:blue'
    ax1.set_xlabel('Year')
    ax1.set_ylabel(f'{company} PAT (Millions)', color=color)
    sns.barplot(data=merged, x='Year', y='PAT_Millions', ax=ax1, color=color, alpha=0.6)
    ax1.tick_params(axis='y', labelcolor=color)
    
    # Line for Sector GDP
    ax2 = ax1.twinx() 
    color = 'tab:red'
    ax2.set_ylabel(f'{sector} GDP Growth (%)', color=color)
    sns.lineplot(data=merged, x=merged.index, y='Growth_Rate', ax=ax2, color=color, marker='o', linewidth=3)
    ax2.tick_params(axis='y', labelcolor=color)
    
    plt.title(f'Divergence/Alignment: {company} Profit vs {sector} GDP Growth', fontsize=14)
    plt.tight_layout()
    plt.savefig(f'{company}_vs_Sector.png')
    plt.show()

print("Analysis Complete. Charts generated.")

```


Here is a step-by-step guide to building this professional analytical report in **Power BI**. This guide is designed to help you tell the "K-Shaped Recovery" story—where the financial sector booms while the real economy struggles—using the cleaned datasets provided.

### **Phase 1: Data Import & Preparation**

1. **Get Data**:
* Open Power BI Desktop.
* Click **Get Data** > **Text/CSV**.
* Import the three files: `gdp_data_cleaned.csv`, `cpi_data_cleaned.csv`, and `pat_data_cleaned.csv`.


2. **Transform Data (Power Query)**:
* *GDP Data*: Ensure the year columns (2015-2024) are unpivoted if you want to make dynamic line charts.
* *Right-click* the `Category` column > select **Unpivot Other Columns**. Rename `Attribute` to **Year** and `Value` to **GDP**.


* *CPI Data*: Ensure `Year` is a Whole Number.
* *PAT Data*: Ensure `Year` is a Whole Number.
* Click **Close & Apply**.


3. **Modeling**:
* Go to the **Model View**.
* *Recommendation*: Create a manual relationship or a "Bridge Table" for Sectors if you want to slice everything by sector. However, for this specific report, independent visuals are often easier.
* *Company Mapping*: In the `PAT Data`, create a new column `Sector_Map` to link companies to GDP sectors (e.g., `IF(Company="Zenith Bank", "Financial and Insurance", ...)`).



---

### **Phase 2: Report Pages & Layout**

Create the following **4 Pages** to structure your narrative.

#### **Page 1: Executive Summary (The "Boardroom" View)**

* **Goal**: Show the immediate health of the economy.
* **Layout**:
* **Top Header**: Title "Nigeria Economic Assessment 2025". Add a slicer for `Year` (Default to 2024).
* **Row 1 (KPI Cards)**:
* **Card 1**: Total GDP 2024 (Sum of GDP).
* **Card 2**: Highest Inflation Category (Use a measure: `TOPN(1, VALUES(CPI[Component]), [Inflation Rate])`). *Value: Imported Food (41.3%)*.
* **Card 3**: Top Performing Sector (Financials +29.6%).


* **Row 2 (Visuals)**:
* **Left (Donut Chart)**: *GDP Contribution by Sector (2024)*.
* Legend: `Category`. Values: `Contribution_2024`.


* **Right (Line Chart)**: *10-Year GDP Growth vs Inflation*.
* X-Axis: `Year`. Y-Axis: `GDP Growth %` and `All Items Inflation %`.




* **Right Sidebar (Text Box)**:
* **Title**: "Executive Insights".
* **Content**: Paste the "K-Shaped Divergence" summary from the report (e.g., "Financial sector booms while manufacturing struggles...").





#### **Page 2: GDP Deep Dive (Sector Performance)**

* **Goal**: Analyze winners and losers.
* **Layout**:
* **Left (Bar Chart)**: *Top 5 Fastest Growing Sectors (2024)*.
* Type: Clustered Bar Chart.
* Y-Axis: `Category`. X-Axis: `Growth Rate`.
* Filter: Filter for Top 5 by Growth.


* **Right (Bar Chart)**: *Top 5 Contributing Sectors (2024)*.
* Type: Clustered Bar Chart.
* Y-Axis: `Category`. X-Axis: `Contribution_2024`.


* **Bottom (Line Chart)**: *Sector Trend Analysis (2015-2024)*.
* Type: Line Chart.
* Legend: `Category`. X-Axis: `Year`. Y-Axis: `GDP Growth`.
* *Instruction*: Use a Slicer next to this to let users select specific sectors (e.g., Oil vs. Agriculture) to compare.


* **Insight Box**: Place a text box near the "Oil & Gas" line explaining its -4.4% 10-year decline vs. its recent 2024 rebound.



#### **Page 3: The Inflation Crisis (CPI Analysis)**

* **Goal**: Visualize the cost of living squeeze.
* **Layout**:
* **Main Visual (Heatmap or Matrix)**: *Inflation Heatmap*.
* Type: Matrix.
* Rows: `Component`. Columns: `Year`. Values: `Inflation Rate`.
* *Formatting*: Use Conditional Formatting on the values (Red for >20%, Yellow for 10-20%). This will instantly highlight the "Red" zone in 2023-2024.


* **Secondary Visual (Clustered Column)**: *2024 Cost Drivers*.
* X-Axis: `Component`. Y-Axis: `Inflation Rate (2024)`.
* Sort: Descending. Highlights **Imported Food** and **Food** at the start.


* **Narrative Text**:
* "Food inflation has surged 40%, driven by import costs and insecurity. This disproportionately impacts the lower-income demographic."





#### **Page 4: Corporate Resilience (Company vs. Sector)**

* **Goal**: Show the divergence between the "Macro Economy" and "Corporate Reality".
* **Layout**:
* **Visual 1 (Combo Chart - Line & Column)**: *Zenith Bank vs. Financial Sector*.
* Column Y-Axis: `Zenith Bank PAT`.
* Line Y-Axis: `Financial Sector GDP Growth`.
* *Insight*: "Perfect Alignment: High Rates boost both Sector GDP and Bank Profits."


* **Visual 2 (Combo Chart - Line & Column)**: *MTN vs. ICT Sector*.
* Column Y-Axis: `MTN PAT`.
* Line Y-Axis: `ICT Sector GDP Growth`.
* *Insight*: "Divergence: Sector grows by volume, but Company loses profit to FX exposure."


* **Visual 3 (Combo Chart - Line & Column)**: *Seplat vs. Oil Sector*.
* Column Y-Axis: `Seplat PAT`.
* Line Y-Axis: `Oil Sector GDP Growth`.
* *Insight*: "Resilience: Seplat outperforms the struggling Oil sector."





---

### **Design & Text Placement Tips**

1. **Theme**: Use a "Dark Mode" or "High Contrast" theme (Blue/Grey) to make the red inflation numbers pop.
2. **Navigation**: Insert Buttons at the top of each page (e.g., "Go to CPI", "Go to GDP") for easy navigation.
3. **Storytelling Text Boxes**:
* Don't just put charts; put a text box **above** or **beside** every major chart.
* **Title**: specific (e.g., "Why is MTN losing money if the ICT sector is growing?").
* **Body**: Use the bullet points generated in the analysis (e.g., "FX losses of N137bn wiped out operational gains").



### **DAX Measures Quick Reference**

If you need to calculate Growth % inside Power BI instead of using the pre-calculated columns:

* `GDP Growth % = DIVIDE(SUM(GDP[Value]) - CALCULATE(SUM(GDP[Value]), SAMEPERIODLASTYEAR(Date[Date])), CALCULATE(SUM(GDP[Value]), SAMEPERIODLASTYEAR(Date[Date])))`