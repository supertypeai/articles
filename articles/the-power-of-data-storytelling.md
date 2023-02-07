---

title: The power of data storytelling, and how to do it with Tableau
post_excerpt: Secrets to start making your first visualization in Tableau
taxonomy:
    category:
        - knowledge
        - notes

---

# The power of data storytelling, and how to do it with Tableau

How much memory space are you familiar with? Megabytes? Gigabytes? Terrabytes? But do you know what a Zettabyte is? One trillion Gigabytes is equal to one Zettabyte of memory. In reality, you won't have to wait long to get quite familiar with the Zettabyte unit since, according to International Data Corp's (IDC) research, 163 zetabytes of data will be generated in 2025, a huge increase in volume. Because of this extremely quick development, the phrase "Data is the new oil" has become so prevalent. But underlying all of that, there is one critical step that data professionals sometimes overlook or ignore, namely Data Visualization.

![Difference between Visualization & Data Table](/_images/tableau_1.png)

How much time did it take you to comprehend the significance of the information in the table on the left? 10 minutes probably won't be enough, especially with that complexity.  However, as soon as you contrast it with the data visualization to the right, it becomes clear that this data pertains to the growth of MSMEs in Indonesia. You will receive information on which MSME items are selling the best on the market, etc., in just a few minutes. That is how people gather information. People are able to concentrate on a certain area since they are helped to comprehend the essential details necessary for their line of work by the visualization. Data visualization not only simplifies difficult-to-understand data tables but also enables thorough data analysis and the making of prudent decisions.

“All roads to the Rome”. There are several approaches to accomplish our objectives, and there are numerous tools available for data visualization that we may utilize to create visually appealing representations, including Tableau, Power BI, and Google Data Studio. Although each tool has its benefits, Tableau is the finest ones. This is clear from the fact that the Gartner Magic Quadrant has awarded the category of Analytics and Business Intelligence Platform apps with the title of best application for 8 years running. Data blending, real-time analysis, and data collaboration are some outstanding features that are exclusive to Tableau. These are, generally speaking, the three benefits of Tableau that I have experienced:
- Interactive visualization: There are several visualization choices available, and Tableau can also automatically suggest any visualization that corresponds to the data we have. Tableau also provides a wide variety of moving/non-static visualization choices.

- Handling several sources and data types: Data can occasionally originate from sources other than a single spreadsheet file. Even more complicated, you may now utilize file extensions other than just xlsx, such as csv, json, etc. You can handle data in Tableau extremely easily because of the data blending capability.

- Mobile-friendly dashboard: Tableau offers the Tableau Desktop application (use on a PC/Laptop) as well as Tableau Mobile, which you can use on an iPhone, iPad, Android tablet, or mobile browser for free. In addition to making it simple for you to create visualizations, Tableau also makes it easy for your audience to view them.
Is Tableau free given all these benefits? Yes! You may start learning using Tableau Public for nothing at all to hone your Tableau operational abilities.

In conclusion, data visualization is an essential component that must be included in any data processing process. However, thanks to Tableau, we no longer have to worry about developing a visualization dashboard from the ground up. After this, I'll show you how to create a visualization dashboard, which is shown in the first paragraph from scratch. Though it appears complicated, it's actually pretty simple to comprehend. Let's work on this together!

## 1. Discover the "What" and “Who”

Who is the target audience and what are you trying to convey? Make sure that the criteria have been established from the beginning since visualization is more than simply an aesthetic art; it is also a source of information. You need to be able to decide at this point what information you want to provide, why it is significant, and how you will persuade your audience. I find that using the 5W + 1H question formula is the simplest method. You could start from the dataset. Focus on every row and column, then begin formulating a single question that captures the urgency of the facts. In my situation, I immediately wondered, "How are MSMEs developing and contributing in Indonesia?" If you have never understood a complicated dataset, you will probably be perplexed since you must comprehend the significance of each column, search for links between variables, and carry out other essential analyses that, of course, take a lot of time. It's very natural, and there's no need to worry since I have a lot easier solution. Close all the dataset files you had open previously, sit down, and try to come up with one major subject, specifically "Indonesian MSMEs." Using the same 5W + 1H formula, my natural curiosity led me to consider the following issues in less than 5 minutes,:
- How is the development of MSMEs in Indonesia?
- What is the contribution of MSMEs in Indonesia?
- What are Indonesian MSME products?
- How is the digital economy ecosystem in Indonesia?

