---
title: Turning Market Sentiment into Data - Developing Fear and Greed Index (Part 2)
post_excerpt: In the second part of this article, we will walk through the calculations for each individual index and discuss the modeling approach for deriving the final Fear and Greed Index for the Indonesian stock market.
taxonomy:
    category:
        - internal-guides
        - notes
---

# A Step by Step Guide to Market Sentiment Analysis - Engineering Fear and Greed Index (Part 2)

In the first part of this article, we examined the concept of the Fear and Greed Index, its mathematical foundation, and the data collection process. We identified eight indices—comprising both direct stock market indicators and indirect macroeconomic factors—that contribute to the overall sentiment index for the Indonesian stock market. With the data now organized into dataframes, we are ready to proceed with calculating each individual index. In this article, we will compute these indices and explore the final model for the overall Fear and Greed Index using a weighted average approach. Note that calculations involving external data sources are based on saved CSV files; the backend data processing pipeline will be covered in a separate article.

![CNN-Fear-Greed-Example](/_images/cnn-f&g-example.png)

### Market Momentum Index - IDX Composite Momentum 

The [Market Momentum Index](https://www.investopedia.com/terms/m/marketmomentum.asp) measures the strength and direction of recent price movements in a stock market, indicating whether the market is trending upward or downward. It’s typically calculated using the rate of change in an index or stock prices over a set period, such as the difference in the IDX Composite over the past week or month. In the context of the Fear and Greed Index, high market momentum reflects investor optimism or “greed,” while negative or weak momentum signals caution or “fear” among investors, as it suggests a potential market downturn.

```python 

# Create a copy from API dataframe result generated from SectorsAPI
daily_data_df_momentum = df_history_idx30.copy()

# Set up SMA period and min max scaling range 
sma_period=7
min_momentum=-5
max_momentum=5

# Calculate the percentage difference from the SMA & handle zero division error 
epsilon = 1e-9
daily_data_df_momentum['momentum_sma'] = ((daily_data_df_momentum['close'] - daily_data_df_momentum['sma']) / (daily_data_df_momentum['sma'] + epsilon)) * 100

# Remove any NaNs in case any
daily_data_df_momentum = daily_data_df_momentum.dropna(subset=['momentum_sma'])

# Scale the momentum to a 0-100 range for the Fear and Greed Index
def scale_to_100(value, min_val, max_val):

  scaled_value = (value - min_val) / (max_val - min_val) * 100

  return np.clip(scaled_value, 0, 100)

# Generate scaled market momentum index
daily_data_df_momentum['momentum_scaled'] = daily_data_df_momentum['momentum_sma'].apply(lambda x: scale_to_100(x, min_momentum, max_momentum))

# Calculate the market momentum Index on a daily basis
daily_momentum_index = daily_data_df_momentum.groupby('date')['momentum_scaled'].mean().reset_index()

# Print today's market momentum fear and greed index 
print('Today´s Market Momentum Fear and Greed index is: ', daily_momentum_index.iloc[-1,-1])

```
From the code above we will have the result of daily market momentum fear and greed index and we can easily access today's market momentum fear and greed index through the iloc function. 

```json

	date	momentum_scaled
0	2024-08-07	50.000000
1	2024-08-08	49.692673
2	2024-08-09	53.665698
3	2024-08-12	60.751728
4	2024-08-13	63.810015
...	...	...
58	2024-10-29	30.791101
59	2024-10-30	31.420214
60	2024-10-31	34.289018
61	2024-11-01	27.821455
62	2024-11-04	31.211632
```

We can plot the results using 50 as a threshold: values below 50 indicate fear, while values above 50 signify greed. Here we will show the visualization using market momentum index result as an example. The techniques to plot rest of the index is the same. 

```python 
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

fig, ax = plt.subplots(figsize=(12, 6))
ax.plot(daily_momentum_index['date'], daily_momentum_index['momentum_scaled'], color='blue', linewidth=2, label='Market Momentum Index')

# Highlight the "Fear" and "Greed" zones
ax.fill_between(daily_momentum_index['date'], 0, 50, color='red', alpha=0.2, label='Fear Zone')
ax.fill_between(daily_momentum_index['date'], 50, 100, color='green', alpha=0.2, label='Greed Zone')

# Formatting the x-axis
ax.xaxis.set_major_formatter(mdates.DateFormatter('%b %d'))
ax.xaxis.set_major_locator(mdates.WeekdayLocator(interval=1))
plt.xticks(rotation=45)

# Adding labels and title
plt.title("Market Momentum: Fear and Greed Index Over Time", fontsize=16, fontweight='bold')
plt.xlabel("Date", fontsize=12)
plt.ylabel("Momentum Scaled Index", fontsize=12)


plt.grid(visible=True, linestyle='--', alpha=0.6)
plt.legend()
plt.tight_layout()
plt.show()
```
![market-momentum-result](/_images/mm_fear_greed.png)

### Stock Price Strength

The [Stock Price Strength Index](https://www.investopedia.com/articles/active-trading/042114/overbought-or-oversold-use-relative-strength-index-find-out.asp) reflects the proportion of stocks trading above their moving averages, such as the 50-day or 200-day average, indicating the overall health of the market. It’s calculated by dividing the number of stocks above their moving average by the total number of stocks in the index. In the context of the Fear and Greed Index, a high price strength suggests optimism or “greed” as more stocks are performing well, while a low value indicates “fear,” showing that a majority of stocks are trending downward.

```python 
# Create a copy of the original DataFrame
df_copy_ps = df_history_idx30.copy()

# Calculate the daily percentage change in 'close' price for each symbol on the copy
df_copy_ps['daily_return'] = df_copy_ps.groupby('symbol')['close'].pct_change() * 100

# Calculate the 7-day SMA of the daily returns on the copy
df_copy_ps['sma_7d_return'] = df_copy_ps.groupby('symbol')['daily_return'].transform(lambda x: x.rolling(window=7).mean())

# Handle zero division by replacing any infinite or NaN values in 'sma_7d_return' with zeros
df_copy_ps['sma_7d_return'].replace([np.inf, -np.inf], np.nan, inplace=True)
df_copy_ps['sma_7d_return'].fillna(0, inplace=True)

# Calculate the min and max of the SMA to normalize
min_sma = df_copy_ps['sma_7d_return'].min()
max_sma = df_copy_ps['sma_7d_return'].max()

# Avoid division by zero in scaling
if max_sma - min_sma == 0:
    df_copy_ps['scaled_strength_index'] = 0
else:
    df_copy_ps['scaled_strength_index'] = ((df_copy_ps['sma_7d_return'] - min_sma) / (max_sma - min_sma)) * 100

df_copy_ps[['symbol', 'date', 'scaled_strength_index']].dropna()

# Group by 'date' and calculate the mean of 'scaled_strength_index' for each day
daily_mean_strength_index = df_copy_ps.groupby('date')['scaled_strength_index'].mean().reset_index()

print ('Today´s Stock Price Strength Fear and Greed index is: ', daily_mean_strength_index.iloc[-1,-1])

```
After running this code, we should see this result below. 

```json
date	scaled_strength_index
0	2024-08-07	42.589840
1	2024-08-08	42.589840
2	2024-08-09	42.589840
3	2024-08-12	42.589840
4	2024-08-13	42.589840
...	...	...
58	2024-10-29	35.441381
59	2024-10-30	33.525886
60	2024-10-31	32.744771
61	2024-11-01	29.398301
62	2024-11-04	30.021671
```

### Stock Price Strength

The [Volatility Index](https://www.investopedia.com/terms/v/vix.asp) measures the degree of variation in stock prices over a specified period, often using the standard deviation of daily returns to quantify market uncertainty. It’s calculated by analyzing recent price fluctuations, typically over the past 30 days. In the context of the Fear and Greed Index, high volatility reflects “fear” as investors react unpredictably, while low volatility suggests “greed,” indicating a stable or confident market sentiment.

```python 
# Create a copy of the original DataFrame
df_copy_vol = df_history_idx30.copy()

# Calculate the daily percentage change in 'close' price for each symbol
df_copy_vol['daily_return'] = df_copy_vol.groupby('symbol')['close'].pct_change()

# Calculate the 7-day rolling standard deviation (volatility) of the daily returns for each symbol
df_copy_vol['volatility_7d'] = df_copy_vol.groupby('symbol')['daily_return'].transform(lambda x: x.rolling(window=7).std())

# Handle zero division by replacing any infinite or NaN values in 'volatility_7d' with zeros
df_copy_vol['volatility_7d'].replace([np.inf, -np.inf], np.nan, inplace=True)
df_copy_vol['volatility_7d'].fillna(0, inplace=True)

# Scale the volatility on a 0 to 100 scale for the fear and greed index context
min_vol = df_copy_vol['volatility_7d'].min()
max_vol = df_copy_vol['volatility_7d'].max()

# Avoid division by zero in scaling
if max_vol - min_vol == 0:
    df_copy_vol['scaled_volatility_index'] = 0
else:
    df_copy_vol['scaled_volatility_index'] = ((df_copy_vol['volatility_7d'] - min_vol) / (max_vol - min_vol)) * 100

# Handle boundary values
def transform_extremes_to_mean(df):


    mean_value = df[(df['scaled_volatility_index'] != 0) & (df['scaled_volatility_index'] != 100)]['scaled_volatility_index'].mean()

    df['scaled_volatility_index'] = df['scaled_volatility_index'].replace([0, 100], mean_value)

    return df


df_copy_vol_transformed = transform_extremes_to_mean(df_copy_vol)

print(df_copy_vol_transformed)
print('Today´s Volatility Fear and Greed index is: ', df_copy_vol_transformed.iloc[-1,-1])
```
Upon running the code above, we will see this result. 

```json 
date  scaled_volatility_index
0  2024-08-07                15.320666
1  2024-08-08                15.320666
2  2024-08-09                15.320666
3  2024-08-12                15.320666
4  2024-08-13                15.320666
..        ...                      ...
58 2024-10-29                14.162872
59 2024-10-30                13.904232
60 2024-10-31                14.530508
61 2024-11-01                16.206552
62 2024-11-04                16.469431
```
### Volume Breadth

The [Volume Breadth Index](https://www.investopedia.com/terms/b/breadthindicator.asp) assesses the strength of market participation by comparing the trading volume of advancing stocks to that of declining stocks. It’s calculated by taking the difference between the volume of advancing and declining stocks, often expressed as a ratio or percentage. In the context of the Fear and Greed Index, high volume breadth reflects “greed,” showing broad investor confidence, while low or negative volume breadth indicates “fear,” as fewer stocks experience strong trading activity.

```python 
# Create a copy of the dataframe result generated from SectorsAPI
df_vb_copy = df_history_idx30.copy()

# Identify advancing and declining stocks
# Calculate whether the stock is advancing or declining based on the 'close' price
df_vb_copy['advancing'] = df_vb_copy.groupby('symbol')['close'].diff() > 0
df_vb_copy['declining'] = df_vb_copy.groupby('symbol')['close'].diff() < 0

# Calculate total volume of advancing and declining stocks for each date
# Advancing volume
df_vb_copy['advancing_volume'] = df_vb_copy['volume'] * df_vb_copy['advancing']
# Declining volume
df_vb_copy['declining_volume'] = df_vb_copy['volume'] * df_vb_copy['declining']

# Group by date to get total advancing and declining volumes for each day
daily_volume = df_vb_copy.groupby('date').agg({
    'advancing_volume': 'sum',
    'declining_volume': 'sum'
}).reset_index()

# Calculate Volume Breadth
# Avoid division by zero by handling cases where declining volume is zero
daily_volume['volume_breadth'] = daily_volume.apply(
    lambda row: row['advancing_volume'] / row['declining_volume'] if row['declining_volume'] != 0 else np.nan,axis=1
)

# Apply the 7-day SMA to the Volume Breadth to smooth the values
daily_volume['sma_7d_vb'] = daily_volume['volume_breadth'].rolling(window=7).mean()

# Handle NaN or infinite values in the SMA with zeros (for stability)
daily_volume['sma_7d_vb'].replace([np.inf, -np.inf], np.nan, inplace=True)
daily_volume['sma_7d_vb'].fillna(0, inplace=True)

# Scale the Volume Breadth for the fear and greed index context
# Min and max for scaling
min_vb = daily_volume['sma_7d_vb'].min()
max_vb = daily_volume['sma_7d_vb'].max()

# Scale between 0 and 100, with protection against division by zero
if max_vb - min_vb == 0:
    daily_volume['scaled_vb'] = 0
else:
    daily_volume['scaled_vb'] = ((daily_volume['sma_7d_vb'] - min_vb) / (max_vb - min_vb)) * 100

# Replace 0 and 100 values with the mean
mean_scaled_vb = daily_volume['scaled_vb'].mean()
daily_volume['scaled_vb'] = daily_volume['scaled_vb'].apply(lambda x: mean_scaled_vb if x == 0 or x == 100 else x)

daily_volume[['date', 'scaled_vb']]
```

After running this code, we will see this result below. 

```json 
date	scaled_vb
0	2024-08-07	27.120935
1	2024-08-08	27.120935
2	2024-08-09	27.120935
3	2024-08-12	27.120935
4	2024-08-13	27.120935
...	...	...
58	2024-10-29	17.862489
59	2024-10-30	15.629806
60	2024-10-31	8.794240
61	2024-11-01	5.512156
62	2024-11-04	5.372818
```
Different from visualizing with only the matplotlib library, here we used seaborn to create a clean, grid-style background with sns.set(style="whitegrid"), which enhances readability and presentation. This approach helps to visually separate the “Fear” and “Greed” zones with shaded areas, making the sentiment shifts in the Volume Breadth Index easier to interpret.

```python 
sns.set(style="whitegrid")
fig, ax = plt.subplots(figsize=(12, 6))

# Plot the daily mean volume breadth
ax.plot(daily_volume['date'], daily_volume['scaled_vb'], color='blue', linewidth=2, label='Mean Volume Breadth Index')

# Highlight the "Fear" and "Greed" zones
ax.fill_between(daily_volume['date'], 0, 50, color='red', alpha=0.2, label='Fear Zone')
ax.fill_between(daily_volume['date'], 50, 100, color='green', alpha=0.2, label='Greed Zone')

# Show dates on a weekly basis
ax.xaxis.set_major_formatter(mdates.DateFormatter('%b %d'))
ax.xaxis.set_major_locator(mdates.WeekdayLocator(interval=1))  # Show weekly dates
plt.xticks(rotation=45)


plt.title("Daily Mean Volume Breadth Index - Fear and Greed Context", fontsize=16, fontweight='bold')
plt.xlabel("Date", fontsize=12)
plt.ylabel("Scaled Volume Breadth Index (0-100)", fontsize=12)
plt.grid(visible=True, linestyle='--', alpha=0.6)
plt.legend(loc='upper left')
plt.tight_layout()
plt.show()
```
![volume-breadth](/_images/volume_breadth.png)

### Safe Heaven Demand

The [Safe Haven Demand Index](https://www.investopedia.com/terms/s/safe-haven.asp#:~:text=A%20safe%20haven%20is%20a,the%20event%20of%20market%20downturns.) measures investors’ preference for safer assets, like gold or government bonds, over stocks, indicating risk aversion. It’s calculated by tracking the price changes or trading volumes of these safe-haven assets compared to the broader stock market. In the context of the Fear and Greed Index, high demand for safe havens reflects “fear,” as investors shift away from stocks, while low demand suggests “greed,” indicating a stronger appetite for risk and confidence in the market.

```python 
# Create a copy of API generated dataframe 
df_idx_shd = df_history_idx30.copy()

# Calculate average daily stock return
average_daily_return = df_idx_shd.groupby('date')['stock_return'].mean().reset_index()
average_daily_return.rename(columns={'stock_return': 'average_stock_return'}, inplace=True)

# import and load government bonds data
gov_bond_df = pd.read_csv('/content/temp_bonds_rate.csv')

# Ensure the dates columne are date format
average_daily_return['date'] = pd.to_datetime(average_daily_return['date'])
gov_bond_df['date'] = pd.to_datetime(gov_bond_df['date'])

# Perform inner join based on common dates
merged_data = pd.merge(average_daily_return, gov_bond_df, on='date', how='inner')

# Store the columns we need for calculation in one dataframe
merged_data

# Calculate the 7-day SMA for average_stock_return and rate
sma_period = 7
merged_data['sma_stock'] = merged_data['average_stock_return'].rolling(window=sma_period, min_periods=1).mean()
merged_data['sma_rate'] = merged_data['rate'].rolling(window=sma_period, min_periods=1).mean()

# Compute percentage distance from SMA for both stock returns and rates
merged_data['safe_haven_index'] = (
    (merged_data['average_stock_return'] - merged_data['sma_stock']) / (merged_data['sma_stock'] + epsilon) -
    (merged_data['rate'] - merged_data['sma_rate']) / (merged_data['sma_rate'] + epsilon)
) * 100

# Scale the Safe Haven Demand Index to 0-100 for Fear and Greed context
min_val, max_val = -5, 5  # Assume min/max range for scaling to 0-100
def scale_to_100(value, min_val, max_val):
    scaled_value = (value - min_val) / (max_val - min_val) * 100
    return np.clip(scaled_value, 0, 100)

# Replace any out-of-bounds values with the mean of the scaled index
mean_value = merged_data['safe_haven_scaled'].mean()
merged_data['safe_haven_scaled'] = merged_data['safe_haven_scaled'].apply(lambda x: mean_value if x < 0 or x > 100 else x)

merged_data[['date', 'average_stock_return', 'rate', 'safe_haven_scaled']]
```

The code outputs a dataframe that contains the average stock return, government bond rate, and the scaled Safe Haven Demand Index.

```json
date	average_stock_return	rate	safe_haven_scaled
0	2024-08-07	0.067625	6.806	50.000000
1	2024-08-08	-0.048310	6.775	54.205938
2	2024-08-09	0.587302	6.767	54.205938
3	2024-08-12	1.078921	6.800	54.205938
4	2024-08-13	0.656614	6.778	54.205938
...	...	...	...	...
58	2024-10-29	-0.576492	6.856	54.205938
59	2024-10-30	-0.527557	6.829	64.974123
60	2024-10-31	-0.105976	6.777	54.205938
61	2024-11-01	-1.487444	6.774	54.205938
62	2024-11-04	-0.582527	6.767	54.205938
```

### Exchange Rate and Interest Rate

The Exchange Rate measures the relative strength of the Indonesian Rupiah (IDR) against foreign currencies, particularly the US Dollar (USD), providing insights into economic stability and investor confidence. It’s calculated by tracking daily fluctuations in the IDR-USD exchange rate. In the context of the Fear and Greed Index, a weakening IDR indicates “fear,” as investors may perceive increased economic risk, while a strengthening IDR suggests “greed,” reflecting economic stability and greater confidence in the market.

The Interest Rate reflects the central bank’s monetary policy stance and its influence on borrowing costs, economic growth, and inflation. It’s measured by tracking the daily policy rate set by Bank Indonesia. In the Fear and Greed Index, higher interest rates signal “fear,” as increased borrowing costs can dampen economic growth and market optimism. Conversely, lower interest rates reflect “greed,” encouraging investment and boosting confidence in the stock market.

To engineer the Fear and Greed Index for both Exchange Rate and Interest Rate, we analyze their trends over a fixed period by examining percentage changes that signal shifts in investor sentiment. For the exchange rate, we calculate the percentage change in the IDR-USD rate over time, with a weakening IDR suggesting “fear” and a strengthening IDR indicating “greed.” For the interest rate, which is typically issued monthly, we transform it into daily data by assigning the same rate for each day within the month. A rise in interest rates reflects “fear,” as higher borrowing costs can impact economic growth, while a decrease signals “greed,” encouraging investment. Both indicators are then scaled to a standardized 0-100 range for integration into the overall Fear and Greed Index, contributing to a comprehensive measure of investor sentiment.

Here is an example using exchange rate. 

```python 
# load exchange rate csv file
er_df = pd.read_csv('/content/exchange_rate.csv')

# Apply SMA method
er_df['SMA_7'] = er_df['rate'].rolling(window=7).mean()

# Handle missing value
er_df['SMA_7'].fillna(er_df['SMA_7'].mean(), inplace=True)

# Scale the value from 0 to 100
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler(feature_range=(0, 100))
er_df['scaled_fear_greed'] = scaler.fit_transform(er_df[['SMA_7']])

#Check today's exchange rate fear and greed index
print('Today´s Fear and Greed Index for Exchange Rate is: ', er_df.iloc[-1,-1])
```

### Buffett Indicator

To engineer the Fear and Greed Index for the [Buffett Indicator](https://www.longtermtrends.net/market-cap-to-gdp-the-buffett-indicator/), we calculate the Buffett Indicator as the ratio of the stock market’s total market capitalization to Indonesia’s GDP, a commonly used measure to assess market valuation relative to the overall economy. A high Buffett Indicator suggests that the market is overvalued compared to economic output, signaling “fear” due to increased caution and risk of a potential correction. Conversely, a low Buffett Indicator implies the market is undervalued, indicating “greed” as stocks appear favorable relative to economic fundamentals. Since GDP is typically reported quarterly, we convert it to a daily measure by carrying forward the latest GDP figure until the next release, using Indonesia’s [projected 2024 GDP of $1.49 trillion](https://en.wikipedia.org/wiki/Economy_of_Indonesia#:~:text=%2B%243.46%20billion%20(2021%20est.)&text=%E2%88%920.89%25%20(of%20GDP)%20(2024%20est.)&text=%24182.1%20billion%20(2024%20est.)) as a baseline. The Buffett Indicator ratio is then normalized to a 0-100 scale to align with the broader Fear and Greed Index, ensuring it accurately contributes to the overall sentiment measure.

```python 
# Select useful input column 
market_cap_df = market_cap_df[['date', 'market_cap']]

# Indonesian GDP for 2024 in Indonesian Rupiah (IDR)
# Using an approximate exchange rate: 1 USD = 15,000 IDR
usd_to_idr_exchange_rate = 15000
indonesia_gdp_idr = 1.47 * 10**12 * usd_to_idr_exchange_rate  # 1.47 trillion USD to IDR

# Calculate Buffett Indicator
market_cap_df['buffett_indicator'] = (market_cap_df['market_cap'] / indonesia_gdp_idr) * 100

# Drop rows with NaN values in the 'buffett_indicator' column
market_cap_df = market_cap_df.dropna(subset=['buffett_indicator'])

# Use MinMaxScaler to scale the Buffett Indicator values between 0 and 100
scaler = MinMaxScaler(feature_range=(0, 100))
market_cap_df['buffett_indicator_scaled'] = scaler.fit_transform(market_cap_df[['buffett_indicator']])

market_cap_df[['date', 'buffett_indicator_scaled']]
```
### Modelling for Final Indonesian Fear and Greed Index 

Now that we have calculated all eight individual indices for the Fear and Greed Index, we have consolidated these results into a single dataframe, allowing us to model the final combined index. Our goal is to determine the weight or contribution of each index to the overall sentiment score, capturing both explicit contributions and any latent patterns among the indices. By identifying these weights, we aim to quantify how each index—such as market momentum, exchange rate, or safe haven demand—uniquely influences the Indonesian stock market’s sentiment landscape. This modeling step is critical to ensure that the final Fear and Greed Index accurately reflects the combined impact of diverse economic and market indicators.

Given the absence of a pre-defined “ground truth” for an overall fear and greed score, we employ unsupervised learning to explore and derive these patterns without labeled data. Unsupervised methods like PCA, ICA, and Factor Analysis are well-suited for this problem, as they allow us to capture the underlying structure and relationships among the indices. PCA helps to maximize variance, revealing the most significant contributors and reducing dimensionality without losing essential information. ICA, in contrast, emphasizes statistical independence, which can be valuable if each index captures distinct sentiment drivers. Factor Analysis, meanwhile, identifies latent factors, assuming correlations among indices that reflect broader, shared sentiment influences. By applying these methods, we can construct a well-rounded and empirically-based Fear and Greed Index, balancing variance, independence, and latent factors.

*** Principle Component Analysis ***

```python 
from sklearn.preprocessing import StandardScaler
# Standardize the data to have mean 0 and standard deviation 1
scaler = StandardScaler()
numeric_df_scaled = scaler.fit_transform(numeric_df)

from sklearn.decomposition import PCA

# Initialize PCA and fit to the scaled data
pca = PCA(n_components=8)  # We are keeping all indices initially
pca.fit(numeric_df_scaled)

# Get the explained variance ratio for each principal component
explained_variance = pca.explained_variance_ratio_

# Display the explained variance
for i, variance in enumerate(explained_variance, 1):
    print(f"Principal Component {i}: {variance:.2%} of the variance explained")
```
We will see below how each component contributes to the variance. 

```json
Principal Component 1: 36.42% of the variance explained
Principal Component 2: 26.88% of the variance explained
Principal Component 3: 12.76% of the variance explained
Principal Component 4: 10.65% of the variance explained
Principal Component 5: 5.47% of the variance explained
Principal Component 6: 4.51% of the variance explained
Principal Component 7: 1.95% of the variance explained
Principal Component 8: 1.36% of the variance explained
```
Next, we will get the loadings and calculate the absolute contributions and normalize them to get percentage. 

```python
# Get the loadings (components) and focus on the first component
loadings = pca.components_[0]  # First principal component
variable_names = numeric_df.columns

# Calculate the absolute contributions and normalize them to get percentages
loadings_percent = 100 * abs(loadings) / abs(loadings).sum()

# Display the contribution of each variable
contributions = pd.DataFrame({'Variable': variable_names, 'Contribution (%)': loadings_percent})
contributions.sort_values(by='Contribution (%)', ascending=False, inplace=True)
```
The PCA analysis shows that market momentum, exchange rate, and price strength are the top contributors to the Fear and Greed Index, with contributions of around 18%, 17.6%, and 17% respectively. These high values suggest that market sentiment is strongly influenced by recent price movements, currency stability, and the strength of stock prices. Volume breadth and the Buffett Indicator also have moderate contributions (approximately 15% and 13%), indicating that broader market participation and valuation relative to GDP play important roles. In contrast, volatility, safe haven demand, and especially interest rate have lower contributions, suggesting they have a more limited impact on the overall sentiment in this model. This distribution highlights that investor sentiment in the Indonesian market is driven primarily by direct market performance indicators and exchange rate stability.

```json
Variable	Contribution (%)
0	market_momentum	18.538131
6	exchange_rate	17.586832
2	price_strength	17.038441
4	volumn_breadth	14.983672
3	buffett_indicator	13.195301
1	volatility	9.487410
5	safe_heaven	6.170997
7	interest_rate	2.999218
```
*** Independent Component Analysis ***

```python 
from sklearn.decomposition import FastICA
from sklearn.preprocessing import StandardScaler
import pandas as pd

# Standardize the data to mean 0 and standard deviation 1
scaler = StandardScaler()
numeric_df_scaled = scaler.fit_transform(numeric_df)

# Apply ICA
ica = FastICA(n_components=8, random_state=42)
ica_components = ica.fit_transform(numeric_df_scaled)  # Transformed independent components

# Get the mixing matrix, which contains the contributions of each index to each independent component
ica_mixing = ica.mixing_

# Extract and normalize the first component to get the contribution percentage for each index
ica_contributions = 100 * abs(ica_mixing[:, 0]) / abs(ica_mixing[:, 0]).sum()
ica_contributions_df = pd.DataFrame({'Variable': variable_names, 'ICA Contribution (%)': ica_contributions})

# Sort and display the contributions
ica_contributions_df.sort_values(by='ICA Contribution (%)', ascending=False, inplace=True)
print("Independent Component Analysis (ICA) Contributions:\n", ica_contributions_df)

```
The ICA analysis indicates that volume breadth and exchange rate are the dominant contributors to the Fear and Greed Index, with contributions around 29% each. This suggests that broad market participation and currency stability are key independent drivers of sentiment, likely reflecting investor confidence and economic stability. The Buffett Indicator also contributes significantly (18%), indicating that overall market valuation relative to GDP is an important factor in market sentiment. In contrast, price strength, market momentum, and volatility have smaller contributions (under 9%), while safe haven demand and interest rate have minimal influence in this independent component framework. This result implies that sentiment in the Indonesian market is most strongly shaped by broad market dynamics and macroeconomic stability rather than specific price movements or safe-haven assets.

```json
Independent Component Analysis (ICA) Contributions:
             Variable  ICA Contribution (%)
4     volumn_breadth             29.250553
6      exchange_rate             29.155509
3  buffett_indicator             18.314223
2     price_strength              8.941634
0    market_momentum              6.097687
1         volatility              5.561563
5        safe_heaven              2.060825
7      interest_rate              0.618005
```
***Factor Analysis**

```python
# Apply Factor Analysis
factor_analyzer = FactorAnalysis(n_components=2, random_state=42)  # Assume 2 latent factors for simplicity
factor_analyzer.fit(numeric_df_scaled)

# Extract factor loadings
factor_loadings = factor_analyzer.components_

# Create a DataFrame of factor loadings
factor_loadings_df = pd.DataFrame(factor_loadings.T, index=variable_names, columns=['Factor1', 'Factor2'])

factor_loadings_df
```
```json
	Factor1	Factor2	Factor1 Contribution (%)
market_momentum	-0.883596	0.137230	23.126012
volatility	0.188058	0.695190	4.921962
price_strength	-0.884621	0.308945	23.152834
buffett_indicator	0.354879	0.751056	9.288099
volumn_breadth	-0.708119	0.220054	18.533305
safe_heaven	0.232037	-0.025614	6.072998
exchange_rate	-0.567351	-0.696712	14.849061
interest_rate	-0.002129	0.340527	0.055730
```
```python
# Calculate contribution percentages for the first factor to interpret as potential weights
factor1_contributions = 100 * abs(factor_loadings_df['Factor1']) / abs(factor_loadings_df['Factor1']).sum()
factor_loadings_df['Factor1 Contribution (%)'] = factor1_contributions

# Display factor loadings and contributions
print("Factor Analysis Loadings:\n", factor_loadings_df)
```
The Factor Analysis results indicate that market momentum and price strength are the strongest contributors to Factor 1, with contributions of approximately 23% each. This suggests that these indices are highly correlated with a shared sentiment factor, likely reflecting overall market optimism or pessimism. Volume breadth and exchange rate also show notable contributions (18.5% and 14.8%, respectively), indicating their relevance to this common sentiment factor, especially in terms of market participation and currency stability. The Buffett Indicator and volatility contribute more significantly to Factor 2, suggesting that they might reflect a separate latent factor related to valuation and economic risk. Safe haven demand has a modest contribution to Factor 1, while interest rate contributes minimally, indicating it has little influence on the primary sentiment factor in this model. Overall, the results imply that sentiment is strongly tied to direct market performance and economic indicators, with secondary influences from broader economic risk factors.

```json
Factor Analysis Loadings:
                     Factor1   Factor2  Factor1 Contribution (%)
market_momentum   -0.883596  0.137230                 23.126012
volatility         0.188058  0.695190                  4.921962
price_strength    -0.884621  0.308945                 23.152834
buffett_indicator  0.354879  0.751056                  9.288099
volumn_breadth    -0.708119  0.220054                 18.533305
safe_heaven        0.232037 -0.025614                  6.072998
exchange_rate     -0.567351 -0.696712                 14.849061
interest_rate     -0.002129  0.340527                  0.055730
```

*** Final Decision ***

In conclusion, after comparing the results of the PCA, ICA, and Factor Analysis models, the final decision is to use the mean of the three sets of weights as the basis for constructing the overall Fear and Greed Index for the Indonesian stock market. This approach balances the distinct insights each model offers, combining variance maximization, independence detection, and latent factor representation to create a more comprehensive sentiment index.

Each model provides unique advantages. PCA highlights the indices that explain the most variance in the data, underscoring the influence of indices like market momentum and exchange rate on the overall sentiment. This approach captures the dominant factors but may overlook unique contributions from less correlated indices. ICA, on the other hand, identifies independent sources of sentiment, emphasizing the influence of volume breadth and exchange rate as distinct drivers of market sentiment. While ICA offers valuable insights into independent sentiment components, it may understate the influence of interrelated factors that affect multiple indices simultaneously. Factor Analysis provides a middle ground by identifying latent factors shared among indices, showing that market momentum, price strength, and volume breadth align closely with a common sentiment factor, while volatility and Buffett Indicator represent a separate dimension of market risk.

By averaging the weights derived from these three methods, we benefit from the comprehensive view provided by PCA, the independence-focused insights of ICA, and the factor correlation perspectives of Factor Analysis. This blended approach creates a balanced and robust Fear and Greed Index that is sensitive to dominant, independent, and correlated drivers of market sentiment, ensuring that the final model is both versatile and representative of diverse market influences. Using the mean weights allows us to incorporate the strengths of each model, resulting in a more resilient and nuanced measure of sentiment for the Indonesian stock market.

```json
     Variable	   PCA(%)	ICA(%)	Factor1(%)	Final Weight (%)
4	volumn_breadth	14.98	29.25	18.53	20.920000
6	exchange_rate	17.59	29.16	14.85	20.533333
2	price_strength	17.04	8.94	23.15	16.376667
0	market_momentum	18.54	6.10	23.13	15.923333
3	buffett_indicator	13.20	18.31	9.29	13.600000
1	volatility	9.49	5.56	4.92	6.656667
5	safe_heaven	6.17	2.06	6.07	4.766667
7	interest_rate	3.00	0.62	0.06	1.226667
```













