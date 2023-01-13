---
title: Applying data analytics to digital advertising (A workflow recap)
post_excerpt: How Supertype's data scientist help a digital advertising client increase response rate by more than 20% (a workflow recap)
taxonomy:
    category:
        - knowledge
        - notes
---

Digital advertising is a booming industry in the world, and it is expected to grow even more in the future, reaching US$701.20bn in 2023 -- more than half of which coming from mobile advertising.

The Interactive Advertising Bureau (IAB) states that business that uses data to target their digital analytics achieve three times of click-through rate (CTR) over their advertising peers who have yet to use data.

> Data is the fuel that powers the engine of digital advertising. Without data, digital advertising is just a guessing game.

How does data analytics and data science assist digital advertisers? This forms the main topic of this article, and coming off the back of a successful project where my team and I were able to **increase the response rate of the advertisement by more than 20%**, I will be sharing some of the insights that we have gathered.

## Background: Turning data into analytics for our digital advertising clients

Being a data analytics company that provides data analytics services to our clients, our most recent project is to analyze the data of our clients' digital advertising campaigns and provide them with insights that can help them improve their advertisement performance. The client brief was open-ended, and we were given a lot of freedom in terms of the analysis that we could do.

The instinct of many data scientists is to use the most advanced analytics method to solve the problem, but in this case, we started by diving into the data and understanding the business perspective of the client. We then used the business perspective to guide our analysis and to identify the parameters that affect the performance of the advertisement. Through this all, the client was able to achieve a 20% increase in response rate. 

A key lesson, and probably a tired cliché, is that **data science is not just about the data**. It is about the empathy in understanding the business objectives, and the creativity in finding the right solution to the problem. Instead of finding a business case where deep learning is the solution, start from the business value and decide if deep learning is the right tool for the job.

What you want to avoid is putting in the number of hours into building a complex model that end up on the shelf, never to be productionized or used.

## 1. Inherent familiarity with digital advertising

To keep our analysis on track, the data scientist and analysts need to be intimately familiar with the digital advertising industry. This is because the data that we are given is raw data, and the data scientist will comb through tens of thousands of variables that might or might not be relevant to the problem at hand. Terms like click through rate (CTR), impressions, revenue per thousand (RPM), and cost per thousand (CPM) are common in the any digital advertising campaign, and it is important to understand what these terms mean. 

Our client operates in the supply side platform (SSP) industry, which is a platform that connects advertisers and publishers. The client's main business is to sell ad space to advertisers, and the client's revenue is generated from the advertisers' ad spend. Since the primary clientele are in the mobile advertising space, the way advertising attribution work involves the use of identifiers such as the IDFAs (Identifier for Advertisers) and the GAIDs (Google Advertising IDs). These identifiers are used to track the user's journey from the initial click to the final conversion, and are distinctly different from how cookies work in the desktop advertising space. Add to this the real-time bidding (RTB) element of the supply side platform, and you have a complex system that requires a deep understanding of the industry. RTB, for the uninitiated,
is a process where advertisers bid for ad space in real time, and the advertiser with the highest bid wins the ad space. The advertiser's bid is based on the value that they place on the ad space, and the value is determined by the advertiser's target audience and the ad's relevance to the target audience.

Data scientists and analysts lacking a fundamental understanding of how ad exchanges operate will find it difficult to understand the data that they are given, and their analysis will quickly reflect this gap in knowledge. A fundamental phase of any data science project is thus to first understand the business landscape, and keep a clear lane of two-way communication with the client to ensure that the analysis is on track.

Here are some of the questions that we asked the client to help us understand the business landscape:

- What is the client's business model?
- What is the (current) role of data in the client's supply side platform?
- How are advertisement campaigns set up?
- How is an inventory priced?
- How would an increase in conversion rate or click through rate affect the client's revenue?

Your goal is to have a clear direction of what you are trying to achieve, and a method in filling your knowledge gap systematically -- by directing the right questions to the right personnel.

## 2. Identify, then double down on key parameters that affect advertising performance

The next phase is to mobilize the data that we have received and formulate a plan for the analysis, rooted on a solid understanding of the business landscape and business model. 

In the case of our client, we did some preprocessing and enriched the data with additional information such as the app category, the app name, and the app publisher. We gather information on important variables in the data, with one such example being the **app_site** -- the app that the advertisement is served on, most likely one where a **valuable event** (registration, first deposit, tutorial completion etc) has occurred.

