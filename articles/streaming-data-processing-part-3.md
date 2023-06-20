---
title: Building a Streaming Data Pipeline with Open Source Stacks | Transactional Data Ingestion and Processing (Part 3)
post_excerpt: In the third part of the article, we will demonstrate how to ingest and process transactional data using Kafka and Spark Streaming.
taxonomy:
  category:
    - knowledge
    - notes
---

In the previous article (part 2), we have designed and implemented the OLTP and OLAP databases using Cassandra and MySQL respectively. In this article, we will demonstrate how to ingest and process transactional data streams using Kafka and Spark Streaming.

Kafka is a distributed streaming platform that excels in handling high-throughput, fault-tolerant, and real-time data streams. It provides publish-subscribe messaging system where data is organized into topics and distributed across multiple partitions. Many organizations such as LinkedIn, Netflix, and Uber use Kafka to build real-time data pipelines and streaming applications.

Spark Streaming is a powerful component of the Apache Spark ecosystem that enables scalable, fault-tolerant and (near) real-time processing of streaming data. It extends the core capabilities of Apache Spark to handle continuous streams of data in a microbatch fashion, which means that the data is processed and analyzed in small and continuous intervals. Spark Streaming can be used to process data streams from various sources such as Kafka, Flume, and Kinesis.

In our data pipeline, we utilize the power of Kafka to efficiently load the raw transactional data into Kafka topics. We will then leverage Spark Streaming to process the data in a microbatch fashion and write the processed data to the OLTP database. Simultaneously, the aggregated transactional data will be directed to the OLAP databases, allowing us to perform insightful analytical queries on the data. 

By combining the strengths of Kafka, Spark Streaming, and our database infrastructure, we establish a robust and scalable data pipeline that enables (near) real-time data processing, seamless data storage, and insightful analytics.

### Getting Started with Kafka (Producing and Consuming Messages within a Topic)

First, make sure that you have the Docker containers running for all services specified in the Docker Compose file. If not, execute the following command to start the containers:
```{bash}
docker-compose up -d
```

Next, we will connect to the Kafka container and launch the Kafka shell by executing the following command:
```{bash}
docker exec -it kafka bash
```

Now, we will create two new topics: `test_topic` (for testing purposes) and `sales_topic` (for storing the raw transactional data). We will create the topics by executing the following commands:
```{bash}
kafka-topics.sh --create --topic test_topic --bootstrap-server kafka:9092 --partitions 1 --replication-factor 1
kafka-topics.sh --create --topic sales_orders --bootstrap-server kafka:9092 --partitions 1 --replication-factor 1
```

`kafka-topics.sh` is a command-line tool that is implemented as a shell script. It is used to create, alter, list, and describe topics in Kafka. Below are a bried explanation of the flags used in the above commands:
* `--create` flag is used to create a new topic. 
* `--topic` flag is used to specify the name of the topic.
* `--bootstrap-server` flag is used to specify the address and port of the Kafka broker to connect to. We set it to `kafka:9092` because in the `docker-compose.yml` file we set the `KAFKA_CFG_ADVERTISED_LISTENERS` as `INTERNAL://kafka:9092,EXTERNAL://localhost:29092`. Note that using `localhost:29092` will work as well because the `kafka` container could connect to both the internal and external listeners.
* `--partitions` flag is used to specify the number of partitions for the topic. The number of partitions determines the parallelism and scalability of data processing in Kafka. In our case, we set the number of partitions to 1 because we are running a single Kafka broker. In a production environment, we would typically have multiple Kafka brokers and we would set the number of partitions to a value greater than 1 to ensure higher throughput and better resource distribution.
* `--replication-factor` flag is used to specify the number of replicas for the topic. A replication factor of 1 is sufficient for our purposes because we are running a single Kafka broker. In a production environment, we would typically have multiple Kafka brokers and we would set the replication factor to a value greater than 1 to ensure fault-tolerance.

You can verify that the topics have been created by executing the following command:
```{bash}
kafka-topics.sh --list --bootstrap-server kafka:9092
```

