# Superstore Sales — Visualization Code

All charts below share one consistent font family and sizing scheme, set once
at the top so every chart looks the same without repeating style code.

**Consistent style used throughout:**
- Font family: `DejaVu Sans` (matplotlib default, explicit for consistency)
- Axis labels: `fontsize=12`
- Titles: `fontsize=14`
- Tick labels: `fontsize=10`
- Bar color: `skyblue`, white edge, `width=0.6`

## Setup — run this once before any chart below

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# --- Global style settings (applied to every chart in this file) ---
# Setting these once here means we don't need to repeat fontsize=... on every
# plt.xlabel / plt.ylabel / plt.title call below.
plt.rcParams['font.family'] = 'DejaVu Sans'
plt.rcParams['axes.labelsize'] = 12      # x/y axis label size
plt.rcParams['axes.titlesize'] = 14      # chart title size
plt.rcParams['xtick.labelsize'] = 10     # x-axis tick label size
plt.rcParams['ytick.labelsize'] = 10     # y-axis tick label size

# Load the cleaned dataset
file_path = 'cleaned_SuperStore_data.csv'

df = pd.read_csv(file_path)
```

---

## 1. Sales by Category

```python
# Group total Sales by Category, sorted highest to lowest
sum_sales = df.groupby('Category')['Sales'].sum().sort_values(ascending=False)

bars = plt.bar(
    x=sum_sales.index,
    height=sum_sales.values,
    color='skyblue',
    width=0.6,
    edgecolor='white'
)

# Label each bar with its dollar value
plt.bar_label(bars, padding=3, fmt='$%.2f')

# Axis labels and title
plt.xlabel('Categories')
plt.ylabel('Sales')
plt.title('Sale By Category')

# Add 15% headroom above the tallest bar so labels aren't cut off
max_val = max(sum_sales)
plt.ylim(0, max_val * 1.15)

plt.show()
```

---

## 2. Sales by Sub-Category

```python
# Group total Sales by Sub-Category (not sorted — keeps original category grouping order)
sum_sales = df.groupby('Sub-Category')['Sales'].sum()

# Wider figure since there are many sub-category labels to fit
plt.figure(figsize=(14, 7))

bars = plt.bar(
    x=sum_sales.index,
    height=sum_sales.values,
    color='skyblue',
    width=0.6,
    edgecolor='white'
)

plt.bar_label(bars, padding=3, fmt='$%.2f')

plt.xlabel('Sub-Categories', labelpad=10)
plt.ylabel('Sales')
plt.title('Sale By Sub-Category', pad=15)

# Rotate labels 45 degrees so long sub-category names don't overlap
plt.xticks(rotation=45, ha='right')

max_val = max(sum_sales)
plt.ylim(0, max_val * 1.15)

plt.show()
```

---

## 3. Sales by Region

```python
# Group total Sales by Region, sorted highest to lowest
sum_sales = df.groupby('Region')['Sales'].sum().sort_values(ascending=False)

bars = plt.bar(
    x=sum_sales.index,
    height=sum_sales.values,
    color='skyblue',
    width=0.6,
    edgecolor='white'
)

plt.bar_label(bars, padding=3, fmt='$%.2f')

plt.xlabel('Regions')
plt.ylabel('Sales')
plt.title('Sale By Region')

max_val = max(sum_sales)
plt.ylim(0, max_val * 1.15)

plt.show()
```

---

## 4. Sales by Customer Segment

```python
# Group total Sales by Segment, sorted highest to lowest
sum_sales = df.groupby('Segment')['Sales'].sum().sort_values(ascending=False)

bars = plt.bar(
    x=sum_sales.index,
    height=sum_sales.values,
    color='skyblue',
    width=0.6,
    edgecolor='white'
)

plt.bar_label(bars, padding=3, fmt='$%.2f')

plt.xlabel('Customer Segment')
plt.ylabel('Sales')
plt.title('Sale By Customer Segment')

max_val = max(sum_sales)
plt.ylim(0, max_val * 1.15)

# Force plain number formatting on the y-axis (no scientific notation)
plt.ticklabel_format(style='plain', axis='y')

plt.show()
```

---

## 5. Monthly Sales Trend (Latest 12 Months)

```python
# --- Step 1: Convert dates and filter to the most recent 12 months ---

# Ensure Order Date is a proper datetime object (needed for date math)
df['Order Date'] = pd.to_datetime(df['Order Date'])

