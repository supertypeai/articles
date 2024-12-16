---
title: Turning Market Sentiment into Data - Developing Fear and Greed Index (Part 1)
post_excerpt: In the first part of the article, we will explore the methodology behind designing a Fear and Greed Index, from conceptualization and mathematical techniques to data gathering, as we quantify market sentiment in real time.
taxonomy:
  category:
    - knowledge
    - notes
---

# A Step by Step Guide to Market Sentiment Analysis - Conceptualization and Methodology (Part 1)

### Understanding the concept of CNN's Fear and Greed Index

The Fear and Greed Index, first introduced by [CNNMoney](https://edition.cnn.com/markets/fear-and-greed), is a tool that measures investor sentiment to help gauge overall market conditions. It’s built on the notion that excessive fear often drives asset prices down, while excessive greed can inflate them, potentially creating bubbles. The index combines seven market indicators, each contributing to a composite score ranging from 0 to 100. Lower scores indicate extreme fear, while higher scores reflect extreme greed.

The seven components of the index cover key areas of the market:

- **Stock Price Momentum**: looks at the S&P 500’s movement relative to its 125-day average, showing bullish or bearish trends.
- **Stock Price Strength**: examines the number of stocks reaching 52-week highs versus lows, while Stock Price Breadth measures market volume, indicating the level of participation in market moves.
- **Put and Call Ratio**: assess sentiment by analyzing the trading volume of options for buying versus those for selling.
- **Market Volatility**: represented by the VIX Index, reflects uncertainty or stability in the market.
- **Safe Haven Demand**: compares stock performance with that of bonds, a safe haven asset, to indicate risk appetite.
- **Junk Bond Demand**: examines the difference between junk bonds and safer government bonds, with narrower spreads indicating greed and wider spreads indicating fear.

![CNN-Fear-Greed-Example](/_images/cnn-f&g-example.png)

Each of these components is given a score from 0 to 100, and the total score offers a snapshot of the market’s emotional state. For investors, the Fear and Greed Index can serve as a valuable tool for identifying potential turning points. Extreme fear may signal a buying opportunity as prices drop, while extreme greed could suggest caution, as markets might be overextended. By tracking sentiment, the Fear and Greed Index acts as a useful complement to fundamental and technical analysis, offering an additional layer for assessing market risks and opportunities.

### Developing the Index for the Indonesian Stock Market

Inspired by CNN’s methodology, our objective is to develop a Fear and Greed Index specifically tailored to the Indonesian stock market. This index will rely on relevant data and indicators that effectively capture the unique dynamics of Indonesia’s financial environment. While CNN’s index provides a broad view of global market sentiment, adapting it for Indonesia will allow us to incorporate local factors that influence market behavior, especially those related to regional economic conditions and investor psychology. In particular, adding macroeconomic indicators such as interest rates, exchange rates, and the Buffett Indicator offers insights into the market’s sensitivity to broader economic shifts. These additional factors are essential for accurately gauging sentiment in a developing economy, where external economic influences can have pronounced effects on local market sentiment.

Data availability is a key consideration in defining this index. For instance, while put and call ratios are integral to CNN’s index, the Indonesian stock market lacks sufficient options trading volume and reporting standards for these metrics. As a result, we’ll focus on accessible, high-impact indicators that offer direct and indirect measures of investor sentiment.

Direct Indicators of Stock Market Sentiment:

- **Market Momentum**: IDX Composite momentum, capturing broader market direction.
- **Stock Price Strength**: Proportion of stocks hitting new highs versus lows.
- **Volatility**: Measuring market stability or uncertainty.
- **Volume Breadth**: Reflecting the level of participation across trades.
- **Safe Haven Demand**: Balance between equity and bond demand, indicating risk appetite.

Indirect Indicators of Macroeconomic Influence:

- **Exchange Rate**: Impact of currency stability on investor confidence.
- **Interest Rate**: Reflects the cost of capital and economic health.
- **Buffett Indicator**: Market cap-to-GDP ratio, providing a valuation benchmark for market sentiment.

Together, these indicators will offer a comprehensive, data-driven view of sentiment in the Indonesian stock market, enabling more informed analysis and potentially predictive insights.

### Establishing the Mathematical Methodology

To create an effective Fear and Greed Index tailored for the Indonesian stock market, we first define the relevant indices and then outline a clear mathematical methodology. Each individual index, such as market momentum or volatility, will have its own Fear and Greed score. These scores are then combined to create a comprehensive sentiment measure for the overall market.

**Step 1: Understanding Simple Moving Average Method (SMA)**

The Simple Moving Average (SMA) method is a valuable tool in constructing a Fear and Greed Index as it smooths out short-term fluctuations, revealing overall trends. By averaging the last N (as you define) data points, SMA helps to reduce market noise, making it easier to detect real shifts in sentiment rather than reacting to daily volatility. This approach is adaptable to different types of data: for example, in market momentum, SMA is applied to stock closing prices, while in volatility, it may be calculated using daily changes in price or index values.

![SMA Formula](/_images/sma_formula.png)

For our purposes, we chose a 7-day SMA to capture market trends without overemphasizing daily fluctuations. This period balances responsiveness with trend stability and provides insight into sentiment shifts over a manageable short-term period. By consistently applying a 7-day average, we can more accurately gauge investor sentiment changes in the Indonesian market within a week’s context, enabling us to maintain sensitivity to recent market behavior.

**Step 2: Calculating Distance from SMA**

In this step, we calculate the distance from the SMA based on the percentage change of each indicator rather than using absolute values. This approach standardizes the data, making it easier to interpret relative deviations across different indicators.

![Distance SMA Formula](/_images/distance_sma_formula.png)

For each indicator, the daily percentage change is firstly calculated, and then the SMA is applied to these percentage changes over a selected period - 7 days. The distance between the current daily percentage change and the SMA reflects the strength of deviation, with larger positive values indicating heightened “greed” and larger negative values indicating “fear.”

**Step 3: Scaling the Distance to a 0-100 Fear and Greed Index**

Now that we have calculated the percentage change for each indicator, the next step is to normalize these values to fit within a 0-100 scale. This step, known as min-max scaling, ensures that each indicator aligns with the Fear and Greed Index structure, where 0 represents extreme fear and 100 represents extreme greed.

![Min Max Scale Formula](/_images/min_max_scale.png)

This formula takes the current value and scales it based on its position between the predefined minimum and maximum. Any values below the minimum threshold are clipped to 0, and any values above the maximum are clipped to 100. This ensures that the indicator always falls within the 0-100 Fear and Greed range.

**Step 4: Modelling the Final Fear and Greed Index**

Ultimately, we will obtain the daily scaled Fear and Greed Index for each selected indicator. These individual indices will then be combined using unsupervised machine learning models to determine the weight of each indicator, resulting in the overall Fear and Greed Index for the Indonesian market.

Note that in our methodology, the SMA calculation often uses percentage-based scaling, so the min-max scaling should apply to these percentage values. However, for some indicators, rather than using percentage changes from the SMA, we may simply calculate the absolute distance from the SMA and scale this using relative min and max values. The next article in this series will dive deeper into the specific calculations for each index as well as unsupervised machine learning modelling.

### Data Sources for Calculating Sentiment Indices

**Step 1: Identify Input Data Sources**

To begin, we will gather the necessary input data required to calculate each of the predefined indices. Below is an overview of the key data points needed for these calculations:

Direct Indicators of Stock Market Sentiment:

- **Market Momentum**: daily stock prices changes for IDX composite
- **Stock Price Strength**: daily closing prices of individual stocks within the IDX Composite
- **Volatility**: daily closing prices of the IDX Composite
- **Volume Breadth**: daily trading volume data for stocks in the IDX Composite.
- **Safe Haven Demand**: 10 year indonesian government bond daily rate & Indonesian stock market capitalization

Indirect Indicators of Macroeconomic Influence:

- **Exchange Rate**: daily exchange rate of the Indonesian Rupiah (IDR) to the US Dollar (USD), issued by Bank Indonesia (the central bank)
- **Interest Rate**: daily interest rate or central bank policy rate issued by Bank Indonesia
- **Buffett Indicator**: market capitalization of the IDX Composite and Indonesia’s GDP

To gather the necessary data for calculating our predefined indices, we categorize the data sources into two main groups:

1. Internal Data Sources (via [Sectors Financial API](https://sectors.app/api))

- Daily Stock Closing Price for IDX Composite: Used to calculate Market Momentum, Price Strength, and Volatility indices.
- Daily Stock Trading Volume for IDX Composite: Utilized in calculating the Volume Breadth index.
- Daily Stock Market Capitalization: Essential for the Buffett Indicator, allowing us to compute market cap-to-GDP ratios.

2. External Data Sources (via Web Scraping)

- [Interest Rate](https://tradingeconomics.com/indonesia/interest-rate): The daily interest rate issued by the Indonesian central bank (Bank Indonesia), used as an indicator of macroeconomic sentiment.
- [Exchange Rate](https://tradingeconomics.com/indonesia/currency): The daily IDR-to-USD exchange rate from the Indonesian central bank, which reflects economic stability.
- [10-Year Indonesian Government Bonds](https://tradingeconomics.com/indonesia/government-bond-yield): Yield data for 10-year government bonds, representing a benchmark for Safe Haven Demand in the market.

Data from external sources are linked above. In the next step, we will utilize [Sectors Financial API](https://sectors.app/api) to retrieve data from internal sources.

**Step 2: Gathering Input Data from [Sectors Financial API](https://sectors.app/api)**

[Sectors Financial API](https://sectors.app/api) offers comprehensive insights into the Indonesian stock market, allowing us to efficiently access the data needed for our analysis. First we use the company API to rend the company information to identify the stock tickers.

```python
# Retrieve Stock index from "Companies by Index" API
import time
import requests
from google.colab import userdata

# Retrieve the API key securely
api_key = userdata.get('SECTORS_API_KEY')

# Define the API URL
url = "https://api.sectors.app/v1/index/idx30/"

# Pass the API key in the header
headers = {"Authorization": api_key}

# Make the API request
response_company_index = requests.get(url, headers=headers)

print(response_company_index.text)
```

We will see the result of a list of response below.

```json
[
  {
    "symbol": "ACES.JK",
    "company_name": "PT Aspirasi Hidup Indonesia Tbk"
  },
  {
    "symbol": "ADRO.JK",
    "company_name": "Adaro Energy Indonesia Tbk"
  },
  {
    "symbol": "AKRA.JK",
    "company_name": "PT AKR Corporindo Tbk."
  }
  ...
]
```

We will now use the “Daily Transaction Data” API and create a function that loops through company tickers to retrieve the historical market data for the IDX Composite.

```python
# Retrieve date and price from "Daily Transaction Data" API
from datetime import datetime, timedelta

# Function to calculate the date 90 days ago from today
def calculate_start_date(days_ago=90):
    return (datetime.now() - timedelta(days=days_ago)).strftime('%Y-%m-%d')

# Calculate the start date 90 days ago
start_date = calculate_start_date()

# Looping API info
history_idx30 = []

for i, x in enumerate(response_company_index.json()):

  # Define the URL for the API endpoint
  url = "https://api.sectors.app/v1/daily/" + response_company_index.json()[i]['symbol'] + "/"

  # Define the query string with the calculated start date
  querystring = {"start": start_date, "end": datetime.now().date()}

  headers = {"Authorization": userdata.get('SECTORS_API_KEY')}

  response_daily_transaction_data = requests.request("GET", url, headers=headers, params=querystring)

  # Append the result into target list
  history_idx30.append(response_daily_transaction_data.json())

  time.sleep(1)
```

Now if we check the result historical IDX30 data, we will have all the information we need.

```json

[[{'symbol': 'ACES.JK',
 'date': '2024-08-07',
 'close': 735,
 'volume': 43056400,
 'market_cap': 12583493959680},
{'symbol': 'ACES.JK',
 'date': '2024-08-08',
 'close': 720,
 'volume': 52492200,
 'market_cap': 12326688260096},
{'symbol': 'ACES.JK',
 'date': '2024-08-09',
 'close': 720,
 'volume': 55467000,
 'market_cap': 12326688260096},
...
]]

```

**Step 3 Scraping External Data and Storing inside database**

External data sources are scraped using the BeautifulSoup library, with a GitHub Actions workflow set up to automate this process daily. This automation retrieves daily values for the interest rate, exchange rate, and 10-year Indonesian government bond rate. Since the interest rate is updated monthly, we interpolate it to a daily frequency by filling each day within the same month with the same rate.

The backend data pipeline setup will be discussed in detail in a separate article.

With the methodology, calculations, and data sourcing now complete, we are ready to proceed to the next part of this article, where we will begin computing the indices.
