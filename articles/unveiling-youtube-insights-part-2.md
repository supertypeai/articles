---
title: Unveiling YouTube Insights - Django REST Framework, API, and AI Chatbot (Part 2)
post_excerpt: In this post, we will develop a website that integrates sentiment analysis techniques and a Large Language Model to provide a comprehensive understanding of YouTube comments, enabling users to extract meaningful information effortlessly.
taxonomy:
    category:
        - knowledge
        - notes
---


# Unveiling YouTube Insights - Django REST Framework, API, and AI Chatbot (Part 2)

![Part 2 Overview](/_images/ra_chatbot.png)

In retrospect to Part 1, we've successfully developed a fully functional website capable of scraping, processing, and storing YouTube data into our database. As we culminate that endeavor, I left you a small challenge—to integrate Django REST Framework. Before delving into our exploration of Django REST Framework, it's crucial to understand the significance of APIs and why Django REST Framework holds such importance. APIs serve as conduits of seamless communication between software systems, enabling data and functionality exchange, a critical component that elevates our application's capabilities. Now, introducing Django REST Framework, a robust toolkit amplifying Django's prowess, simplifying Web API creation with features like serialization, authentication, and streamlined CRUD operations. This dynamic duo, API integration, and DRF, equips us to traverse the digital landscape with flexibility, scalability, and innovation, setting the stage for our exciting exploration in Part 2.

### Django REST Framework
Begin by installing the Django REST Framework with a simple command

```python
pip install djangorestframework
```

Then, go to settings.py and add 'rest_framework' in INSTALLED_APPS

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'ypa',
    'rest_framework'
]
```

Create a new file named serializers.py and define a class named ResultSerializer inside it.

```python
from rest_framework import serializers
from .models import Result

class ResultSerializer(serializers.ModelSerializer):
    class Meta:
        model = Result
        fields = '__all__'
```


Then, connect the serializers by importing them into views.py, and add some functionality from Django REST Framework
```python
from .serializers import ResultSerializer
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework import status
```

If you still remember, previously we store data directly to database, like this:
```python
current_user = request.user
videoid = source['videoid']

Check if a record with the same videoid exists
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
```

Right now, we can simply delete it, and change it like this

```python
serializer = ResultSerializer(data=source)
if serializer.is_valid():
    # Check if a record with the same videoid exists
    try:
        result = Result.objects.get(user=request.user, videoid=source['videoid'])
        serializer.update(result, serializer.validated_data)
    except Result.DoesNotExist:
        result = serializer.save()

    return redirect(reverse('chat') + f'?id={result.id}')
else:
    print(serializer.errors) 