How about you? Are you thinking the same thing? We were able to create concise goals from these four first queries. These inquiries, however, are insufficient, so we must work to assemble as much data as we can. Imagine that you are the audience as a business owner and that one of your employees is presenting a line chart with the simple statement, "Our sales have increased for a year." Does that suffice? No, you will undoubtedly answer with a variety of questions, such as what percentage the growth is, why there may be an increase, when sales started to show a good trend, what is our future plan, and how it relates to the state of the world economy. The same goes for creating visualizations; make sure your work is as thorough as you can. Here is an example of how to formulate the four questions mentioned above:

-	What is the development trend of MSMEs in Indonesia? What are the factors that have the most influence on the progress of MSMEs? 
-	How much does MSMEs contribute to Indonesia in Gross Domestic Product, export, and employment?
-	What is the largest MSME sector in Indonesia?
-	How many MSMEs in Indonesia have adopted e-commerce? Is there any support provided by the Indonesian government for the growth of MSMEs there?

We will use these 4 questions as a guide while creating visualizations. It's crucial to keep in mind that visualizations are not intended for you. As a result, you must adapt to what your intended audience wants to know. It is also desirable to additionally solicit the opinions of others, particularly those who are knowledgeable about the issue at hand. Speaking in more depth about the target audience, this is truly crucial for identifying the problem's core and affects the visualization we plan to design. The design outcomes of our visualization for the same issue will undoubtedly change based on the audience. As simple as, use sophisticated charts, such as waterfall charts, with strong and readable fonts to express your ideas clearly when creating visualizations for coworkers, colleagues, and other data scientists. However, when creating visualizations for children, it is preferable to use simple visualizations like bar charts, with appealing font styles and vibrant colors. In my case, I chose to utilize a brown color tone because the people who will be viewing my visualization are West Java government officials, academics, and MSME players. In addition to being neutral, brown is a sophisticated color to use in visualization. I write in the classic font, which is often used in writing for publications, posters, etc. Because there will be MSME players in the future who don't really understand statistics, I also employ some straightforward visuals like line charts and bar charts. Make sure you also do the same thing, because the audience is everything. No matter how valuable the information is, it won't reach them if we can't get their attention. Don't overlook this step since locating the roots of the problem is crucial to a visualization's effectiveness.


## 2. Understand each **graph's function**

Charts are inseparable when we talk about visualization, and each chart has its function, so make sure you choose the right chart type. Here are the 5 most popular types of visualizations that are frequently used in Tableau

| Graph | Function |
|-------|----------|
|Bar Chart|Quickly compare data from all existing categories|
|Line Chart|Time series data can be displayed yearly, quarterly, monthly, or even days to show data patterns. It is simpler for the user to interpret the data and determine if it has risen or reduced when the visualization results are presented as lines
|Maps|All forms of location, zip code, and country information may be easily seen using maps. Maps are an easy and appealing approach to display how geography corresponds with trends in your data if consumers have geographic information connected to geospatial data
|Scatter Plot| a statistical tool for analyzing the correlation between two numerical variables|
|Pie Chart|It will split the circle into many sections according to our data. If there are no more than five sections, this chart will be more suitable for usage since if there are more, the pie chart becomes more challenging to read|

Have you ever imagined what happens when we use a chart that is not in accordance with its function? Let's look at the comparison below:

![Difference between Line Chart & Bar Chart](/_images/tableau_2.png)

We occasionally wish to see trends or patterns based on the statistics on the growth of MSMEs in Indonesia that was mentioned before. I created a bar chart on the left side and a line chart on the right side to represent the data. Isn't the trend more obvious in the second visualization? It is crucial that we utilize the correct chart since the incorrect chart increases the likelihood that the audience will not understand the information you are trying to express.

## 3. Create a **design layout**

Have you ever considered how a newspaper is written? Instead of being printed horizontally to complete a newspaper sheet, writing is split by column. The image in the headline news is often almost a third of a page in size, and it is followed by a title in a bold, huge font. At the top of the page are some advertisements. Why should that be the case is the question. Why not have advertisements on the bottom? Why not make the picture smaller?
Layout design is the answer to each of these issues. Because most people read from top to bottom, advertisements are always at the top of the page where they will be immediately visible to readers. Because most readers require a hook to pique their interest, the picture virtually fills a third of the page. Writing is divided into columns because most readers are better able to concentrate while reading a tiny portion as opposed to something that appears large and complicated. Writing is also from left to right since that is how people read. That is a straightforward instance of using layout design.

