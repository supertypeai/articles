---

title: The power of data storytelling, and how to do it with Tableau
post_excerpt: Leveraging Tableau to tell compelling stories that inspire action
taxonomy:
    category:
        - knowledge
        - notes

---

# The power of data storytelling, and how to do it with Tableau

According to International Data Corp's (IDC) research, 163 zettabytes of data will be generated in 2025, a huge increase in volume. Lending to this extremely quick development, companies are stepping into new, mostly uncharted territories, having to make sense with a new face of reality in business and finding new tools to doing just that. 


![Difference between Visualization & Data Table](/_images/tableau_1.png)

Take a look at the data table in the left portion of the image above. Now internalize the complexity of the data -- these are cognitive load and possibly information overload for the uninitiated.

However, as soon as you contrast it with the data visualization to the right, it becomes clear that this data pertains to the growth of MSMEs (Micro, Small & Medium Enterprises) in Indonesia. The visual elements are much easier to comprehend, and the data is much more accessible. This is an illustration of how data visualization can be used to offset the cognitive load otherwise required to process, and comprehend data. Consequently, that ease of understanding translates into a more thorough analysis of the data, and an increase in the quality of the decision-making process.

There are several approaches to perform data visualization, and there are numerous at the disposal of those wanting to create visually appealing representations. Famous examples are Tableau, Microsoft PowerBI, and Google Data Studio.

I've picked Tableau for this article for the three distinct advantages it offers:

- A large selection for interactive visualization: Tableau offers a wide variety of visualization choices, including static and moving (non-static) visualizations; it even automatically suggests the best visualization for the data you have.

- Handling several sources and data types: Data can occasionally originate from sources other than a single spreadsheet file. With Tableau, you can combine data from multiple sources, including Excel, CSV, and SQL, and even from a variety of data types, such as text, numbers, and dates.

- Mobile-friendly dashboard: Tableau offers the Tableau Desktop application as well as Tableau Mobile, which you can use on an iPhone, iPad, Android, or mobile browsers for free. In addition to making it simple for you to create visualizations, Tableau also makes it easy for your audience to view them.

Now that we've covered my tool of choice, let's get started with the nitty gritty of data storytelling.

## 1. Discover the "What" and “Who”

Starts from your target audience. Who is your audience and what are you trying to convey? Make sure that the criteria have been established from the beginning since effective visualization, at its core, is really a medium for information. 

You need to be able to decide at this point what information you want to provide, why it is significant, and how you will persuade your audience. I find the 5W + 1H question formula to be helpful in this process.

Data storytelling is about formulating a story that is both interesting and informative. In this particular example, I start with the thought of, "How are MSMEs developing and contributing in Indonesia?" 

This process might feel a bit overwhelming at first, but it's important to remember that you don't have to answer all of the questions at once. You can start with the most important features or variables, then move on to looking at relationships between them.

At times, you may find that you have too much information to convey. In that case, you can narrow down your focus to a specific aspect of the data. For example, in this project I wanted to center the theme around MSMEs in West Java, Indonesia.

Applying the 5W + 1H formula, this led me to the following questions:
- How is the development of MSMEs in Indonesia?
- What is the contribution of MSMEs in Indonesia?
- What are Indonesian MSME products?
- How is the digital economy ecosystem in Indonesia?

You might, at this point, pause to brainstorm for a bit. Having these inquiries are great prompts, but they're insufficient. You will want to probe deeper and generate more questions.

Here are examples of the questions stemming from the initial prompts:

-	What is the development trend of MSMEs in Indonesia? What are the factors that have the most influence on the progress of MSMEs? 
-	How much does MSMEs contribute to Indonesia in Gross Domestic Product, export, and employment?
-	What is the largest MSME sector in Indonesia?
-	How many MSMEs in Indonesia have adopted e-commerce? Is there any support provided by the Indonesian government for the growth of MSMEs in this sector?

We will use these 4 questions as a guide for our data storytelling.

