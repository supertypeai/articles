---
title: Unveiling YouTube Insights - Introduction, Data Collection, Data Processing, and Database (Part 1)
post_excerpt: In this post, we will develop a website that integrates sentiment analysis techniques and a Large Language Model to provide a comprehensive understanding of YouTube comments, enabling users to extract meaningful information effortlessly.
taxonomy:
    category:
        - knowledge
        - notes
---


# Unveiling YouTube Insights - Introduction, Data Collection, Data Processing, and Database (Part 1)

![Part 1 Overview](/_images/ra_homepage.png)

### Introduction

We, as programmers, are witnessing the rise of the Large Language Model (LLM) as the latest trend. Data scientists, software engineers, and even website or mobile developers are all captivated by its potential. When we can seamlessly integrate LLM into our products, it becomes a truly exciting prospect. Now, you might wonder, "How can we achieve this?" or "Isn't building AI products costly?" Well, let me assure you, it's a resounding "NO"! After reading this article, we'll discover how we can create our very own AI app for free and with utmost ease. Let's dive in!

For this first project, we will be utilizing data from YouTube. Now, for a second, I want us to positioning ourself as a YouTube creator. Nowadays we can track metrics like views, watch time, audience demographics, traffic sources, and more in ease through YouTube studio. However, to grow our YouTube channel we need to know what people say about our content through their comment, right? What’s their feedback, what they like from our video, and many more. On the other hand, right now Youtube Studio hasn’t provided a tool to summarize those comment, if we only have 50 comments in our content, we can read it one by one. But how if our channel keeps growing, you upload more than 3 videos each week and more than 100 comments we will receive everyday. If we keep analyzing manually, we’ll waste our valuable time. 