Next, we will prepare a Python script that will be used to generate the raw transactional data. Before we do that, we will create a new virtual environment and install the required Python packages. To do so, navigate to your project directory and execute the following commands:
```{bash}
python3 -m venv project_env
source ./project_env/bin/activate
python -m pip install kafka-python==2.0.2 cassandra-driver==3.28.0
```

`kafka-python` is a Python client library for Kafka that offers convenient high-level `Producer` and `Consumer` classes. These classes allow us to easily send and receive messages to and from Kafka within Python applications. On the other hand, `cassandra-driver` is a Python client library designed for interacting with Cassandra databases, which allows us to easily establish connections to Cassandra and execute queries on the database.

Now, let's create a new Python script called `producer.py` in the project directory and add the following code to it:
```{python}
import random
import sys
import time
from datetime import datetime
from json import dumps

from cassandra.cluster import Cluster
from kafka import KafkaProducer

CASSANDRA_HOST = 'localhost'
CASSANDRA_KEYSPACE = 'sales'
CASSANDRA_TABLE = 'orders'
KAFKA_BOOTSTRAP_SERVER = 'localhost:29092'

def get_last_order_id():
    cluster = Cluster([CASSANDRA_HOST])
    session = cluster.connect(CASSANDRA_KEYSPACE)
    query = f"SELECT MAX(order_id) AS last_order_id FROM {CASSANDRA_TABLE}"
    result = session.execute(query)
    last_order_id = result.one().last_order_id
    return last_order_id if last_order_id else 0
        
def produce_message(order_id, max_product_id, max_platform_id):
    message = {}
    message["order_id"] = order_id
    message["created_at"] = datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S")
    message['platform_id'] = random.randint(1, max_platform_id)
    message['product_id'] = random.randint(1, max_product_id)
    message['quantity'] = random.randint(1, 10)
    message['customer_id'] = random.randint(1, 1000)
    message['payment_method'] = random.choice(['credit card', 'debit card', 'bank transfer', 'paypal'])
    return message

def main():
    KAFKA_TOPIC = sys.argv[1]
    producer = KafkaProducer(bootstrap_servers=KAFKA_BOOTSTRAP_SERVER,
                            value_serializer=lambda x: dumps(x).encode('utf-8'))
    last_order_id = get_last_order_id()
    print("Kafka producer application started.")
    
    try:
        while True:
            order_id = last_order_id + 1
            max_product_id = 20
            max_platform_id = 4
            message = produce_message(order_id, max_product_id, max_platform_id)
            print(f"Produced message: {message}")
            producer.send(KAFKA_TOPIC, message)
            time.sleep(0.2)
            last_order_id = order_id    
    except KeyboardInterrupt:
        producer.flush()
        producer.close()
        print("Kafka producer application completed.")

if __name__ == "__main__":
    main()
```

The script above serves as a Kafka producer, which generates and sends messages to a Kafka topic using the Kafka Python client library. Let's explore the script in more detail!

First we define the `CASSANDRA_HOST`, `CASSANDRA_KEYSPACE`, `CASSANDRA_TABLE`, and `KAFKA_BOOTSTRAP_SERVER` variables.
```{python}
CASSANDRA_HOST = 'localhost'
CASSANDRA_KEYSPACE = 'sales'
CASSANDRA_TABLE = 'orders'
KAFKA_BOOTSTRAP_SERVER = 'localhost:29092'
```

We can simply use 'localhost' for the `CASSANDRA_HOST` variable because we have exposed the port of `cassandra` container to the host machine. The values assigned to the `CASSANDRA_KEYSPACE` and `CASSANDRA_TABLE` variables correspond to the names we previously defined in the article when setting up the Cassandra database. For the `KAFKA_BOOTSTRAP_SERVER` variable, we set it to 'localhost:29092' based on the `KAFKA_ADVERTISED_LISTENERS` configuration specified in the `docker-compose.yml` file. It's important to note that we are connecting to the Kafka broker from the host machine, so we need to use the hostname and port for the external client.

