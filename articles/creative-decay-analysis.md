---
title: Decoding the Decline of Advertisement Creatives in Advertising -- An Analysis
post_excerpt: Key visual representation that can provide significant insights into the rate at which advertisements lose their impact and methods to improve it
taxonomy:
    category:
        - knowledge
        - notes
---

![decay rate analysis](/_images/header_decay.jpeg)

In this article, I will be sharing another piece on data analysis in the digital marketing industry. If you're interested, you can also check out our <u>[previous post](https://supertype.ai/notes/ads-optimization-analytics/)</u>, to see our approach to consulting projects, particularly in the digital marketing field.

The digital marketing industry is a vast and expanding market, with projections for the global digital marketing industry to reach $671.86 billion by 2028, and with a forecasted compound annual growth rate of 13.1% from 2023 to 2028. The integration of data analysis has the potential to further enhance its growth. According to research conducted by the Data & Marketing Association, companies that adopt a data-driven approach to marketing outperformed those that don't by a significant margin, with a **20% increase in revenue growth and a 15% improvement in marketing efficiency**. Additionally, according to Gartner, companies in the digital marketing sector that embrace data-driven decision making are **six times more likely to retain customers** than who don't.

So, in a moment, I will delve into the topic of data analysis in the digital marketing industry. I will outline different techniques for generating impactful visualizations that can be applied in the digital marketing field and beyond. The article will also include examples from our data analysis team's projects, such as creating a funnel chart to assess the retention rate of an advertisement and a decay time series chart to identify when a downward trend in an advertisement's performance is occurring, utilizing the Python programming language

# Libraries & Setup

At first let's install and import some libraries that we will be used through this whole visualization production

```python
import numpy as np
import pandas as pd
import re
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
from plotly import graph_objects as go
from scipy.interpolate import interp1d
from plotnine import ggplot, aes, geom_smooth, geom_bar, labs, theme_minimal, scale_x_date, facet_wrap, facet_grid, expand_limits
```
We utilized various libraries including matplotlib, ggplot, and plotly in order to generate the visualizations in our analysis, which required importing a number of libraries. Despite the need for multiple imports, we ensured that the code is simple and efficient in producing the desired visualizations.

## Retention Rate Analysis

When evaluating the performance of advertisement creatives, it is important to understand the retention rate of users from initial impressions to clicks and continued use seven days later. To determine this performance, we created a funnel chart that provides answers to these questions.

![ads retention funnel chart](/_images/funnel_chart.png)

The chart above is a plotly chart that can give you much information only from 1 chart visualization. it can give you the number of users in each phase, the percentage difference between one phase to previous phase, and also the percentage difference between one phase to the initial phase. Using this information only, we can gather some findings such as, which phase has a low retention rate compared to the average in the industry, or does the initial impressions already reached the target or not. We can also measure in general if the number of clicks, impressions start declining or not.

Since, plotly chart is an interactive plot, we can add many more tooltips that we want. The code snippets below is the code that we used to produced the plotly plot above. The data input format in the **x** is the total number of each phase, while for the **y** is the name of the phase.

```python
fig = go.Figure(go.Funnel(
    y = df_join.iloc[0,8:14].index,
    x = df.sum()[4:10],
    marker = {"color": "#D4AF37"}
    ))

fig.update_layout(title={'text':
"Advertisement Funnel", 'y': 0.9,
'x': 0.5, "xanchor":"center",
'yanchor': "top"},
plot_bgcolor='rgb(255,255,255)')

fig.show();
```

## Decay Rate Analysis

After analyzing retention rates, we jumped to the main point of this project which is examining the rate at which advertisements lose their effectiveness over time, also known as decay rate. 

For those who not familiar with the terms `decay rate`, basically It is  a measure of the reduction in magnitude or value of a quantity over time. In the context of advertising, the decay rate refers to the rate at which the performance of an advertisement, such as its click-through rate or conversion rate, declines over time. It is used to evaluate the effectiveness of an advertisement campaign and to identify when it is necessary to refresh or replace the creative material in order to maintain its performance.

