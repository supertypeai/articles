---

title: The power of data storytelling, and how to do it with Tableau
post_excerpt: Four secrets to start making your first visualization in Tableau
taxonomy:
    category:
        - knowledge
        - notes

---

# The power of data storytelling, and how to do it with Tableau

![Dashboard UMKM Indonesia](/_images/tableau_1.jpg)

---
Carli Fiorina once said, *“The goal is to turn data into information, and information into insight”*. This is true because, no matter how thorough our analysis, it will all be for naught if we are unable to communicate it to others. Visualization is a powerful way to overcome this. People prefer to understand the information presented in the form of images or illustrations more readily than words or statistics since we are visual beings. So here are four recommendations for utilizing Tableau to visualize data.

## 1. Discover the **"What"** causing the issue you are facing

Visualization is more than just a picture, it's also information. As a result, we must fully comprehend the issue that has to be resolved. The 5W + 1H question format is the simplest to use. In my case, I made a list of the following questions since I want to solve the Micro, Small and Medium Enterprises(MSME) or well known as Usaha Mikro Kecil dan Menengah (UMKM) problem in Indonesia, such as:

-	What is the development trend of MSMEs in Indonesia?
-	What kind of support is provided by the Indonesian government for the growth of MSMEs there?
-	How much does Indonesia's GDP contribute from MSMEs?
-	What are the factors that have the most influence on the progress of MSMEs?
-	What is the largest MSME sector in Indonesia?
-	How many MSMEs in Indonesia have adopted e-commerce?

It is preferable to have more questions since it shows that you have a thorough understanding of your issue. Ask others for their perspectives, especially those who are knowledgeable about the subject at hand. Don't overlook this step since locating the roots of the problem is crucial to a visualization's effectiveness.

## 2. Be aware of your visualization's **target audience**

The audience is everything. No matter how valuable the information is, it won't reach them if we can't get their attention. Use appealing fonts and colors, and simple visualization (like bar charts) when creating visualizations for kids. However, when making visuals for business presentations, utilize classy colors, readable fonts, and advanced visualization ( like waterfall charts). In my example, I choose to utilize a brown color tone because the people who will be viewing my visualization are West Java government officials, academics, and MSME players. Brown not only appears neutral but is also elegant to employ in visualization. I use the Poppins font, a classic font that is frequently used in writing for publications, posters, etc. To facilitate the audience's understanding of the information from my visualizations, I also included several picture components. Consider themes, colors, fonts, and pictures after you've identified your target market. You may utilize the assistance of some of these websites to make things simpler, [My Fonts](https://www.myfonts.com/pages/whatthefont ) and [Color Pallete](https://colorpalettes.net/)

## 3. Understand each **graph's function**

Each chart has its function, so make sure you choose the right chart type. Here are the 5 most popular types of visualizations that are frequently used in Tableau

| Graph | Function |
|-------|----------|
|Bar Chart|Quickly compare data from all existing categories|
|Line Chart|Time series data can be displayed yearly, quarterly, monthly, or even days to show data patterns. It is simpler for the user to interpret the data and determine if it has risen or reduced when the visualization results are presented as lines
|Maps|All forms of location, zip code, and country information may be easily seen using maps. Maps are an easy and appealing approach to display how geography corresponds with trends in your data if consumers have geographic information connected to geospatial data
|Scatter Plot| a statistical tool for analyzing the correlation between two numerical variables|
|Pie Chart|It will split the circle into many sections according to our data. If there are no more than five sections, this chart will be more suitable for usage since if there are more, the pie chart becomes more challenging to read|

## 4. Create a **design layout**
This phase is my secret technique for coming up with ideas. However, most people usually choose to design at the moment instead of creating the layout first. For me by creating a layout design early on, the concept we are preparing is well developed, eliminating the need for extensive deliberation throughout the implementation phase. Beginning with the layout in mind is also a crucial stage since we can then search for references to improve our ideas. You may look for references on [Dribble](https://dribbble.com/) and [Pinterest](https://id.pinterest.com/). These 2 apps are where artists' works congregate, therefore you'll get a lot of inspiration there. Moreover, you may go to [Tableau Public](https://public.tableau.com/app/discover/viz-of-the-day) there are many people's portfolios there.

![Example of Design Layout](/_images/tableau_2.jpg)

## 5. Don't let your audiece guess

Last but not least, make sure to add some info related to your visualization. It is very important since additional info will help your audience to understand the content easily. There are several ways to add info, but you can start with a title, caption, legends, and tooltip, like this:

![Contoh Visualisasi dengan Informasi Lengkap](/_images/tableau_3.jpg)
 
- **Title** shows the main point in your visualization. From the picture above, we can immediately know that this visualization aims to compare and find the correlation between product description, sold products, and product price
- **Legends** add detail about the meaning of the color we use. In this example, we know that the light brown color indicates 0 or a bad product description, and dark brown indicates 1 or a good product description
- **Axis Label** shows what kind of variable you would to compare or show in the visualization. In this case, we know that x is for “Sold” and y is for “Price”
- **Tooltip** helps your audience to easily see the detail of certain points or things in your visualization

Here is the tutorial for adding information in Tableau:
1. On the worksheet, hover on the title, then click the drop-down arrow on the right-hand side and select Edit Title or Edit Caption from the context menu. By default, you will see "Sheet Name" which means your title will be the same as your sheet name. However, you can change it to your title like this:

![Tampilan Edit Title](/_images/tableau_4.jpg)
 
2. You can also change the font name, font size, alignment, etc. If you want to see how it fits with your design, you can click “Apply” or just simply click “Ok” to see the changes. After you have finished with your title, let’s move to create a caption (right-click à tick the caption)
 
 ![Tampilan Setelah Edit Title](/_images/tableau_5.png)

3. Right after you tick “ Caption”, there will be a space at the bottom of your workspace to add your caption. It looks similar to the process of editing the title in step 1.
 
 ![Tampilan Edit Caption](/_images/tableau_6.png)

4. Next, if you are focusing on a small info box beside the visualization, you will realize that this is a **legend**. By default, a legend will be created whenever your visualization is using color, size, or shape. You can change the color, font, and others by right click on that legend card. There will be some options such as Edit Colors, Format Legends, etc
 
 ![Tampilan Edit Legend](/_images/tableau_7.png)

5. For example, if you click the “Edit Colors” option from the dropdown menu, you will be able to edit the color from a menu like this:

![Tampilan Edit Color Legend](/_images/tableau_8.png)
 
6. Lastly, do not forget to also maximize your visualization through “Marks Box”. You can also add color, size, detail, label, and tooltip there. 

![Tampilan Marks Box](/_images/tableau_9.png)
 
7. For example, if you want to add a label, you can drag your variables in the Tables section ( bottom left of your screen) to the label icon. Then, your design will be shown like this:

![Tampilan Add Label](/_images/tableau_10.png)

8. However, for me this design does not need a label since there is clear information from the legend. So, here is my final design using a scatter plot to visualize the correlation between several variables

![Tampilan Final Design](/_images/tableau_11.png)

---

To sum up, The audience and the problem's core are the visualization's two most important success factors. Make sure your design is in line with the goal you have in mind, and be very clear about the issue you want to raise. Don't forget to familiarize yourself with how each style of visualization works, too. Create a layout design, gather as much inspiration as you can, and don't forget to include as much information in your visualization as you can.