Next, we define the `get_last_order_id()` function, which is used to retrieve the last order ID from the Cassandra table. This function will be called by the `main()` function to determine the order ID of the next message to be produced when the producer application is started.
```{python}
def get_last_order_id():
    cluster = Cluster([CASSANDRA_HOST])
    session = cluster.connect(CASSANDRA_KEYSPACE)
    query = f"SELECT MAX(order_id) AS last_order_id FROM {CASSANDRA_TABLE}"
    result = session.execute(query)
    last_order_id = result.one().last_order_id
    return last_order_id if last_order_id else 0
```

Then, we define the `produce_message()` function, which is used to generate a random order data.
```{python}
def produce_message(order_id, max_product_id, max_platform_id):
    message = {}
    message["order_id"] = order_id
    message["created_at"] = datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S")
    message['platform_id'] = random.randint(1, max_platform_id)
    message['product_id'] = random.randint(1, max_product_id)
    message['quantity'] = random.randint(1, 10)
    message['customer_id'] = random.randint(1, 1000)
    message['payment_method'] = random.choice(['credit card', 'debit card', 'bank transfer', 'paypal'])
    return message
```

Note that the `created_at` field is generated using the current date and time in UTC timezone, to ensure consistency with the `created_at` field in the Cassandra table. The values for `platform_id` and `product_id` fields are randomly generated based on the `max_platform_id` and `max_product_id` parameters. These parameters should be adjusted according to the number of platforms and products available in our database.

Finally, we define the `main()` function, which is used to start the Kafka producer application.
```{python}
def main():
    KAFKA_TOPIC = sys.argv[1]
    producer = KafkaProducer(bootstrap_servers=KAFKA_BOOTSTRAP_SERVER,
                            value_serializer=lambda x: dumps(x).encode('utf-8'))
    last_order_id = get_last_order_id()
    print("Kafka producer application started.")
    
    try:
        while True:
            order_id = last_order_id + 1
            max_product_id = 20
            max_platform_id = 4
            message = produce_message(order_id, max_product_id, max_platform_id)
            print(f"Produced message: {message}")
            producer.send(KAFKA_TOPIC, message)
            time.sleep(0.2)
            last_order_id = order_id    
    except KeyboardInterrupt:
        producer.flush()
        producer.close()
        print("Kafka producer application completed.")
```

The `main()` function begins by assigning the value of the first command-line argument (`sys.argv[1]`) to the `KAFKA_TOPIC` variable, which will serve as the destination Kafka topic for the messages. The `KafkaProducer` object is created using the previously defined `KAFKA_BOOTSTRAP_SERVER`. We also specify the `value_serializer` parameter to serialize the message value to JSON format. For the first message to be produced, we retrieve the last order ID from the Cassandra table, which is done by calling the `get_last_order_id()` function. 

Within the `while` loop, we continuously generate and send messages to the Kafka topic. The `message` variable is assigned with the value returned by the `produce_message()` function. Note that we set `max_product_id` and `max_platform_id` to 20 and 4 respectively, reflecting the available number of products and platforms in our database. The messages are send to the Kafka topic using the `send()` method of the `KafkaProducer` object. The `while` loop will continue indefinitely until the user interrupts the process by pressing `Ctrl+C` in the terminal. When this happens, the producer application will be terminated.

To verify if the `producer.py` script is working properly, we can do the following steps:
1. Set up a Kafka consumer within the `kafka` container to consume messages from the `test_topic` topic. To do so, open a new terminal window and execute the following command:
  ```{bash}
  docker exec -it kafka bash
  kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic test_topic
  ```
2. Concurrently run the `producer.py` script in the host machine to produce messages to the `test_topic` topic. To do so, open another terminal window and execute the following command:
  ```{bash}
  python producer.py test_topic
  ```
3. Observe that the message produced by the `producer.py` script appears on the Kafka consumer in the first terminal. The message should look something like this:
  ```{json}
  {'order_id': 1, 'created_at': '2023-06-18 12:33:12', 'platform_id': 1, 'product_id': 5, 'quantity': 7, 'customer_id': 928, 'payment_method': 'paypal'}
  ```
4. To stop the `producer.py` script, press `Ctrl+C` in the second terminal.

