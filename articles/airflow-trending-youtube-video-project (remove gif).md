---
title: Using Airflow and YouTube API to Automatically Retrieve Trending Videos
post_excerpt: This article explains how to set up a workflow using Apache Airflow within a Docker environment and YouTube Data API v3 to automatically retrieve trending videos from YouTube on a daily basis. The retrieved data will be stored in BigQuery and visualized using Looker Studio.
taxonomy:
    category:
        - knowledge
        - notes
---

Nowadays, data is very important as they help us make informed decisions and gain insights into a wide range of topics. With the increasing amount of data generated every day, it has become crucial to automate data retrieval and processing. In this article, we will walk through the process of setting up a workflow using Apache Airflow within a Docker environment and YouTube Data API v3 to automatically retrieve daily trending videos from YouTube. The retrieved data will be stored in BigQuery and visualized using Looker Studio.

Apache Airflow is an open-source platform for automating tasks in a data pipeline. It allows data engineers to author, schedule, and monitor workflows. If you wish to learn more about the basics of Airflow, you can check out an article written by my colleague, Stane, <u>[here](https://supertype.ai/notes/intro-to-apache-airflow/)</u>. The article also explains how to install and run Airflow on Docker, which is the approach we will be using for our workflow.

The YouTube Data API v3 is a powerful tool that allows us to fetch data from YouTube. With this API, we can retrieve information about channels, videos, playlists, and more. In this article, we will focus on using the API to fetch data about trending videos in Indonesia.

BigQuery is a fully-managed, cloud-native data warehouse that allows us to store and analyze large amounts of data. BigQuery offers a free tier with a usage limits of 1 TB of querying per month and 10 GB of storage per month, which is adequate for our purpose.

Finally, Looker Studio is a cloud-based business intelligence and analytics platform that allows us to create and share data visualizations. Looker Studio makes it easy to create interactive dashboards and reports, and it integrates seamlessly with BigQuery.

Now let's start building our workflow!

### Setting up the environment
If you are using Windows, I recommend you to use Windows Subsystem for Linux (WSL) to avoid any compatibility issues. You can follow the steps in this <u>[video](https://www.youtube.com/watch?v=JTVww711fIY)</u> to install WSL. For this project, I will be using Ubuntu 22.04.2 LTS.

First we will make a new project directory named `airflow_youtube_project`. Then we will follow the steps in the <u>[article](https://supertype.ai/notes/intro-to-apache-airflow/)</u> mentioned earlier to install and run Airflow on Docker (note: some modifications have been made to the original article to make it more suitable for our project.).

> To run Airflow in Docker, you will need to have Docker and Docker Compose installed beforehand. To do so, simply download and install Docker Desktop in your local machine depending on your OS via the <u>[official Docker website](https://docs.docker.com/get-docker/)</u>. Once you have installed Docker Desktop, ensure that you have Docker Compose with v1.27.0 or newer installed by running this command
>
> ```{bash}
> $ docker compose version
> Docker Compose version v2.7.0
> ```
> 
> Afterwards, you need to download `docker-compose.yaml` by executing this command in the project directory (`airflow_youtube_project` for our case)
> 
> ```{bash}
> $ curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.6.0/docker-compose.yaml'
> ```
> 
> If you open the `docker.compose.yaml` file, you will see that it defines many services and composes them in a proper way. Now we need to initialize the environment. If you are running Linux OS, you will need to execute these following commands beforehand
> 
> ```{bash}
> $ mkdir ./dags ./logs ./plugins
> $ echo -e "AIRFLOW_UID=$(id -u)" > .env
> ```
> 
> Now you can run database migrations and create the first user account by using this command
> 
> ```{bash}
> $ docker compose up airflow-init
> ```
> 
> It will create an account with username `airflow` and password `airflow`. Run Airflow by using `docker compose up`, then navigate to `localhost:8080` and login using the credentials.

Next we will create a new virtual environment and install the required packages in the virtual environment.
```{bash}
$ python3 -m venv airflow_youtube_env
$ source airflow_youtube_env/bin/activate
$ python3 -m pip install apache-airflow google-api-python-client python-dotenv isodate google-cloud-bigquery
```

Now we need to create a new project in Google Cloud Platform. First go to the <u>[Google Cloud Console](https://console.cloud.google.com/)</u>, then click on the project dropdown menu and select `New Project`. We will name the project as `airflow-youtube-project` and click `Create`. Wait a few moment until the project is sucessfully created. Note that the project ID may be different from the project name.

Once the project is created, select the project from the dropdown menu and go to the <u>[APIs & Services Dashboard](https://console.cloud.google.com/apis/dashboard)</u>. Click on `Enable APIs and Services`, then search for `Youtube Data API v3` and enable them. By default, the BigQuery API should already be enabled (in case it is not, please enable it as well). After enabling the `Youtube Data API v3`, click on the `Credentials` tab. Click on the `Create Credentials` and select `API key`. Copy the API key and save it in a `.env` file in the `airflow_youtube_project/dags` directory. We will use this API key later to fetch data from YouTube.

The content of the `.env` file should be similar to this.
```
YOUTUBE_API_KEY=AIzaSyBiHhiP0_NeL1js7IC3w1ZPWaAt_gCdrUY
```

Next we will create a new service account and download the service account key. Go to the <u>[Service Accounts](https://console.cloud.google.com/iam-admin/serviceaccounts)</u> page and click on `Create Service Account`. Give your service account a name and click `Create and Continue`. Select `BigQuery Admin` as the role, then click `Done`. Click on `Action` and select `Manage keys`. Click `Add key` and select `Create new key`. Select `JSON` as the key type and click `Create`. Save the downloaded JSON file in the `airflow_youtube_project/dags` directory. Let's rename the json file to `service_account_key.json` for simplicity. We will use this JSON file to authenticate our BigQuery client.

Note that we are saving the `.env` and JSON files in the `airflow_youtube_project/dags` directory. This is done so the Docker containers are able access those files as well. In the `docker-compose.yaml` file, the volume sharing is specified using the volumes parameter, which maps the local directories to the directories inside the container. E.g. the files inside the `airflow_youtube_project/dags` directory will be shared with the `/opt/airflow/dags` directory inside the container.
```{yaml}
volumes:
- ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags
- ${AIRFLOW_PROJ_DIR:-.}/logs:/opt/airflow/logs
- ${AIRFLOW_PROJ_DIR:-.}/plugins:/opt/airflow/plugins
```

Now let's create a new BigQuery dataset using the python code below. This will also let us test if the service account key is working properly.
```{python}
from google.cloud import bigquery
from google.oauth2 import service_account

# Set the path to your service account key file
key_path = 'dags/service_account_key.json'

# Set the credentials using the service account key
credentials = service_account.Credentials.from_service_account_file(
    key_path,
    scopes=["https://www.googleapis.com/auth/cloud-platform"],
)

# Instantiate the BigQuery client with the credentials
client = bigquery.Client(credentials=credentials)

# Set the ID of the dataset to create
dataset_id = 'youtube'

# Construct a full Dataset object to be send to the API
dataset = bigquery.Dataset(f"{client.project}.{dataset_id}")

# Send the API request to create the dataset
dataset = client.create_dataset(dataset)
```

To confirm that the dataset has been created, we can run the following python code.
```{python}
datasets = client.list_datasets()
for dataset in datasets:
    print(dataset.dataset_id)
```
You should see `youtube` in the output if the dataset has been created successfully.

By now your project structure should look similar to this.
```
airflow_youtube_project
├── airflow_youtube_env
├── dags
│   └── .env (containing YOUTUBE_API_KEY)
│   └── service_account_key.json
├── logs
├── plugins
├── .env (containing AIRFLOW_UID)
└── docker-compose.yaml
```

### Fetching and Processing Data from YouTube Data API
Earlier we have obtained the API key for YouTube Data API v3, which enables us to fetch data from YouTube. Below is a sample code to fetch the first 50 results of the trending YouTube videos in Indonesia. I encourage you to test the code below to see the response returned by the API. If you wish to read more about the parameters of the API request, you can refer to this <u>[documentation page](https://developers.google.com/youtube/v3/docs/videos/list)</u>. And if you wish to read more about the details of the items returned in the response, you can refer to this <u>[documentation page](https://developers.google.com/youtube/v3/docs/videos#resource)</u>. 
```{python}
from dotenv import load_dotenv
import os
from googleapiclient.discovery import build

# Load API key from .env file
load_dotenv("dags/.env")
api_key = os.environ.get("YOUTUBE_API_KEY")

# Create YouTube API client
youtube = build("youtube", "v3", developerKey=api_key)

# Fetch videos until max_results is reached or there are no more results

# Make API request for videos
request = youtube.videos().list(
    part="snippet,contentDetails,statistics",
    chart="mostPopular",
    regionCode="id",
    maxResults=50,
)
response = request.execute()
```

Now we will make a function to fetch a list of trending videos from the YouTube Data API. The function takes the region code, maximum number of results, and the path to the target file as the arguments. The function will extract the relevant information from the API response and save it in a list of dictionaries, which will then be written to a JSON file.
```{python}
from dotenv import load_dotenv
import os
from googleapiclient.discovery import build
import json
from datetime import datetime

def fetch_trending_videos(region_code: str, max_results: int, target_file_path: str):
    """Fetches trending videos from YouTube for a specific region.

    Args:
        region_code: A string representing the ISO 3166-1 alpha-2 country code for the desired region.
        max_results: An integer representing the maximum number of results to fetch.
        target_file_path: A string representing the path to the file to be written.
    """

    # Load API key from .env file
    load_dotenv("dags/.env")
    api_key = os.environ.get("YOUTUBE_API_KEY")

    # Create YouTube API client
    youtube = build("youtube", "v3", developerKey=api_key)

    # Fetch videos until max_results is reached or there are no more results
    videos_list = []
    next_page_token = ""
    while len(videos_list) < max_results and next_page_token is not None:
        # Make API request for videos
        request = youtube.videos().list(
            part="snippet,contentDetails,statistics",
            chart="mostPopular",
            regionCode=region_code,
            maxResults=50,
            pageToken=next_page_token,
        )
        response = request.execute()

        # Extract videos from response
        videos = response.get("items", [])

        # Update next_page_token for the next API request
        next_page_token = response.get("nextPageToken", None)
        
        # Extract relevant video details and append to list
        infos = {'snippet':['title', 'publishedAt', 'channelId', 'channelTitle',
                            'description', 'tags', 'thumbnails', 'categoryId', 'defaultAudioLanguage'],
                    'contentDetails':['duration', 'caption'],
                    'statistics':['viewCount', 'likeCount', 'commentCount']}
        now = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ')
        for video in videos:
            video_details = {
                'videoId': video["id"],
                'trendingAt': now
            }
            
            for k in infos.keys():
                for info in infos[k]:
                    # use try-except to handle missing info
                    try:
                        video_details[info] = video[k][info]
                    except KeyError:
                        video_details[info] = None
                        
            videos_list.append(video_details)

    # Write fetched videos data to a json file
    with open(target_file_path, "w") as f:
        json.dump(videos_list, f)
```

If we run the function below, we should see a JSON file named `tmp_file.json` in the current directory.
```{python}
fetch_trending_videos("ID", 200, "tmp_file.json")
```

Below is a snippet of `tmp_file.json` content.
```{json}
[
    {
        "videoId": "aVrzOIDB5uc",
        "trendingAt": "2023-05-15T02:47:17Z",
        "title": "TERHARU🔥🔥🔥! Perjuangan Dramatis Indonesia (3) VS (2) Vietnam | SEA GAMES 32 CAMBODIA",
        "publishedAt": "2023-05-13T12:12:24Z",
        "channelId": "UCeM5Nksgv9_FXTuZ8jkPJPg",
        "channelTitle": "RCTI - ENTERTAINMENT",
        "description": "SEMUANYA TERHARU SEKALI!!!melihat garuda muda berjuang dikala hanya bermain dengan 10 orang pemain, namun tak membuat mental mereka gentar. Ini pertandingan yang seru dengan diwarnai dengan berbagai konflik, peluang emas & gol yang indah dari Timnas Indonesia!\n\n#SeaGames #Seagames32Cambodia #Indonesia\n\nPerjuangan Atlet Atlet Indonesia dalam ajang Sea Games 2023 sudah dimulai✨! Pasukan Garuda akan memulai langkahnya di ajang Sea Games 32 Cambodia, Dukung perjuangan mereka untuk meraih kemenangan di Sea Games dan saksikan semua pertandingan Sea Games 32 Cambodia",
        "tags": ["RCTI"],
        "thumbnails": {
            "default": {
            "url": "https://i.ytimg.com/vi/aVrzOIDB5uc/default.jpg",
            "width": 120,
            "height": 90
            },
            "medium": {
            ...
            },
            "high": {
            ...
            },
            "standard": {
            ...
            },
            "maxres": {
            ...
            }
        },
        "categoryId": "17",
        "defaultAudioLanguage": "id",
        "duration": "PT10M57S",
        "caption": "false",
        "viewCount": "3712309",
        "likeCount": "47450",
        "commentCount": "6770"
    }, 
    ...
]
```

From the result above, we can see that some of the data need further processing. First we will focus on the `categoryId`, which is a numeric value. We will need to convert it to the corresponding category name. We can find the category mapping by performing a YouTube API request as follows. We will also save the category mapping to a JSON file for later use.
```{python}
from dotenv import load_dotenv
import os
from googleapiclient.discovery import build
import json

load_dotenv('dags/.env')
api_key = os.environ.get("YOUTUBE_API_KEY")

youtube = build("youtube", "v3", developerKey=api_key)
request = youtube.videoCategories().list(
    part="snippet",
    regionCode="ID"
)
response = request.execute()

categories = {item['id']:item['snippet']['title'] for item in response['items']}

# save categories to json file
with open('dags/categories.json', 'w') as f:
    json.dump(categories, f)
```

The content of `categories.json` should look like this.
```{json}
{"1": "Film & Animation", "2": "Autos & Vehicles", "10": "Music", "15": "Pets & Animals", "17": "Sports", "18": "Short Movies", "19": "Travel & Events", "20": "Gaming", "21": "Videoblogging", "22": "People & Blogs", "23": "Comedy", "24": "Entertainment", "25": "News & Politics", "26": "Howto & Style", "27": "Education", "28": "Science & Technology", "30": "Movies", "31": "Anime/Animation", "32": "Action/Adventure", "33": "Classics", "34": "Comedy", "35": "Documentary", "36": "Drama", "37": "Family", "38": "Foreign", "39": "Horror", "40": "Sci-Fi/Fantasy", "41": "Thriller", "42": "Shorts", "43": "Shows", "44": "Trailers"}
```

Next, we will make a function to process the json data obtained from `fetch_trending_videos` function. The function will do the following things:
- Convert the `duration` from ISO 8601 standard to seconds
- Convert the `tags` list to string
- Map the `categoryId` to the corresponding category name
- Parse the thumbnail URL from `thumbnails`
- Convert `viewCount`, `likeCount`, and `commentCount` to integer; and `caption` to boolean
- Save the processed data to a new json file
```{python}
import json
import isodate

def data_processing(source_file_path: str, target_file_path: str):
    """Processes the raw data fetched from YouTube.
    
    Args:
        source_file_path: A string representing the path to the file to be processed.
        target_file_path: A string representing the path to the file to be written.
    """
    # Load the fetched videos data from the json file
    with open(source_file_path, 'r') as f:
        videos_list = json.load(f)
    
    # Load the categories dictionary from the json file
    with open('dags/categories.json', 'r') as f:
        categories = json.load(f)
    
    # Process the fetched videos data
    for video in videos_list:
        # Convert ISO 8601 duration to seconds
        video['durationSec'] = int(isodate.parse_duration(video['duration']).total_seconds()) if video['duration'] is not None else None
        del video['duration']
        
        # Convert tags list to string
        video['tags'] = ', '.join(video['tags']) if video['tags'] is not None else None
        
        # Convert categoryId to category based on categories dictionary
        video['category'] = categories.get(video['categoryId'], None) if video['categoryId'] is not None else None
        del video['categoryId']

        # Parse the thumbnail url
        video['thumbnailUrl'] = video['thumbnails'].get('standard', {}).get('url', None) if video['thumbnails'] is not None else None
        del video['thumbnails']

        # Convert viewCount, likeCount, and commentCount to integer
        video['viewCount'] = int(video['viewCount']) if video['viewCount'] is not None else None
        video['likeCount'] = int(video['likeCount']) if video['likeCount'] is not None else None
        video['commentCount'] = int(video['commentCount']) if video['commentCount'] is not None else None
        
        # Convert caption to boolean
        video['caption'] = True if video['caption'] == 'true' else False if video['caption'] == 'false' else None
    
    # Save the processed videos data to a new file
    with open(target_file_path, "w") as f:
        json.dump(videos_list, f)
```

Now let's try to run the `data_processing` function to process the json file (`tmp_file.json`) we generated earlier. We will save the processed data to a new file called `tmp_file_processed.json`.
```{python}
data_processing('tmp_file.json', 'tmp_file_processed.json')
```

Below is the snippet of `tmp_file_processed.json` content. Notice the difference in the `tags`, `durationSec` (formerly `duration`), `category` (formerly `categoryId`) and `thumbnailUrl` (formerly `thumbnails`). The data type for `viewCount`, `likeCount`, `commentCount`, and `caption` have also been converted appropriately.
```{json}
[
    {
        "videoId": "aVrzOIDB5uc",
        "trendingAt": "2023-05-15T02:47:17Z",
        "title": "TERHARU🔥🔥🔥! Perjuangan Dramatis Indonesia (3) VS (2) Vietnam | SEA GAMES 32 CAMBODIA",
        "publishedAt": "2023-05-13T12:12:24Z",
        "channelId": "UCeM5Nksgv9_FXTuZ8jkPJPg",
        "channelTitle": "RCTI - ENTERTAINMENT",
        "description": "SEMUANYA TERHARU SEKALI!!!melihat garuda muda berjuang dikala hanya bermain dengan 10 orang pemain, namun tak membuat mental mereka gentar. Ini pertandingan yang seru dengan diwarnai dengan berbagai konflik, peluang emas & gol yang indah dari Timnas Indonesia!\n\n#SeaGames #Seagames32Cambodia #Indonesia\n\nPerjuangan Atlet Atlet Indonesia dalam ajang Sea Games 2023 sudah dimulai✨! Pasukan Garuda akan memulai langkahnya di ajang Sea Games 32 Cambodia, Dukung perjuangan mereka untuk meraih kemenangan di Sea Games dan saksikan semua pertandingan Sea Games 32 Cambodia",
        "tags": "RCTI",
        "defaultAudioLanguage": "id",
        "caption": false,
        "viewCount": 3712309,
        "likeCount": 47450,
        "commentCount": 6770,
        "durationSec": 657,
        "category": "Sports",
        "thumbnailUrl": "https://i.ytimg.com/vi/aVrzOIDB5uc/sddefault.jpg"
    },
    ...
]
```

By now your project structure should look similar to this.
```
airflow_youtube_project
├── airflow_youtube_env
├── dags
│   └── .env (containing YOUTUBE_API_KEY)
│   └── service_account_key.json
│   └── categories.json
├── logs
├── plugins
├── .env (containing AIRFLOW_UID)
├── docker-compose.yaml
├── tmp_file.json
└── tmp_file_processed.json
```

### Loading Data to BigQuery
Earlier we have created a BigQuery dataset called `youtube`. Now we will create two new tables called `trending_videos` and `trending_videos_test` in the `youtube` dataset. The `trending_videos` table will be used to store the processed data generated by the Airflow pipeline, while the `trending_videos_test` table will be used for testing purposes.
```{python}
from google.oauth2 import service_account
from google.cloud import bigquery

# Set the path to your service account key file
key_path = 'dags/service_account_key.json'

# Set the credentials using the service account key
credentials = service_account.Credentials.from_service_account_file(
    key_path,
    scopes=["https://www.googleapis.com/auth/cloud-platform"],
)

# Instantiate the BigQuery client with the credentials
client = bigquery.Client(credentials=credentials)

# Define the table schema
schema = [
    bigquery.SchemaField('videoId', 'STRING'),
    bigquery.SchemaField('trendingAt', 'TIMESTAMP'),
    bigquery.SchemaField('title', 'STRING'),
    bigquery.SchemaField('publishedAt', 'TIMESTAMP'),
    bigquery.SchemaField('channelId', 'STRING'),
    bigquery.SchemaField('channelTitle', 'STRING'),
    bigquery.SchemaField('description', 'STRING'),
    bigquery.SchemaField('tags', 'STRING'),
    bigquery.SchemaField('category', 'STRING'),
    bigquery.SchemaField('defaultAudioLanguage', 'STRING'),
    bigquery.SchemaField('durationSec', 'INTEGER'),
    bigquery.SchemaField('caption', 'BOOLEAN'),
    bigquery.SchemaField('viewCount', 'INTEGER'),
    bigquery.SchemaField('likeCount', 'INTEGER'),
    bigquery.SchemaField('commentCount', 'INTEGER'),
    bigquery.SchemaField('thumbnailUrl', 'STRING')
]

# Create the table references
dataset_ref = client.dataset('youtube')
table1_ref = dataset_ref.table('trending_videos')
table2_ref = dataset_ref.table('trending_videos_test')

# Define the table objects
table1 = bigquery.Table(table1_ref, schema=schema)
table2 = bigquery.Table(table2_ref, schema=schema)

# Create the tables in BigQuery
client.create_table(table1)
client.create_table(table2)
```

To confirm that the table has been created, we can run the following code to list all the tables in the `youtube` dataset.
```{python}
tables = client.list_tables('youtube')
for table in tables:
    print(table.table_id)
```
You should see `trending_videos` and `trending_videos_test` in the output if the table has been created successfully.

Once the table has been created, we will be able to load the data from our json file to the table. We will make a function called `load_to_bigquery` to do this. 
```{python}
from google.oauth2 import service_account
from google.cloud import bigquery
import json

def load_to_bigquery(source_file_path: str, table_name: str):
    """
    Loads the processed data to BigQuery.
    
    Args:
        source_file_path: A string representing the path to the file to be loaded.
        table_name: A string representing the name of the table to load the data to.
    """
    
    # Set the path to your service account key file
    key_path = 'dags/service_account_key.json'

    # Set the credentials using the service account key
    credentials = service_account.Credentials.from_service_account_file(
        key_path,
        scopes=["https://www.googleapis.com/auth/cloud-platform"],
    )

    # Instantiate the BigQuery client with the credentials
    client = bigquery.Client(credentials=credentials)
    
    # Refer to the table where the data will be loaded
    dataset_ref = client.dataset('youtube')
    table_ref = dataset_ref.table(table_name)
    table = client.get_table(table_ref)
    
    # Load the data from the json file to BigQuery
    with open(source_file_path, 'r') as f:
        json_data = json.load(f)
    job_config = bigquery.LoadJobConfig()
    job_config.source_format = bigquery.SourceFormat.NEWLINE_DELIMITED_JSON
    job = client.load_table_from_json(json_data, table, job_config = job_config)
    job.result() # Waits for the job to complete
    
    # Log the job results
    print(f"Loaded {job.output_rows} rows to {table.table_id}")
```

Now let's load the data to BigQuery by running the following code.
```{python}
load_to_bigquery('tmp_file_processed.json', 'trending_videos_test')
```

You should see something like `Loaded 200 rows to airflow-youtube-project-386703.youtube.trending_videos_test` in the output if everything is running properly. If you want to verify the loaded data through the BigQuery UI, you can go to the <u>(BigQuery Console)[https://console.cloud.google.com/bigquery]</u> and select the `youtube` dataset. You should see the `trending_videos_test` table in the list of tables. Click on the table, then click on the `Preview` tab to see the data.
![BigQuery table preview](/_images/ayp_bq_table_preview.png)

### Building the Airflow DAG
Now that we have created the functions to fetch, process and load the trending YouTube videos data to BigQuery, we can start building the Airflow DAG. To begin, we will create a new file named `trending_youtube_dag.py` within the `dags` folder. We will use the TaskFlow API to make the DAG, by adding `@dag` decorator before the DAG definition and `@task` decorators before the task definitions. This DAG will encompass a series of tasks (`fetch_trending_videos`, `data_processing` and `load_to_bigquery`), each corresponding to the functions we previously created. Note that we have changed all the paths to absolute paths (e.g. "/opt/airflow/dags/.env" instead of "dags/.env") to ensure that the file locations are explicitly specified and can be correctly recognized and accessed by the Docker container running the code. By establishing this DAG, we can effortlessly execute the pipeline on a defined schedule, ensuring the continuous retrieval and processing of daily trending YouTube videos data. Below is the entire code for the `trending_youtube_dag.py` file.
```{python}
from airflow.decorators import dag, task
from googleapiclient.discovery import build
from google.cloud import bigquery
from google.oauth2 import service_account
import json
from datetime import datetime, timedelta, timezone
import isodate
import os
from dotenv import load_dotenv

default_args = {
    'owner': 'tmtsmrsl',
    'retries': 3,
    'retry_delay': timedelta(minutes=5)
}

@dag(dag_id='trending_youtube_dag_v1',
    default_args=default_args,
    description='A pipeline to fetch trending YouTube videos',
    start_date=datetime(2023, 5, 17, tzinfo=timezone(timedelta(hours=7))),
    schedule_interval='0 10 * * *',
    catchup=False)
def trending_youtube_dag():
    @task()
    def fetch_trending_videos(region_code: str, max_results: int, target_file_path: str):
        """Fetches trending videos from YouTube for a specific region.

        Args:
            region_code: A string representing the ISO 3166-1 alpha-2 country code for the desired region.
            max_results: An integer representing the maximum number of results to fetch.
            target_file_path: A string representing the path to the file to be written.
        """
        
        # Load API key from .env file
        load_dotenv("/opt/airflow/dags/.env")
        api_key = os.environ.get("YOUTUBE_API_KEY")

        # Create YouTube API client
        youtube = build("youtube", "v3", developerKey=api_key)

        # Fetch videos until max_results is reached or there are no more results
        videos_list = []
        next_page_token = ""
        while len(videos_list) < max_results and next_page_token is not None:
            # Make API request for videos
            request = youtube.videos().list(
                part="snippet,contentDetails,statistics",
                chart="mostPopular",
                regionCode=region_code,
                maxResults=50,
                pageToken=next_page_token,
            )
            response = request.execute()

            # Extract videos from response
            videos = response.get("items", [])

            # Update next_page_token for the next API request
            next_page_token = response.get("nextPageToken", None)
            
            # Extract relevant video details and append to list
            infos = {'snippet':['title', 'publishedAt', 'channelId', 'channelTitle',
                                'description', 'tags', 'thumbnails', 'categoryId', 'defaultAudioLanguage'],
                        'contentDetails':['duration', 'caption'],
                        'statistics':['viewCount', 'likeCount', 'commentCount']}
            now = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ')
            for video in videos:
                video_details = {
                    'videoId': video["id"],
                    'trendingAt': now
                }
                
                for k in infos.keys():
                    for info in infos[k]:
                        # use try-except to handle missing info
                        try:
                            video_details[info] = video[k][info]
                        except KeyError:
                            video_details[info] = None
                            
                videos_list.append(video_details)

        # Write fetched videos data to a json file
        with open(target_file_path, "w") as f:
            json.dump(videos_list, f)
    
    @task()
    def data_processing(source_file_path: str, target_file_path: str):
        """Processes the raw data fetched from YouTube.
        
        Args:
            source_file_path: A string representing the path to the file to be processed.
            target_file_path: A string representing the path to the file to be written.
        """
        # Load the fetched videos data from the json file
        with open(source_file_path, 'r') as f:
            videos_list = json.load(f)
        
        # Load the categories dictionary from the json file
        with open('/opt/airflow/dags/categories.json', 'r') as f:
            categories = json.load(f)
        
        # Process the fetched videos data
        for video in videos_list:
            # Convert ISO 8601 duration to seconds
            video['durationSec'] = int(isodate.parse_duration(video['duration']).total_seconds()) if video['duration'] is not None else None
            del video['duration']
            
            # Convert tags list to string
            video['tags'] = ', '.join(video['tags']) if video['tags'] is not None else None
            
            # Convert categoryId to category based on categories dictionary
            video['category'] = categories.get(video['categoryId'], None) if video['categoryId'] is not None else None
            del video['categoryId']

            # Parse the thumbnail url
            video['thumbnailUrl'] = video['thumbnails'].get('standard', {}).get('url', None) if video['thumbnails'] is not None else None
            del video['thumbnails']
            
            # Convert viewCount, likeCount, and commentCount to integer
            video['viewCount'] = int(video['viewCount']) if video['viewCount'] is not None else None
            video['likeCount'] = int(video['likeCount']) if video['likeCount'] is not None else None
            video['commentCount'] = int(video['commentCount']) if video['commentCount'] is not None else None
            
            # Convert caption to boolean
            video['caption'] = True if video['caption'] == 'true' else False if video['caption'] == 'false' else None
        
        # Save the processed videos data to a new file
        with open(target_file_path, "w") as f:
            json.dump(videos_list, f)
            
    @task()
    def load_to_bigquery(source_file_path: str, table_name: str):
        """
        Loads the processed data to BigQuery.
        
        Args:
            source_file_path: A string representing the path to the file to be loaded.
            table_name: A string representing the name of the table to load the data to.
        """
        
        # Set the path to your service account key file
        key_path = '/opt/airflow/dags/service_account_key.json'

        # Set the credentials using the service account key
        credentials = service_account.Credentials.from_service_account_file(
            key_path,
            scopes=["https://www.googleapis.com/auth/cloud-platform"],
        )

        # Instantiate the BigQuery client with the credentials
        client = bigquery.Client(credentials=credentials)
        
        # Refer to the table where the data will be loaded
        dataset_ref = client.dataset('youtube')
        table_ref = dataset_ref.table(table_name)
        table = client.get_table(table_ref)
        
        # Load the data from the json file to BigQuery
        with open(source_file_path, 'r') as f:
            json_data = json.load(f)
        job_config = bigquery.LoadJobConfig()
        job_config.source_format = bigquery.SourceFormat.NEWLINE_DELIMITED_JSON
        job = client.load_table_from_json(json_data, table, job_config = job_config)
        
        # Waits for the job to complete and log the job results
        job.result()  
        print(f"Loaded {job.output_rows} rows to {table.table_id}")
    
    file_path = '/opt/airflow/dags/tmp_file.json'
    fetch_trending_videos_task = fetch_trending_videos(region_code='ID', max_results=200, target_file_path=file_path)
    processed_file_path = '/opt/airflow/dags/tmp_file_processed.json'
    data_processing_task = data_processing(source_file_path=file_path, target_file_path=processed_file_path)
    load_to_bigquery_task = load_to_bigquery(source_file_path=processed_file_path, table_name='trending_videos')
    
    fetch_trending_videos_task >> data_processing_task >> load_to_bigquery_task
    
dag = trending_youtube_dag()
```

Now, let's explore some specific sections of the code above in more detail. The section of the code below defines the default configuration options for an Airflow DAG. It specifies various attributes that apply to all tasks within the DAG unless overridden at the task level. With the configuration below, a failed task will be retried for 3 times with 5 minutes delay between each retry.
```{python}
default_args = {
    'owner': 'tmtsmrsl',
    'retries': 3,
    'retry_delay': timedelta(minutes=5)
}
```

The next section of the code defines the DAG itself. Below are the explanations of the parameters included in the `@dag` decorator:
- `dag_id` parameter is a unique identifier for the DAG, which will be shown in the Airflow Webserver UI 
- `default_args` parameter refers to the configuration options defined above
- `description` parameter provides a description or summary of the DAG's purpose
- `start_date` parameter is the date when the DAG is eligible to run tasks, which you can adjust to the current date
- `schedule_interval` parameter defines the schedule for the DAG. Here, it is set to run daily at 10:00 AM GMT+7 according to the cron expression '0 10 * * *' and the timezone specified in the `start_date`
- `catchup` parameter is a boolean that indicates whether the DAG should run for the past dates or not (in our case, we set it to `False` to prevent the DAG from running for the past dates)
```{python}
@dag(dag_id='trending_youtube_dag_v1',
    default_args=default_args,
    description='A pipeline to fetch trending YouTube videos',
    start_date=datetime(2023, 5, 17, tzinfo=timezone(timedelta(hours=7))),
    schedule_interval='0 10 * * *',
    catchup=False)
def trending_youtube_dag():
    ...
```

After defining the DAG and tasks, we need to specify the dependencies between the tasks. In our case, the `fetch_trending_videos_task` task should be executed first, followed by the `data_processing_task`, and finally the `load_to_bigquery_task`. We can specify the dependencies between the tasks by using the `>>` operator, which indicates that the task on the left side should be executed before the task on the right side. The code below shows how we can specify the dependencies between the tasks.
```{python}
@dag(...)
def trending_youtube_dag():
    @task()
    def fetch_trending_videos(region_code: str, max_results: int, target_file_path: str):
        ...
    
    @task()
    def data_processing(source_file_path: str, target_file_path: str):
        ...
            
    @task()
    def load_to_bigquery(source_file_path: str, table_name: str):
        ...
    
    file_path = '/opt/airflow/dags/tmp_file.json'
    fetch_trending_videos_task = fetch_trending_videos(region_code='ID', max_results=200, target_file_path=file_path)
    processed_file_path = '/opt/airflow/dags/tmp_file_processed.json'
    data_processing_task = data_processing(source_file_path=file_path, target_file_path=processed_file_path)
    load_to_bigquery_task = load_to_bigquery(source_file_path=processed_file_path, table_name='trending_videos')
    
    fetch_trending_videos_task >> data_processing_task >> load_to_bigquery_task
```

The final line of the code creates the DAG instance and registers it in Airflow.
```{python}
dag = trending_youtube_dag()
```

By now your project structure should look similar to this. You can delete the `tmp_file.json` and `tmp_file_processed.json` files if you want to.
```
airflow_youtube_project
├── airflow_youtube_env
├── dags
│   └── .env (containing YOUTUBE_API_KEY)
│   └── service_account_key.json
│   └── categories.json
│   └── trending_youtube_dag.py
├── logs
├── plugins
├── .env (containing AIRFLOW_UID)
├── docker-compose.yaml
├── tmp_file.json
└── tmp_file_processed.json
```

### Monitoring the DAG through Airflow Webserver UI
Now that we have created the DAG, it's time to check the Airflow Webserver UI to see if the DAG has been registered successfully. I assume that you have followed the instructions in the earlier section to install and run Airflow using Docker. First open Docker Desktop and and ensure that the containers (except for `airflow-init-1`) under `airflow_youtube_project` are running. You can verify this by checking the container status, as shown in the screenshot below.
![Checking containers status on Docker Desktop](/_images/ayp_docker_containers_status.png)

If the containers are not running, you can start them by navigating to the `airflow_youtube_project` directory and executing the following command.
```{bash}
$ docker compose up
```

As seen from the last screenshot, the port for `airflow-webserver-1` is shown as 8080:8080. This means port 8080 on the host machine is mapped to port 8080 on the container. Now open your browser and go to `http://localhost:8080` to access the Airflow Webserver UI. Enter your username and password, which by default are both `airflow`. After logging in, you should be able to see the list of DAGs registered in Airflow. Search for the DAG that we have created earlier (`trending_youtube_dag_v1`) and click on it to see the details.
![Details of trending_youtube_dag_v1 DAG](/_images/ayp_airflow_dag_details.png)

To unpause the DAG, click on the toggle button on the top left corner of the page. The color of the toggle background should change from grey to blue. To check when the next DAG run will be, hover to the light gray box containing `Next Run: ...`. A dark gray box will appear showing details such as `Run After`, `Next Run` and `Date Interval`. In the case below, the dark gray box indicates that the next DAG run is scheduled to occur in 18 hours. 
![Schedule of next DAG run](/_images/ayp_airflow_next_run.png)

If you wish to visualize the dependencies between the tasks, you can click on the `Graph` tab. The screenshot below shows the dependencies between the tasks in our DAG.
![Graph view of dependencies between tasks](/_images/ayp_airflow_graph.png)

After the DAG has been executed, we can can monitor the status of the DAG run and individual tasks to ensure everything ran as expected. The screenshot below shows the status of our first DAG run and the corresponding tasks. In the Airflow UI, successful tasks are denoted by green boxes, while any failed tasks will be denoted by red boxes. Additionally, hovering over each task provides additional information such as the execution date and duration, allowing us to analyze the performance of our DAG. If you are interested to see the logs of each task, simply click on the respective task box and select the `Logs` option.
![Status of DAG run and the corresponding tasks](/_images/ayp_airflow_dag_status.png)

Notice that earlier we have only installed the dependecies in the `youtube_airflow_env` virtual environment on our local machine. You might be curious how the DAG can run successfully within the Docker environment even though we have not installed any of the dependencies inside the Docker containers. Well, this is because the Docker containers (which are created from the `docker-compose.yaml` file provided by Airflow) come with a set of pre-installed Python libraries by default, which happen to include all of our required dependencies. We can verify this by opening the terminal within the `airflow-worker-1` container and checking if our dependencies are installed using the command below. 
```{bash}
$ container_id=$(docker ps --filter name=airflow-worker-1 --format "{{.ID}}")
$ docker exec -it $container_id bash
$ pip list | grep -E 'google-api-python-client|python-dotenv|isodate|google-cloud-bigquery'
google-api-python-client                 1.12.11
google-cloud-bigquery                    2.34.4
google-cloud-bigquery-datatransfer       3.7.0
google-cloud-bigquery-storage            2.14.1
isodate                                  0.6.1
python-dotenv                            0.21.1
```

### Visualizing the Data in BigQuery using Looker Studio
Most people are not interested in examining the raw data directly within BigQuery. Instead, they prefer a more user-friendly and visually appealing representation of the data. In this section, we will use Looker Studio to create a dashboard to visualize the data that we have loaded into BigQuery earlier.

First navigate to <u>[Looker Studio](https://lookerstudio.google.com/)</u>. Our initial step is to establish a new connection to BigQuery within Looker Studio. To accomplish this, click on the `Create` button on the left panel and select `Data Source`. Then select `BigQuery` as the desired data source. If this is your first time connecting to BigQuery from Looker Studio, you may need to click the "Authorize" button. Following that, we'll need to select the appropriate `Project`, `Dataset` and `Table`, then click on the `Connect` button. Finally, click on the `Create Report` button to create a new report, which will be used to create our dashboard.

I encourage you to explore the different options available in Looker Studio and use your creativity to design your own dashboard. It might take some time to be familiar with Looker Studio, but if you have used other dashboarding tools, you'll quickly get the hang of it. For this article, I'll share my completed dashboard as an example. Note that I have been retrieving data from YouTube API on a daily basis since 8th May 2023 up to 18th May 2023 (which is the date of writing this article). It includes three charts, a table and a date filter, which are shown below.
![Preview of Looker dashboard](/_images/ayp_looker_dashboard.png)

The dashboard above aims to address the following questions:
- Which video categories are the most popular?
- What is the average view count for trending videos in each category?
- Is publication day related to the likelihood of a video becoming trending?
- Which videos appeared the most frequently in the trending list?

You can also access my dashboard directly <u>[here](https://lookerstudio.google.com/reporting/d29391f8-334a-4e13-add2-d4b6ffcb1f15)</u>. You'll be able to interact with the latest data and experiment with different date filter to observe how the charts and table dynamically respond to your selections.

### Conclusion
In this article, we have explored how to use Youtube Data API v3 to retrieve data about trending videos on YouTube. We have also learned how to use Airflow within a Docker environment to automate our data pipeline. Specifically, we have used Airflow to automate the ETL (extract, transform, and load) process from the YouTube Data API into BigQuery. Furthermore we have seen how Looker Studio can be utilized to visualize the data in BigQuery. Hopefully, this article provides you with valuable insights on automating your data retrieval, processing, loading, and visualization processes, allowing you to streamline your workflows effectively.

If you wish to check out the code and folder structure for this project, it is available on my <u>[Github Repository](https://github.com/tmtsmrsl/airflow_youtube_project)</u>. And if you want to see an example of extensive analysis on the trending youtube videos data, you can check out my older project <u>[here](https://github.com/tmtsmrsl/TrendingYoutubeVideos)</u>. Below are some of the visualizations that I have created in my older project.
![Visualiztions from older project](/_images/ayp_older_project_viz.jpg)