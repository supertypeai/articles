---
title: Serving PyTorch Models Using TorchServe
post_excerpt: How to use TorchServe to serve your PyTorch model (detailed TorchServe tutorial)
taxonomy:
    category:
        - knowledge
        - notes
---



# Serving PyTorch Models Using TorchServe

Model serving has always been a crucial process in MLOps as it decides whether an AI product will be accessible to the user. Upon developing a model that can perform a certain task, the next step is to serve the model so that it is accessible through an API, hence enabling applications to incorporate AI into the system. This process also includes model monitoring and management, which gives the ability to ensure that the model can function properly and scale the model on demand.

Various tools have been built as a solution to serve models. One of the most popular ones is **TorchServe**, which allows users to easily serve their PyTorch models within few simple steps:

1. Develop and export PyTorch model
2. Create a model handler and other additional files for the model
3. Generate model archive
4. Serve the model using TorchServe
5. Monitor and manage the model

Don't worry if some of the terms does not make any sense to you yet. This article will explain every step in detail, including all the additional files you need to personally create based on your pipeline. To better demonstrate the process, this article will be using a pre-trained model designed for sentiment analysis task obtained from [Hugging Face](https://huggingface.co/finiteautomata/bertweet-base-sentiment-analysis). This article will cover sufficient materials for deploying your PyTorch model, but will not dive into the advanced features of TorchServe.

When I was learning about TorchServe, I was troubled because lots of tutorials and documentations out there didn't provide much information about the additional files for supporting the models. They did mention about creating your own model handler, label mapper, and any other stuffs; but they did not explain how exactly could you create it on your own. Hence, I wrote this article, hoping that you wouldn't encounter any of the issue I had back then.

## Installation & Setup

First of all, TorchServe is implemented in Java, so you need to have Java 11 installed on your local machine. You can install JDK (Java Development Kit) 11 through the [Oracle](https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html) website. After installing it, verify that Java is properly installed in your local machine by running the following code in your terminal

```{bash}
java -version
```

If JDK 11 is properly installed in your local machine, you should get an output similar to this in your terminal

```
openjdk version "11.0.15" 2022-04-19
OpenJDK Runtime Environment (build 11.0.15+10-Ubuntu-0ubuntu0.22.04.1)
OpenJDK 64-Bit Server VM (build 11.0.15+10-Ubuntu-0ubuntu0.22.04.1, mixed mode, sharing)
```