Please note that if you repeatedly stop and restart the `producer.py` script, the first order ID of the produced message will always start from 1. This behavior is due to the logic in the `producer.py` script, where it queries the `orders` table in the `sales` keyspace to retrieve the last `order_id` and increments it by 1 to generate the next message. However, since we have not yet established a pipeline to load the raw transactional data into the `orders` table, the table remains empty, and the `producer.py` script will consistently obtain a null value for the last order ID (which is treated as 0 within the script).

### Ingesting and Processing Transactional Data with Kafka and Spark Streaming

To recap, our project directory structure should now look like this:
```{bash}
streaming_data_processing
├── docker-compose.yml
├── producer.py
├── spark-defaults.conf
├── project_env
└── spark_script
```

The `spark-defaults.conf` file is bind-mounted to the `spark` and `spark-worker` containers but is currently empty. The `spark_script` folder is bind mounted to the `spark` containers, but currently does not contain any files. In this section, we will populate the `spark-defaults.conf` file with the necessary configurations and the `spark_script` folder with the Spark Streaming application script.

First, let's add the following configurations to the `spark-defaults.conf` file:
```{conf}
spark.jars.packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.4.0,org.apache.kafka:kafka-clients:3.4.0,com.datastax.spark:spark-cassandra-connector_2.12:3.3.0,mysql:mysql-connector-java:8.0.26
```

The above configurations specify the external dependencies that need to be downloaded and added to the Spark environment, so we can use the corresponding functionality in our Spark Streaming application. Here are a brief explanation for each of the dependencies:
* `org.apache.spark:spark-sql-kafka-0-10_2.12:3.4.0` - Enables Spark to use Spark Structured Streaming API when reading and writing Kafka topics.
* `org.apache.kafka:kafka-clients:3.4.0` - Enables Spark to interact with Kafka.
* `com.datastax.spark:spark-cassandra-connector_2.12:3.3.0` - Enables Spark to interact with Cassandra database.
* `mysql:mysql-connector-java:8.0.26` - Enables Spark to interact with MySQL database.

Next, create a new file named `data_streaming.py` in the `spark_script` folder. Place the following code inside the `data_streaming.py` file:
```{python}
import signal
import sys
import time

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum, to_date, current_timestamp

CASSANDRA_HOST = 'cassandra'
CASSANDRA_PORT = 9042
CASSANDRA_KEYSPACE = 'sales'
CASSANDRA_TABLE = 'orders'

MYSQL_HOST = 'mysql'
MYSQL_PORT = 3306
MYSQL_DATABASE = 'sales'
MYSQL_TABLE = 'aggregated_sales'
MYSQL_USERNAME = 'root'
MYSQL_PASSWORD = 'root'

KAFKA_BOOTSTRAP_SERVER = 'kafka:9092'
KAFKA_TOPIC = 'sales_orders'

def write_to_cassandra(df, epoch_id):
    df.write \
        .format("org.apache.spark.sql.cassandra") \
        .options(keyspace=CASSANDRA_KEYSPACE, table=CASSANDRA_TABLE) \
        .mode("append") \
        .save()

def write_to_mysql(df, epoch_id):
    agg_df = df.withColumn("order_date", to_date(col("created_at"))) \
        .groupBy("date", "platform_id", "product_id") \
        .agg(sum("quantity").alias("total_quantity")) \
        .withColumn("processed_at", current_timestamp())

    agg_df.write \
        .jdbc(url=f"jdbc:mysql://{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DATABASE}", 
                table=MYSQL_TABLE, 
                mode="append", 
                properties= {
                    "driver": "com.mysql.cj.jdbc.Driver",
                    "user": MYSQL_USERNAME,
                    "password": MYSQL_PASSWORD}) 

def signal_handler(signal, frame):
    print("Waiting for 90 seconds before terminating the Spark Streaming application...")
    time.sleep(90)
    sys.exit(0)
        
def main():
    spark = SparkSession.builder \
        .appName("Spark-Kafka-Cassandra-MySQL") \
        .config("spark.cassandra.connection.host", CASSANDRA_HOST) \
        .config("spark.cassandra.connection.port", CASSANDRA_PORT) \
        .getOrCreate()
        
    spark.sparkContext.setLogLevel("ERROR")

    spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", KAFKA_BOOTSTRAP_SERVER) \
        .option("subscribe", KAFKA_TOPIC) \
        .option("startingOffsets", "latest") \
        .load() \
        .createOrReplaceTempView("tmp_table")
        
    query = """
        SELECT FROM_JSON(
                CAST(value AS STRING), 
                    'order_id INT, 
                    created_at TIMESTAMP, 
                    platform_id INT, 
                    product_id INT, 
                    quantity INT,
                    customer_id INT,
                    payment_method STRING'
                ) AS json_struct 
        FROM tmp_table
    """
    
    tmp_df = spark.sql(query).select("json_struct.*")
    
    tmp_df.writeStream \
        .foreachBatch(write_to_cassandra) \
        .outputMode("append") \
        .trigger(processingTime='10 seconds') \
        .start() \
            
    tmp_df.writeStream \
        .foreachBatch(write_to_mysql) \
        .outputMode("append") \
        .trigger(processingTime='60 seconds') \
        .start() 
    
    signal.signal(signal.SIGINT, signal_handler)
    spark.streams.awaitAnyTermination()
    
if __name__ == "__main__":
    main()
```

