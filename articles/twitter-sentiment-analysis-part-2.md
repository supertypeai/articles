---
title: Twitter Sentiment Analysis - Data Preprocessing and Model Building (Part 2)
post_excerpt: End-to-end machine learning project on sentiment analysis. In this post, we will walk through the steps of data preprocessing and model building.
taxonomy:
    category:
        - knowledge
        - notes
---


# End-to-End Machine Learning Project: Twitter Sentiment Analysis - Data Preprocessing and Model Building (Part 2)

In the <u>[previous post](https://supertype.ai/notes/twitter-sentiment-analysis-part-1/)</u>, I have explained how to collect the training data. Now, we will preprocess the collected data and use it to build a Long Short-Term Memory (LSTM) model that can predict the sentiment of a tweet.

### Reading the Dataset

The <u>[original dataset](https://github.com/tmtsmrsl/TwitterSentimentAnalyzer/blob/main/dataset/labeled_tweets.csv)</u> contains the following columns: username, date, tweet, and sentiment. We will only use the tweet and sentiment columns for our purpose.


```python
import pandas as pd
import numpy as np
import plotly.express as px

df = pd.read_csv("/content/drive/MyDrive/dataset/labeled_tweets.csv")
df = df.drop(columns=["username", "date"])
```

Let's check the distribution of the sentiment labels in the dataset.

![sentiment distribution plot](/_images/tsa_sentiment_dist.png)

### Text Preprocessing

First, we will create a set of stopwords which contain common words that we want to remove from the tweets. Removing stopwords can reduce the size of the dataset and also minimize the noise in the dataset, however we should choose the stopwords carefully for sentiment analysis. For example, the word "not" is often regarded as a stopword, but it is actually important for sentiment analysis. We do not want the sentences "I am not happy" and "I am happy" to be treated as the same by our model. You can view the custom stopwords that we use <u>[here](https://github.com/tmtsmrsl/TwitterSentimentAnalyzer/blob/main/static/en_stopwords.txt)</u>. We also create an instance of `WordNetLemmatizer` from the NLTK library, which will be used to convert words into their base form while considering the context.


```python
import nltk
nltk.download("punkt", "wordnet", "omw-1.4", "averaged_perceptron_tagger", "universal_tagset")
from nltk.stem import WordNetLemmatizer
from nltk.tag import pos_tag
from nltk.tokenize import word_tokenize
import re

stopwords = set()
with open("/content/drive/MyDrive/static/en_stopwords.txt", "r") as file:
    for word in file:
        stopwords.add(word.rstrip("\\n"))
        
lemmatizer = WordNetLemmatizer()
```

The text prepocessing consists of the following steps:
1) Converting the text to lowercase
2) Expanding negative contractions (n't to not)
3) Removing URLs
4) Removing usernames
5) Removing HTML entities
6) Removing characters that are not letters
7) POS tagging the words (to get better lemmatization results)
8) Excluding words with only 1 character and stopwords
9) Lemmamatizing the remaining words


```python
def text_preprocessing(text):
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

df["cleaned_tweet"] = df["tweet"].apply(text_preprocessing)
```

Some examples of the preprocessed tweets are shown below.

tweet | cleaned_tweet
----- | -------------
I am not very happy with the quality of the coffee from @ExampleShop, wouldn't be buying it anymore | not very happy quality coffee not buy anymore
www.examplestore.com is running a discount promotion, so I bought several t-shirts there. Quite satisfied with the quality of the products. | run discount promotion buy several shirt quite satisfied quality product

### Splitting and Tokenization

Before splitting the dataset, we will drop any rows where the `cleaned_tweet` column is empty. Then, we will divide the dataset into training set (80%) and test set (20%). We will use the training set to train the model, and the test set to evaluate the performance of the final model.


```python
import pickle
from sklearn.model_selection import train_test_split
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences

df = df[df["cleaned_tweet"].notna() & (df["cleaned_tweet"] != "")]

train_df = df.sample(frac=0.8, random_state=42)
test_df = df.drop(index=train_df.index)
```

The distribution of the sentiment labels in the training set and test set are relatively similar.  

---

**Training set sentiment distribution**  
Negative: 283011  (50.9%)  
Positive: 272608  (49.1%)

