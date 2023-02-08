## Technical Writing
This is an instructional guide for anyone relatively new to technical writing. It is not a comprehensive guide, but it encapsulates the more important aspects of writing and offer some practical tips you can immediately incorporate into your writing. 

### Techniques of Technical Writing
- Use the active voice
    - "RStudio will display a message" rather than "A message will be displayed by RStudio."
    - "RStudio will display a message" is more direct and concise.
    - "Wrap the following python code with a decorator" rather than "The following python code should be wrapped in a decorator."

- Use "we" instead of "you" or "I"
    - "Next, we will apply our transformation to the input tensor" rather than "Next, you will apply your transformation to the input tensor."
    - Instead of "take a look at this dashboard, what do you think about its user-friendliness?" ask, "let us take a look at this dashboard and ask ourselves what we think of its user-friendliness"
    - "We" is more inclusive and less confrontational. It also makes the reader feel like they are part of the process. Use "we", "ask ourselves" and "let us" to help the reader feel more involved.

- Refactor long sentences
    - "After you have installed the package, you want to also make sure it has been added to the dependencies of your project, which could mean verifying it in package.json for a JS project, or requirements.txt for a python project, but make sure there is a separation between development and production dependencies." is an awfully long sentence, and hard to follow. Refactor it into shorter sentences.
    - """Install the package, then add it to the dependencies of your project. \n
    For a JS project, verify it in package.json. For a python project, verify it in requirements.txt. \n
    Ensure a separation between development and production dependencies."""

- Pick a consistent tone
    - "Now, this dashboard looks a lot prettier, right?" is very casual and informal. "This dashboard looks a lot prettier now, doesn't it?" is less casual but arguably still informal. Both of these senteces are conversational. They mirrors how you speak, and it ends with asking a question. You wouldn't write that if you were writing a technical document, or a paper, or a coursebook meant for students. If you would like a more formal tone, aim to end the statement with a period. "You may find that this dashboard looks prettier than the earlier version". 
        - "Now that we have a model, how do we know it's not overfitting?" is a question (conversational, engaging). "Once we have obtained the model, we want to make sure it isn't overfitting" is a statement (non-engaging).
    - A more formal tone doesn't try to engage the reader, it just presents the information. It doesn't ask questions, it just states facts. It also doesn't ask the reader to do anything, such as "ask yourself", or "let's take a look", or "by looking at the code snippet above, you will find that ...". It states facts, such as "the code snippet above shows that our refactored code is 2x shorter."
    - There are no hard rules. **Pick a consistent tone and stick to it. One might choose to be casual, conversational, formal or even academic. Just be consistent.**
    - I could have wrote the above line in a different tone. If I was going for a more conversational tone, this is how I would have wrote the line above.
        - _Since there are no hard rules, you can pick any tone you like. You may choose a casual, conversational, formal or even academic style, but you want to stay consistent. _
        - Both convey the same message. The first one is more formal, the second one is more conversational.

- Use bullet points to present lists and sequences
    - "After you have installed the package, you want to also make sure it has been added to the dependencies of your project, which could mean verifying it in package.json for a JS project, or requirements.txt for a python project, but make sure there is a separation between development and production dependencies." can be refactored into bullet points:
        - Install the package, and add it to the dependencies of your project.
            - `.js`: verify it in package.json
            - `.py`: verify it in requirements.txt
        - Note: Ensure a separation between development and production dependencies.
    - It could also be written as a sequence:
        1. Install the package
        2. Add it to the dependencies of your project
            - `.js`: verify it in package.json
            - `.py`: verify it in requirements.txt
        3. Separate your development and production dependencies.

### Pedagogical Strategies
> 'I apologize for such a long letter - I didn't have time to write a short one.'
> - Mark Twain

1. Almost always re-write your introduction after you've completed the rest of the article. It's easier to write an introduction after you've written the rest of the article. You have opportunities to assert your key points, and use the introduction to summarize your arguments and conclusions _because_ you have completed the article. 

