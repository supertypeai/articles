---
title: Decoding the Decline of Advertisement Creatives in Advertising -- An Analysis
post_excerpt: Key visual representation that can provide significant insights into the rate at which advertisements lose their impact and methods to improve it
taxonomy:
    category:
        - knowledge
        - notes
---

Do you know that almost **two-thirds** of the current world's population are connected to the internet on a daily basis? Not to mention that this number will keep growing as mobile phones have provided more ease of access to the internet. Combined with the emergence of data-driven paradigm, businesses realize that a shift has occurred in the marketing industry — specifically on how marketing should be conducted in order to promote a brand, convey information regarding their products, and lure in potential customers. A recent research by Data & Marketing Association has shown that integrating the use of digital communications and adopting a data-driven approach enables a marketing strategy to outperform those that don't by a significant margin — a **20% increase in revenue growth** and a **15% improvement in marketing efficiency**. Not only that, companies in the digital marketing sector that embrace data-driven decision making are proven to be **six times more likely to retain customers**.

Despite all of the astonishing benefits of data in the digital marketing industry, lots of businesses still face a major issue. *"Precisely what kind of data do we need? What kind of analysis do we need to conduct? How can the result help us leverage our marketing strategy?"* So, in a moment, I will delve into the topic of data analysis in the digital marketing industry, outlining different techniques for generating impactful analysis and visualizations that can be applied in the digital marketing and other similar industries. The demonstration that will be shown in this article is based on our real-world data analysis projects, putting into spotlight the fact that generating a thorough analysis from which numerous valuable insights could be derived is actually simple and straightforward, given the proper analysis method was conducted.

# Libraries & Setup

Let's first import some libraries that will be used throughout the demonstration. You can use the `pip install` command to install any of the libraries that are not installed in your virtual environment.

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

If you have some prior knowledge regarding the aforementioned libraries, you might have guessed that the upcoming demonstration will be heavily focused on conducting analysis by generating visualizations. A question that pops up on your mind might be *"why do we need to import multiple visualization libraries?"* This question is natural, considering that we are importing multiple visualization libraries such as **matplotlib**, **ggplot**, and **plotly** — it seems very redundant! However, throughout the visualization, I am going to show you that the orchestration between these libraries can produce powerful, effective visualizations with minimal code.

## Retention Rate Analysis

Suppose that you are trying to evaluate the performance of advertisement creatives, what metric do you think will fulfill the task best? The majority of people would immediately answer *"it would be the number of clicks, as it shows whether the viewers are interested in our advertisement"*. While that answer was not completely incorrect, take a moment to think of the main purpose of advertisements. In my opinion, as the main purpose of advertisement creatives is to obtain users, the number of clicks alone is not sufficient to measure its success — there might be users that only click and see the ads, but decide not to do anything else! This signifies the importance of understanding the retention rate of users, which can immediately answer 3 questions in one go:

1. Does the advertisement creative have good impressions?
2. Are the users interested to "click" upon seeing the advertisement creative?
3. Do we get continuous usage within a period of time?

A simple way to conduct retention analysis is creating a funnel chart! Take a close look at the following visualization:

![ads retention funnel chart](/_images/funnel_chart.png)

That interactive chart alone can provide a lot of information, such as the number of the number of users in each phase, the percentage difference between a phase to previous phase, and also the percentage difference between a phase to the initial phase. By utilizing this information further, we can generate some meaningful insights:

- Which phase has a low retention rate compared to the average in the industry?
- Have the initial impressions already reached the target?
- In general, does the number of clicks and impressions start declining within a specific period of time?

The best thing about that chart is in addition to the information it provides, it is fairly easy to make. We can use plotly to create a plot and add multiple tooltips of our choice, making the plot highly interactive. Consider the following lines of code that were used to generate the previous chart. Refer to the comments for details regarding the data input format.

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

We conducted a retention rate analysis and ended up with some meaningful information, but how do we take a step further and dive into more details? A good way of moving on is by understanding what analysis complements the prior. Following this thought process, we can jump to the main point of this demonstration — examining the rate at which advertisements lose their effectiveness over time, also known as the **decay rate**.