**Test set sentiment distribution**  
Negative: 71121  (51.2%)  
Positive: 67784 (48.8%)

---

We then extract the feature (X) and target (y) from the training set. The prior training set is further split into training set (80%) and validation set (20%). The training set will be fitted to the model. The validation set will be used to evaluate the model's performance during training and to make decisions about the training process, e.g. when to stop training (to avoid overfitting) and which model to choose as the final model.


```python
X = train_df.cleaned_tweet
y = train_df.sentiment
X_train, X_val, y_train, y_val = train_test_split(
    X, y, train_size=0.8, random_state=42
)
```

Next we will create an instance of `Tokenizer` from the Keras library, which will be used to convert the text into a sequence of integers. The tokenizer is fitted to `X_train` to identify the unique words in the training set and map them to the corresponding integers.


```python
tokenizer = Tokenizer()
tokenizer.fit_on_texts(X_train)    
vocab_size = len(tokenizer.word_index) + 1
print("Number of Unique Words: {}".format(vocab_size))

# saving tokenizer
with open("/content/drive/MyDrive/static/tokenizer.pickle", "wb") as handle:
    pickle.dump(tokenizer, handle, protocol=pickle.HIGHEST_PROTOCOL)
```

    > Number of Unique Words: 111001

Machine learning models do not understand text, so we need to convert both X and y into numerical form. We use the fitted tokenizer to convert `X_train` and `X_val` into sequences of integers. The sequences are then padded so every sequences have the same length. Notice that we use `X_train.shape[1]` as the maximum length for `X_val` to ensure that the length of sequences in X_train and X_val are the same (required by the model). We then perform one-hot encoding on the target labels (`y_train` and `y_val`).


```python
X_train = pad_sequences(tokenizer.texts_to_sequences(X_train))
input_len = X_train.shape[1]
X_val = pad_sequences(tokenizer.texts_to_sequences(X_val), maxlen=input_len)
y_train = y_train.replace({"Negative": 0, "Positive": 1})
y_val = y_val.replace({"Negative": 0, "Positive": 1})
```

### Word Embedding and Model Building

We will use a pre-trained GloVe word embedding to create the embedding matrix, which will serve as the weights for the embedding layer. The embedding layer will convert the sequences of integers into sequences of vectors, which will be used as the input to the LSTM layer. To ensure that the pre-trained weights are not modified during training, we set trainable to False for the embedding layer. We keep the pre-trained weights fixed because the GloVe embedding has already been trained on a large corpus of tweets and we want to utilize that pre-trained knowledge.


```python
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Embedding, LSTM, SpatialDropout1D
from tensorflow.keras.callbacks import EarlyStopping, ModelCheckpoint
from tensorflow.keras.optimizers import Adam

# download the pre-trained GloVe embedding (27B tokens based on 2B tweets)
file_url = "https://nlp.stanford.edu/data/glove.twitter.27B.zip"    
!wget $file_url
!unzip /content/glove.twitter.27B.zip

# load the pre-trained GloVe embedding with 200 dimensions into a dictionary
embeddings_index = {}
with open("/content/glove.twitter.27B.200d.txt", "r") as file:
    for line in file:
        values = line.split()
        word = values[0]
        coefs = np.asarray(values[1:], dtype='float32')
        embeddings_index[word] = coefs

# create our embedding matrix
embed_dim = 200
embedding_matrix = np.zeros((vocab_size, embed_dim))
for word, index in tokenizer.word_index.items():
    if word in embeddings_index:
        embedding_matrix[index] = embeddings_index[word]

# create the embedding layer
embedding_layer = Embedding(input_dim=vocab_size, 
                            output_dim=embed_dim, 
                            weights=[embedding_matrix], 
                            input_length=input_len, 
                            trainable=False)
```

Now we will create our LSTM model. The model consists of the following layers:
1) Embedding layer (converts sequences of integers into sequences of vectors)
2) SpatialDropout1D layer (prevents overfitting by randomly deactivating neurons)
3) LSTM layer (extracts features from the sequences of vectors while considering their order)
4) Dense layer with one neuron and sigmoid activation function (outputs the probability of the tweet being positive)

The model is then compiled with the Adam optimizer and binary crossentropy loss function.