You also need to install the dependencies in your virtual environment. The first dependency you need is [PyTorch](https://pytorch.org/get-started/locally/). Navigate to that link and choose the installation command based on your OS, preferred package manager, and compute platform (whether you want to use GPU/CPU). Then you need to install **TorchServe** and **Torch Model archiver** using the following command.

```(bash)
pip install torchserve torch-model-archiver torch-workflow-archiver
```

If you are creating your own model from scratch, you're already good to go. For this article, since I'm using a pre-trained model from Hugging Face, I need to install `transformers`. Furthermore, since I'm planning to utilize my GPU, I need to also install the `nvgpu` package.

```{bash}
pip install transformers nvgpu
```

## Exporting Model to Your Local Machine

As the content of this article is focused on *how to serve PyTorch model*, I expect that you already have a trained model on your end. For this article, as I previously mentioned, I will be using the [BERTweet-Based Sentiment Analysis](https://huggingface.co/finiteautomata/bertweet-base-sentiment-analysis) model from Hugging Face. In order for this model to work, I need the tokenizer and the model on my local machine. So, load them using the following lines of code in a notebook

```{python}
from transformers import AutoTokenizer, AutoModelForSequenceClassification

# get tokenizer
tokenizer = AutoTokenizer.from_pretrained("finiteautomata/bertweet-base-sentiment-analysis")

# get model
model = AutoModelForSequenceClassification.from_pretrained("finiteautomata/bertweet-base-sentiment-analysis")
```

After loading the model and tokenizer, we can save them into our local machine using the following lines of code

> *if you are not using transformers, you might need to use `torch.save(model.state_dict(), PATH)` instead*

```{python}
# save tokenizer
tokenizer.save_pretrained('./my_tokenizer')

# save model
model.save_pretrained('./my_model')
```

This will save the tokenizer into a folder named **my_tokenizer** and the model into a folder named **my_model** in the same directory as your notebook. Currently, your working directory should have the following structure

```
workdir
├── my_model
│   ├── pytorch_model.bin
│   └── config.json
│
└── my_tokenizer
    ├── added_tokens.json
    ├── bpe.codes
    ├── special_tokens_map.json
    ├── tokenizer_config.json
    └── vocab.txt
```

Knowing the function of each file is unnecessary. What you need to know is that you need all of the files to be able to load the saved model and tokenizer.

## Creating Your JSON Request & Label Mapper

Creating a JSON file that will be attached upon sending a request to the model endpoint before actually deploying it might seem odd. However, you will need to create a script that tells TorchServe how to process the data sent to the endpoint later on. If you don't know the shape of that data, then it will be really hard to build the pipeline. Hence, you need to create your sample JSON file beforehand. The JSON file sent to a model endpoint might look like this

```
# sample_input.json

{
    "input" : ["texts 1", "texts 2"]
}
```

For this article, I'm going to stick with the previous example, but you are free to define your own input. An additional file you might want to create is the `index_to_name.json`, which contains a mapper for your model output. When you train your own model, the labels are usually encoded. You can use this file to convert the encoded labels back to human-readable string. The following is the label mapper I created for the model.

```
# index_to_name.json

{
    "0" : "Negative",
    "1" : "Neutral",
    "2" : "Positive"
}
```

## Initiating Model Handler

Moving on to the next step, we need to create a model handler. Model handler is basically a pipeline for transforming the input data that is sent via HTTP request into the desired output. It is the one who is responsible to generate a prediction using your model. TorchServe has provided many default handlers which you can directly use for various machine learning tasks. However, there is a high possibility that you have your own way of processing the input data to the desired output, so it is very likely that you need to create your own handler. A good practice for creating your own handler is by extending the `base handler` provided by TorchServe. Create a file named `handler.py` and use the following lines of code to initiate your handler.

```{python}
# handler.py

import torch
import logging
import transformers
import os
import json

from ts.torch_handler.base_handler import BaseHandler
from transformers import AutoModelForSequenceClassification, AutoTokenizer

logger = logging.getLogger(__name__)
logger.info("Transformers version %s", transformers.__version__)

class ModelHandler(BaseHandler):
    pass
```

In the code above, we are creating a new class named `ModelHandler` which inherits from the `BaseHandler` class. The `BaseHandler` class has implemented the necessary functions for generating predictions. Usually, when you are creating your own handler, you only need to modify the following functions:

- `initialize`: initialization function for loading the model and dependencies such as tokenizer
- `preprocess`: preprocessing function for transforming the request input
- `inference`: function for generating predictions
- `postprocess`: postprocessing function to transform the model output

Note that we also created a logger before creating the `ModelHandler` class. This logger can be configured to print any information into the terminal once the model is served. Now, let's create the initialization function for our handler.

```{python}
class ModelHandler(BaseHandler):

    def initialize(self, context):
        """Initialize function loads the model and the tokenizer

        Args:
            context (context): It is a JSON Object containing information
            pertaining to the model artifacts parameters.
        """

        properties = context.system_properties
        self.manifest = context.manifest

        logger.info(f'Properties: {properties}')
        logger.info(f'Manifest: {self.manifest}')
```

The `initialize` function takes a `context`, a JSON object containing information pertaining to the model artifacts parameters. It has 2 important attributes: `system_properties` and `manifest`. For the time being, we will serve the model using this handler so that we can see what exactly the `context.system_properties` and `context.manifest` contain through the logs.

## Serving the Model in Your Localhost

Once you have all the necessary files, serving the model in your localhost is very straightforward. Basically you just need to package all your model artifacts and complementary assets into a single model archive (`.mar`) file. This file can then be used to register the model into TorchServe. You can also share this file so that anyone can directly serve your model without even knowing what you have created. At this point, you should have the following files and directories in your working dir

```
workdir
├── my_model
│   └── ...
├── my_tokenizer
│   └── ...
├── index_to_name.json
└── handler.py
```

### Generating Model Archive

Create a new folder named **model_store**. Then, from your terminal, navigate to your working dir and run the following command

```{bash}
torch-model-archiver \
--model-name BERTweetSentimentAnalysis \
--version 1.0 \
--model-file my_model/pytorch_model.bin \
--handler handler.py \
--extra-files "my_model/config.json,my_tokenizer/added_tokens.json,my_tokenizer/bpe.codes,my_tokenizer/special_tokens_map.json,my_tokenizer/tokenizer_config.json,my_tokenizer/vocab.txt,index_to_name.json" \
--export-path model_store
```

The command will use `torch-model-archiver` to create a file named `BERTweetSentimentAnalysis.mar` in the **model_store** directory. It requires us to pass on some arguments, which you can see in detail by running the following command.

```{bash}
torch-model-archiver -h
```

### Registering & Serving the Model

Upon generating the model archive, we need to register our model into TorchServe. Run the following command to register the model and serve it using TorchServe

```{bash}
torchserve --start --model-store model_store --models my_model=BERTweetSentimentAnalysis.mar --ncs
```

This will start the model-server in your localhost. The arguments `--model-store` is used to specify the location from which the models can be loaded. `--models MODEL_NAME=<PATH_TO_MAR_FILE>` is used to register the models, you can define a model name of your own choice. Lastly, `--ncs` prevents the server from storing config snapshot files. Upon running the command, you should see some logs like this in your terminal

![TorchServe start logs](/_images/torchserve_start_logs.png)

The important things to see here is the **Inference address**, **Management address**, and **Metrics address**. These addresses show the URLs that you access to generate predictions, manage models, and see the model metrics, respectively. Take the inference address as ane example, you can send a POST request containing your JSON file to `localhost:8080/predictions/my_model` using any app such as **Postman** to get a prediction. To stop the model-server, simply use the following command

```{bash}
torchserve --stop
```

## Model Handler Revisited

Previously, we have only defined a little part of the initialization function for our model handler. The reason why I stopped there and showed you how to serve the model beforehand was because for building and debugging your model handler, you might need to iteratively generate the model archive and serve it. If you look closely in the logs that were generated when serving the model earlier on, there are these 2 important lines present.

```{bash}
[INFO ] MODEL_LOG - Properties: {'model_dir': '/tmp/models/806bc5b85e3c40198a766b4371db964f', 'gpu_id': 0, 'batch_size': 1, 'server_name': 'MMS', 'server_version': '0.6.0', 'limit_max_image_pixels': True}

[INFO ] MODEL_LOG - Manifest: {'createdOn': '14/07/2022 21:49:55', 'runtime': 'python', 'model': {'modelName': 'BERTweetSentimentAnalysis', 'handler': 'handler.py', 'modelFile': 'pytorch_model.bin', 'modelVersion': '1.0'}, 'archiverVersion': '0.6.0'}
```

This is the logging we set on our model handler, with the purpose of inspecting `context.system_properties` and `context.manifest`. Apparently, `context.system_properties` can be used to set our compute platform. Additionally, it provides an information about the name of our model directory. This model directory contains all the files we passed on as arguments when generating the model archive. On the other hand, `context.manifest` provides the details of our model.

### Initialization Function

Upon inspecting the `context` object, building the initialization function is just a matter of playing with dictionary in Python. In this function, we need to:

- utilize the GPU if available
- load the model, tokenizer, and mapping file
- model, tokenizer, and mapping file can be loaded from `model_dir` specified in `context.properties`


```{python}
def initialize(self, context):
    """Initialize function loads the model and the tokenizer

    Args:
        context (context): It is a JSON Object containing information
        pertaining to the model artifacts parameters.

    Raises:
        RuntimeError: Raises the Runtime error when the model or
        tokenizer is missing

    """
    
    properties = context.system_properties
    self.manifest = context.manifest
    model_dir = properties.get("model_dir")

    # use GPU if available
    self.device = torch.device(
        "cuda:" + str(properties.get("gpu_id"))
        if torch.cuda.is_available() and properties.get("gpu_id") is not None
        else "cpu"
    )
    logger.info(f'Using device {self.device}')
    
    # load the model
    model_file = self.manifest['model']['modelFile']
    model_path = os.path.join(model_dir, model_file)

    if os.path.isfile(model_path):
        self.model = AutoModelForSequenceClassification.from_pretrained(model_dir)
        self.model.to(self.device)
        self.model.eval()
        logger.info(f'Successfully loaded model from {model_file}')
    else:
        raise RuntimeError('Missing the model file')

    # load tokenizer
    self.tokenizer = AutoTokenizer.from_pretrained(model_dir)
    if self.tokenizer is not None:
        logger.info('Successfully loaded tokenizer')
    else:
        raise RuntimeError('Missing tokenizer')

    # load mapping file
    mapping_file_path = os.path.join(model_dir, 'index_to_name.json')
    if os.path.isfile(mapping_file_path):
        with open(mapping_file_path) as f:
            self.mapping = json.load(f)
        logger.info('Successfully loaded mapping file')
    else:
        logger.warning('Mapping file is not detected')

    self.initialized = True
```

Feel free to modify the code if you have a different way of loading any asset. I wrote some comments in the code snippet so that it is easier for you to understand what each part does.

### Preprocessing Function

The next function we need to create is the `preprocess` function. This function takes a request, unpacks the data, and preprocess it using the tokenizer so that it can be passed into the model. Recall that we have defined our data as follows

```
# sample_input.json

{
    "input" : ["texts 1", "texts 2"]
}
```

You can see what this request object will look like when passed into the `preprocess` function by implementing a logging in your model handler and sending a POST request containing your **sample_input.json** to `localhost:8080/predictions/<MODEL_NAME>`. It will return an `Error 503 Prediction failed` response, but the `request` object should be printed in the logs.

```{bash}
[INFO ] Request object: [{'body': {'input': ['texts 1', 'texts 2']}}]
```

The request object is a list containing a dictionary `{'body': sample_input.json}`. Sometimes, the dictionary key might be `'data'` instead of `'body'`. Now we can create the preprocessing function that unpacks this data and tokenize it.

```{python}
def preprocess(self, requests):
    """Tokenize the input text using the suitable tokenizer and convert 
    it to tensor
    
    Args:
        requests: A list containing a dictionary, might be in the form
        of [{'body': json_file}] or [{'data': json_file}]
    """
    
    # unpack the data
    data = requests[0].get('body')
    if data is None:
        data = requests[0].get('data')

    texts = data.get('input')
    logger.info(f'Received {len(texts)} texts. Begin tokenizing')

    # tokenize the texts
    tokenized_data = self.tokenizer(texts,
                                    padding=True,
                                    return_tensors='pt')
    
    logger.info('Tokenization process completed')

    return tokenized_data
```

### Inference Function

The `inference` function is quite straightforward, it takes `inputs`, which is a tensor containing the preprocessed data, and pass it into the model.

```{python}
def inference(self, inputs):
    """Predict class using the model
        
    Args:
        inputs: tensor of tokenized data
    """

    outputs = self.model(**inputs.to(self.device))
    probabilities = torch.nn.functional.softmax(outputs.logits, dim=-1)
    predictions = torch.argmax(probabilities, axis=1)
    predictions = predictions.tolist()
    logger.info('Predictions successfully created.')

    return predictions
```

Remember that when you generate an output from the model, it returns the non-normalized probability of each class, i.e. [logits](https://developers.google.com/machine-learning/glossary/#logits). You need to pass these logits into a **softmax** function to get the vector of normalized probabilities.

### Postprocessing Function

The last function we need to build is the `postprocess` function. It is used to convert the model output to a list of string labels. It takes `outputs`, a list of integer label produced by the model, and returns a list of string labels.

```{python}
def postprocess(self, outputs: list):
    """
    Convert the output to the string label provided in the label mapper (index_to_name.json)

    Args:
        outputs (list): The integer label produced by the model

    Returns:
        List: The post process function returns a list of the predicted output.
    """
    
    predictions = [self.mapping[str(label)] for label in outputs]
    logger.info(f'PREDICTED LABELS: {predictions}')

    return [predictions]
```

## Inference & Management

At this point, you should already be able to serve the model in your localhost by rebuilding the model archive file and serving it in TorchServe. The inference address that was previously mentioned offers some APIs which are useful for checking the health status of a running TorchServe server, as well as generating predictions. While the management address offers some APIs that can be used for registering a new model, deleting old model, scaling workers, etc. The list of available APIs can be seen on the [official TorchServe GitHub page](https://github.com/pytorch/serve/blob/master/docs/rest_api.md).

To check the health status of a running TorchServe server, you can send a GET request to `localhost:8080/ping` using any method you prefer (Postman, curl command line, etc). It should return the status of the server. If it returns anything than `Healthy`, you should check the logs to see if there's any error mesagge.

![health status of a server](/_images/health_status.png)

To get a prediction, you can send a POST request to `localhost:8080/predictions/{model_name}` while attaching a JSON file in the body. Check if the model can properly returns the desired output.

![prediction example](/_images/prediction_example.png)

## Containerizing TorchServe

At this point, your model should be running properly in your local machine. If you want to take a step further and deploy your model in the cloud, you need to make a container image for your application. Fortunately, TorchServe has provided us a base image for creating our custom container image in [Docker Hub](https://hub.docker.com/r/pytorch/torchserve). The base image has already been properly configured — we just need to provide our model archive (`.mar`) file and install any additional packages we are using. Remember that we store our model archive file in the **model_store** directory. In your working dir, create a file `Dockerfile` and paste the following lines of code

```
# base image
# use FROM pytorch/torchserve:latest-gpu for GPU support
FROM pytorch/torchserve:latest

# install dependencies
RUN pip3 install transformers

# copy model archive
COPY model_store /home/model-server/model-store/

# run Torchserve upon running the container
CMD ["torchserve", \
     "--start", \
     "--models my_model=BERTweetSentimentAnalysis.mar", \
     "--ts-config /home/model-server/config.properties"]
```

We are basically sending a list of instructions to the Docker to use TorchServe's base image and install `transformers` package within it. Then copy every file inside the **model_store** directory on our local machine to the **model-store** directory inside the container. Upon running the container, run the command to start TorchServe. Wait a few minutes until Docker has already finished building your image, then try to run the container using the following command

```{bash}
docker run -dp 8080:8080 \
    -p 8081:8081 \
    --name torchserve \
    torch
```

Notice that the command only exposes port 8080 and 8081 of the container (you can change the command to expose other ports). Upon running the command, you should be able to send a post request to `localhost:8080/predictions/my_model` and get a prediction response. If it behaves as expected and you are able to generate predictions, then you can already push this container image to the cloud and deploy your PyTorch model.