For those who are yet to be familiar with the term decay rate, consider it as a measure of the reduction in magnitude or value of a quantity over time. In the context of advertising, the decay rate refers to the rate at which the performance of an advertisement, such as its click-through rate or conversion rate, declines over time. It is used to evaluate the effectiveness of an advertisement campaign and to identify when it is necessary to refresh or replace the creative material in order to maintain its performance.

Decay rate analysis aids creative creators in determining when they need to create new advertisements and what type of advertisements tends to have a longer lifespan. It can also help in creating more effective advertisements by analyzing the lifespan of previous advertisements. The following charts are some of the decay rate visualizations that we have made to analyze our clients' advertisement lifespan.

![decay rate analysis](/_images/ctr_decay.jpeg)
![decay rate analysis](/_images/d1_decay.jpeg)

As you might have already noticed, the chart actually represents the *"trend"* of the advertisement campaign. Since the X axis is the time signal, i.e. how many days have passed since the advertisement was published, it can be used as an indicator showing when should the creators refresh their creatives. The following code snippets were used to make those insightful charts:

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

As a quick elaboration, the `matplotlib` library was used to create 5 subplots (1 subplot for each video) with the purpose of enabling quick comparison between each video's decay rate. Additionally, we applied a smoothing process to the CTR values using the **"inter1d"** function from the `scipy` library to make the trend line, which makes the plots easier to be interpreted.

## Actionable Insights

Upon analyzing the rate of decay using a time series decay chart and evaluating the performance of all creatives through a funnel chart, we can already provide our client with actionable insights to enhance their creative performance.

The first set of actionable items was derived from the decay rate analysis and identified patterns where the creatives showed a downward trend. We can advise the client to create new creatives before the downward trend begins, allowing them to determine the optimal and effective time to generate new content.

Additionally, we performed other digital marketing data analysis, such as tracking the Click Through Rate (CTR), D1 Retention, and D7 Retention over time for each group of creatives. All of these analysis will be shown in the following image, try to take a closer look at these following charts!

![retention rate analysis](/_images/d1_d7_retention.jpeg)

![CPM analysis](/_images/cpm_value.jpeg)
![CTR analysis](/_images/ctr_value.jpeg)

Did you find the results from the four charts above intriguing? If you thought they were created using similar code, then you are completely correct! Although each chart provides different insights, they can be produced by using similar commands, just different parameters. So if you intend to use the code in the future, you can easily create a custom Python function to streamline the process by turning the input into parameters. As a side note, I created these multi-line charts using the `ggplot` library. Here is the code used to generate the charts.

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

As it was previously mentioned, all of the charts are quite similar, and can be created by simply changing the y parameter in the `aes()` function. The code above, `df_plot.d1_retention_rate`, generates a plot for the d1 retention rate. By simply changing the value of y to `df_plot.d7_retention_rate`, `df_plot.cpm`, or `df_plot.ctr`, you can obtain the other three charts. However, remember to also change the title and y in the `labs()` function to ensure the chart accurately displays the relevant information.

In addition to the visualizations we made, we can also provide better insights by conducting a cluster analysis using neural network models on the `creative thumbnail` to determine which type of thumbnail is more likely to perform well in terms of CTR, CPM, and retention rate. The result of the clustering can then be promptly analyzed based on various performance metrics (such as clicks and impressions). By examining the results (as shown in the picture below), our client will have a better understanding of which type of thumbnail performs the best and can use that information as a factor when creating new creative materials.

![cluster analysis](/_images/clustering.jpeg)

Concluding everything that we have done, this article demonstrates a guide on some visualizations and analysis that we can make to generate a report on creative performance, with a specific focus on retention rate and decay rate analysis. We hope that this article provides some inspirations on what to analyze when working with data, as well as enhancing your thought process when conducting analysis, be it in the digital marketing industry or other similar industries. Always remember that with the right tools and techniques, data analysis can unlock valuable insights and drive better decision-making 😉.

> If you are highly interested in the digital marketing industry, I highly recommend checking out our previous <u>[previous post](https://supertype.ai/notes/ads-optimization-analytics/)</u>. The article contains our detailed approach to a consulting project in the digital marketing field, providing essential information regarding the industries along with a workflow recap.