2. The 25% rule: In most articles, you could remove the first 25% of the article and the article would still make sense with 100% information retention. The first 25% of most articles is padded with generic opening remarks. In the data science industry, the first 25% of most articles is usually about how we're all living in an era of big data with an ever-increasing rate of new tools, innovations, yada yada. It's generic opening remark that isn't necessary.

3. Staying on-topic: It's easy to write a highly elaborate article, with lots of citation and references, and plenty of examples while not actually answering the question. It's easy to write an article that is 100% on-topic, but 0% useful. It's also easy to write an article that is 100% useful, but 0% on-topic. You may want to solicit feedback early on in the writing process to ensure you're staying on-topic. You want want to produce an outline for your article or content to solicit feedback from your editor before writing the article.

4. Writing is a social contract: By luring me in with your headline, or title, you're making a promise. "The importance of analytics in the manufacturing process" offers a glimpse at _your_ expertise and insights in a sector you _claim_ to know a lot of. That expertise is implied. It is implicit. You might end up writing something thoroughly useful, but if it isn't on-point, it may tell me a lot about analytics and tell me a lot about the manufacturing process, but you still fail to deliver on that initial, implicit promise. "Importance of analytics in the manufacturing process" is an offer. Relating to point (3), this is much harder than you think.
    - Many articles with the topic asserting a claim fails to deliver on this social contract completely
    - Articles like "Generative AI will disrupt the gaming industry forever" makes a very bold, very strong promise. It made a stronger implicit promise than "Potential of Generative AI for the gaming industry". 
        - More than half of the articles with such title will fail on their promise and violate rule (3) as well. they think they're writing about the potential of Generative AI on the entire industry, and end up writing about the applications of Generative AI for a very specific use-case in the gaming industry, such as art assets creation, or character generation for a very specific type of game. Boring. Dishonest. Violates the social contract.
    - Similarly, an article titled "The advantages of the modern data engineering stack" very often regurgitates content about data engineering, and possibly the use-cases, or some of the tools involved. It might even go into details about data engineering best practices, or make a compelling argument about cloud vs on-premise data engineering stacks. But "advantages" is a promise: it's about cost, it's about speed and efficiency. If you can't quantify those, you are letting the reader down by making a promise you can't deliver on.

5. Be uniquely qualified to solve an unmet need: Write something that you can offer a unique perspective on. "Python or R" articles are boring, and there are 3,104,092 articles on that already. Why become the 3,104,093rd article on this topic? If you're writing about a tool (dbt, BigQuery, Tableau etc) what makes you uniquely qualified on that article over the 500+ articles on the vendor's official documentation and training portals? It's very competitive out there, so you want to offer something unique. 
    - it's easier to offer something unique by doing a fusion of two or more topics. "How to use dbt to build a data warehouse on BigQuery" has a bigger chance of being a unique perspective, than "How to use dbt". "A shallow network approach to Remaining Useful Lifecycle" offers a promise that draws me, as a reader in, more than "Remaining Useful Lifecycle in R". 
    - you have unique experiences, whether by working on a project individually or as a team, that you can bring into the article. This is a great way to offer a unique perspective. "Solving the unpredictable nature of water level on the Barito river in Kalimantan: Perspectives and Lessons" is highly unique, and if you worked on that project, you're more uniquely qualified than a MIT graduate or Harvard professor to write about it.
    - If you find it difficult to offer unique perspectives, do less projects and spend more time on average immersing yourself in one or two domains. If you're haphazardly working on 10 projects (or learning 5 different technologies) in 2 weeks, you're not going to be able well versed enough in any one specific domain to offer a unique contribution.
    - When bringing your content into the world, start from asking yourself, what is an article I'm in the top 1% of people uniquely qualified to offer my perspective on. Solve an unmet need. 