Notes: The sentences "I am *not happy* because today is *raining*" and "I am *happy* becuase today is *not raining*" would be treated the same by a model that does not consider the order of the words (e.g. Naive Bayes algorithm using a unigram bag-of-words model). However, LSTM model can recognize the difference between the two sentences.


```python
model = Sequential()
model.add(embedding_layer)
model.add(SpatialDropout1D(0.4))
model.add(LSTM(150, dropout=0.25, recurrent_dropout=0.25))
model.add(Dense(1, activation="sigmoid"))
model.compile(loss="binary_crossentropy", optimizer="adam", metrics=["accuracy"])
print(model.summary())
```

![model summary table](/_images/tsa_model_summary.png)

Now we are ready to train our model. We set two callbacks to monitor the training process: `EarlyStopping` and `ModelCheckpoint`. The `EarlyStopping` callback will reduce overfitting by stopping the training process if the validation loss does not decrease after 3 epochs. The `ModelCheckpoint` callback will save the model whenever a new best model is found (based on the validation loss).


```python
es_callback = EarlyStopping(monitor="val_loss", patience=3)
mc_callback = ModelCheckpoint(
    filepath="/content/drive/MyDrive/static/lstm_model-{epoch:02d}.h5",
    monitor="val_loss",
    mode="min",
    save_best_only=True,
)
history = model.fit(
    X_train,
    y_train,
    epochs=10,
    validation_data=(X_val, y_val),
    callbacks=[es_callback, mc_callback],
    batch_size=32,
)
```

After 10 epochs, the model stops training because we only set the epochs to 10. The accuracy and loss of the model on the training set and validation set are shown below.  The validation accuracy and loss kind of level off after 7 or 8 epochs.  

Notice that the validation accuracy is always higher than the training accuracy, while the validation loss is always lower than the training loss. This is common when using Dropout because regularization mechanism such as Dropout is not applied when evaluating the validation set.

![train and validation evaluation plot](/_images/tsa_eval_plot.png)

### Evaluation on Test Set

We will choose the model with the lowest validation loss as the final model (model on epoch 8), then evaluate its performance on the test set. Remember that we need to convert the cleaned tweets in the test set into sequences of integers and pad them before feeding them to the model. The output of the model is the probability of the tweet having a positive sentiment, so we will convert the probability into a label by setting the probability threshold to 0.50.


```python
from tensorflow.keras.models import load_model
from sklearn.metrics import accuracy_score, confusion_matrix

model = load_model("/content/drive/MyDrive/static/lstm_model-08.h5")
sequences = pad_sequences(
    tokenizer.texts_to_sequences(test_df["cleaned_tweet"]), maxlen=input_len
)
test_df["score"] = model.predict(sequences)
test_df["pred_sentiment"] = test_df["score"].apply(
    lambda x: "Positive" if x >= 0.50 else "Negative"
)
```

The accuracy on test set (78.6%) is similar to the accuracy of the validation set (79.0%), which means the model can generalize well to unseen data.


```python
accuracy_score(test_df["sentiment"], test_df["pred_sentiment"])
```

    > 0.7855224793923905

The true negative rate is higher than the true positive rate, which means that the model is a bit biased towards predicting negative sentiment.


```python
tn, fp, fn, tp = confusion_matrix(
    test_df["sentiment"], test_df["pred_sentiment"]
).ravel()
tnr = tn / (tn + fp)
tpr = tp / (tp + fn)
print("True Negative Rate: {:.3f}".format(tnr))
print("True Positive Rate: {:.3f}".format(tpr))
```

    > True Negative Rate: 0.812
    > True Positive Rate: 0.758

We have performed text preprocessing and build a LSTM model that can predict the sentiment of tweets. In the next post, we will create a Streamlit app that can predict the sentiment of tweets and show some visualizations based on a given search term. Hope you enjoyed the post!

### Relevant Links
* Project Github: <u>[github.com/tmtsmrsl/TwitterSentimentAnalyzer](https://github.com/tmtsmrsl/TwitterSentimentAnalyzer)</u>  
* Streamlit App: <u>[twitter-sentiment.streamlit.app/](https://twitter-sentiment.streamlit.app/)</u>
