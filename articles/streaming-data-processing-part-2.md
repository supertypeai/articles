---
title: Streaming Data Processing - Building an OLTP, OLAP and Sales Dashboard with Open Source Stacks | OLTP and OLAP Databases Setup (Part 2)
post_excerpt: This project showcases the implementation of a streaming data processing pipeline using open-source technologies (Kafka, Spark Streaming, Cassandra, and MySQL). In the second part of the article, we will walk through the setup of OLTP and OLAP databases.
taxonomy:
  category:
    - knowledge
    - notes
---

In the previous article (part 1), we have explored the overview of the project and also walk through the setup of the environment. If you have followed the steps in the previous article, you should have a project directory which looks like the following:
```{bash}
streaming_data_processing
├── docker-compose.yml
├── spark-defaults.conf
└── spark_script
```

In this article, we will design the OLTP (Online Transaction Processing) database schema and implement it in Cassandra. Cassandra is a NoSQL database that excels at handling high volumes of writes and reads, making it well-suited for OLTP workloads. Its distributed architecture and ability to scale horizontally enable efficient data storage and retrieval for real-time transactional applications.

Additionally, we will design the OLAP (Online Analytical Processing) database schema and implement it in MySQL. MySQL is a widely used relational database management system that excels in handling multidimensional tables, making it well-suited for OLAP workloads. Its robust support for complex queries and aggregations makes it an ideal choice for analytical applications.

By leveraging the strengths of Cassandra for OLTP and MySQL for OLAP, we can build a comprehensive data processing pipeline that handles both real-time transactional data and analytical queries effectively.

### OLTP Database Design and Implementation

First, we need to build and run all of the services defined in the `docker-compose.yml` file. To do so, navigate to the project directory and execute the following command:
```{bash}
docker compose up -d
```

This command will initiate the containers in detached mode, allowing them to run in the background.

Next, we will access the Cassandra container and launch the CQL shell by executing the following command:
```{bash}
docker exec -it cassandra cqlsh
```

This command will open an interactive session within the Cassandra container, allowing you to execute CQL queries and interact with the database.

Now, we will create a new keyspace named `sales` by executing the following command:
```{cql}
CREATE KEYSPACE sales WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
```

Note that the replication class of `SimpleStrategy` and a replication factor of one are only suitable for development purposes. In a production environment with multiple data centers, it is essential to use other replication strategies (i.e. `NetworkTopologyStrategy`) and replication factors (greather than one) to ensure fault tolerance and high availability.

Next, we will create a new table named `orders` in the `sales` keyspace by executing the following command:
```{cql}
USE sales;

CREATE TABLE sales.orders (
  order_id INT PRIMARY KEY,
  created_at TIMESTAMP,
  platform_id INT,
  product_id INT,
  quantity INT,
  customer_id INT,
  payment_method VARCHAR
  );
```

This table will be part of our OLTP database, which will store transactional data (raw order data) for our application. The orders table will have several columns including `order_id`, `created_at`, `platform_id`, `product_id`, `quantity`, `customer_id`, and `payment_method`. The order_id column will serve as the primary key for the table, ensuring uniqueness for each order. 

To illustrate, here is an example of how the row in the orders table will look like:
| order_id | created_at | product_id | platform_id | quantity |
| - | - | - | - | - |
| 1 | '2023-06-06T13:52:00' | 2 | 3 | 10 | 

### OLAP Database Design and Implementation

Now, we will access the MySQL container and launch the MySQL shell by executing the following command:
```{bash}
docker exec -it mysql mysql -u root -p
```

You will be prompted to enter the password for the root user. We have set the password to `root` in the `docker-compose.yml` file, so enter `root` as the password.

After entering the password, you will be logged into the MySQL shell. Now, we will create a new database named `sales` by executing the following command:
```{sql}
CREATE DATABASE sales;
```

We will create a data model consisting of a fact table called `aggregated_sales` and three dimension tables: `dim_platform`, `dim_product`, and `dim_category`. The entity relationship diagram (ERD) for the data model is shown below:
![Sales ERD](/_images/sdp_sales_erd.png)

The `aggregated_sales` table will store aggregated sales data with the following attributes: `batch_id`, `date`, `platform_id`, `product_id`, and `total_quantity`. The `batch_id` serves as the primary key for this table. It also has foreign keys `platform_id` and `product_id` that reference the primary keys of the `dim_platform` and `dim_product` tables, respectively. 