At all times, it is crucial to keep in mind that you may produce more compelling, convincing narratives by rigorously researching the field and soliciting the opinions of others, particularly those who are knowledgeable about the issue at hand. You will also want to tailor your aesthetic to your audience, lending it a more personal touch. In my case, I kept my visualizations simple and straightforward, using a brown color tone and classic font style. My primary audience is the government of West Java, other academics, and MSMEs interested in the growth of this sector. Brown, I find, is a neutral color that is easy on the eyes, and the classic font style is easy to read and immediately familiar to the audience. Understanding that a segment of my audience are less statistically inclined, I made generous use of commentary to explain the data and employ simple, straightforward visuals like line charts and bar charts. No matter how valuable your insights might be, if you can't get your audience's attention, they won't be able to see, and consequently act on them.

## 2. Understand each **graph's function**

Each type of graph has its own function and role in data storytelling. Here are 5 of the most common types of graphs you will come across:

| Graph | Function |
|-------|----------|
|Bar Chart|Quickly compare data from all existing categories|
|Line Chart|Time series data can be displayed yearly, quarterly, monthly, or even days to show data patterns. It is simpler for the user to interpret the data and determine if it has risen or reduced when the visualization results are presented as lines
|Maps|All forms of location, zip code, and country information may be easily seen using maps. Maps are an easy and appealing approach to display how geography corresponds with trends in your data if consumers have geographic information connected to geospatial data
|Scatter Plot| a statistical tool for analyzing the correlation between two numerical variables|
|Pie Chart|It will split the circle into many sections according to our data. Typically better suited when there are fewer categories to compare and when the proportions are more important than the absolute values|

Have you ever imagined what happens when we use a chart that is not in accordance with its function? Let's look at the comparison below:

![Difference between Line Chart & Bar Chart](/_images/tableau_2.png)

Let's say we wish to see trends or patterns in the growth of MSMEs in Indonesia. I created a bar chart on the left side and a line chart on the right side to represent the data. 

It should be notable that the trend is more obvious in the second visualization. When you employ the right chart, your visualization tells a strong, persuasive, and convincing story. It increases the likelihood that your audience will understand, and resonate with the story you are trying to tell.
## 3. Create a **design layout**

Have you ever considered how a newspaper is written? Instead of being printed horizontally to complete a newspaper sheet, writing is split by column. The image in the headline news is often almost a third of a page in size, and it is followed by a title in a bold, huge font. At the top of the page are some advertisements. Why is that? Why aren't the advertisements placed at the bottom of the page?

The crux of the matter is something known as layout design. Layout design is the arrangement of the elements of a page, such as text, images, and advertisements. It is the way in which the elements are organized on a page. Because most people read from top to bottom, advertisements are always at the top of the page where they will be immediately visible to readers. Because most readers require a hook to pique their interest, the picture virtually fills a third of the page. Writing is divided into columns because most readers are better able to concentrate while reading a tiny portion as opposed to something that appears large and complicated. Writing is also from left to right since that is how people read. That is a straightforward instance of using layout design.

Similarly, the arrangement of charting and narrative elements on a dashboard or business intelligence report is crucial because it affects the way the reader perceives, consumes, and subsequently, digests the information.

Here are useful tips for creating a design layout:

-	Research: Always do as much research as you can before creating a design. Find out how other data storytellers / illustrators present information effectively. 

-	Know the Hierarchy: Similar to what you have seen in the hierarchy of newspaper writing as an example. Make sure the most crucial information is conveyed first in the build up of your story. 

-	Review, ask for feedback and improve: Once you've settled on a layout, ask for feedback from your peers. You may also ask yourself the following questions:
    1. Have I informed the reader of the most important information?
    2. Have I used attributes like font and color consistently in your work and does the information flows naturally from one section to the next?
    3. Are there information I can remove to streamline the design, resulting in a more straightforward and direct story?

The three suggestions above are quite helpful and may act as guidelines for you as you continue to create layout designs. In my dashboard design, I have used a simple layout design, illustrated below:

![Example of Design Layout](/_images/tableau_3.png)

## 4. Color palette & fonts

Colors play an important role in data storytelling. They can be used to convey a message, evoke an emotion, and even influence the way we think. The use of color in data storytelling is crucial because it can help you convey your message more effectively. Here are some tips:

1. Utilize the company's or target's logo color as inspiration: For example, if you're creating a visualization for Garuda Indonesia, you may use the fundamental color palette of white, tosca, and blue. You can utilize the foundation colors of purple and blue when producing visualizations for Baskin Robbins, or red and yellow when creating visualizations for Lego.
 
![Example of Company Logo Color](/_images/tableau_4.png)

2. Learn the significance of each color:Employ colors in accordance with the feelings and objectives you wish to portray. You can use a tool like ColorPalettes to quickly scan for color combinations and bring it into your storytelling.

![Example of Color Pallete website](/_images/tableau_5.png)

An illustration of a website presentation using color palettes is shown above. I want to utilize green in this case, but I'm not sure what colors will look well with it. I can use a tool to find complementary colors that will work well with green.

Alongside colors, font is also an important element in data storytelling. Broadly speaking, there are 4 font families, and they're Sans Serif, Serif, Decorative, and Script respectively.

![Example of Typography](/_images/tableau_6.png)

While the second font has a heavier accent and seems stiffer, the first appears extremely clear and is easy to read. A classic typeface with an air of elegance can be seen on the right side, along with the second font from the bottom right that is aesthetically pleasing but a little challenging to read. 

# Creating the Charts: A Tableau tutorial

Now that we have the building blocks of our visualization, let's put all of this together with Tableau. Make sure you have downloaded Tableau Public, open the application, click "File" in the upper left corner, then click "New" to create a new file 

![chart tutorial 1](/_images/tableau_7.png)

2. As you click "New," a screen similar to this one will appear. Select Data Source next to "Sheet1" in the bottom left corner.

![chart tutorial 2](/_images/tableau_8.png)
 
3. After selecting "Data Source", the screen shown below will appear. Pay attention to the blue "Connect" tab on the left side of the screen. We can access the data we have from here. Tableau supports processing a variety of file types, including Microsoft Excel, text files, JSON, PDF, and others. We'll utilize an Excel-extension file in this lesson. As we pick "Microsoft Excel," a new tab will instantly open, allowing us to choose the intended file.
 
![chart tutorial 3](/_images/tableau_9.png)

4. The analysis is the next stage. Tableau sees the "Year" column as an integer even though it should be of the datetime data type. We will need to convert the data type from integer to datetime. In addition to datetime, we may also perform other data transformations as required, converting data types into the appropriate types wherever necessary.

![chart tutorial 4](/_images/tableau_10.png)

5. When we have finished processing the data, we can start the visualization by clicking onto “Sheet1” tab; our data is automatically loaded in Tableau and ready for us!
 
![chart tutorial 5](/_images/tableau_11.png)

6. Tableau has a relatively gentle learning curve. At the top of Tableau, there are areas for columns and rows. As the name suggests, we may drag the variables we wish to see in this area. I drag the year variable into the column area and the number variable into the row section because I want to observe the trend in the number of MSMEs over the years.
 
![chart tutorial 6](/_images/tableau_12.png)

7. Tableau will suggest a chart that it believes best illustrates our data by default, but we can easily select a different graphic by selecting "Show Me" in the top right corner. We may utilize a variety of visualization techniques there. I decided to try replacing it with a bar chart this time, and this is what we got: 

![chart tutorial 7](/_images/tableau_13.png)

8. This is the graph that we used as a comparison in the previous step, and of course it is not optimal as it doesn't convey the trend as effectively as a line chart could. In addition to have a line chart, you may also try altering the color by clicking on the color area picking your choice. You might want to experiment with the opacity as well to see what works best for your visualization.

![chart tutorial 8](/_images/tableau_14.png)
 
9. Last but not least, make sure to add commentary relating to your visualization. Commentary is a crucial part of data storytelling. It helps you to explain the data and the story behind it. You can also add a title, axis label, tooltip, and legends to your visualization.
 
![chart tutorial 9](/_images/tableau_15.png)

- **Title** communicates the main story in your visualization. On the picture above, we can tell the purpose of this visualization and resonate with it.
- **Axis Label** helps your reader understand the meaning of each axis. In this case, we know that x corresponds to the Year axis and y is the total number of MSMEs.
- **Tooltip** helps your audience understand the meaning of each data point.
- **Legends** adds detail about the meaning of color we use. By default, a legend will be created whenever your visualization is using color, size, or shape to represent a dimension. You can customize the legend by right-clicking on the legend card and selecting the option you want.

