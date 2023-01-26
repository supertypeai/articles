---
title: Twitter Sentiment Analysis - Creating Dashboard and Deploying Model with Streamlit (Part 3)
post_excerpt: End-to-end machine learning project on sentiment analysis. In this post, we will walk through the steps of creating a dashboard and deploying our model as a web app with Streamlit.
taxonomy:
    category:
        - knowledge
        - notes
---


# End-to-End Machine Learning Project: Twitter Sentiment Analysis - Creating Dashboard and Deploying Model with Streamlit (Part 3)

In the <u>[previous post](https://supertype.ai/notes/twitter-sentiment-analysis-part-2/)</u>, we have build a LSTM model that can predict the sentiment of a tweet. Now we will create a dashboard and deploy our model using Streamlit. Our Streamlit app will automatically scrape the latest tweets based on the entered search term, then perform inference in real-time and visualize the results.

## Creating Dashboard with Streamlit

First we will make a folder with the following structure:
```
TwitterSentimentAnalyzer
├── app.py
├── helper_functions.py
├── requirements.txt
└── static
    ├──lstm_model.h5
    ├──tokenizer.pickle
    ├──en_stopwords.txt
    ├──en_stopwords_viz.txt
    ├──twitter_mask.png
    └──quartzo.ttf 
```

The `app.py` will contain the code for the dashboard design, while the `helper_functions.py` will contain the code for retrieving the tweets, preprocessing the tweets, performing inference and visualizing the results. The `requirements.txt` will contain the list of libraries that we will need to install. The `static` folder contain supporting files, such as the trained model (`lstm_model.h5`), tokenizer (`tokenizer.pickle`), stopwords for sentiment prediction pipeline (`en_stopwords.txt`), stopwords for visualization purpose (`en_stopwords_viz.txt`), twitter image for wordcloud (`twitter_mask.png`) and font type for wordcloud (`quartzo.ttf`). All of the supporting files are available in the <u>[GitHub repository](https://github.com/tmtsmrsl/TwitterSentimentAnalyzer/tree/main/static)</u>.

**helper_functions.py**  

We will populate the `helper_functions.py` with the necessary functions. First we will import all necessary libraries.


```python
import numpy as np
import pandas as pd

# for scraping tweets
import snscrape.modules.twitter as sntwitter

# for loading the model and tokenizer
from tensorflow.keras.models import load_model
import pickle

# for text processing
from tensorflow.keras.preprocessing.sequence import pad_sequences
import nltk
nltk.download(
    ["punkt", "wordnet", "omw-1.4", "averaged_perceptron_tagger", "universal_tagset"]
)
from nltk.stem import WordNetLemmatizer
from nltk.tag import pos_tag
from nltk.tokenize import word_tokenize
import re
from sklearn.feature_extraction.text import CountVectorizer

# for visualization
import plotly.express as px
import plotly.io as pio
import matplotlib as mpl
import matplotlib.pyplot as plt
from wordcloud import WordCloud
from PIL import Image
```

Then we will create `get_latest_tweet_df` function which will retrieve the latest tweets based on the entered search term and number of tweets requested. We will use the `snscrape` library to retrieve the tweets, which does not require using the Twitter API. The `get_latest_tweet_df` function will return a dataframe containing the username of the poster, date, number of likes and the tweet itself.


```python
def get_latest_tweet_df(search_term, num_tweets):
    tweet_data = []
    # only scrape tweets in English
    for i, tweet in enumerate(
        sntwitter.TwitterSearchScraper("{} lang:en".format(search_term)).get_items()
    ):
        # the number of tweets scraped are limited to 5000
        if i >= num_tweets or i >= 5000:
            break
        tweet_data.append(
            [tweet.user.username, tweet.date, tweet.likeCount, tweet.content]
        )

    tweet_df = pd.DataFrame(
        tweet_data, columns=["Username", "Date", "Like Count", "Tweet"]
    )
    return tweet_df
```

Next we will create `text_preprocessing` function which will preprocess the tweets. The preprocessing steps have been explained in the <u>[previous post](https://supertype.ai/notes/twitter-sentiment-analysis-part-2/)</u>.


```python
def text_preprocessing(text):
    # load the stopwords and lemmatizer
    stopwords = set()
    with open("static/en_stopwords.txt", "r") as file:
        for word in file:
            stopwords.add(word.rstrip("\\n"))
    lemmatizer = WordNetLemmatizer()
    
    try:
        url_pattern = r"((http://)[^ ]*|(https://)[^ ]*|(www\\.)[^ ]*)"
        user_pattern = r"@[^\\s]+"
        entity_pattern = r"&.*;"
        neg_contraction = r"n't\\W"
        non_alpha = "[^a-z]"
        cleaned_text = text.lower()
        cleaned_text = re.sub(neg_contraction, " not ", cleaned_text)
        cleaned_text = re.sub(url_pattern, " ", cleaned_text)
        cleaned_text = re.sub(user_pattern, " ", cleaned_text)
        cleaned_text = re.sub(entity_pattern, " ", cleaned_text)
        cleaned_text = re.sub(non_alpha, " ", cleaned_text)
        tokens = word_tokenize(cleaned_text)
        # provide POS tag for lemmatization to yield better result
        word_tag_tuples = pos_tag(tokens, tagset="universal")
        tag_dict = {"NOUN": "n", "VERB": "v", "ADJ": "a", "ADV": "r"}
        final_tokens = []
        for word, tag in word_tag_tuples:
            if len(word) > 1 and word not in stopwords:
                if tag in tag_dict:
                    final_tokens.append(lemmatizer.lemmatize(word, tag_dict[tag]))
                else:
                    final_tokens.append(lemmatizer.lemmatize(word))
        return " ".join(final_tokens)
    except:
        return np.nan
```

Next we will create `predict_sentiment` function which acts as the inference pipeline, with tweet dataframe (ouput of `get_latest_tweet_df` function) as the input. The function will first preprocess the tweets by applying the `text_preprocessing` function we have created earlier, then convert the preprocessed tweets into sequences of integers using the pre-fitted tokenizer. The sequences of integers are then padded to the same length as the training data. The padded sequences are then passed to the pre-trained model to perform inference. The `predict_sentiment` function will return the original dataframe with the predicted score and sentiment.


```python
def predict_sentiment(tweet_df):
    # load the model and tokenizer
    model = load_model("static/lstm_model.h5")
    with open("static/tokenizer.pickle", "rb") as handle:
        custom_tokenizer = pickle.load(handle)
        
    # preprocess the tweets
    temp_df = tweet_df.copy()
    temp_df["Cleaned Tweet"] = temp_df["Tweet"].apply(text_preprocessing)
    temp_df = temp_df[temp_df["Cleaned Tweet"].notna() & temp_df["Cleaned Tweet"] != ""]
    
    # tokenize the tweets then pad the sequences (54 is the maxlen of the training data)
    sequences = pad_sequences(
        custom_tokenizer.texts_to_sequences(temp_df["Cleaned Tweet"]), maxlen=54
    )
    
    # predict the sentiment by setting the probability threshold to 0.50
    score = model.predict(sequences)
    temp_df["Score"] = score
    temp_df["Sentiment"] = temp_df["Score"].apply(
        lambda x: "Positive" if x >= 0.50 else "Negative"
    )
    return temp_df
```

Now we will create some functions related to visualization. First we will create `plot_sentiment` function which will plot the number of positive and negative tweets in a pie chart. The function will take the tweet dataframe (output of the `predict_sentiment` function) as input and return a plotly figure.


```python
def plot_sentiment(tweet_df):
    # count the number tweets based on the sentiment
    sentiment_count = tweet_df["Sentiment"].value_counts()
    
    # plot the sentiment distribution in a pie chart
    fig = px.pie(
        values=sentiment_count.values,
        names=sentiment_count.index,
        hole=0.3,
        title="<b>Sentiment Distribution</b>",
        color=sentiment_count.index,
        # set the color of positive to blue and negative to orange
        color_discrete_map={"Positive": "#1F77B4", "Negative": "#FF7F0E"},
    )
    fig.update_traces(
        textposition="inside",
        texttemplate="%{label}<br>%{value} (%{percent})",
        hovertemplate="<b>%{label}</b><br>Percentage=%{percent}<br>Count=%{value}",
    )
    fig.update_layout(showlegend=False)
    return fig
```

Next we will create `plot_wordcloud` function which will show the top words in form of a wordcloud. The function will take the tweet dataframe (output of the `predict_sentiment` function) and color map as inputs and return a matplotlib figure. First we load the stopwords from `en_stopwords_viz.txt`, which is a list of words that we want to remove from the wordcloud. We use a different stopwords for visualization and text preprocessing because we want to remove more words for visualization purpose. We also load the image from `twitter_mask.png` to be used as the mask for the wordcloud, and the font type from `quartzo.ttf`. Next we create a custom colormap using the `LinearSegmentedColormap` function from the `matplotlib` library. We do not use the default colormap because we want to control the color intensity of the wordcloud. Finally, we will generate a wordcloud based on the words frequencies of the combined preprocessed tweets. The wordcloud is returned as a matplotlib figure.


```python
def plot_wordcloud(tweet_df, colormap="Greens"):
    # load the stopwords
    stopwords = set()
    with open("static/en_stopwords_viz.txt", "r") as file:
        for word in file:
            stopwords.add(word.rstrip("\\n"))
            
    # load the mask image and font type
    mask = np.array(Image.open("static/twitter_mask.png"))
    font = "static/quartzo.ttf"
            
    # generate custom colormap
    cmap = mpl.cm.get_cmap(colormap)(np.linspace(0, 1, 20))
    cmap = mpl.colors.ListedColormap(cmap[10:15])
    
    # combine all the preprocessed tweets into a single string
    text = " ".join(tweet_df["Cleaned Tweet"])
    
    # create the WordCloud instance
    wc = WordCloud(
        background_color="white",
        font_path=font,
        stopwords=stopwords,
        max_words=90,
        colormap=cmap,
        mask=mask,
        random_state=42,
        collocations=False,
        min_word_length=2,
        max_font_size=200,
    )
    
    # generate and plot the wordcloud
    wc.generate(text)
    fig = plt.figure(figsize=(8, 8))
    ax = fig.add_subplot(1, 1, 1)
    plt.imshow(wc, interpolation="bilinear")
    plt.axis("off")
    plt.title("Wordcloud", fontdict={"fontsize": 16}, fontweight="heavy", pad=20, y=1.0)
    return fig
```

We also want to visualize the top words and bigrams in form of a bar chart. First we create `get_top_n_gram` function which takes the tweet dataframe (output of the `predict_sentiment` function), range of n-gram (e.g. (1,1) for word and (2,2) for bigram) and number of top words as inputs and return a dataframe containing the top n-grams and their frequencies. We then create `plot_top_n_gram` function which takes the n-gram dataframe (output of the `get_top_n_gram` function), title, and color as inputs; and then plot the top n-grams in form of a bar chart.


```python
def get_top_n_gram(tweet_df, ngram_range, n=10):
    # load the stopwords
    stopwords = set()
    with open("static/en_stopwords_viz.txt", "r") as file:
        for word in file:
            stopwords.add(word.rstrip("\\n"))
    
    # load the corpus and vectorizer
    corpus = tweet_df["Cleaned Tweet"]
    vectorizer = CountVectorizer(
        analyzer="word", ngram_range=ngram_range, stop_words=stopwords
    )
    
    # use the vectorizer to count the n-grams frequencies
    X = vectorizer.fit_transform(corpus.astype(str).values)
    words = vectorizer.get_feature_names_out()
    words_count = np.ravel(X.sum(axis=0))
    
    # store the results in a dataframe
    df = pd.DataFrame(zip(words, words_count))
    df.columns = ["words", "counts"]
    df = df.sort_values(by="counts", ascending=False).head(n)
    df["words"] = df["words"].str.title()
    return df


def plot_n_gram(n_gram_df, title, color="#54A24B"):
    # plot the top n-grams frequencies in a bar chart
    fig = px.bar(
        x=n_gram_df.counts,
        y=n_gram_df.words,
        title="<b>{}</b>".format(title),
        text_auto=True,
    )
    fig.update_layout(plot_bgcolor="white")
    fig.update_xaxes(title=None)
    fig.update_yaxes(autorange="reversed", title=None)
    fig.update_traces(hovertemplate="<b>%{y}</b><br>Count=%{x}", marker_color=color)
    return fig
```

**app.py**  

We will design our dashboard by populating the `app.py` file. First we will import the necessary libraries, which include the `helper_functions` that we have created earlier.


```python
import streamlit as st
import pandas as pd
import helper_functions as hf
```

Next we will set the page configuration (title, icon and layout) of the app. 


```python
st.set_page_config(
    page_title="Twitter Sentiment Analyzer", page_icon="📊", layout="wide"
)
```

We also want to adjust the layout of the app to make the interface more visually appealing. We will use HTML and CSS to adjust the top padding of the block container.


```python
adjust_top_pad = """
    <style>
        div.block-container {padding-top:1rem;}
    </style>
    """
st.markdown(adjust_top_pad, unsafe_allow_html=True)
```

Now we will define the callback function for the 'Search' button (which we will discuss next). Whenever the 'Search' button is clicked, the `search_callback` function will be called. `search_callback` function will retrieve the latest tweets (in form of dataframe) by passing the entered search term and number of tweets to `get_latest_tweet_df` function; and then perform inference by calling `predict_sentiment` function. 


```python
def search_callback():
    st.session_state.df = hf.get_latest_tweet_df(
        st.session_state.search_term, st.session_state.num_tweets
    )
    st.session_state.df = hf.predict_sentiment(st.session_state.df)
```

Next we will use the `st.sidebar` to create a sidebar for the app. First we will include some information about the app in the top of the sidebar. Then we will create a form (using `st.form`) for the user to enter the search term (using `st.text_input`) and number of tweets (using `st.slider`). By using `st.form`, the input widgets will be grouped and submitted in batch once the user click the 'Search' button (`st.form_submit_button`). Clicking the 'Search' button will update the session state (associated with the widget key) and trigger the `search_callback` function that we have created earlier. Last we will include the Github link of the project and the author's name in the bottom of the sidebar.


```python
with st.sidebar:
    st.title("Twitter Sentiment Analyzer")

    st.markdown(
        """
        <div style="text-align: justify;">
            This app performs sentiment analysis on the latest tweets based on 
            the entered search term. Since the app can only predict positive or 
            negative sentiment, it is more suitable towards analyzing the 
            sentiment of brand, product, service, company, or person. 
            Only English tweets are supported.
        </div>
        """,
        unsafe_allow_html=True,
    )

    # create a form to obtain the search parameters
    with st.form(key="search_form"):
        st.subheader("Search Parameters")
        # session_state.search_term will be updated when the form is submitted
        st.text_input("Search term", key="search_term")
        # session_state.num_tweets will be updated when the form is submitted
        st.slider("Number of tweets", min_value=100, max_value=2000, key="num_tweets")
        # search_callback will be called when the form is submitted
        st.form_submit_button(label="Search", on_click=search_callback)
        st.markdown(
            "Note: it may take a while to load the results, especially with large number of tweets"
        )

    st.markdown("[Github link](https://github.com/tmtsmrsl/TwitterSentimentAnalyzer)")
    st.markdown("Created by Timotius Marselo")
```

Now we will design the main part of the app. First we will check if `st.session_state.df` exists. If it exists, it means that the user has already clicked the 'Search' button and the `search_callback` function has been called, so we will display the dashboard. The dashboards are created using the `make_dashboard` function, which takes the tweet dataframe, bar chart color and wordcloud color as inputs. The `make_dashboard` function will create a container with two rows. The first row will contain the pie chart of sentiment distribution, bar chart of the top 10 words, and bar chart of the top 10 bigrams. The second row will contain the dataframe (containing the tweets and their sentiments) and the wordcloud. We rely on the functions from the `helper_functions` to create the visualizations. We will use `st.tabs` to create 3 tabs in the dashboard, which are 'All', 'Positive', and 'Negative'. Each tab will display the dashboard for the tweets in that sentiment category.   


```python
if "df" in st.session_state:
    # function to make the dashboard
    def make_dashboard(tweet_df, bar_color, wc_color):
        # make 3 columns for the first row of the dashboard
        col1, col2, col3 = st.columns([28, 34, 38])
        with col1:
            # plot the sentiment distribution
            sentiment_plot = hf.plot_sentiment(tweet_df)
            sentiment_plot.update_layout(height=350, title_x=0.5)
            st.plotly_chart(sentiment_plot, theme=None, use_container_width=True)
            
        with col2:
            # plot the top 10 occuring words 
            top_unigram = hf.get_top_n_gram(tweet_df, ngram_range=(1, 1), n=10)
            unigram_plot = hf.plot_n_gram(
                top_unigram, title="Top 10 Occuring Words", color=bar_color
            )
            unigram_plot.update_layout(height=350)
            st.plotly_chart(unigram_plot, theme=None, use_container_width=True)
            
        with col3:
            # plot the top 10 occuring bigrams
            top_bigram = hf.get_top_n_gram(tweet_df, ngram_range=(2, 2), n=10)
            bigram_plot = hf.plot_n_gram(
                top_bigram, title="Top 10 Occuring Bigrams", color=bar_color
            )
            bigram_plot.update_layout(height=350)
            st.plotly_chart(bigram_plot, theme=None, use_container_width=True)

        # make 2 columns for the second row of the dashboard
        col1, col2 = st.columns([60, 40])
        with col1:
            # function to color the sentiment column
            def sentiment_color(sentiment):
                if sentiment == "Positive":
                    return "background-color: #1F77B4; color: white"
                else:
                    return "background-color: #FF7F0E"
            
            # show the dataframe containing the tweets and their sentiment
            st.dataframe(
                tweet_df[["Sentiment", "Tweet"]].style.applymap(
                    sentiment_color, subset=["Sentiment"]
                ),
                height=350
            )
            
        with col2:
            # plot the wordcloud
            wordcloud = hf.plot_wordcloud(tweet_df, colormap=wc_color)
            st.pyplot(wordcloud)

    # increase the font size of text inside the tabs
    adjust_tab_font = """
    <style>
    button[data-baseweb="tab"] > div[data-testid="stMarkdownContainer"] > p {
        font-size: 20px;
    }
    </style>
    """
    st.write(adjust_tab_font, unsafe_allow_html=True)

    # create 3 tabs for all, positive, and negative tweets
    tab1, tab2, tab3 = st.tabs(["All", "Positive 😊", "Negative ☹️"])
    with tab1:
        # make dashboard for all tweets
        tweet_df = st.session_state.df
        make_dashboard(tweet_df, bar_color="#54A24B", wc_color="Greens")
        
    with tab2:
        # make dashboard for tweets with positive sentiment
        tweet_df = st.session_state.df.query("Sentiment == 'Positive'")
        make_dashboard(tweet_df, bar_color="#1F77B4", wc_color="Blues")
        
    with tab3:
        # make dashboard for tweets with negative sentiment
        tweet_df = st.session_state.df.query("Sentiment == 'Negative'")
        make_dashboard(tweet_df, bar_color="#FF7F0E", wc_color="Oranges")

```

**requirements.txt**  

We will populate the `requirements.txt` with the libraries required for the Streamlit app. The version of the libraries are included to ensure that the app will run as expected.

```
streamlit==1.16.0
pandas==1.4.3
numpy==1.24.1
plotly==5.9.0
tensorflow==2.11.0
nltk==3.7
scikit-learn==1.1.1
matplotlib==3.5.1
wordcloud==1.8.2.2
snscrape
```

## Deploying the Streamlit App

### Testing the App Locally

To test our Streamlit app locally, we will change the directory to the folder containing the `app.py` and run the following command in the terminal:

```
streamlit run app.py
```

If the app runs successfully, the browser will automatically open the app.

![local app 1](/_images/tsa_local_app_1.png)

We can test the app by entering a search term and number of tweets to retrieve. After we click the 'Search' button, the app should show the dashboard.

![local app 2](/_images/tsa_local_app_2.png)

### Deploying the App to Streamlit Community Cloud

First we need to upload the project folder to GitHub. Then we will go to the <u>[Streamlit Community Cloud](https://streamlit.io/cloud)</u> and click the 'Get Started' button. After signing in and connecting our GitHhub account, we will click the 'New App' button. We will then enter the repository that we have created earlier, along with the branch and main file path.

![deploy streamlit cloud](/_images/tsa_deploy_streamlit_cloud.png)

After clicking the 'Deploy' button, the app will be deployed to the Streamlit Community Cloud (might take a few minutes for the initial deployment). We can open the app by clicking the app name.

In this post, we have successfully made our Streamlit app and deploy it on Streamlit Community Cloud. If you are following this series from part 1, we have gone through an end-to-end machine learning project, starting from data collection and preprocessing, model building, creating a dashboard, and finally deploying our model and dashboard as an online application. Hopefully you learned something from the series!

## Relevant Links
* Project Github: <u>[github.com/tmtsmrsl/TwitterSentimentAnalyzer](https://github.com/tmtsmrsl/TwitterSentimentAnalyzer)</u>  
* Streamlit App: <u>[twitter-sentiment.streamlit.app/](https://twitter-sentiment.streamlit.app/)</u>