We then use the **app_site** to identify the top 10 apps that bring the biggest monetary value to the client, whether in the way of revenue, or one of the predefined events ("valuable event"). Different strategies are formulated to help the advertisers on the SSP to allocate advertising budget toward their respective top apps. Further analytical techniques such as scaling and clustering are used, to avoid the bias of the top apps being the ones with the most impressions. 

![ads optimization top occurrence](/_images/top_occurrence.jpeg)
![ads optimization scaled occurrence](/_images/top_occurrence.jpeg)

Beyond that, our analysis also highlight a significant opportunity for advertisers to further improve their ad performance by taking into consideration the time where an ad impression is served ("serve time") and correlating it with the possible of a valuable event occurring ("valuable event"). We sent the analysis back to the client, and after a quick trial, notice an uptick in the conversion rate of the advertisements. Since each **valuable event** has its own distinctive pattern of occurrence and its own correlation with the **serve time** parameter, the combinatorial possibilities of targeting the right audience at the right time is exciting and reap for some multivariate analysis.

It is key to communicate these analysis plainly and clearly to the client, to ensure that the outcome of your data analysis is understood and actionable. An example of our communication is "An ad serve time of 14:00 to 18:00 has the highest conversion rate for all valuable events, and the serve time extending to include 00:00 to 05:00 will yield the highest conversion rate for the install event per thousand impressions".

![dsp optimize hour parameter](/_images/hour_parameter.jpeg)

Other, less orthodox parameters we found include the longitude and latitude of the ad impression, and when we incorporate these into the analytics model, this opens up new perspectives on how locations affect the performance of an ad.

![dsp optimize place parameter](/_images/place_parameter.jpeg)
![dsp optimize category parameter](/_images/category_parameter.jpeg)

Equally as important is to not get caught up in a never-ending cycle of analysis and data exploration. Once we've identified the key parameters and test for their significance, we communicate the results to the client to move these into "real world" testing.

## Deliver Your Findings

Delivering your analysis requires a different set of skills, and as it has been repeated above, plain and clear communication is key to ensuring that your analysis is understood and actionable.

You might choose to present your information in the form of a report, or a presentation. The report should be concise and to the point, and action items should be easily gleaned from the report. In many cases, you might also have to pick and choose which findings to present, as there might be too many findings that are unactionable, inconsequential, or might already be common knowledge to the client (eg. "first person shooter games have a strong male bias, and ad impressions convert better with the 25-35 age group"). Picking the right findings to present, and then organizing them in a way that is easy to action upon is a skill dependent on your experience, as well as the time investment you make into understanding the business domain (point 1) and the data (point 2).

Here are my suggestions on delivering your findings:
- Provide a section upfront with a summary of your findings, and a list of actionable items. This will help the reader to quickly understand the key takeaways of your analysis, and also provide a list of tasks to complete or actions to take.
- Demonstrate an emphasis on business value using quantitative measures, such as the increase in conversion rate or the increase in revenue. This will show the reader that you care about the impact of your analysis on the business.
- Know your audience; Use simple charts and graphs that are easy to understand, and avoid using charts that are too complex or have multiple facets requiring interpretation beyond the comfort level of your audience.

## Follow Up on your Analysis

Your journey doesn't end here. The data analyst should promptly and periodically follow up on the actions that the client has taken based on your analysis. This is to ensure that the actions taken are effective, and to also identify any new opportunities for improvement. 

If the actions taken are not effective, corrective actions and explanations should be offered and discussed with the client. Data scientists should be highly engaged in this feedback cycle, and not take a hands off approach beyond the initial analysis.


## Closing Notes

While this article demonstrates the work cycle that we have implemented for a specific client, the workflow is applicable to any data analysis project. At Supertype, we obsess over the client's business model and we never take on a project where we cannot be confident of our ability to deeply understand the business domain. 

Once a project is on board and a baseline knowledge of the business value has been agreed, we then proceed to gather the data, and then formulate a plan for the analysis. We then deliver our findings to the client, and follow up on the actions taken by the client. A side effect of this all is the ease in producing well-documented and well-communicated analysis, supplemented by a clear and actionable plan for the client to follow, or any follow-up training that might be required.

If you are interested in learning more about our work, check out our articles section or hop over to the case study section to read more about how Supertype might be able to help you with your data science and analytics needs.