Before you move on, be sure to Edit Title or Edit Caption using the context menu. With the appropriate use of each visual element, we have created a visualization that is easy to understand and tells a story.

![chart tutorial 15](/_images/tableau_21.png)

# Telling the Story: Doing more with Tableau

Let's start by using the layout design as a guide. We divide the dashboard into 2 parts vertically and made space to incorporate the visualizations we've created.

![dashboard tutorial 1](/_images/tableau_25.png)

We can then bring in our text object and the various charting elements we've created. 

![dashboard tutorial 4](/_images/tableau_28.png)

To better resonate with the reader, you may bring in a picture of the person or subject you're talking about. In this dashboard, I've added a picture of Mr. Teten Masduki, Indonesia's Minister of Cooperatives and Small and Medium-Sized Enterprises. This helps draw attention to the subject of the dashboard, and adds flair to your story.

![dashboard tutorial 5](/_images/tableau_29.png)

Our dashboard is almost ready. I added a big, bold font to the left of the dashboard to draw the audience's attention to that title. I added a subheading, _Bangkit Bersama, Ekonomi Maju_ which roughly translates to _Rise Together, Economy Flourishes_. This a slogan that evokes emotions and helps the reader feel the urgency of our work in helping MSMEs thrive, not just survive, coming out of the Covid-19 pandemic.


![dashboard tutorial 6](/_images/tableau_30.png)

Speaking of emotions and urgency, I added a picture and a quote from the Indonesian President, Joko Widodo. This marvelously capture the spirits of the Indonesian people and the government's long term commitment to helping MSMEs thrive.
 
![dashboard tutorial 7](/_images/tableau_31.png)

### Winning the Audience's Conviction

How else do we rally the audience to our cause? One way is to show them the role that MSMEs play in the Indonesian economy. Here we have a small dilemma: adding more charts to the dashboard might dilute the key message and make the dashboard look cluttered. We could have a section containing tables to illustrate the roles of MSMEs, but tables tend to be ignored when compared to charts. 

I opted for a third option, using large, bold statistics to convey the message. This engages the audience and makes them curious about the data, which draws their attention to the smaller fonts describing each statistic.

![dashboard tutorial 8](/_images/tableau_32.png)

Displaying the importance of MSMEs in the Indonesian economy is a great way of winning conviction, but humans are visual beings and we are creatures of empathy. Here is an opportunity to include the faces of the subjects, or some illustrations of the very people that are at the center of this story. 

![dashboard tutorial 9](/_images/tableau_33.png)

The dashboard at this point feels sufficiently rich in information density, but sometimes less is more. Upon further inspection, I decided to eliminate the caption for the line chart and bar chart. The title on this dashboard has clearly states its purpose, so by removing the caption, the story is more focused and the dashboard is less cluttered.

Even better, without everything clearly spelled out of the audience, they are encouraged to internalize the story and fill in the tiny details themselves. Small and medium businesses are the backbone of the Indonesian economy, and they are the ones that will help Indonesia recover from the Covid-19 pandemic. By using a combination of visual elements and direct, straightforward statistic, we have made our case and left enough room for the audience to ponder and reflect on the story we've told.
 
![dashboard tutorial 10](/_images/tableau_34.png)

### Data Storytelling: The Purpose 

![dashboard final](/_images/tableau_35.png)

In the "Before" image, we have a dashboard with charts, lines and bars, but no coherent story. To the more statistically inclined, this dashboard serve an educational purpose -- it informs, but it does not inspire.

In the "After" image, we add details and narrative elements to guide our audience as they navigate through each section, similar to a reader moving from one chapter of a book to another. Data storytelling, when done well, can be a powerful tool to help us understand the world around us. In this case, it can help us empathize with the people we are trying to help, creates urgency in the mind of the readers, and rally them to our cause in a persuasive, engaging manner. 

Data storytelling, at its core, is about creating a narrative that is memorable and impactful. It is as much art as it is science, and you will have to call on your creativity and empathy. It's what makes this a rewarding experience, and I hope my article has inspired you to try your hand at it.

