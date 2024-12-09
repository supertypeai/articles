---
title: Turning Market Sentiment into Data - Developing Fear and Greed Index (Part 3)
post_excerpt: In the third part of this article, we will walk through the data handling process - from sourcing into production-ready data.
taxonomy:
    category:
        - internal-guides
        - notes
---

# A Step by Step Guide to Market Sentiment Analysis - Engineering Fear and Greed Index (Part 3)

In the first and second part of this article, we explored the concept of Fear and Greed Index, and walked through the theoretical and mathematical concepts used for the calculation. This part of the article will focus more on the technical and operation aspect of the data engineering process. We will begin by exploring the data sources, the considerations that rise between different data sources, and the challenges for automation that requires some workarounds.

## Data Sourcing

### Planning & Identifying Data Sources
To identify what data are going to be needed, we first take a look at the indices we are using.
- **Market Momentum**: IDX daily data
- **Stock Price Strength**: IDX daily data
- **Volatility**: IDX daily data
- **Volume Breadth**: IDX daily data
- **Safe Haven Demand**: Indonesian bonds data & IDX daily data
- **Exchange Rate**: USD/IDR exchange rate data & IDX daily data
- **Interest Rate**: IDR interest rate & IDX daily data
- **Buffett Indicator**: IDX market cap data

0. The general process for the data are identifying its attributes that match the requirement for the Fear and Greed indices, identify its completeness (since we are working with historical data), then start preparing the transformation for the readily available data. The source also has to pass the "BE0T24M" test (I forgot the full name for it, but it basically means that the pipeline will work for the next 24 months - expecting the upstream source does not introduce potentially breaking changes to the format we are using)

1. Based on the data sources required, we can start by identifying which data are available in our data warehouse, then start preparing the transformation for the readily available data.

2. For the data that is not yet available in the data warehouse, a good starting point for potential sources would be the current data sources currently consumed by our data pipeline. This is beneficial for some reasons:
    - Effectivity: We do not have to subscribe to more services by reusing existing subscriptions.
    - Reduced complexity and by extension, increased reliability: Reduced dependency simply reduce the need to maintain the pipeline when a source changes. This means less possibly breaking changes hence more reliability.
    - Consistency: Same data sources accross multiple pipelines means better consistency and reduces the need for extra transformation.

3. For the data not yet available in the data warehouse, not yet sourced for the current pipeline, or does not meet the required attributes, we have to find new source for them.

### Scouting the Data Sources
Based on the plan discussed previously, it is time to explore the data sources.