The creation of a layout design will increase the effectiveness of your work in addition to making it simpler for the audience to grasp the flow of your design. Maybe you're wondering what the term "effective" actually means. While designing something, h ave you ever repeatedly searched Google for ideas? Have you ever continually changed images because you couldn't decide which one to use? Have you ever spent a lot of time deciding on the perfect color for your design? Or have you ever been unsure of the best font to use? Everything transpired as a result of your lack of an initial clear strategy. It's challenging to think and design at the same time, I assure you.  So, here are some things that I usually apply in making layout designs:

-	Research: Always do as much research as you can before creating a design. Find out how other designer / illustrator present information in an appealing and artistic manner. You may look for references on Dribble and Pinterest. These 2 apps are where artists' works congregate, therefore you'll get a lot of inspiration there. Moreover, you may go to tableau public there are many people's portfolios there.

-	Know the Hierarchy: Similar to what you have seen in the hierarchy of newspaper writing as an example. Make sure the most crucial information is conveyed first in your design. When using visualization, you may begin with the title or even the main visualization, which serves as the fundamental building block of the message you want to communicate. Second, check the composition of the text, graphics, and visualizations you employ. You can grasp the hierarchical design using a number of characteristics, including grid, alignment, color, and font. You may place your pieces using a grid to keep the columns and rows in the right order. Alignment, meanwhile, will make your design seem tidy and appealing. In the next phase, we'll become more specific regarding colors and typography.

-	Review, ask for feedback and improve: We occasionally manage to obtain one layout that is both excellent and attractive. As a result, after successfully creating your layout design draft , you may ask some of the questions below as an evaluation.
    1. Have I already informed you of what I was about to do in step 1?
    2. Have you used attributes like font and color consistently in your work?
    3. Is there information that is unnecessary and may be removed to make the design more straightforward and appealing?

The three suggestions above are quite helpful and may act as guidelines for you as you continue to create layout designs. As of right now, I have the following design layout:


![Example of Design Layout](/_images/tableau_3.png)

## 4. Color palette & fonts

A visualization is made more beautiful by the use of color. However, picking the incorrect hue can just exacerbate it. Use of dark and contrast hues is the most fundamental color-selection advice. Why? Because the audience can instantly notice it in ancient colors. Use just distinct colors; stay away from pastels. After that, adapt to your audience. There are two methods to choose a color:

1. Utilize the company's or target's logo color as inspiration: For example, if you're creating a visualization for Garuda Indonesia, you may use the fundamental color palette of white, tosca, and blue. You can utilize the foundation colors of purple and blue when producing visualizations for Baskin Robbins, or red and yellow when creating visualizations for Lego.
 
![Example of Company Logo Color](/_images/tableau_4.png)