This type of analysis (decay rate analysis) can aid creative creators in determining when to create new advertisements and what types of advertisements tend to have a longer lifespan. It can also help in creating more effective advertisements by analyzing the lifespan of previous advertisements.

The charts below is some of the decay rate plots that we have made to analyze our clients advertisement lifespan. We made these plots using `matplotlib` library.

![decay rate analysis](/_images/ctr_decay.jpeg)
![decay rate analysis](/_images/d1_decay.jpeg)

From those decay chart, we can see the time when a creatives start to have a downward trend, which also could be used as a warning alarm for the creator to make a new creatives.We made those insightful chart using this code below:

```python
figure, axis = plt.subplots(5, 1, figsize=(18, 30))

for i in range (0,5):
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

We created facet grid plots using the matplotlib subplots function to enable easy comparison of each video. The subplots function allows for a specified number of rows and columns in the plots, in this case **5** rows and **1** column. Additionally, we applied a smoothing process to the CTR values using the **"inter1d"** function from the `scipy` library to make the visualizations easier to interpret.

## Actionable Insights

Having analyzed the rate of decay using a time series decay chart and evaluated the performance of all creatives through a funnel chart, it is now time to provide our client with actionable insights to enhance their creative performance.

The first set of actionable items was derived from the decay rate analysis and identified patterns where the creatives showed a downward trend. Our advice to the client is to create new creatives before the downward trend begins, allowing them to determine the optimal and effective time to generate new content.

Additionally, we performed other digital marketing data analyses, such as tracking the Click Through Rate (CTR), D1 Retention, and D7 Retention over time for each group of creatives. All of these analysis, will be shown in the image below, and please pay attention to these four charts.

![retention rate analysis](/_images/d1_d7_retention.jpeg)

![CPM analysis](/_images/cpm_value.jpeg)
![CTR analysis](/_images/ctr_value.jpeg)

Did you find the results from the four charts above intriguing? If you thought they were created using similar code, then you are correct. Although each chart provides different insights, they can be produced using similar code, just by adjusting the parameters. Furthermore, a custom Python function can be created to streamline the process, allowing for easy reuse in the future by simply specifying the parameters. As a side note, I created these multi-line charts using the `ggplot` library. Here is the code used to generate the charts.

```python
(ggplot(df_plot, aes(x=df_plot.index, y=df_plot.d1_retention_rate, color=df_plot.videoTopic)) +
  geom_smooth()+
  labs(
      title="Average D1 Retention Rate through Time per Video Topic",
      x="Date",
      y="Average D1 Retention Rate",
      color="Video Topic"
  )+
 scale_x_date(date_breaks = "1 month", date_labels =  "%b %Y")+
 theme_minimal() +
 expand_limits(y=[0,0.44])
)
```

As previously mentioned, all of the charts are quite similar, and can be created by simply changing the y parameter in the `aes()` function. The code above, `df_plot.d1_retention_rate`, generates a plot for the d1 retention rate. By simply changing the value of y to `df_plot.d7_retention_rate`, `df_plot.cpm`, or `df_plot.ctr`, you can obtain the other three charts. However, remember to also change the title and y in the `labs()` function to ensure the chart accurately displays the relevant information.

Not quite related to visualizations topic, but we also attempted a cluster analysis using neural network models on the `creative thumbnail` to determine which type of `thumbnail is more likely to perform well in terms of CTR, CPM, and retention rate. The result of the clustering was then promptly analyzed based on various performance metrics (such as clicks and impressions). By examining the results (as shown in the picture below), our client will have a better understanding of which type of thumbnail performs the best and can use that information as a factor when creating new creative materials.

![cluster analysis](/_images/clustering.jpeg)

In conclusion, these are some of the visualizations and analysis we utilized to generate a report on creative performance, with a specific focus on the decay rate analysis. We hope that this article provides some inspiration on what to analyze when working with data, whether in the digital marketing industry or any other industry. By using the right tools and techniques, data analysis can help unlock valuable insights and drive better decision-making 😉.