1. #### IDX Daily Data
    Since Sectors already has complete daily data of IDX stock prices, we can directly use the data stored in the database. The data is complete and has been quality checked by the team. To limit the data size however, we only pick the IDX30 stocks for the calculation. This index list is accessed from [sectors_indices_company_list repo](https://github.com/supertypeai/sectors_indices_company_list)

2. #### Indonesian Bonds Data
    The current data warehouse has not dealt with Indonesian bonds data, so it doesn't store any bonds data, and it doesn't have any pipeline sourcing bonds data yet. The best quality of data that we can get is from [PHEI](https://www.phei.co.id/Data/Indeks), the authoritative Indonesian bonds indexer. However, PHEI does not provide historical data and so we can only collect one data point a day. After searching for data sources online, we came across [Indonesia's 10Y bond yield from investing.com](https://investing.com/rates-bonds/indonesia-10-year-bond-yield-historical-data). This data source seemed good since it reflects the government bonds that we manually cross checked from bond brokers. We chose to go with this source for the initial calculation while we wait for data from PHEI accumulates over time in with our pipeline.

3. #### USD/IDR Exchange Rate Data
    There are some data sources for USD/IDR exchange rate that we use in our data processing pipeline, but all of them does not provide the data we need. There is an existing pipeline for [USD/IDR conversion rate](https://github.com/supertypeai/sectors_get_conversion_rate), but this pipeline does not store historical data (at least not conveniently, we can still dig through the commit histories to collect the historical data, but that would be impractical). The data source itself - [exchangerate-api](https://www.exchangerate-api.com/) - does not provide historical data that we need (at least not for free).\
    \
    The next best thing we found is [openexchangerates.org](https://docs.openexchangerates.org/reference/historical-json). Although we can get the historical data from the new source and switch to the existing data pipeline for less service dependency, the data between these services does not show high consistency, so we decided to continue with [openexchangerates.org](https://docs.openexchangerates.org/reference/api-introduction) because we primarily need the consistent trend between time series. A noteworthy source to mention is [BI JISDOR](https://www.bi.go.id/en/statistik/informasi-kurs/jisdor/Default.aspx) - good quality data (provided by Bank Indonesia, the authoritative source of IDR exchange rate) - but this source has some downsides: less reliable method (by parsing the HTML tags, which is more likely break if an update occurs compared to dedicated production grade API).

4. #### IDR Interest Rate Data
    There are no IDR interest rate (bank interest rate) data in our data warehouse yet, and we haven't really had the need to use it so far, so our only option is to look for a new data source. There aren't many source for this data except for Bank Indonesia's reference interest rate directly from their [website](https://www.bi.go.id/id/statistik/indikator/BI-Rate.aspx). Since this is the only data source we came across - and this is the authoritative data source - we chose this as our source for IDR Interest Rate.

5. #### IDX Market Cap Data
    Just like [IDX daily data](#idx-daily-data), this data is readily available in the database and can be processed directly from the same dataset of IDX daily data. However, since we are running on resource constraint (the batch process is run on Github Actions), we choose to fetch the data from [Sector](https://sectors.app/api)'s [market cap API](https://api.sectors.app/v1/idx-total/). Not only does this saves on computational costs, but we expect that using this API can save on costs overall since the API results can be cached by the Sectors App.

## Data Fetching & Processing
After identifying the data sources for the data pipeline, it is now time to fetch and process them

0. ### Setting Up & Design
    The main components that play huge roles for the data processing are:
    * Python\
    This is the main tool we've been using for our data pipelines, we choose this for its extensive libraries and for tech consistencies across data pipelines.

    * Supabase\
    Supabase client is used for fetching data from the database.

    * Pandas\
    The main tool for data processing in Python. Pandas can do virtually any data processing methods, and this is widely used in our data pipelines.

    * Datetime\
    This is used for virtually every data processing in this pipeline since we are dealing with time series.

    * id_ID & en_US System Locale\
    The locales are needed for parsing human format datetime from various sources that does not provide ISO format datetime. This removes the need for manual parsing and allows for cleaner code.

    As mentioned in the system locale, we made the `override_locale` function to switch the system locale into another locale temporarily using Python's `contextmanager`. This allows the data processing module to parse human time formats from different locales by safely managing the state of locales since the locale state is shared across the whole program.

    Another pattern which we'll repeatedly cover is management of data not stored in the database, which is mentioned in the [Planning & Identifying Data Sources](#planning--identifying-data-sources) section. This is done by storing the data we fetched from the remote source into a file stored in the local filesystem, then commits the data after each update (which is done through the Github Action with cron schedule). This utlizes the code repository as the "cache" for the data, or even as a form of database.

1. ### IDX Daily Data
    `fetch_daily_data(timeframe: int)`

    The data size for the daily data can be rather huge when we are just plainly accessing all stocks data on each requests. This does not work well for the pipeline and also for the database. To get around this, we use only the stocks listed in the IDX30 since this represents the largest market movement. This also reduces the noises of "penny stocks" that causes has huge volatility, but only makes up a fraction of the market cap.

    An important caveat is the limit on the amount of data per request made. This limit is a safety mechanism to prevent malicious or accidental huge query, and we shouldn't disable this just for the sake of this pipeline. So, to get around this, we turn the timeframe into chunks of smaller timeframes, which is then used for subsequent requests into the database. Although this is not a good sign of data access (e.g. not caching them in the same way as other data that we'll discuss later), this is rarely done since data access for huge enough timeframe is done only to train models. The actual data pipeline will use small enough timeframes that doesn't hit the limit specified in the Supabase settings.

    The data processing consists of
    1. Fetch IDX30 list from github. This can be done by accessing the raw content via HTTP request to get the code in text format, then parse the result from CSV.

    2. Create a Supabase request using `in_()` method (equivalent to `IN` query in SQL) with the IDX30 list.

    3. Compute the start and end date in 90 days chunk, then include the date parameter by chaining `gte()` and `lte()`.

    4. Execute request repeatedly with different start and end date until the maximum past date is reached with the Supabase client.

    5. Parse the JSON responses into Pandas DataFrame, and return the final result

2. ### Indonesian Bonds Data:
    As we have discussed in the [Indonesian Bonds Data](#indonesian-bonds-data) section, there are two data sources used initially for the data pipeline. Each has its own API, which will be discussed separately below.

    `fetch_bonds_rate(timeframe: int, force_refresh: bool)`

    This API fetches data from [PHEI](https://www.phei.co.id/Data/Indeks).

    Since we do not have a table for Indonesian bonds data in our database, we fetch the bonds data from PHEI regularly and store the results in the repository. PHEI does not provide historical data (they only provide the data for the current date and the previous date), so the data fetching is run every working day to collect the daily data.

    The data processing consists of:

    1. Read the locally stored data into Pandas DataFrame

    2. Fetch the current date's data from the website

    3. Parse the data from HTML tags, then convert the data into Pandas DataFrame

    4. Combine the two DataFrame

    5. Write the combined result into the same file

    6. Filter the result to be within the timeframe, then return the final result

    Note that this data source provides the current date in human format, specifically in the Indonesian date format. To parse the date, we first switch the locale into Indonesian, parse the date, and return the locale to its original state with the `override_locale` function.

    There is also an unusual behavior of the data listed on PHEI: the "current date" data of the previous date sometimes do not match the "previous date" data of the current date (i.e. the previous date data can sometimes get updated even after the data no longer gets an update after the working hour). To get around this, the API provides the `force_refresh` parameter to replace the "previous date" data with the current data (which is true by default, since we believe that the correction made by PHEI is the authoritative result).

    `fetch_temp_bonds_rate(timeframe: int)`

    This API fetches data from [Indonesia's 10Y bond yield from investing.com](https://investing.com/rates-bonds/indonesia-10-year-bond-yield-historical-data).

    Similar to PHEI, we do not have a table in our database to store the bonds data. Although investing.com does provide historical data that we can access anytime, the page provides the data with javascripts for the dynamic query, and does not provide dynamic query within the URI. For data that is out of the default time range within the page, this is problematic. This can be circumvented by analyzing the network traffic from the browser, and identify the endpoints for the data API - which is simpler, and provides the data in structured form, but since this is only used temporarily, and this behavior might trigger unwanted behavior (e.g. getting blacklisted), we choose to go with the original page URI and parse the HTML tag manually and give up using the data endpoint directly. To provide the older historical data that doesn;t get loaded into the single page, we cache the data into local files, just like data processing for the one from PHEI.

    The data processing consists of:

    1. Read the locally stored data into Pandas DataFrame

    2. Check today's date vs latest date in the file

    3. If today's date > latest date, make a request fot the webpage

    4. Parse the data from HTML tags, then convert the data into Pandas DataFrame

    5. Combine the two DataFrame

    6. Write the combined result into the same file

    7. Filter the result to be within the timeframe, then return the final result

3. ### USD/IDR Exchange Rate Data
    `fetch_idr_usd_rate(timeframe: int)`

    The openexchangerate API provides historical data, but the API provided only allows for single datapoint in the timeseries. This means that querying big sets of data can burn our limited API calls quota pretty quickly, all requesting the same data over and over again. To handle this, we turn back to our local file storage mechanism to cache the older data for future use.

    Just like the previous examples, the data processing consists of:

    1. Read the locally stored data into Pandas DataFrame

    2. Make requests from the latest date in the file until today's date to update until today's data

    3. Transform the new data into a Pandas DataFrame

    4. Combine the two DataFrame

    5. Write the combined result into the same file

    6. Filter the result to be within the timeframe, then return the final result

4. ### IDR Interest Rate Data
    `fetch_idr_interest_rate(timeframe: int)`

    Similar to the data from investing.com in the [Indonesian Bonds Data](#indonesian-bonds-data-1) section, we do not have a table in the database to store these data, and the website does not allow dynamic querying through the URI directly. Analyzing the network seems to yield no endpoints for the data, and the website most likely provides the data alongside the page rendering from the server side. This means that we have to cache older data (for queries that are outside of the default time range of the webpage) locally.

    The data processing consists of:

    1. Read the locally stored data into Pandas DataFrame

    2. Check today's month vs latest month in the file

    3. If today's date > latest date, make a request fot the webpage

    4. Parse the data from HTML tags, then convert the data into Pandas DataFrame

    5. Combine the two DataFrame

    6. Write the combined result into the same file

    7. Filter the result to be within the timeframe, then return the final result

    Just like the data from PHEI in the [Indonesian Bonds Data](#indonesian-bonds-data-1) section, the website provides the date in human readable format, specifically in the Indonesian date format. To parse the date, we first switch the locale into Indonesian, parse the date, and return the locale to its original state with the `override_locale` function.

5. ### IDX Market Cap Data
    `fetch_mcap_data(timeframe: int = _ONE_WEEK)`

    The market cap data does not require local caching (as it takes precious storage in the repository) since we are using Sectors API. The API provides historical data for the market cap calculation, and provides the data in JSON format already. The only problem we faced during fetching the API is the time range limit of each request. The API provides maximum of 90 days range between start date and end date. This can be solved by turning the timeframe into chunks of smaller timeframes, which is then used for subsequent requests into the database.

    The data processing consists of:

    1. Compute the start and end date in 90 days chunk for the API request parameter.

    2. Execute request repeatedly with different start and end date until the maximum past date is reached.

    3. Parse the JSON responses into Pandas DataFrame, and return the final result.


## Fear and Greed Index Processing

### Programming Paradigm
Calculating the Fear and Greed's intermediary indices is done using functions (including the data fetching APIs). Although this causes the import of the functions to be too verbose, this prevents the unintended usage of object oriented (OOP) approach in the future when we need to extend this in the future. We avoid using OOP approach for the calculation because it does not go well with the nature of the calculation, where we only require a determined output of some inputs, without changing any states (or put another way, causing "side effects"). This is safer and easier to maintain, especially for a pipeline where multiple calculations are done independently.

The output of several indices can then be fed into another function to calculate the final Fear and Greed Index. This however, requires a huge amount of parameter since we are dealing with multiple indices. Another approach would be to wrap around all the indices calculation into a single function, accepting the raw data as input, and outputting the final Fear and Greed Index while the calculation of the intermediary indices is hidden within the function. This also allows the intermediary indices parameters to be configured by the Fear and Greed-related parameters, separating isolating the concern to within the function.

The previous approach worked well and is simple to understand, until we have the need for more granular control over the parameters in the function. Passing more and more parameters would make the code less elegant and less readable. To work around this, we finally decided to use OOP approach for the Fear and Greed Index calculation. This approach is used to isolate and abstract the parameters as state of the Fear and Greed Index "calculator". This state can then be changed accordingly, which is helpful when we need to compare results with different parameters.

### Configurations Handling
As mentioned in the previous section, there are configurable parameters that we can use to enhance the Fear and Greed Index calculation. The parameters we use for the Fear and Greed Index pipeline are:

* Data period\
    The period in which the data is going to be fetched and fed into the Fear and Greed Index calculation.

* Moving average period\
    The window period used for the moving average calculation. Currently implemented as a single parameter used across all intermediary indices calculation.

* Moving average method\
    The moving average method used for the intermediary indices calculation, currently implemented for Simple Moving Average (SMA) and Exponential Moving Average (EMA). These parameters are stored and loaded from JSON file.

* Intermediary indices weights\
    The weight of each intermediary indices' result towards the final Fear and Greed Index result. This is currently implemented for linear weights that sums up to 1.

Hard-coding the parameters is somewhat OK for this pipeline, since the code is very likely going to be used solely for the Fear and Greed Index calculation, and there will be very small changes to the parameter. However, implementing it as configuration from another file or as the script arguments are still good practices generally. Not only does this help with refactoring in the future, but allows for flexibility in the early stage where we want to find out which parameters work best.

## Extra
As a bonus for the article, we also tinker around with the Github Action locale. Since the environment Github provided for its Actions are limited, we went with Ubuntu since it offers the best flexibility and portability (I personally prefer Linux environment for production as well). It would have been great if Github Action supported container image since it offers even better portability, and possibly easier cache management, but Github Actions doesn't support it by this time.

One of the satisfying feat we performed when creating the Action is caching the locale for the Ubuntu system. In a nutshell, we cache the locale by:
1. Find out the locale installation directory for Ubuntu using apt.

2. Create the caching logic: By listing the installed locale and match if it contains the desired locale (in this case, id_ID)

3. Finally, we found a caveat where it doesn't provide permission for the locale directory by default, so we change the access for the cache file and it solved the problem.

## Conclusion
To finalize the whole journey through the data processing, we can take several points that we can learn through the process:

* Selecting data sources for the indices can be a challenge when you don't have the data readily available in your data warehouse, and this challenge is even bigger when you have to find a way to manually store them with limited resource constraint. This however, challenge you to be efficient and find clever ways to approach the problem.

* Selecting the design for the code is better done by starting simple. That way you can remove unnecessary complexity from the start, and helps you with building your way up faster rather than concern yourself with possibly unnecessary stuffs with complex designs.

* Optimize when you can, and fix bugs as soon as possible. Don't let the technical debts pile up - not everyone is going to read your code, and most of the time, your code is only revisited when a problem rise or it needs a new feature.