2. Learn the significance of each color: The second option is to employ colors in accordance with the feelings and objectives you wish to portray. For those of us who aren't familiar with design, tip number two could be pretty challenging. But don't worry; I always have a simpler answer, which is to use [ColorPalettes](https://colorpalettes.net). We can quickly look for color combinations on this website, copy the color code, and utilize it in our Tableau application.

![Example of Color Pallete website](/_images/tableau_5.png)

An illustration of a website presentation using color palettes is shown above. I want to utilize green in this case, but I'm not sure what colors will look well with it. Therefore, there are 5 possible color combinations when using this colour palette. If you look closely, each color has a special code at the bottom, such as #1A2902. You may copy or write this hex color into the Tableau app's color selection.

Not only color, font also provides an important essence in a visualization. Broadly speaking, there are 4 font families namely Sans Serif, Serif, Decorative, and Script. What is the difference? Let's see in detail through the example below: 

![Example of Typography](/_images/tableau_6.png)

While the second font has a heavier accent and seems stiffer, the first appears extremely clear and is easy to read. A classic typeface with an air of elegance can be seen on the right side, along with the second font from the bottom right that is extremely aesthetically pleasing but a little challenging to read. You may begin selecting fonts after you are aware of the differences between utilizing them [MyFonts](https://www.myfonts.com). This website is really simple to use, and while trying with several fonts, you may get inspiration quickly.

## Creating the Charts

Now that you understand the fundamentals of data visualization, let's get hands on with Tableau. Make sure you have downloaded Tableau Public, and open the application, click "File" in the upper left corner, then click "New" to create a new file 

![chart tutorial 1](/_images/tableau_7.png)

2. When you click "New," a screen similar to this one will appear. Although there are many alternatives available to you, in this lesson we will only cover the fundamental method. To begin, select Data Source next to "Sheet1" in the bottom left corner.

![chart tutorial 2](/_images/tableau_8.png)
 
3. After selecting "Data Source," the screen shown below will appear. Pay close attention to the blue "Connect" tab on the left side of the screen. We can access the data we have from here. Tableau supports processing a variety of file types, including Microsoft Excel, text files, JSON, PDF, and others. We'll utilize an excel-extension file in this lesson. When we pick "Microsoft Excel," a new tab will instantly open, allowing us to choose the appropriate file.
 
![chart tutorial 3](/_images/tableau_9.png)

4. The analysis is the next stage. If you'll note, Tableau sees the "Year" column as an integer even though it should be of the datetime data type. As a result, we must choose the options below to convert the data type from integer to datetime. In addition to datetime, we may also switch to other data types as needed, like strings, Booleans, and geographic roles. 

![chart tutorial 4](/_images/tableau_10.png)

5. When we have finished processing the data, we can start the visualization by clicking the “Sheet1” tab again, and our data is automatically distributed in Tableau and ready to be used
 
![chart tutorial 5](/_images/tableau_11.png)

6. Tableau is incredibly easy to use. At the top of Tableau, there are areas for columns and rows. As the name suggests, we may drag the variables we wish to see in this area. I drag the year variable into the column area and the number variable into the row section because I want to observe the trend in the number of MSMEs
 
![chart tutorial 6](/_images/tableau_12.png)

7. Tableau will suggest a chart that it believes best illustrates our data by default, but we can easily select a different graphic by selecting "Show Me" in the top right corner. We may utilize a variety of visualization techniques there. I decided to try replacing it with a bar chart this time, and this is what happened: 

![chart tutorial 7](/_images/tableau_13.png)

8. How looks it? This is the graph that we used as a comparison in the previous step, and of course it is so bad. You may basically experiment and have fun with the charts that are available, but always utilize the chart in accordance with your aims. To make the line chart more interesting, we need to include certain elements like altering the color. It's quite easy; all you have to do is click on the color area and then make your choice. You may also experiment with opacity and other effects to make your display more attractive.

![chart tutorial 8](/_images/tableau_14.png)
 
9. Last but not least, make sure to add some info related to your visualization. It is very important since additonal info will help your audience to understand the content easily. There are several ways to add info, but you can start with title, caption, legends and tooltip, like this:
 
![chart tutorial 9](/_images/tableau_15.png)

- **Title** to shows the main point in your visualization. On the picture above, we can immediately know that the aim of this visualization is to see the development trend of MSME in Indonesia
- **Axis Label** to shows what kind of variable that you would to compare or show in the visualization. In this case, we know that x is for “Tahun” and y is for “Jumlah”
- **Tooltip** to helps your audience to easily see the detail of certain points or thing in your visualization
- **Legends** to adds detail about the meaning of color we use. By default, a legend will be created whenever your visualization is using color, size, or shape. In Tableau, you can change the color, font, and others by right click at that legend card. There will be some options such as Edit Colors, Format Legends, etc.

Here is the tutorial of adding information in Tableau:
1. On the worksheet, hover on the title, then click the drop-down arrow on the right-hand side and select Edit Title or Edit Caption from the context menu. By default, you will see "Sheet Name" that means your title will be same as your sheet name. However, you can change it to your own title like this:
 
![chart tutorial 10](/_images/tableau_16.png)

2. You can also change font name, font size, alignment, etc. If you want to see how it fits with your design, you can click “Apply” or just simply click “Ok” to see the changes. After you have finished with your title, let’s move to create a caption (right click, then tick the caption)
 
![chart tutorial 11](/_images/tableau_17.png)

3. Right after you tick “ Caption”, there will be a space in the bottom of your workspace to add your caption. It looks similar with the process of editing title in step 1.

![chart tutorial 12](/_images/tableau_18.png)
 
4. After you give a caption, you can also edit and add axis label, by double click on the axis label, and change it in the upcoming tab like this
 
![chart tutorial 13](/_images/tableau_19.png)

5. Lastly, do not forget to also maximize your visualization through “Marks Box”. You can also add color, size, detail, label and tooltip there. Back to our line chart visualization, we will add some label to it, by dragging selected variable to give it some details. For example, if you want to add label, you can drag your variables in the Tables section ( bottom left of your screen) to the label icon. Then, your design will be shown like this:
 
![chart tutorial 14](/_images/tableau_20.png)

However, for me personally this design does not need a label since there it will ruin the view, then our audience can use tooltip feature to gain the info. Horay, we have done our first task in this tutorial. So easy isn’t it? Next, let’ s make another chart using the same step like what we have done before. Here is the example of other chart that we could make in Tableau.

![chart tutorial 15](/_images/tableau_21.png)

