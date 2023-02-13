---
title: Data-driven advertising | 3 use-cases
post_excerpt: How I use data analytics to deliver value to my advertising clients; A case study on creative decay analysis, retention rate analysis, and exploratory data analysis (python code provided) 
taxonomy:
    category:
        - knowledge
        - notes
---

## Applications of data analytics in digital advertising

Recent researches on the digital marketing industry estimate that businesses adopting a data-driven approach in their marketing strategy are outperforming their peers by 20% in revenue growth and are **six times more able to retain customers**. 

In this article, we will be exploring some usage patterns of data analytics for modern marketing and advertising teams. We will dive into some data analysis from the lens of a digital marketing professional, and look at three ways you can incorporate data science into your growth and advertising strategy:
- **Retention Rate Analysis**: Understand the rate at which users are retained over time, and an evaluation metric for the performance of our advertising creatives.
- **Creative Decay Analysis**: Understand the rate at which advertisements lose their effectiveness over time, also known as the **decay rate**.
- **Exploratory Data Analysis**: Determining characteristics of effective ad creatives, and grouping ads into clusters with machine learning

Examples and demonstrations of the analysis are drawn from our earlier work with [clients in the creative advertising industry](https://supertype.ai/notes/creadits-x-supertype/) and [programmatic advertising industry](https://supertype.ai/notes/opera-adcolony-supertype-case-study/), which you can read more about in the case study section of our website.

If you are a Python practitioner, or are interested in learning Python for marketing analytics, I've interspersed the article with some Python code snippets wherever it is meaningful to do so. 

Python is a popular programming languages for data science and machine learning, and is a great addition to any marketing analytics stack, as we'll see in the article. 

### Libraries & Setup
Let's start off by importing some Python libraries that will be used throughout the demonstration.

```{python}
import numpy as np
import pandas as pd
import re
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as 

from plotly import graph_objects as go
from scipy.interpolate import interp1d
from plotnine import ggplot, aes, geom_smooth, geom_bar, labs, theme_minimal, scale_x_date, facet_wrap, facet_grid, expand_limits
```

Many of the imported libraries above are centered around data visualization, which is a key component of data analytics. We will be using plotly and plotnine's ggplot2-like syntax to create interactive and aesthetically pleasing visualizations.

## Retention Rate Analysis

When evaluating advertising campaigns and ad creatives, the natural inclination is to observe the number of clicks and impressions. However, in order to quantify the impact of advertisements on the company's growth, we might go a step further and examine the number of users that are retained over time. This is known as the **retention rate**, and is a concept first popularized by the video game industry. 

In the context of advertising, the retention rate is the percentage of users that continue to use the product or service after the point of initial contact ("acquisition"). It is an indication of the effectiveness of an advertisement campaign, as it correlates with the relevance of the advertisement to the user. A holistic marketing stack consider retention rate alonside other key performance indicators such as click-through rate (CTR) and conversion rate (CVR). These metrics are used to evaluate the performance of an advertisement campaign and they might take the form of the following questions:

1. Does the advertisement creative have good impressions?
2. Are the users interested to "click" upon seeing the advertisement creative?
3. Do we get continuous usage over a period of time?

There are numerous ways to conduct retention rate analysis, and we'll start off with the cohort retention table. A cohort retention table is a table that shows the percentage of users retained over time, grouped by the time of their initial contact with the product or service. The following table is an example of a cohort retention table:

![cohort retention table](
/_images/cohortanalysis.png
)

The table describes the proportion of users who, upon their initial contact (the "New Users" column), continue to consume our product or service over the next, say, 17 days. This is helpful for the data analyst or marketing analyst because it serves as a reference point to how well the advertisement is at resonating to the core intended audience. An ad that is great at converting impressions to clicks, but has a low retention rate compared to the benchmark group, might indicate a mismatch between the audience's expectation and the advertisement's content.

If we want to visualize the funnel of the average user, we can shape the data up to be presented in a funnel chart. This loses some of the information that the cohort retention table provides, but it can be an interactive medium to visualize the retention rate of the average user.

![ads retention funnel chart](/_images/funnel_chart.png)

The funnel chart above breaks down the number of users in each phase, the percentage difference between a phase to previous phase, and also the percentage difference between a phase to the initial phase. The data analyst, or marketing analyst, can begin to formulate thoughtful questions based on the retention table and funnel charts:

- At which phase do most users drop off?
- How does the retention rate compare to the benchmark group or industry standard?
- Are the impressions strongly, or weakly correlated to long term retention of the acquired user?
- Can the advertising message and creative be improved to resonate better with the target audience, and in turn appeal to them to continue using the product or service?

The chart is generated using plotly to add a level of interactivity, and the code snippet is provided below. Refer to the comments for details regarding the data input format.

```{python}
# Data Input Format
# x: total number of each phase
# y: name of the phase

fig = go.Figure(go.Funnel(
  y = df_join.iloc[0,8:14].index,
  x = df.sum()[4:10],
  marker = {"color": "#D4AF37"}
  ))

fig.update_layout(title={
  'text' : "Advertisement Funnel",
  'y' : 0.9,
  'x' : 0.5,
  "xanchor" : "center",
  'yanchor': "top"},
  plot_bgcolor = 'rgb(255,255,255)')

fig.show();
```

## Decay Rate Analysis

Advertising relevance and message appeal are not the only factors that determine the effectiveness of an advertisement. The lifespan of an advertisement creative is an equally important factor that impact performance of an advertisement campaign.

We can quantify the rate at which an advertisement loses its effectiveness over time by examining the number of impressions and click-through rate (CTR) over time. We'll call this quantity the **decay rate**.
 
A reliable decay rate analysis can help the advertising team determine when to refresh or replace an advertisement creative, and empower data analysts on the team to hypothesize the reasons behind advertising creatives that have a longer lifespan.

![decay rate analysis](/_images/ctr_decay.jpeg)

Advertising creatives with a longevity much shorter than that of the benchmark group might indicate that the creative elements are influenced by the current trends, and are not as timeless. These creative elements should be refreshed more regularly to maintain the same appeal to its target audience.

![decay rate analysis](/_images/d1_decay.jpeg)

These charts can be produced with relative ease using `matplotlib`. The code snippets are provided below.

```{python}
figure, axis = plt.subplots(5, 1, figsize=(18, 30))

for i in range (0, 5):
  id = top_videos[i]
  df_vid_one = df_decay[df_decay.video_id == id]
  cubic_interpolation_model = interp1d(df_vid_one.index, df_vid_one.click_through_rate, kind="cubic")
  x = np.linspace(df_vid_one.index.min(), df_vid_one.index.max(), 500)
  y = cubic_interpolation_model(x)
  y = np.where(y < 0 , 0.01 , np.where(y > 1, 0.99, y))

  axis[i].plot(x,y)
  axis[i].xaxis.set_major_locator(ticker.MaxNLocator(integer=True))
  axis[i].set_xlim(1,30)
  axis[i].set_ylim(0,1)
  axis[i].set_xlabel('Days from Published Date')
  axis[i].set_ylabel('Click Through Rate (CTR)')
  axis[i].set_title(f"CTR Decay Analysis for Video ID {id}")
```

Some short remarks on the code snippet above:
- we use `matplotlib`, a Python library for data visualization, to create 5 subplots (1 subplot for each video) with the purpose of enabling quick comparison between each video's decay rate
- we apply a smoothing process to the CTR values using the `inter1d` function from the `scipy` library; this is to ensure that the decay rate is not overly influenced by the noise in the data, and ease the interpretation of our visualization 

## Exploratory Analysis on Ad Creatives

At this point, the data analyst can also build on the exercises above by grouping and reshaping these data differently. For example, one might group the data by the video topic, and examine the retention rate of each topic. This can be done by using the `groupby` function in `pandas`.

One might also group them into different groups based on the video length, and examine the retention rate of each group. With a bit of creativity and exploration, the data analyst might be able to uncover some interesting insights that can be used to improve the performance of the advertisement campaign.

![retention rate analysis](/_images/d1_d7_retention.jpeg)

![CPM analysis](/_images/cpm_value.jpeg)
![CTR analysis](/_images/ctr_value.jpeg)

The visualizations displayed above are created using an identical code base, with a few parameters changed between each chart. This is a programming pattern known as **parameterization**, where a function (or process, or algorithm) is defined once, and then reused with different parameters, promoting code reuse and reducing the amount of code that needs to be written.

```python
def create_plot(df=df_plot, x="index", y="d1_retention", col="videoTopic"):
  return 
    (ggplot(df, aes(x=df[x], y=df[y], color=df[col])) +
      geom_smooth() +
      scale_x_date(date_breaks = "1 month", date_labels =  "%b %Y")+
      theme_minimal() +
      expand_limits(y=[0,0.44])
    )
```

Calling `create_plot` with the default parameters will generate the chart for the d1 retention rate. However, by changing the value of y to `d7_retention`, `cpm`, or `ctr`, you can obtain the other three charts. 

## Cluster Analysis

I've written about [dimensionality reduction](https://supertype.ai/notes/decision-boundaries-with-pca-famd/) in a previous article, and how it was used to cluster the data into different groups, based on the similarity of the data points. 

In our case, a data scientist might want to cluster the creatives into different groups, based on the similarity of their thumbnails. This can be done by using a neural network model -- the network extracts the features of the thumbnails, and subsequent use these features in its training process to cluster the thumbnails into meaningful groups. 

This shifts the data analysis process from a purely descriptive one, to a more predictive one. Each image is automatically, according to the weights of the neural network, assigned to a cluster. The data scientist can then examine the characteristics of each cluster, and compare the average performance of each cluster against the population CTR, CPM, and retention rate.

Manually assigning tens of hundreds of ad thumbnails and video ads to different clusters is a tedious and time-consuming process. However, with the help of a neural network model, this process is fully automatic, and the data scientists on the team can focus on the higher-level tasks of interpreting the results and making recommendations to the advertising team.

![cluster analysis](/_images/clustering.jpeg)

Data scientists and data analysts working in the digital advertising industry are tasked with the immense responsibility of sifting through large amount of data (tabular, images, videos, or even text, i.e [sentiment analysis on Tweets](https://supertype.ai/notes/twitter-sentiment-analysis-part-1/)), to tease out valuable insights. With the right tools and techniques, some of these analyses can be parameterized, or even automated, to allow the analysts more time on high value activities and recommendations.

While our analyses here are centered on three specific data science use-cases in the digital advertising industry, I've also written a broader [workflow recap on applying data analytics](https://supertype.ai/notes/ads-optimization-analytics/) in the digital advertising industry, which you might also find useful. 

Until next time, may your data be clean, and your findings be insightful!