# Find the most recent order date in the dataset dynamically
latest_date = df['Order Date'].max()

# Calculate the cutoff: exactly 12 months before the latest date
cutoff_date = latest_date - pd.DateOffset(months=12)

# Keep only rows within the last 12 months
df_filtered = df[df['Order Date'] > cutoff_date].copy()

# Shift each date to the end of its month, so all orders in the same month
# group together under one consistent date
df_filtered['Month_End_Order_Date'] = df_filtered['Order Date'] + pd.offsets.MonthEnd(0)

# Sum sales for each month
sum_sales = df_filtered.groupby('Month_End_Order_Date')['Sales'].sum()

# Sort chronologically so the trend line moves left-to-right through time
sum_sales = sum_sales.sort_index()

# Convert the datetime index into readable string labels for the x-axis
x_labels = sum_sales.index.strftime('%Y-%m-%d')

# --- Step 2: Plot ---

plt.figure(figsize=(14, 7))

bars = plt.bar(
    x=x_labels,
    height=sum_sales.values,
    color='skyblue',
    width=0.6,
    edgecolor='white'
)

# Format y-axis as dollar amounts with commas, no scientific notation
plt.gca().yaxis.set_major_formatter(plt.FuncFormatter(lambda x, p: f"${x:,.0f}"))

# Label each bar with a clean rounded dollar amount
plt.bar_label(
    bars,
    padding=4,
    labels=[f"${val:,.0f}" for val in sum_sales.values],
    fontsize=9
)

plt.xlabel('Month End Date', labelpad=10)
plt.ylabel('Sales')
plt.title('Monthly Sales Trend (Latest 12 Months)', pad=15)

# Rotate x-axis labels so month-end dates don't overlap
plt.xticks(rotation=45, ha='right')

max_val = max(sum_sales)
plt.ylim(0, max_val * 1.15)

# Prevent axis labels from being cut off at the figure edge
plt.tight_layout()

plt.show()
```

---

## 6. Shipping Delay by Ship Mode

```python
# Ensure both date columns are proper datetime objects
df['Ship Date'] = pd.to_datetime(df['Ship Date'])
df['Order Date'] = pd.to_datetime(df['Order Date'])

# Calculate the number of days between order and shipment
df['Days_To_Ship'] = (df['Ship Date'] - df['Order Date']).dt.days

# Some rows have a negative shipping time (Ship Date before Order Date),
# which is a data quality issue. Cap those at 0 rather than letting them
# distort the average.
df['Cleaned_Days_To_Ship'] = np.where(df['Days_To_Ship'] < 0, 0, df['Days_To_Ship'])

# Average shipping delay per Ship Mode, sorted highest to lowest
avg_days = df.groupby('Ship Mode')['Cleaned_Days_To_Ship'].mean().sort_values(ascending=False)

bars = plt.bar(
    x=avg_days.index,
    height=avg_days.values,
    color='skyblue',
    width=0.6,
    edgecolor='white'
)

plt.bar_label(bars, padding=3, fmt='%.2f')

plt.xlabel('Ship Mode')
plt.ylabel('Days')
plt.title('Shipping Delay by Ship Mode')

# Add headroom above bars using margins instead of a fixed ylim
plt.margins(y=0.20)

plt.show()
```

---

## 7. Top 10 Customers by Sales

```python
# Group total Sales by Customer Name, sort descending, keep top 10 only
sum_sales = df.groupby('Customer Name')['Sales'].sum().sort_values(ascending=False).head(10)

# Wider figure so all 10 customer names have room to breathe
plt.figure(figsize=(12, 6))

bars = plt.bar(
    x=sum_sales.index,
    height=sum_sales.values,
    color='skyblue',
    width=0.6,
    edgecolor='white'
)

# Clean rounded dollar labels (no cents) to save space above bars
plt.bar_label(
    bars,
    padding=4,
    labels=[f"${val:,.0f}" for val in sum_sales.values],
    fontsize=9
)

plt.xlabel('Customer')
plt.ylabel('Sales')
plt.title('Top 10 Customers by Sales', pad=15)

# Rotate customer names so long names don't overlap each other
plt.xticks(rotation=45, ha='right', fontsize=9)

# Let margins handle headroom instead of a fixed ylim
plt.margins(y=0.20)

# Prevent rotated bottom labels from being cut off at the figure edge
plt.tight_layout()

plt.show()
```

---