The above code defines the Spark Streaming application that will be used to process the data stream from Kafka and write the processed data to Cassandra and MySQL. We will go through the code section by section to understand the logic behind the application.

First, we define the configurations for the Cassandra, MySQL, and Kafka connections. 
```{python}
CASSANDRA_HOST = 'cassandra'
CASSANDRA_PORT = 9042
CASSANDRA_KEYSPACE = 'sales'
CASSANDRA_TABLE = 'orders'

MYSQL_HOST = 'mysql'
MYSQL_PORT = 3306
MYSQL_DATABASE = 'sales'
MYSQL_TABLE = 'aggregated_sales'
MYSQL_USERNAME = 'root'
MYSQL_PASSWORD = 'root'

KAFKA_BOOTSTRAP_SERVER = 'kafka:9092'
KAFKA_TOPIC = 'sales_orders'
```

The hostname for Cassandra and MySQL are based on the container names that we have defined in the `docker-compose.yml` file, and the ports are based on the default ports for Cassandra and MySQL. For the `KAFKA_BOOTSTRAP_SERVER` variable, we set it to 'kafka:9092' based on the `KAFKA_ADVERTISED_LISTENERS` configuration specified for the internal client in the `docker-compose.yml` file. 

Next, we define the `write_to_cassandra` function which will be used to write the raw transactional data to the Cassandra table.
```{python}
def write_to_cassandra(df, epoch_id):
    df.write \
        .format("org.apache.spark.sql.cassandra") \
        .options(keyspace=CASSANDRA_KEYSPACE, table=CASSANDRA_TABLE) \
        .mode("append") \
        .save()
```

The `epoch_id` parameter is required for the function passed to the `foreachBatch` method in Spark Streaming. It is used to uniquely identify each batch of data processed by Spark Streaming. We set the mode to `append` because we want to append the data to the existing data in the Cassandra table. 

Next, we define the `write_to_mysql` function which will be used to write the aggregated data to the MySQL table.
```{python}
def write_to_mysql(df, epoch_id):
    agg_df = df.withColumn("order_date", to_date(col("created_at"))) \
        .groupBy("order_date", "platform_id", "product_id") \
        .agg(sum("quantity").alias("total_quantity")) \
        .withColumn("processed_at", current_timestamp())

    agg_df.write \
        .jdbc(url=f"jdbc:mysql://{MYSQL_HOST}:{MYSQL_PORT}/{MYSQL_DATABASE}", 
                table=MYSQL_TABLE, 
                mode="append", 
                properties= {
                    "driver": "com.mysql.cj.jdbc.Driver",
                    "user": MYSQL_USERNAME,
                    "password": MYSQL_PASSWORD}) 
```

To create the aggregated dataframe, we first convert the `created_at` column (which contains the date and time) to a `order_date` column (which contains only the date) using the `to_date` function. Then, we aggregate the sales data and determine the total quantity of each product sold within that specific microbatch, by grouping the data based on the `order_date`, `platform_id`, and `product_id`. Note that we include the `order_date` column in the group by clause to ensure that order data from different day are not aggregated together. We also add a `processed_at` column to the aggregated dataframe to indicate the time when the data is processed, using the `current_timestamp` function which by default returns the current time in UTC timezone. Finally the aggregated dataframe is written to the MySQL table using the `write.jdbc` method.