```
This code snippet is used to serialize data and interact with the `Result` model in Django using the `ResultSerializer` class. Let's break it down step by step:

1. `serializer = ResultSerializer(data=source)`: This line initializes an instance of the `ResultSerializer` class and provides it with data from the `source` dictionary. The `ResultSerializer` is responsible for converting complex data types, like the `source` dictionary, into a format that can be stored in the `Result` model.

2. `if serializer.is_valid():`: This conditional statement checks if the data provided to the serializer is valid according to the validation rules defined in the `ResultSerializer` class.

3. `try:`: This marks the beginning of a try-except block, where the code attempts to perform certain operations, and if an error occurs, it is caught and handled in the `except` block.

4. `result = Result.objects.get(user=request.user, videoid=source['videoid'])`: This line attempts to retrieve a `Result` object from the database that matches both the logged-in user and the `videoid` from the `source` dictionary. If a match is found, it means there's an existing record with the same `videoid`.

5. `serializer.update(result, serializer.validated_data)`: If an existing record is found, this line updates the fields of the existing `result` object with the new validated data from the serializer. This is a way of updating the existing record with new information.

6. `except Result.DoesNotExist:`: If no matching record is found in the database, this block is executed, indicating that a new record needs to be created.

7. `result = serializer.save()`: This line saves the data from the serializer to the `Result` model, creating a new record.

8. `return redirect(reverse('chat') + f'?id={result.id}')`: After either updating an existing record or creating a new one, the code redirects the user to a view named `'chat'` with a query parameter `id` that corresponds to the ID of the saved `Result` object.

9. `else:`: If the serializer data is not valid, this block is executed.

10. `print(serializer.errors)`: This line prints out any validation errors that occurred during the serialization process. This can help in identifying and debugging issues with the data being processed.

In summary, this code segment serializes data using `ResultSerializer`, updates existing records or creates new ones based on the validity of the data, and handles redirection and error reporting. It's a key part of the logic for managing and interacting with `Result` objects in the application.

Our final  `getoutput` function in views.py will be like this:

```python
@api_view(['POST'])
@permission_classes([IsAuthenticated])
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

        username = request.user.username
        recipient = User.objects.get(username=username)
        recipient_email = recipient.email

        async def run_async():
            stats, df, videoid, positive, negative, neutral = await get_result(url, youtubeapikey, username, recipient_email)

            source = {
                # if using API 'user' should be added
                'user': request.user.pk,
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

        serializer = ResultSerializer(data=source)
        if serializer.is_valid():
            # Check if a record with the same videoid exists
            try:
                result = Result.objects.get(user=request.user, videoid=source['videoid'])
                serializer.update(result, serializer.validated_data)
            except Result.DoesNotExist:
                result = serializer.save()


            return redirect(reverse('chat') + f'?id={result.id}')
        else:
            print(serializer.errors) 


    else:
        return render(request, "home.html", {'context':context})

```
Before we jump to the next step, if you look at the top of the code, we utilize a decorator (@) provided by Django REST Framework (DRF) to define the behavior and permissions associated with a specific API view. Let's break down each line:

- @api_view(['POST']): This decorator, @api_view, is used to specify which HTTP methods are allowed for this API view. In this case, it's set to ['POST'], meaning this view will only respond to HTTP POST requests. This decorator ensures that the view is properly configured for handling POST data. It's not only limited to POST method, we can use DELETE, POST, etc.

- @permission_classes([IsAuthenticated]): This decorator, @permission_classes, is used to define the permissions required to access the view. In this case, [IsAuthenticated] is specified, which means that only authenticated users are allowed to access this view. The IsAuthenticated permission class is a built-in DRF permission class that restricts access to authenticated users.

Our setup of Django REST Framework is nearly complete. Up to this point, we've primarily utilized serializers to populate our model with data. However, if we intend to retrieve data from our model through an API endpoint, we need to take an additional step. To begin, we must establish a function that provides access to our API, followed by seamlessly incorporating this new pathway into our `urls.py` configuration.

```python
@api_view(['GET', 'PUT', 'DELETE'])
def result_list_by_user(request,user):

    try:
        result = Result.objects.filter(user=user)
    except Result.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = ResultSerializer(result, many = True)
        return Response(serializer.data)
```
This function defines an API endpoint that retrieves a list of Result objects associated with a specific user. The user parameter is received from the URL. The code attempts to retrieve Result objects filtered by the provided user. If no results are found, a 404 Not Found response is returned. If the HTTP method is GET, the ResultSerializer is used to serialize the results and return them as a response.

```python 
@api_view(['GET', 'PUT', 'DELETE'])
def result_details(request,user,id):

    try:
        result = Result.objects.filter(user=user,id= id)
    except Result.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = ResultSerializer(result, many = True)
        return Response(serializer.data)

    elif request.method == 'DELETE':
        result.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
This function defines an API endpoint to retrieve, update, or delete a specific Result object identified by both the user and the id parameter from the URL. Similar to the previous function, it tries to retrieve a Result object based on the user and id. If the object is not found, a 404 Not Found response is returned. For GET requests, the serialized data of the result is returned. For DELETE requests, the result is deleted and a 204 No Content response is returned.

Overall, these views provide API endpoints to retrieve lists of Result objects for a specific user and to perform CRUD operations (GET, PUT, DELETE) on individual Result objects based on the user and id. The use of DRF decorators and serializers simplifies the process of building these API endpoints while adhering to best practices for API design. Next step is registering a new path in urls.py

```python 
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name="home"),
    path('getoutput',views.getoutput, name = "getoutput"),
    path('chat',views.chat,name = "chat"),
    path('login/', views.loginpage, name='login'),
    path('signup/', views.signuppage, name='signup'),
    path('logout/', views.logoutpage, name='logout'),
    path('result/<int:user>', views.result_list_by_user),
    path('result/<int:user>/<int:id>', views.result_details)
]
```
We have successfully integrated Django REST Framework. Now, upon entering `http://127.0.0.1:8000/result/1` into your browser, the `result_list_by_user` view corresponding to the 'result/<int:user>' path pattern will be executed. Ultimately, you will be presented with a Django Rest Framework template showcasing your data, as follows:

![django rest framework example](/_images/ra_restapi.png)

### Develop an AI Chatbot

This phase represents the pinnacle of sophistication within our project. Prepare to construct a cutting-edge chatbot that will enable our users to engage in dynamic interactions and seek information pertinent to their YouTube videos. If you recall the progression laid out in Part 1, we will replicate those steps for this endeavor as well. Our initial stride involves crafting a foundational code that will serve as the essence of our chatbot, leveraging the capabilities of Langchain and Large Language Model. Following this, we'll proceed to elevate our Django `views.py` by incorporating a new view aptly named 'chat'. Let's get started!

First, let's revisit our `maincodes.py`, where we previously composed our scraping code. As a swift reminder, if you have completed Part 1, you will encounter crucial components such as the imported libraries and our prominent large language model, depicted as follows:

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

Secondly, create a new function named answer_question with a set of parameters, including question, videoid, videotitle, view, like, comment, total_positive_comment, positive_comment, total_negative_comment, negative_comment, total_neutral_comment, and neutral_comment.

```python
def answer_question(question,videoid, videotitle, view, like, comment, total_positive_comment, positive_comment, total_negative_comment, negative_comment, total_neutral_comment, neutral_comment):
```

Next, let's proceed by crafting a template—a command that will guide our Large Language Model. Here's the format you can follow:

```python
    template = """
    You are an intelligent chatbot that act as a senior data consultant. 
    Here is report of youtube video performance from your client:
    The video title is {videotitle} with video id is {videoid}. 

    Here are some statistic of the video performance:
    Total view (people viewing client's video): {view}
    Total like (people liking client's video): {like}
    Total comment (people commenting client's video): {comment}
    Total negative comment (people giving negative comments to client's video): {total_negative_comment}
    Total positive comment (people giving positive comments to client's video): {total_positive_comment}
    Total neutral comment (people giving neutral comments to client's video): {total_neutral_comment}

    Positive comments: {positive_comment}
    Negative comments: {negative_comment} 
    Neutral comments: {neutral_comment} 

    Answer the following question with the fact based on the report provided above. Don't hallucinating! Be concise and don't repeating the same thing
    Question: {question}
    Answer: your answer here be concise and specific
    
    """
```

With the template in place, the subsequent step involves integrating it into a `PromptTemplate` from Langchain and storing it within a variable named `prompt`. The `PromptTemplate` constructor accepts two crucial parameters: `template` and `input_variables`. We'll provide our formulated template for the `template` parameter, while the `input_variables` parameter encompasses the variables employed within our template. 

```python 
    prompt = PromptTemplate(
        template=template, 
        input_variables=[
            "question","videoid","videotitle","view","like","comment",
            "total_positive_comment", "positive_comment", "total_negative_comment",
            "negative_comment", "total_neutral_comment", "neutral_comment"
            ]
        )
```

Subsequently, it becomes imperative to establish a chain model using `LLMChain`, requiring two pivotal parameters. Initially, the `prompt` - the template we've devised - assumes the forefront. Meanwhile, the second parameter necessitates specifying the Large Language Model of our choice. Given that we've stored our LLM under the variable `falcon_llm`, we can readily reference it within this context. 

```python
    llm_chain = LLMChain(prompt=prompt, llm=falcon_llm)
```

Lastly, to get the result, we can use llm_chain.run command with adding some parameter as below:
```python
    answer = llm_chain.run(
        question=question, videoid = videoid, 
        videotitle = videotitle, view = view, like = like, 
        comment = comment, total_positive_comment = total_positive_comment, 
        positive_comment = positive_comment, total_negative_comment = total_negative_comment,
        negative_comment = negative_comment, total_neutral_comment = total_neutral_comment,
        neutral_comment = neutral_comment)
    
    return answer
```
Our final `answer_question`` function would be like this

```python
def answer_question(question,videoid, videotitle, view, like, comment, total_positive_comment, positive_comment, total_negative_comment, negative_comment, total_neutral_comment, neutral_comment):

    template = """
    You are an intelligent chatbot that act as a senior data consultant. 
    Here is report of youtube video performance from your client:
    The video title is {videotitle} with video id is {videoid}. 

    Here are some statistic of the video performance:
    Total view (people viewing client's video): {view}
    Total like (people liking client's video): {like}
    Total comment (people commenting client's video): {comment}
    Total negative comment (people giving negative comments to client's video): {total_negative_comment}
    Total positive comment (people giving positive comments to client's video): {total_positive_comment}
    Total neutral comment (people giving neutral comments to client's video): {total_neutral_comment}

    Positive comments: {positive_comment}
    Negative comments: {negative_comment} 
    Neutral comments: {neutral_comment} 

    Answer the following question with the fact based on the report provided above. Don't hallucinating! Be concise and don't repeating the same thing
    Question: {question}
    Answer: your answer here be concise and specific
    
    """

    prompt = PromptTemplate(
        template=template, 
        input_variables=[
            "question","videoid","videotitle","view","like","comment",
            "total_positive_comment", "positive_comment", "total_negative_comment",
            "negative_comment", "total_neutral_comment", "neutral_comment"
            ]
        )

    llm_chain = LLMChain(prompt=prompt, llm=falcon_llm)

    print("start working")

    answer = llm_chain.run(
        question=question, videoid = videoid, 
        videotitle = videotitle, view = view, like = like, 
        comment = comment, total_positive_comment = total_positive_comment, 
        positive_comment = positive_comment, total_negative_comment = total_negative_comment,
        negative_comment = negative_comment, total_neutral_comment = total_neutral_comment,
        neutral_comment = neutral_comment)
    
    print("answer ready")
    
    return answer
```

One last thing is to update our views.py. Since we already implement Django REST Framework and created API endpoints, let's utilize it in our chat function.

```python
last_id = None
@login_required(login_url='login')
def chat(request):

    context = {
        'user_id': request.user.pk
    }

    global last_id

    user = request.user.pk
    username = request.user

    button_id = request.POST.get('button_id')
    print(button_id)

    id = request.GET.get('id')
    print(id)


    if button_id is not None and button_id != '':
        print('y')
        last_id = button_id 
        print(button_id)
        api_url = f'http://127.0.0.1:8000/result/{user}/{button_id}'
        print(api_url)
    else:
        print('n')
        if id != None:
            api_url = f'http://127.0.0.1:8000/result/{user}/{id}'
            print(api_url)
            last_id = id
        else:
            api_url = f'http://127.0.0.1:8000/result/{user}/{last_id}'
            print(api_url)

    response = requests.get(api_url)

    if response.status_code == 200:
        data = response.json()
        source = data[0]
        print(source)
    else:
        print('Error:', response.status_code)

    videoid = source['videoid'], 
    videotitle= source['videotitle'],
    view = source['view'],
    like = source['like'],
    comment = source['comment'],
    total_positive_comment = source['total_positive_comment'],
    positive_comment = source['positive_comment'],
    total_negative_comment = source['total_negative_comment'],
    negative_comment = source['negative_comment']
    total_neutral_comment = source['total_neutral_comment'],
    neutral_comment = source['neutral_comment']

    if request.method == 'POST' or request.method == 'GET':
        user_input = request.POST.get('user_input')
        print(user_input)

        if user_input is not None and user_input != '':

            answer = answer_question(
              user_input, videoid, videotitle, view, like, comment,
              total_positive_comment, positive_comment,
              total_negative_comment, negative_comment,
              total_neutral_comment, neutral_comment)

            print(answer)
        
        else:
            answer = f'''Hey there, {username}! Let's dive deep into your report together. 
                    Feel free to ask me anything you'd like, whether it's seeking advice, 
                    summarizing the comments on your video, or exploring other fascinating insights! 
                    We're here to make your experience as engaging and informative as possible!'''
            print(answer)

    return(render(request, "home.html", {"response":answer, "source":source, "context":context}))

```

To gain a better understanding, let's break down its functionality:

1. `@login_required(login_url='login')`: This decorator ensures that only authenticated users can access the `chat` view. If a user is not authenticated, they will be redirected to the specified login page (`'login'`).

2. `context`: A dictionary containing the current user's ID is created, which will be passed to the template for rendering.

3. `global last_id`: Declares a global variable named `last_id`, presumably intended to keep track of a specific ID for user interactions.

4. Various variables are assigned using the request's POST and GET data, such as `button_id` and `id`, both of which are print-debugged for monitoring.

5. The code checks whether `button_id` is provided and updates `last_id` accordingly, forming an API URL to fetch data.

6. If no `button_id` is provided, the code checks for the existence of `id`, and if present, constructs an API URL with `id`. If neither `button_id` nor `id` is available, the code uses `last_id` to generate the API URL.

7. The application sends a GET request to the API URL and retrieves data, storing it in the `source` dictionary.

8. Key data elements (e.g., `videoid`, `videotitle`, etc.) are extracted from the `source` dictionary.

9. The view handles both POST and GET requests. In case of a user input, the function calls the `answer_question` function to generate a response. If no user input is provided, a default response is generated.

10. The view renders the `home.html` template, passing the generated response, source data, and context for rendering.

In summary, this view function handles user interactions with the web application's chat feature. It manages user input, communicates with an API to retrieve data, generates responses using the `answer_question` function, and ultimately renders the response along with other relevant data in the `home.html` template.

In reference to our previous discussion on APIs, during the journey of developing this project, I came across a highly beneficial approach: implementing the Large Language Model as a separate API. The decision to compartmentalize the model has proven to have significant advantages, particularly when dealing with large-sized models. By segregating the model into an API, we have effectively optimized scalability, security, and the ease of model updates. This strategic separation has proven to be an immensely advantageous choice in our project's development journey.

The way we transform a Large Language Model into an API endpoint is by using Langcorn and FASTAPI. I found a great tutorial on using Langcorn and FASTAPI in this [video](https://www.youtube.com/watch?v=iFvCZD4iS2w&t=392s) by Assembly AI. Langcorn is also well-documented; please take a look at this [Langcorn Github Repository](https://github.com/msoedov/langcorn). The process is quite straightforward, so please watch the tutorial and follow the steps one by one. While I won't include the entire sequence in this article, I will provide you with some essential code snippets that were not covered in the video. These snippets explain how to call the API endpoint and integrate it into our chat view. Before proceeding with the modification of our chat function in `views.py`, ensure that when you enter your API endpoint in your browser, you receive a result similar to the following:

![ra_langcorn](/_images/ra_langcorn.png)


If you haven't encountered this view, there's no need to worry. I have also provided a code for you. Please kindly check my [Github Repository](https://github.com/projectwilsen/ReviewAnalyzerLangchainApp). One last step of this project, let's update our chat function in `views.py` become like this:


```python
last_id = None
@login_required(login_url='login')
def chat(request):

    context = {
        'user_id': request.user.pk
    }

    global last_id

    user = request.user.pk
    username = request.user

    button_id = request.POST.get('button_id')
    print(button_id)

    id = request.GET.get('id')
    print(id)


    if button_id is not None and button_id != '':
        print('y')
        last_id = button_id 
        print(button_id)
        api_url = f'http://127.0.0.1:8000/result/{user}/{button_id}'
        print(api_url)
    else:
        print('n')
        if id != None:
            api_url = f'http://127.0.0.1:8000/result/{user}/{id}'
            print(api_url)
            last_id = id
        else:
            api_url = f'http://127.0.0.1:8000/result/{user}/{last_id}'
            print(api_url)

    response = requests.get(api_url)

    if response.status_code == 200:
        data = response.json()
        source = data[0]
        print(source)
    else:
        print('Error:', response.status_code)


    if request.method == 'POST' or request.method == 'GET':
        user_input = request.POST.get('user_input')
        print(user_input)

        if user_input is not None and user_input != '':
            
            
            headers = {
                "accept":"application/json",
                "Content-Type":"application/json"
            }

            json_data = {
                "question": user_input,
                "videoid" : source['videoid'],
                "videotitle": source['videotitle'], 
                "view" : source['view'],
                "like" : source['like'],
                "comment" : source['comment'],
                "total_positive_comment" : source['total_positive_comment'],
                "positive_comment" : source['positive_comment'],
                "total_negative_comment" : source['total_negative_comment'],
                "negative_comment" : source['negative_comment'],
                "total_neutral_comment" : source['total_neutral_comment'],
                "neutral_comment" : source['neutral_comment']
            }


            response = requests.post(
                "your_langchain_model_api_endpoint_here", headers = headers, json = json_data
            )

            data = response.text
            parsed_data = json.loads(data)
            print(parsed_data)
            answer = parsed_data['output']

            print(answer)
        
        else:
            answer = f'''Hey there, {username}! Let's dive deep into your report together. 
                    Feel free to ask me anything you'd like, whether it's seeking advice, 
                    summarizing the comments on your video, or exploring other fascinating insights! 
                    We're here to make your experience as engaging and informative as possible!'''
            print(answer)

    return(render(request, "home.html", {"response":answer, "source":source, "context":context}))
```
Once again, don't forget to change `your_langchain_model_api_endpoint_here` in

```python
    response = requests.post(
        "your_langchain_model_api_endpoint_here", headers = headers, json = json_data
   )
```
with your api endpoint. Right after we change the chat function, we can delete or comment the `answer_question` function in maincodes.py since we don't need it anymore.


### Conclusion

As we conclude our project journey, we recognize that our implementation merely scratches the surface of the expansive capabilities offered by Large Language Models (LLMs). I encourage you to delve deeper into the myriad features that LLMs, especially those harnessed through Langchain, have to offer. While certain aspects remained unexplored within this project, you can broaden your horizons by exploring a playlist titled [LangChain & LLM tutorials (ft. gpt3, chatgpt, llamaindex, chroma)](https://www.youtube.com/playlist?list=PLXsFtK46HZxUQERRbOmuGoqbMD-KWLkOS) explained by Samuel Chan. I hope that this project has laid a strong foundation for your endeavors, and I eagerly anticipate witnessing the innovative AI creations that you will undoubtedly develop. Until our next article, stay inspired and keep innovating!

### Relevant Links

* Project: <u>[https://github.com/projectwilsen/ReviewAnalyzer/](https://github.com/projectwilsen/ReviewAnalyzer/)</u>  
* Langchain API Model using Langcorn: <u>[https://github.com/projectwilsen/ReviewAnalyzerLangchainApp](https://github.com/projectwilsen/ReviewAnalyzerLangchainApp)</u>