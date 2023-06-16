---
title: Streaming Data Processing - Building an OLTP, OLAP and Sales Dashboard with Open Source Stacks | Project Overview and Environment Setup (Part 1)
post_excerpt: This project showcases the implementation of a streaming data processing pipeline using open-source technologies (Kafka, Spark Streaming, Cassandra, and MySQL). In the first part of the article, we will discuss the overview of the project and how to set up the environment.
taxonomy:
  category:
    - knowledge
    - notes
---

In the era of big data where the volume and velocity of data are increasing at an exponential rate, the ability to efficiently process and analyze streaming data has become increasingly important. As quoted from an <u>[article](https://supertype.ai/notes/streaming-pipeline-for-warehouse-inventory-management-system/)</u> written by my colleague regarding the importance of streaming analytics in businesses:
> Businesses need a way to process this data (stream of events) in the shortest time possible to avoid the staleness problem, as well as making decisions right on time. This is where streaming analytics comes into the picture. It allows the continuous stream of data to be processed as soon as they are generated.

In this project, we will explore the power of open-source technologies (Kafka, Spark Streaming, Cassandra and MySQL) to build a robust and scalable streaming data processing pipeline. We will begin by producing simulated raw order data using a producer and sending it to Kafka, a distributed streaming platform. Leveraging the micro-batch processing capabilities of Spark Streaming, we will load the raw data into Cassandra, a distributed NoSQL database, for real-time transaction processing (OLTP). Simultaneously, we will aggregate the data and store it in MySQL, a relational database, for analytical processing (OLAP). To bring the insights to life, we will visualize the aggregated data in the form of a dynamic sales dashboard using Streamlit, an open-source Python library to build custom web apps. This comprehensive architecture allows organizations to extract (near) real-time insights from streaming data, enabling informed decision-making and improved business performance.

Now, let's start our project by setting up the environment. To ensure reproducibility, we will utilize Docker Compose to run Kafka, Spark, Cassandra, and MySQL in Docker containers. If you are unfamiliar with Docker or Docker Compose, I highly recommend watching the following two videos to gain a better understanding of them: <u>[Hands on Introduction to Docker & Data Persistence w/ Docker Volumes](https://www.youtube.com/watch?v=bRyuhBJtJ6M&list=PLXsFtK46HZxUUAQSZRMP3g2YSvVJYoGgd)</u> and <u>[Hands on introduction to Docker Compose V2 (tutorial ft. wordpress, mysql, phpmyadmin)](https://www.youtube.com/watch?v=ZlLwDN9_Gwg&list=PLXsFtK46HZxUUAQSZRMP3g2YSvVJYoGgd&index=3)</u>.

### Setting up the environment

First we will create a new project directory named `streaming_data_processing`. Then we will make a new file named `docker-compose.yml` in the project directory and copy the following code into the file.
```{yaml}
version: '3.8'

services:
  zookeeper:
    image: bitnami/zookeeper:3.8.1
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    volumes:
      - zookeeper-data:/bitnami/zookeeper

  kafka:
    image: bitnami/kafka:3.2.3
    container_name: kafka
    ports:
      - "29092:29092"
    environment:
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_ENABLE_KRAFT=no
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_CFG_LISTENERS=INTERNAL://:9092,EXTERNAL://:29092
      - KAFKA_CFG_ADVERTISED_LISTENERS=INTERNAL://kafka:9092,EXTERNAL://localhost:29092
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_INTER_BROKER_LISTENER_NAME=INTERNAL
      
    volumes:
      - kafka-data:/bitnami/kafka
    depends_on:
      - zookeeper
    restart: always
      
  spark:
    image: bitnami/spark:3.4.0
    container_name: spark
    environment:
      - SPARK_MODE=master
    ports:
      - "8080:8080"
      - "4040:4040"
    volumes:
      - /home/tmtsmrsl/streaming_data_processing/spark_script:/spark_script
      - /home/tmtsmrsl/streaming_data_processing/spark-defaults.conf:/opt/bitnami/spark/conf/spark-defaults.conf
    depends_on:
      - zookeeper
      - kafka
      - cassandra
    command: bash -c "python -m pip install py4j==0.10.9.7 && tail -f /dev/null"

  spark-worker:
    image: docker.io/bitnami/spark:3.4.0
    container_name: spark-worker
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark:7077
      - SPARK_WORKER_MEMORY=1G
      - SPARK_WORKER_CORES=1
    ports:
      - "8081:8081"
    volumes:
      - /home/tmtsmrsl/streaming_data_processing/spark-defaults.conf:/opt/bitnami/spark/conf/spark-defaults.conf
    depends_on:
      - zookeeper
      - kafka
      - cassandra
    command: bash -c "python -m pip install py4j==0.10.9.7 && tail -f /dev/null"

  cassandra:
    image: cassandra:4.1.2
    container_name: cassandra
    ports:
      - "9042:9042"
    volumes:
      - cassandra-data:/var/lib/cassandra

  mysql:
    image: mysql:8.0.33
    container_name: mysql
    ports:
      - "3307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  zookeeper-data:
  kafka-data:
  cassandra-data:
  mysql-data:
```

We will discuss the contents of the `docker-compose.yml` file in detail in the next section. For now, let's continue setting up the environment.

Next, we will create a new file named `spark-defaults.conf` in the project directory, which contains the Spark application configuration properties. We will populate the file later, so for now we can leave it empty.

Last, we will create a new folder named `spark_script` in the project directory, which will be used to store the Spark application script. For now, we can leave the folder empty.

Now, your project directory should look like this:
```{bash}
streaming_data_processing
├── docker-compose.yml
├── spark-defaults.conf
└── spark_script
```

### Understanding the Contents of `docker-compose.yml` File

In this section, we will provide a comprehensive explanation of the `docker-compose.yml` file and its contents. While it's not necessary to comprehend every aspect of the `docker-compose.yml` file to successfully complete the project, having a thorough understanding can be useful for troubleshooting purposes and enhance your learning experience. If you're not particularly interested in the finer details, feel free to skip ahead to the next article and revisit the following sections later if you encounter any issues or wish to deepen your understanding.

The `docker-compose.yml` file include seven services: `zookeeper`, `kafka`, `spark`, `spark-worker`, `cassandra`, and `mysql`. For each service, we define the service name and other configurations such as `image`, `container_name`, `ports`, `environment`, `volumes`, `depends_on`, `restart`, and `command`. Below is a brief description of each configuration.
* The service name is used to identify and reference a specific service within the Docker Compose file.
* The `image` configuration specifies the Docker image to be used for the service which will be pulled from <u>[Docker Hub](https://hub.docker.com/)</u>.
* The `container_name` configuration allows us to assign a custom name to the container created from the image. Although it doesn't have to match the service name, it is recommended to keep them consistent.
* The `ports` configuration defines the port mappings between the host machine and the container. It allows us to access services running inside the container from the host using specified ports.
* The `environment` configuration is used to set environment variables specific to the container.
* The `volumes` configuration is used to mount either Docker volumes or bind mounts inside the container. Docker volumes provide a means to persist data generated by the container by creating and managing storage entities that are independent of the host's filesystem. On the other hand, bind mounts establish a direct mapping between a directory on the host and a directory inside the container. This allows us to retain data even after the container is removed.
* The `depends_on` configuration specifies the service that the current service depends on, which ensures that the current service will not start until all the dependencies are started.
* The `restart` configuration determines the restart policy for a container. By default, it is set to `no`, which means the container will not automatically restart if it stops or encounters an error.
* The `command` configuration specifies the command that will be executed when the container starts.

Now, we will discuss each service and some of the important configurations in details, starting from the `zookeeper` service.
```{yaml}
  zookeeper:
    image: bitnami/zookeeper:3.8.1
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    volumes:
      - zookeeper-data:/bitnami/zookeeper
```

The `zookeeper` service is a centralized coordinator for managing metadata and maintaining the overall state of the Kafka cluster. For the `environment` configuration, we set `ALLOW_ANONYMOUS_LOGIN` to `yes` so that we can connect to the `zookeeper` service without authentication. Note that this is not recommended for production environments. In the provided configuration, the `volumes` demonstrates the usage of Docker volumes (not bind mounts). Notice that we specify `zookeeper-data` on the left side of the colon, which is not a path on the host machine but rather managed storage entities created by Docker.

Next, we will discuss the configurations for `kafka` service.
```{yaml}
  kafka:
    image: bitnami/kafka:3.2.3
    container_name: kafka
    ports:
      - "29092:29092"
    environment:
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_ENABLE_KRAFT=no
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_CFG_LISTENERS=INTERNAL://:9092,EXTERNAL://:29092
      - KAFKA_CFG_ADVERTISED_LISTENERS=INTERNAL://kafka:9092,EXTERNAL://localhost:29092
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_INTER_BROKER_LISTENER_NAME=INTERNAL
    volumes:
      - kafka-data:/bitnami/kafka
    depends_on:
      - zookeeper
    restart: always
```

The `kafka` service is used to store and distribute the messages within a Kafka cluster. Below are the explanation for each of the `environment` configurations:
* ALLOW_PLAINTEXT_LISTENER=yes - By allowing plaintext listener, we allow clients to connect to Kafka without any authentication. This is not recommended for production environments.
* KAFKA_ENABLE_KRAFT=no - By disabling Kafka Raft Metadata mode, we can use the traditional Zookeeper-based Kafka cluster.
* KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181 - This variable specifies the hostname and port of the Zookeeper server. Since we are using Docker Compose, we can use the service name `zookeeper` as the hostname and the default Zookeeper port `2181`.

We also need to configure two listeners, one for internal client (other services within the Docker network) and another one for external client (clients outside the Docker network, i.e. the host machine). We will name the listener for the internal client as `INTERNAL`, and the listener for the external client as `EXTERNAL`. These listeners are defined using the following `environment` configurations:
* KAFKA_CFG_LISTENERS=INTERNAL://:9092,EXTERNAL://:29092 - The network interface and port for each listener on which Kafka listens for incoming connections. Here we don't specify any hostname or IP address for both listeners, which means Kafka will listen on all network interfaces within the specified port.
* KAFKA_CFG_ADVERTISED_LISTENERS=INTERNAL://kafka:9092,EXTERNAL://localhost:29092 - The network interface and port for each listener that clients should use to connect to the Kafka broker. Here we specify `kafka` as the hostname for the internal client and `localhost` as the hostname for the external client.
* KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT - The security protocol used for each listener. In our case, we are using the `PLAINTEXT` protocol for both listeners, indicating that no authentication is required.
* KAFKA_INTER_BROKER_LISTENER_NAME=INTERNAL - The name of the listener used for communication between brokers. Typically the listener for the internal client is used for inter-broker communication.

For the `ports` configuration, we need to expose the external listener port `29092` to the host machine so that we can connect to the `kafka` service from the host machine.

Since the `kafka` service depends on the `zookeeper` service, we specify `zookeeper` in the `depends_on` configuration to ensure that the `zookeeper` service is started before the `kafka` service. Personally I have encountered `kafka` service stopping unexpectedly, so I set `restart` configuration to `always` to ensure that the `kafka` service will be restarted automatically if it stops or encounters an error.

Next, we will discuss the configurations for `spark` and `spark-worker` services.
```{yaml}
  spark:
    image: bitnami/spark:3.4.0
    container_name: spark
    environment:
      - SPARK_MODE=master
    ports:
      - "8080:8080"
      - "4040:4040"
    volumes:
      - /home/tmtsmrsl/streaming_data_processing/spark_script:/spark_script
      - /home/tmtsmrsl/streaming_data_processing/spark-defaults.conf:/opt/bitnami/spark/conf/spark-defaults.conf
    depends_on:
      - zookeeper
      - kafka
      - cassandra
    command: bash -c "python -m pip install py4j==0.10.9.7 && tail -f /dev/null"

  spark-worker:
    image: docker.io/bitnami/spark:3.4.0
    container_name: spark-worker
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark:7077
      - SPARK_WORKER_MEMORY=1G
      - SPARK_WORKER_CORES=1
    ports:
      - "8081:8081"
    volumes:
      - /home/tmtsmrsl/streaming_data_processing/spark-defaults.conf:/opt/bitnami/spark/conf/spark-defaults.conf
    depends_on:
      - zookeeper
      - kafka
      - cassandra
    command: bash -c "python -m pip install py4j==0.10.9.7 && tail -f /dev/null"
```

Here the `spark` service is configured as the Spark master node and `spark-worker` service is configured as the Spark worker node, as indicated by the `SPARK_MODE` under `environment` configuration. Besides `SPARK_MODE`, we also specify the following `environment` configurations for the `spark-worker` service:
* SPARK_MASTER_URL=spark://spark:7077 - The URL of the Spark master node. Here we use the service name `spark` as the hostname and the default Spark master port `7077`.
* SPARK_WORKER_MEMORY=1G - The amount of memory to use for the Spark worker node.
* SPARK_WORKER_CORES=1 - The number of cores to use for the Spark worker node.

For `spark`, we expose ports 8080 (Spark master web UI) and 4040 (Spark application web UI). The Spark application web UI shows detailed information about the Spark application's progress and executed tasks. For `spark-worker` service, we only expose port 8081, which is the Spark worker web UI.

For both `spark` and `spark-worker` services, we mount the `spark-defaults.conf` file into the containers by providing a direct mapping between a directory on the host and a directory inside the container. This file contains the configuration properties for the Spark application. We also mount the `spark_script` directory into the container of `spark` service, which will be used to store the Python script that we will submit to the Spark application.

We also add a `command` configuration to both `spark` and `spark-worker` services to install the `py4j` library, which is required to run the Python script within the Spark application. The `tail -f /dev/null` command is used to keep the container running indefinitely.

Note that we can enhance task performance by increasing the number of Spark worker nodes or increasing the number of cores and memory for each Spark worker node. However, we need to ensure that the host machine has sufficient resources to support the desired configuration.

The configurations for `cassandra` service are pretty straightforward and easy to understand, so we will not discuss them in detail.
```{yaml}
  cassandra:
    image: cassandra:4.1.2
    container_name: cassandra
    ports:
      - "9042:9042"
    volumes:
      - cassandra-data:/var/lib/cassandra
```

Last, we will discuss the configurations for `mysql` service.
```{yaml}
  mysql:
    image: mysql:8.0.33
    container_name: mysql
    ports:
      - "3307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - mysql-data:/var/lib/mysql
```

Notice that the port for `mysql` service is set to `3307` instead of the default port `3306`. This is because I already have another `mysql` service running on port `3306` on my host machine. If you do not have a `mysql` service running on your host machine, you can use the default port `3306` instead. We also set the `MYSQL_ROOT_PASSWORD` to `root` for simplicity, which will be used later to connect to the MySQL server using the `root` user.

At the end of the `docker-compose.yml` file, we specify all the Docker volume names to ensure that the volume names used in the services' configurations are recognized properly.
```{yaml}
volumes:
  zookeeper-data:
  kafka-data:
  cassandra-data:
  mysql-data:
```

### Conclusion

In this article, we have discussed the overall architecture of the streaming data processing pipeline that we will be building. We have also walked through the process of setting up the environment and explained the contents of the Docker Compose file, which will be used to run the necessary services for our pipeline. In the next article (part 2), we will dive into the implementation of the pipeline, starting with the creation of the Cassandra and MySQL databases.