Next, we define the `signal_handler` function which handle the interruption signal received by the application. 
```{python}
def signal_handler(signal, frame):
    print("Waiting for 90 seconds before terminating the Spark Streaming application...")
    time.sleep(90)
    sys.exit(0)
```

The objective of this function is to add a delay before terminating the Spark Streaming application. It provides a grace period for any ongoing processing or cleanup tasks before the application is terminated. We set the delay period to 90 seconds, which is slightly longer than the microbatch interval of `write_to_mysql`, which is set to 60 seconds.

Finally, we define the `main` function which will be used to define the Spark Streaming application.
```{python}
def main():
    spark = SparkSession.builder \
        .appName("Spark-Kafka-Cassandra-MySQL") \
        .config("spark.cassandra.connection.host", CASSANDRA_HOST) \
        .config("spark.cassandra.connection.port", CASSANDRA_PORT) \
        .getOrCreate()
        
    spark.sparkContext.setLogLevel("ERROR")

    ...
```

The first part of the `main` function is used to define the Spark session. We set the application name to "Spark-Kafka-Cassandra-MySQL" and configure the Cassandra host and port. Then, we set the log level to "ERROR" to reduce the amount of logs generated to the console.

```{python}
def main():
    ...
    
    spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", KAFKA_BOOTSTRAP_SERVER) \
        .option("subscribe", KAFKA_TOPIC) \
        .option("startingOffsets", "latest") \
        .load() \
        .createOrReplaceTempView("tmp_table")

    query = """
        SELECT FROM_JSON(
                CAST(value AS STRING), 
                    'order_id INT, 
                    created_at TIMESTAMP, 
                    platform_id INT, 
                    product_id INT, 
                    quantity INT,
                    customer_id INT,
                    payment_method STRING'
                ) AS json_struct 
        FROM tmp_table
    """
    
    tmp_df = spark.sql(query).select("json_struct.*")

    ...
```

The next part of the `main` function is used to read the data stream from Kafka. We set the Kafka bootstrap server and topic to the values defined in the variables we defined earlier. The option "startingOffsets" is set to "latest", which means the Spark Streaming application starts consuming the data that is produced after the application is started. Any data that was previously published to the topic before the application is started will not be consumed. Then, we create a temporary view named "tmp_table", which stores the data stream from Kafka in a JSON format. Next, we define a SQL query to convert the JSON data into structured columns. Finally, we execute the SQL query and store it in a dataframe named `tmp_df`.

```{python}
def main():
    ...

    tmp_df.writeStream \
        .foreachBatch(write_to_cassandra) \
        .outputMode("append") \
        .trigger(processingTime='10 seconds') \
        .start() \
            
    tmp_df.writeStream \
        .foreachBatch(write_to_mysql) \
        .outputMode("append") \
        .trigger(processingTime='60 seconds') \
        .start() 

    ...
```

In the next part of the `main` function, we define the two streaming queries by using the `writeStream` method. The first query responsible for writing the raw transactional data to the Cassandra table, while the second query is responsible for writing the aggregated data to the MySQL table. 

For the first query, we use the `write_to_cassandra` function as the processing logic within the `foreachBatch` method. The `outputMode` is set to "append", which means that only the newly generated rows since the last trigger will be written to the sink (Cassandra table). The `trigger` is set to a processing time interval of 10 seconds, which determines the frequency at which microbatches are processed. Note that the lowest limit for the trigger interval is 100 milliseconds, however we should consider the underlying infrastructure and capabilities of the system/ cluster as well.

Simiarly, for the second query, we use the `write_to_mysql` function as the processing logic within the `foreachBatch` method. The `outputMode` is also set to "append". The `trigger` is set to a longer processing time interval of 60 seconds, because we want to accumulate more data before performing the aggregation.

```{python}
def main():
    ...

    signal.signal(signal.SIGINT, signal_handler)
    spark.streams.awaitAnyTermination()
```