The `dim_platform` table represents the dimensions related to platforms and has two attributes: `platform_id` and `platform_name`. The `platform_id` serves as the primary key in this table.

The `dim_product` table represents the dimensions related to products and has attributes such as `product_id`, `product_name`, `cost_price`, and `category_id`. The `product_id` is the primary key, and it has a foreign key `category_id` that references the primary key of the `dim_category` table.

The `dim_category` table represents the dimensions related to the product categories and has attributes such as `category_id`, `category_name`, and `profit_margin`. The `category_id` is the primary key in this table.

With this data model, we can analyze and report on sales data by considering dimensions such as platform, product, and category. By establishing relationships between the fact and dimension tables, we can calculate profit and sales by considering the cost price and profit margin, and analyze the two metrics by doing aggregations accross different dimensions. This enable us to gain insights into various aspects of the business.

Below are the SQL statements to create the tables in the `sales` database:
```{sql}
USE sales;

CREATE TABLE dim_platform (
  platform_id INT PRIMARY KEY,
  platform_name VARCHAR(255)
  );

CREATE TABLE dim_category (
  category_id INT PRIMARY KEY,
  category_name VARCHAR(255),
  profit_margin DECIMAL(3, 2)
  );

CREATE TABLE dim_product (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(255),
  cost_price DECIMAL(10, 2),
  category_id INT,
  FOREIGN KEY (category_id) REFERENCES dim_category (category_id)
  );

CREATE TABLE aggregated_sales (
  batch_id INT AUTO_INCREMENT,
  date DATE,
  platform_id INT,
  product_id INT,
  total_quantity INT,
  PRIMARY KEY (batch_id),
  FOREIGN KEY (platform_id) REFERENCES dim_platform (platform_id),
  FOREIGN KEY (product_id) REFERENCES dim_product (product_id)
  );
```

Now we will populate the dimension tables with some data. Note that the data used in this project is hypothetical and solely for demonstration purposes.
```{sql}
INSERT INTO dim_platform (platform_id, platform_name)
  VALUES
    (1, 'MarketPlaceX'),
    (2, 'EasyShop'),
    (3, 'BuyNow'),
    (4, 'SwiftMart');

INSERT INTO dim_category (category_id, category_name, profit_margin)
  VALUES
    (1, 'Jewelry', 0.25),
    (2, 'Shoes', 0.20),
    (3, 'Clothing', 0.15),
    (4, 'Personal Care', 0.10);

INSERT INTO dim_product (product_id, product_name, cost_price, category_id)
  VALUES
    (1, 'Diamond Ring', 1000.00, 1),
    (2, 'Gold Necklace', 800.00, 1),
    (3, 'Silver Bracelet', 150.00, 1),
    (4, 'Leather Boots', 210.00, 2),
    (5, 'Sports Shoes', 130.00, 2),
    (6, 'High Heels', 80.00, 2),
    (7, 'T-Shirt', 25.00, 3),
    (8, 'Jeans', 50.00, 3),
    (9, 'Dress', 90.00, 3),
    (10, 'Shampoo', 10.00, 4),
    (11, 'Soap', 5.00, 4),
    (12, 'Moisturizer', 15.00, 4),
    (13, 'Gold Earrings', 220.00, 1),
    (14, 'Titanium Bracelet', 120.00, 1),
    (15, 'Hiking Boots', 140.00, 2),
    (16, 'Sandals', 40.00, 2),
    (17, 'Trousers', 70.00, 3),
    (18, 'Skirt', 40.00, 3),
    (19, 'Sunscreen', 20.00, 4),
    (20, 'Hand Lotion', 10.00, 4);
```

Note that the profit margin is represented as a decimal value relative to the cost price. For instance, consider a Diamond Ring with a cost price of $1000 and a profit margin of 0.25 (based on its category). This means that the profit is calculated as 0.25 times the cost price, resulting in $250. And the selling price of the Diamond Ring is determined by adding the cost price and profit together, which is $1250 in this case.

### Conclusion

In this article, we have demonstrated how to implement an OLTP database using Cassandra and an OLAP database using MySQL. By combining the capabilities of both the OLTP and OLAP databases, we can support transactional operations while enabling powerful analytical capabilities for decision-making purposes. In the next article (part 3), we will implement a data pipeline that facilitates the ingestion of real-time transactional data. We'll leverage the capabilities of Kafka and Spark Streaming to seamlessly load this raw data into our OLTP database while simultaneously aggregating and populating the OLAP database.