That’s why today our mission is to help every Youtube content creator out there to easily analyzing their comment and gain insight from it in a minute using help of Large Language Model. Speaking of YouTube data, I highly recommend you checking out <u>[an article on data engineering](https://supertype.ai/notes/airflow-trending-youtube-video-project/)</u> authored by my colleague, Timotius. This series delves into data engineering using Airflow, offering valuable insights into the subject.

### Workflow and Pre-requisites

Okay, let's get back to the topic! Before we start, I want to explain briefly how's our website work

- User input a link of a YouTube video - part 1
- Scrap the statistic and comment - part 1
- Store it to database - part 1
- User can start a conversation with AI bot ( build with Langchain and Hugging Face Open Source Model) - part 2

In this article, I will more focus explaining about the backend side explaining on how it's working. Mainly using Django, Django REST Framework, and Langchain. If you haven't known it yet, I really recommend you to watch this video before continue:

- Langchain: [LangChain & LLM tutorials (ft. gpt3, chatgpt, llamaindex, chroma)](https://www.youtube.com/playlist?list=PLXsFtK46HZxUQERRbOmuGoqbMD-KWLkOS) by Samuel Chan
- Django: [Django Crash Course - Introduction + Python Web Development Tutorial](https://www.youtube.com/watch?v=EuBQU_miReM&list=PL_c9BZzLwBRLCpTc20e2pFT1lmbnevegR&index=1&t=1859s) by Caleb Curry
- Django REST Framework: [Django REST Framework - Build an API from Scratch](https://www.youtube.com/watch?v=i5JykvxUk_A&t=2120s) by Caleb Curry

For the frontend part, we are going to use Tailwind CSS, HTML, and JavaScript. Please feel free to check out my GitHub repository. I encourage you to explore and develop it further with your own design style, as I'm sure the talented audience reading this article can enhance it in amazing ways. If you are ready, let's start with the first part, which focuses on data collection, data processing, and database

### Data Collection, Data Processing, and Database

If you still remember the workflow of our website, the first step is to submit a YouTube video link. Using this link, we will perform a scraping task, which involves the process of data collection. Scraping YouTube data is relatively straightforward; what we actually need is a YouTube API key. Our team at Supertype has provided you with a quick tutorial, which you can access here:

- Part 1: [YouTube Analytics API with Python (May 2022 new Google API Library)](https://www.youtube.com/watch?v=rcE3iL0URhU&t=0s)
- Part 2: [YouTube Comments to CSV](https://www.youtube.com/watch?v=XTjtPc0uiG8)
- Github Repo: [youtube_api_python](https://github.com/onlyphantom/youtube_api_python)

In this section, we will modify some parts of the code to meet our specific requirements.<br>

1. Create a new project in Google Cloud Console
    - Go to [Google Cloud Console](https://console.cloud.google.com/)
    - Click “Select a Project” dropdown beside the Google Cloud icon in the top right of your screen
    - Click “New Project” and fill the form with the project name and location
    ![Create Project in Google Cloud Console](/_images/ra_createproject.png)
    - Click the “create” button
    
2. Create a YouTube API Key

    - Go to https://console.cloud.google.com/apis/library/youtube.googleapis.com 
    - Click “Enable”
    - Click “Create Credentials” and choose “API Key”.
    ![Create YouTube API Key](/_images/ra_youtubeapikey.png)
    Once you click it, you will get a pop up. This is your secret API Key, copy and save it to an environment variable or .env file so it will not leak. 

3. Write a function to scrap video statistic

    ```python
    async def video_stats(youtube, videoIDs, channelID = None, to_csv=False):
        if type(videoIDs) == str:
            videoIDs = [videoIDs]
    
        stats_list = []


        for videoId in videoIDs:
            request = youtube.videos().list(
                part="snippet, statistics, contentDetails",
                id=videoId
            )
            response = request.execute()
            statistics = response['items'][0]['statistics']
            snippet = response['items'][0]['snippet']
            statistics['videoId'] = videoId
            statistics['title'] = snippet['title']
            statistics['description'] = snippet['description']
            statistics['publishedAt'] = snippet['publishedAt']
            statistics['duration'] = response['items'][0]['contentDetails']['duration']
            statistics['thumbnail'] = snippet['thumbnails']['high']['url']
            statistics['channelId'] = channelID
            statistics['likeCount'] = statistics.get('likeCount', 0)


            print(f"Fetched stats for {videoId}")
            stats_list.append(statistics)


        return statistics
    ```

    Let’s walk through the code section by section to gain a better understanding of the implementation.

    ```python
    async def video_stats(youtube, videoIDs, channelID=None, to_csv=False):
    ```

    This line defines an asynchronous function called video_stats. It takes four arguments: youtube (presumably an object representing the YouTube API client), videoIDs (a list of video IDs or a single video ID as a string), channelID (an optional channel ID, defaulting to None), and to_csv (an optional argument, defaulting to False).

    ```python
    if type(videoIDs) == str:
        videoIDs = [videoIDs]
    ```

    This checks if the videoIDs is a string. If it is, it converts it into a list containing just that string. This step ensures that the videoIDs parameter can be treated as a list, regardless of whether it was passed as a single string or a list of strings.

    ```python
    stats_list = []
    ```

    This initializes an empty list called stats_list, which will be used to store the statistics data for each video.

    ```python
    for videoId in videoIDs:
        request = youtube.videos().list(
            part="snippet, statistics, contentDetails",
            id=videoId
        )
        response = request.execute()
    ```

    This starts a loop that iterates through each videoId in the videoIDs list. Within the loop, it creates a request to the YouTube API to get video details (snippet, statistics, and contentDetails) for the current videoId. The request is executed using request.execute() which sends the request to the YouTube API and gets the response.

    ```python
    statistics = response['items'][0]['statistics']
    snippet = response['items'][0]['snippet']
    ```

    These lines extract the 'statistics' and 'snippet' information from the API response for the current video.

    ```python
    statistics['videoId'] = videoId
    statistics['title'] = snippet['title']
    statistics['description'] = snippet['description']
    statistics['publishedAt'] = snippet['publishedAt']
    statistics['duration'] = response['items'][0]['contentDetails']['duration']
    statistics['thumbnail'] = snippet['thumbnails']['high']['url']
    statistics['channelId'] = channelID
    statistics['likeCount'] = statistics.get('likeCount', 0)
    ```

    Here, specific data from the API response is extracted and added to the statistics dictionary for the current video. videoId, title, description, publishedAt, duration, thumbnail, and channelId are taken from the snippet information. likeCount is extracted from the 'statistics' dictionary, and if it is not present, it defaults to 0 using the .get() method.

    ```python
    print(f"Fetched stats for {videoId}")
    stats_list.append(statistics)
    return stat_list
    ```

    This line prints a message indicating that the statistics for the current video with videoId have been fetched. Then, the statistics dictionary for the current video is appended to the stats_list, which collects data for all videos and being returned

4. Write a function to process comment of the video 

    ```python
    def process_comments(response_items, channelID = None, csv_output=False):
        comments = []

        for res in response_items:

            # loop through the replies
            if 'replies' in res.keys():
                for reply in res['replies']['comments']:
                    comment = reply['snippet']
                    comment['commentId'] = reply['id']
                    comments.append(comment)
            else:
                comment = {}
                comment['snippet'] = res['snippet']['topLevelComment']['snippet']
                comment['snippet']['parentId'] = None
                comment['snippet']['commentId'] = res['snippet']['topLevelComment']['id']

                comments.append(comment['snippet'])
            
        keytoremove = ['textDisplay','authorProfileImageUrl','authorChannelUrl','authorChannelId','canRate','viewerRating','likeCount','updatedAt','parentId']
        for i in comments:
            for y in keytoremove:
                del i[y]

        new_comments = []
        for original_dict in comments:
            new_dict = {
                'video_id': original_dict['videoId'],
                'comment_id': original_dict['commentId'],
                'date': original_dict['publishedAt'],
                'author': original_dict['authorDisplayName'],
                'comment_text': original_dict['textOriginal']
            }
            new_comments.append(new_dict)

        new_key = 'channel_id'
        new_value = channelID

        for dictionary in new_comments:
            new_dict = {new_key: new_value}
            new_dict.update(dictionary)
            dictionary.clear()
            dictionary.update(new_dict)

        def sentiment(i):
            blob = TextBlob(i)
            score = blob.sentiment.polarity
            if score == 0:
                sentiment = 'neutral'
            elif score > 0: 
                sentiment = 'positive'
            else:
                sentiment = 'negative'

            return sentiment
        
        for comment in new_comments:
            if type(comment['comment_text']) == 'float':
                comment['sentiment'] == 'no sentiment for numerical values'
            else:
                comment['sentiment'] = sentiment(comment['comment_text'])

        print(f'Finished processing {len(new_comments)} comments.')
        return new_comments
    ```

    This function takes three parameters:
    - `response_items`: A list of YouTube API response items containing comments data.
    - `channelID` (optional): An identifier for the YouTube channel associated with the comments.
    - `csv_output` (optional): A boolean flag indicating whether the comments should be saved to a CSV file.

    The function performs the following tasks:

    - It processes the input `response_items` list to extract relevant information from the comments and replies.
    - It removes unnecessary keys from the comment data using the `keytoremove` list.
    - It constructs a new list of comments in a specific format, containing attributes like video ID, comment ID, date, author, and comment text.
    - It calculates the sentiment of each comment text using the TextBlob library. `sentiment(i)`: This is a nested function within the `process_comments` function. It takes a string `i` as input and calculates the sentiment of the text using TextBlob. The sentiment is categorized as "positive," "negative," or "neutral" based on the polarity score of the text.
    - It prints the number of processed comments.
    - It returns the processed comments as a list of dictionaries.

5. Write a function to scrap comment of the video

    In this function, we will also use process_comment function that has been defined before.

    ```python
    scraped_videos = {}

    async def comment_threads(youtube, videoID, channelID=None, to_csv=False):
    
        comments_list = []
        
        try:
            request = youtube.commentThreads().list(
                part='id,replies,snippet',
                videoId=videoID,
            )
            response = request.execute()
        except Exception as e:
            print(f'Error fetching comments for {videoID} - error: {e}')
            if scraped_videos.get('error_ids', None):
                scraped_videos['error_ids'].append(videoID)
            else:
                scraped_videos['error_ids'] = [videoID]
            return

        comments_list.extend(process_comments(response['items'],channelID))

        # if there is nextPageToken, then keep calling the API
        while response.get('nextPageToken', None):
            request = youtube.commentThreads().list(
                part='id,replies,snippet',
                videoId=videoID,
                pageToken=response['nextPageToken']
            )
            response = request.execute()
            comments_list.extend(process_comments(response['items'],channelID)) 
        
        print(f"Finished fetching comments for {videoID}. {len(comments_list)} comments found.")
        
        if scraped_videos.get(channelID, None):
            scraped_videos[channelID].append(videoID)
        else:
            scraped_videos[channelID] = [videoID]

        comment_df = pd.DataFrame(comments_list)

        return comment_df

    ```

    This is an asynchronous function used to fetch comments for a specific YouTube video and process them using the `process_comments` function. It takes the following parameters:
   - `youtube`: The YouTube API client used to make requests.
   - `videoID`: The ID of the YouTube video for which comments need to be fetched.
   - `channelID` (optional): An identifier for the YouTube channel associated with the video.
   - `to_csv` (optional): A boolean flag indicating whether the comments should be saved to a CSV file.

   The function performs the following tasks:

   - It makes an API request to fetch the comment threads for the given `videoID`.
   - It processes the retrieved comments using the `process_comments` function.
   - If there are more comments available (i.e., additional pages), it continues fetching them until there are no more comments.
   - It prints the number of fetched comments and stores the list of video IDs with their associated channel IDs in the `scraped_videos` dictionary.
   - It returns a pandas DataFrame containing the processed comments.

    Please note that for the code to work, it requires the necessary libraries, such as pandas and TextBlob, and a YouTube API client initialized with the required credentials. Additionally, the `TextBlob` class needs to be imported from the `textblob` library at the beginning of the code.

6. Write a function to summarize positive and negative comments

    In this part, we will not be developing a model from scratch to summarize a text; instead, we will utilize an open-source model from Hugging Face. This approach is entirely free; however, it is important to be aware that there are limitations on the free plan. Therefore, I will present three different ways to summarize the text, along with the pros and cons of each method. But before we proceed, let's set up the Hugging Face model. Please ensure that you have installed `langchain` and Hugging Face libraries.
        
    For this case, I have chosen to use the Falcon 7B instruct model, which is one of the best open-source models available in those parameters. The results it provides are quite good. However, if you desire even better results, you may consider trying the OpenAI model or using higher parameters such as the Falcon 40B instruct model or llama2 (though, be mindful that you need to have compatible resources for these higher-parameter models).

    ```python
    from langchain import HuggingFaceHub
    from langchain.text_splitter import RecursiveCharacterTextSplitter
    from langchain.chains.summarize import load_summarize_chain
    from langchain import PromptTemplate,  LLMChain
    import textwrap
    from transformers import pipeline


    load_dotenv(find_dotenv())
    HUGGINGFACEHUB_API_TOKEN = os.environ["huggingfacehub_api_token"]


    repo_id = "tiiuae/falcon-7b-instruct"  
    falcon_llm = HuggingFaceHub(
        repo_id=repo_id, model_kwargs={"temperature": 0.5, "max_new_tokens": 425}
    )
    ```

    - Using load_summarize_chain from Langchain

        ```python
        async def summary_of_comments(df,things = 'positive'):
            filtered_comment = df[df['sentiment'] == things]
            comment_text = ';'.join(filtered_comment['comment_text']).replace('\n','')


            text_splitter = RecursiveCharacterTextSplitter(chunk_size=500)
            comment_doc = text_splitter.create_documents([comment_text])


            chain = load_summarize_chain(falcon_llm, chain_type="map_reduce", verbose=True)
            print(chain.llm_chain.prompt.template)
            print(chain.combine_document_chain.llm_chain.prompt.template)


            output_summary = chain.run(comment_doc)
            wrapped_text = textwrap.fill(
                output_summary, width=100, break_long_words=False, replace_whitespace=False
            )
            print(wrapped_text)


            return wrapped_text
        ```

        In short, this code will separate the text into chunks. We need to split it since there's a limit on the number of tokens that can be processed at the same time. Once we split it into chunks, we save it as a document using `comment_doc = text_splitter.create_documents([comment_text])`. Then, we use `load_summarize_chain` to summarize the text.

        Using this approach, you will get a very comprehensive summary that doesn't lose the context. However, if your text is too long, this method will incur significant costs in terms of both money and time. If you don't have compatible resources, your website may end up crashing or encountering errors due to the processing time.

    - Using combination of Langchain and Question Answering model

        We have reviewed the drawbacks of the previous method. Now, let's explore an alternative solution that could potentially address the cost and time consumption issues. Instead of sending the entire text to the Langchain and Falcon 7B model, we can leverage a question-answering model to identify key points from each text. For this purpose, we are utilizing the prompt, "What {things} things does the user/audience feel?" However, it's essential to keep in mind that this prompt can be modified in the future to potentially achieve better results.

        Once we have extracted the key points using the question-answering model, we can pass them to our Langchain model for summarization.

        This approach significantly reduces both cost and processing time. However, it's worth noting that the results may not always be as satisfactory compared to the first method, where the Falcon 7B instruct model was applied directly to the entire text.

        The choice between these two methods depends on your specific requirements. If you need highly accurate and satisfying results, particularly for production or when selling it as a product where customer satisfaction is crucial, you might opt for the first method or consider investing more to achieve better results. On the other hand, if you are looking for a more cost-effective and time-efficient solution and the results do not have to be perfect, the second method could be a suitable option. It strikes a balance between cost and accuracy and may suffice for certain use cases.

        ```python 
        rm = 'deepset/roberta-base-squad2'


        question_answerer = pipeline("question-answering", model=rm)


        async def summary_of_comments(df,things = 'positive'):


            print(f"start summary of {things} comments")


            filtered_comment = df[df['sentiment'] == things]


            if len(filtered_comment) != 0:
        
                comment_text = ';'.join(filtered_comment['comment_text']).replace('\n','')


                text_splitter = RecursiveCharacterTextSplitter(chunk_size=500)
                comment_doc = text_splitter.create_documents([comment_text])


                output = {}
                for i in comment_doc:
                    result = question_answerer(question= f"What {things} things does the user/audience feel?", context= i.page_content) #str(i)
                    output[result['answer']] = round(result['score'], 4)
                    print(f"Answer: '{result['answer']}', score: {round(result['score'], 4)}, start: {result['start']}, end: {result['end']}")
                
                keys_set = set(output.keys())
                keys_sentence = '; '.join([key for key in keys_set])
                print(keys_sentence)


                docs = text_splitter.create_documents([keys_sentence])
                print('done splitting')


                chain = load_summarize_chain(falcon_llm, chain_type="map_reduce", verbose=True)
                print(chain.llm_chain.prompt.template)
                print(chain.combine_document_chain.llm_chain.prompt.template)


                output_summary = chain.run(docs)
                wrapped_text = textwrap.fill(
                    output_summary, width=100, break_long_words=False, replace_whitespace=False
                )
                print(wrapped_text)


                print(f"done summary of {things} comments")


                return wrapped_text
        
            else:
                return f"There is no {things} comment"
        ```

7. Write a final function to wrap up the statistics and comment functions

    ```python
    async def get_result(url, youtubeapikey):
        parsed_url = urlparse(url)
        query_params = parse_qs(parsed_url.query)
        videoid = query_params.get('v', [''])[0]


        youtube = build("youtube", "v3", developerKey= youtubeapikey)


        stats = await video_stats(youtube, videoid)
        df = await comment_threads(youtube, videoID=videoid)
        positive = await summary_of_comments(df,'positive')
        negative = await summary_of_comments(df,'negative')
        neutral = await summary_of_comments(df,'neutral')

        return stats,df,videoid,positive,negative, neutral

    ```

    get_result is our final function that will be used in our Django code. As you may recall, in our previous function, we required a YouTube API Key and the link to the YouTube video. These two values are processed when the user submits the form, and then we utilize the get_result function to process them. If you don't fully understand the process now, that's okay; it will be our next step. I want you to keep it in mind. Once we finish this main code, we will explore the form and Django.

8. HTML Form

    ```html
    <form action = "getoutput" method = "POST" id = "form" class = "mt-3 text-center flex">
        {% csrf_token %}
        <input
            type = "text"
            name = "videoid"
            placeholder = "Paste your youtube video url here"
            class = "m-2 px-3 py-1.5 border shadow rounded w-5/6 text-sm placeholder:text-slate-400 focus:outline-none focus:ring-1 focus:ring-red-600 inline lg:text-lg">
        <input
            type = "submit"
            class = "m-2 px-3 py-1.5 rounded text-sm bg-red-600 text-white transition duration-300 ease-in-out hover:bg-black hover:shadow-lg lg:text-lg dark:hover:bg-white dark:hover:text-black">  
    </form>

    ```

    This snippet is from the complete home.html, which includes the form where users can submit a YouTube video link. The key elements that connect this HTML form with our backend are the action, method, and name attributes. Here are the details:

    - Action: "getoutput" is the name of our Django function. By specifying this in the form tag, we inform the HTML which function to execute when the user submits the form.
    - Method: We use "POST" since we are uploading or posting new data or records to our backend. This method serves as a signal to Django, indicating that we want to create a new resource on the server with the data provided in the form. (More details about how Django handles different HTTP methods will be covered later.)
    - Name: "videoid" is the name attribute that represents the value being passed to our backend. This name is used to identify and access the submitted data on the server side.


    When creating a form in Django, it's important to remember that you need a CSRF token. The CSRF (Cross-Site Request Forgery) attack can trick the user's browser into executing unwanted actions on an authenticated website. To prevent such attacks, Django includes a built-in security mechanism called the CSRF Token. This unique and unpredictable token is generated for each user session and is included as a hidden field in the form using the `{% csrf_token %}` template tag. Upon form submission, the CSRF Token is sent back to the server with the form data, and Django checks its validity. If the token is missing or incorrect, the request is rejected, safeguarding against potential CSRF attacks. In summary, including the CSRF Token in your HTML forms is essential to ensure the authenticity of form submissions and protect the integrity of your Django application.

9. Django Model / Database (models.py)

    If you look your folder structure, inside your Django app, you will see models.py. This python file is a place for us to write a Django Model or database. Here are two databases that we will create together.
    - ApiKey Model: This model is used to store API keys associated with users. It has fields for the user (linked through a ForeignKey relationship), YouTube API key, OpenAI API key, and Hugging Face Hub API key.
    - Result Model: This model stores results related to YouTube videos. It includes fields for user, video ID, video title, views, likes, comments, positive/negative/neutral comments, and timestamps for creation and updates.

    ```python
    from django.db import models
    from django.contrib.auth.models import User


    class ApiKey(models.Model):
        user = models.ForeignKey(User, on_delete=models.CASCADE)
        youtube_api_key= models.CharField(max_length=100, null=True, blank=True)
        openai_api_key = models.CharField(max_length=100, null=True, blank=True)
        huggingfacehub_api_key = models.CharField(max_length=100, null=True, blank=True)


    class Result(models.Model):
        user = models.ForeignKey(User, on_delete=models.CASCADE)
        videoid= models.CharField(max_length=20, null=True, blank=True)
        videotitle= models.CharField(max_length=200, null=True, blank=True)
        view = models.CharField(max_length=20, null=True, blank=True)
        like = models.CharField(max_length=10, null=True, blank=True)
        comment = models.CharField(max_length=10, null=True, blank=True)
        total_positive_comment = models.CharField(max_length=10, null=True, blank=True)
        positive_comment = models.CharField(max_length=5000, null=True, blank=True)
        total_negative_comment = models.CharField(max_length=10, null=True, blank=True)
        negative_comment = models.CharField(max_length=5000, null=True, blank=True)
        total_neutral_comment = models.CharField(max_length=10, null=True, blank=True)
        neutral_comment = models.CharField(max_length=5000, null=True, blank=True)
        created_at = models.DateTimeField(auto_now_add=True)
        last_update = models.DateTimeField(auto_now=True)


        def __str__(self):
            return self.videoid
    ```

    After we have create a model or database, we should run two commands in our command prompt:

    - python manage.py makemigrations: This command will analyze the changes we made to the models and create migration files in the migrations directory of our app.
    - python manage.py migrate: This command will execute the migrations and update the database schema to reflect the changes you made in your models.

10. Register our Django model in admin.py

    ```python
    from django.contrib import admin
    from .models import User, ApiKey, Result


    # Register your models here.
    admin.site.register(ApiKey)
    admin.site.register(Result)
    ```

11. Fill out our views.py

    We have register our database, so right now, let’s code. If you still remember the 3 key elements that connect the HTML form with our backend, we will use them here. The first one is the action "getoutput". In our Django code, we create a POST function called "getoutput", like the following code:
    
    ```python
    def getoutput(request):


    context = {
        'user_id': request.user.pk
    }
       
    if request.method == "POST":
        url = request.POST["videoid"]


        try:
            key = ApiKey.objects.get(user=request.user)
            youtubeapikey = key.youtube_api_key
            if youtubeapikey is None:
                youtubeapikey = os.environ.get('youtubeapikey')
        except:
            youtubeapikey = os.environ.get('youtubeapikey')


        async def run_async():
            stats, df, videoid, positive, negative, neutral = await get_result(url, youtubeapikey, username, recipient_email)


            source = {
                'videoid': videoid,
                'videotitle': stats['title'],
                'view': stats['viewCount'],
                'like': stats['likeCount'],
                'comment': stats['commentCount'],
                'total_positive_comment': len(df[df['sentiment'] == 'positive']),
                'total_negative_comment': len(df[df['sentiment'] == 'negative']),
                'total_neutral_comment': len(df[df['sentiment'] == 'neutral']),
                'positive_comment': positive,
                'negative_comment': negative,
                'neutral_comment': neutral,
            }


            return source


        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        source = loop.run_until_complete(run_async())
        loop.close()

        current_user = request.user
        videoid = source['videoid']

        # Check if a record with the same videoid exists
        try:
            result = Result.objects.get(user=current_user, videoid=videoid)
            print(result)
        except Result.DoesNotExist:
            result = None


        # If the record exists, update it; otherwise, create a new record
        if result:
            result.videotitle = source['videotitle']
            result.view = source['view']
            result.like = source['like']
            result.comment = source['comment']
            result.total_positive_comment = source['total_positive_comment']
            result.positive_comment = source['positive_comment']
            result.total_negative_comment = source['total_negative_comment']
            result.negative_comment = source['negative_comment']
        else:
            result = Result(
                user=current_user,
                videoid=videoid,
                videotitle= source['videotitle'],
                view = source['view'],
                like = source['like'],
                comment = source['comment'],
                total_positive_comment = source['total_positive_comment'],
                positive_comment = source['positive_comment'],
                total_negative_comment = source['total_negative_comment'],
                negative_comment = source['negative_comment']
            )


        result.save()

        return redirect(reverse('chat') + f'?id={result.id}')

    else:
        return render(request, "home.html", {'context':context})
    ``` 

    This code defines a Django view function named `getoutput` that handles a POST request. The primary purpose of this function is to fetch data about a YouTube video, perform sentiment analysis on its comments, and then store the results in a Django model named `Result`. Let's break down the code step by step:

    - The `context` dictionary is initialized with the user's ID, retrieved from the request.

    - The function checks if the request method is POST (indicating form submission). If so, it extracts the video ID from the submitted form data.

    - An attempt is made to retrieve the user's YouTube API key from the `ApiKey` model. If not found, it falls back to using an API key from the environment variables.

    - An asynchronous function named `run_async` is defined. This function retrieves statistics and sentiment analysis results for the given video URL and API key.

    - An event loop is created using `asyncio.new_event_loop()` to execute the asynchronous function, and the loop is closed afterward.

    - The `current_user` variable is assigned the user making the request, and the `videoid` is extracted from the `source` dictionary.

    - The code checks if a `Result` record with the same `videoid` exists for the current user. If it exists, the record's fields are updated with new data; otherwise, a new `Result` instance is created.

    - The `Result` instance is saved to the database using `result.save()`.

    - Finally, the function redirects the user to another view named `'chat'`, passing the `id` of the stored `Result` record as a query parameter. More detail on this topic will be explained in part 2.

    - If the request method is not POST (e.g., a GET request), the function renders the "home.html" template with the `context` data.

    This view function serves as the backend logic for processing user input, fetching YouTube data, performing sentiment analysis, and storing the results in a Django model. It demonstrates how Django handles data manipulation, asynchronous operations, and database interactions in a web application.
        
12. Add our getoutput view into urls.py

    ```python
    from django.urls import path
    from . import views


    urlpatterns = [
        path('', views.home, name="home"),
        path('getoutput',views.getoutput, name = "getoutput"),]
    ```

### Conclusion

In this article, we embarked on a challenge to create a comprehensive web application that leverages the power of Django, sentiment analysis, and a Large Language Model to gain invaluable insights from YouTube comments. By blending data collection, analysis, and storage, we have built a tool that empowers content creators to better understand their audience's sentiments and preferences. What's most important, you now truly comprehend how to seamlessly integrate Large Language Models into our product, all for free and with ease.

As we wrap up this initial phase, remember that the journey is far from over. Here's a stimulating bonus challenge that will propel your skills to greater heights. Let's shift gears and modify our approach. Instead of querying our database directly, we're introducing a fascinating twist: the utilization of APIs.

To embark on this challenge, follow these steps:

1. Begin by installing Django REST Framework using the command `pip install djangorestframework`.
2. Create a new file named `serializers.py`.
3. Inside `serializers.py`, define a class named `ResultSerializer`.
4. Integrate your serializers into `view.py` using the import statement `from .serializers import ResultSerializer`.
5. Forge a path to access the API for CRUD operations; name it `result`.
6. Add this new path to your `urls.py` configuration.

Successfully completing this challenge will yield exciting results. When you type `http://127.0.0.1:8000/result/1` into your browser, you'll see a Django Rest Framework template and your data like this

![django rest framework example](/_images/ra_restapi.png)

In our upcoming Part 2, we'll delve even deeper into the potential of Large Language Models and also Django REST Framework. We'll pave the way for users to engage with an AI chatbot, enabling activities like seeking advice and exploring a myriad of other possibilities that an AI bot can seamlessly facilitate. This journey promises to unlock a realm of innovative interactions and dynamic user experiences.


### Relevant Links

* Project Github: <u>[https://github.com/projectwilsen/ReviewAnalyzer/](https://github.com/projectwilsen/ReviewAnalyzer/)</u>  