In the last part of the `main` function, we register a signal handler for the `SIGINT` signal using the `signal.signal` function. This allows us to capture the interrupt signal (`Ctrl+C`) and perform any neccessary additional processing before terminating the application. Finally, we call the `awaitAnyTermination` method on the `SparkSession` streams to ensure that the application continues to run and process data until it is explicitly terminated or encounters an error. Without this method, the program would reach the end of the script and terminate immediately, without giving the streaming queries a chance to process any data.

Now, let's open a new terminal and run the Spark Streaming application inside the `spark` container:
```{bash}
docker exec -it spark bash
python /spark_script/data_streaming.py
```

Wait until all the dependencies are installed sucesfully and you should see the following text in the terminal:
```{bash} 
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
```

Next, open another terminal and run the `producer.py` script in the host machine to produce messages to the `sales_orders` topic:
```{bash}
python producer.py sales_orders
```

Let the producer script to run for around 15 seconds, then open another terminal to check if the data has been successfully written to Cassandra:
```{bash}
docker exec -it cassandra bash
cqlsh
```

```{cql}
USE sales;
SELECT * FROM orders;
```

If the data is successfully written to Cassandra, you should see an output similar to the following:
```{cql}
| order_id |           created_at            | customer_id | payment_method | platform_id | product_id | quantity |
|----------|---------------------------------|-------------|----------------|-------------|------------|----------|
|    53    | 2023-06-18 07:04:45.000000+0000 |     30      |   credit card  |      3      |     8      |    9     |
|    55    | 2023-06-18 07:04:41.000000+0000 |     866     |   debit card   |      2      |    12      |    3     |
|    28    | 2023-06-18 07:04:40.000000+0000 |     233     |   credit card  |      1      |     4      |    5     |
```

And after around one minute, we can also check if the data has been successfully written to MySQL. Open another terminal and run the following commands:
```{bash}
docker exec -it mysql bash
mysql -u root -p
```

Enter the password `root` when prompted (as specified in the `docker-compose.yml` file). Then run the following commands:
```{sql}
USE sales;
SELECT * from aggregated_sales;
```

If the data is successfully written to MySQL, you should see an output similar to the following:
```{sql}
| processing_id |    processed_at     | order_date  | platform_id | product_id | total_quantity |
|---------------|---------------------|-------------|-------------|------------|----------------|
|       1       | 2023-06-18 07:05:00 | 2023-06-18  |      1      |      9     |       16       |
|       2       | 2023-06-18 07:05:00 | 2023-06-18  |      4      |     16     |       9        |
|       3       | 2023-06-18 07:05:00 | 2023-06-18  |      3      |      7     |       4        |
```

Note that each record in the `aggregated_sales` table corresponds to the sum of quantities sold for a particular product on a specific platform and order date within a microbatch, which has a 1-minute interval. The timestamps in the `processed_at` column are stored in UTC timezone, similar to the `created_at` column in the `orders` table in Cassandra.

You can keep the `producer.py` script and the Spark Streaming application running to continuously produce and process the data stream. To stop the operation, plaese follow the steps below.
1. Go to the terminal running the `producer.py` script and press Ctrl + C to stop the producer application. 
2. After the producer application has been stopped, go to the terminal running the Spark Streaming application and press Ctrl + C to terminate the application. Note that the Spark Streaming application will wait around 90 seconds before it terminates.

By following the steps above, you ensure that all the data produced by the producer application are ingested and processed by the Spark Streaming application.

If you want to stop the Docker containers, you can run the following command in the project directory:
```{bash}
docker compose stop
```

### Conclusion
In this article, we have covered the steps involved in producing and consuming messages within a topic using Kafka. Additionally, we have successfully implemented a data pipeline that leverages Kafka and Spark Streaming to ingest and process transactional data streams. The processed data is then loaded into both OLTP and OLAP databases, enabling (near) real-time data processing and insightful analytics. 

In the next article (part 4), we will take our data analysis further by performing analytical queries on the data stored in the OLAP database. Furthermore, we will create a (near) real-time dashboard using Streamlit to visualize the sales and profit data for the current day. This will provide a comprehensive view of the ongoing business performance and facilitate data-driven decision-making.