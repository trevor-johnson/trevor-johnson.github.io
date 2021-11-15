## Introduction

In this report, we will demonstrate how to build a pipeline to enable a data science team to obtain valuable insights on game events. 

The latest mobile game in a game development company has three events: join a guild, purchase a sword, and purchase a horse. To enable analytics for these events and the metadata of each of the events, a pipeline has been created to allow analysts to access the data. 

To start a virtual container is created leveraging docker.

Kafka is then leveraged for a singular topic called "events" has been created to track the events of join a guild, purchase a sword, and purchase a sword. A single topic is intentional to allow filtering. 

A flask app is created to simulate a web app that allows events to be created and stored. In the flask app, events can be triggered with the corresponding metadata and host information for analysis. Values are saved in a dictionary in a single key to value to allow simplicity in reporting and scale. Meaning, if additional values are added or existing values are replaced, the schema is easily adjustable.

Python script filters for each event have been created and submits separated event tables to HDFS depending on the event type. Error handling has also been added to ensure events are added. This approach allows the availability to quickly adjust each event. For new events, a separate python script can be created. For example, if metadata is needed for swords the write_sword_events.py can be edited. If a new event such as, purchase a spellbook is needed in the future, a write_spellbook_events.py should be created.

3 script files have been created to read messages from Kafka and save results in tables written in Hadoop as a stream, with a 10 second batch interval.

<details>
    <summary>write_sword_events.py</summary>

```py


#!/usr/bin/env python
"""Extract events from kafka and write them to hdfs
"""
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType


def purchase_sword_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    |-- color: string (nullable = true)
    |-- quantity: string (nullable = true)

    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
        StructField("color", StringType(), True),
        StructField("quantity", StringType(), True),
    ])


@udf('boolean')
def is_sword_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_sword':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .enableHiveSupport() \
        .getOrCreate()
    
    
    # Create a hive table
    spark.sql("""
        create external table if not exists default.sword_purchases (
            raw_event string,
            timestamp string,
            Accept string,
            Host string,
            User_Agent string,
            event_type string,
            color string,
            quantity string
          )
          stored as parquet 
          location '/tmp/sword_purchases'
    """)        

    raw_events = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .load()

    sword_purchases = raw_events \
        .filter(is_sword_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                     purchase_sword_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')
            
    sink = sword_purchases \
        .writeStream \
        .format("parquet") \
        .option("checkpointLocation", "/tmp/checkpoints_for_sword_purchases") \
        .option("path", "/tmp/sword_purchases") \
        .trigger(processingTime="10 seconds") \
        .start()
        
    sink.awaitTermination()

if __name__ == "__main__":
    main()


```

</details>

<details>
    <summary>write_horse_events.py</summary>

```py

#!/usr/bin/env python
"""Extract only horse purchasing events from kafka and write them to hdfs
"""

# libraries
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType

# set up schema for the horse purchase events
def purchase_horse_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    |-- speed: string (nullable = true)
    |-- size: string (nullable = true)
    |-- quantity: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
        StructField("speed", StringType(), True),
        StructField("size", StringType(), True),
        StructField("quantity", StringType(), True),
    ])


@udf('boolean')
def is_horse_purchase(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'purchase_horse':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .enableHiveSupport() \
        .getOrCreate()
    
    # Create a hive table
    spark.sql("""
        create external table if not exists default.horse_purchases (
            raw_event string,
            timestamp string,
            Accept string,
            Host string,
            User_Agent string,
            event_type string,
            speed string,
            size string,
            quantity string
          )
          stored as parquet 
          location '/tmp/horse_purchases'
    """)      

    raw_events = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .load()

    horse_purchases = raw_events \
        .filter(is_horse_purchase(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          purchase_horse_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')
    
    sink = horse_purchases \
        .writeStream \
        .format("parquet") \
        .option("checkpointLocation", "/tmp/checkpoints_for_horse_purchases") \
        .option("path", "/tmp/horse_purchases") \
        .trigger(processingTime="10 seconds") \
        .start()  
        
    sink.awaitTermination()


if __name__ == "__main__":
    main()

```

</details>

<details>
    <summary>write_guild_events.py</summary>

```py


#!/usr/bin/env python
"""Extract only guild events from kafka and write them to hdfs
"""

# libraries
import json
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import udf, from_json
from pyspark.sql.types import StructType, StructField, StringType

# set up schema for the guild events
def guild_event_schema():
    """
    root
    |-- Accept: string (nullable = true)
    |-- Host: string (nullable = true)
    |-- User-Agent: string (nullable = true)
    |-- event_type: string (nullable = true)
    |-- timestamp: string (nullable = true)
    |-- action: string (nullable = true)
    """
    return StructType([
        StructField("Accept", StringType(), True),
        StructField("Host", StringType(), True),
        StructField("User-Agent", StringType(), True),
        StructField("event_type", StringType(), True),
        StructField("action", StringType(), True)
    ])


@udf('boolean')
def is_guild(event_as_json):
    """udf for filtering events
    """
    event = json.loads(event_as_json)
    if event['event_type'] == 'guild':
        return True
    return False


def main():
    """main
    """
    spark = SparkSession \
        .builder \
        .appName("ExtractEventsJob") \
        .enableHiveSupport() \
        .getOrCreate()

    # Create a hive table
    spark.sql("""
        create external table if not exists default.guild_actions (
            raw_event string,
            timestamp string,
            Accept string,
            Host string,
            User_Agent string,
            event_type string,
            action string
          )
          stored as parquet 
          location '/tmp/guild_actions'
    """)   
    
    raw_events = spark \
        .readStream \
        .format("kafka") \
        .option("kafka.bootstrap.servers", "kafka:29092") \
        .option("subscribe", "events") \
        .load()

    guild_actions = raw_events \
        .filter(is_guild(raw_events.value.cast('string'))) \
        .select(raw_events.value.cast('string').alias('raw_event'),
                raw_events.timestamp.cast('string'),
                from_json(raw_events.value.cast('string'),
                          guild_event_schema()).alias('json')) \
        .select('raw_event', 'timestamp', 'json.*')
    
    sink = guild_actions \
        .writeStream \
        .format("parquet") \
        .option("checkpointLocation", "/tmp/checkpoints_for_guild_actions") \
        .option("path", "/tmp/guild_actions") \
        .trigger(processingTime="10 seconds") \
        .start()  
        
    sink.awaitTermination()
    

if __name__ == "__main__":
    main()


```

</details>

<br>

Metadata to each event:

* purchase_a_sword: accepts color and quantity parameters 
* purchase_a_horse: accepts speed, size, and quantity parameters 
* guild: accepts inputs to join or leave the guild 
* default_response: this does not accept any parameters and is run when the user runs a blank input 

Metadata for the users when an event is triggered outside of events and user agent:

* timestamp: information allowing analysis on when the event occurs. Use this value for understanding state changes over time.
* Host: information for allowing analysis on the User Id.

After each script is ran, the events are successfully landed into HDFS.
```bash
docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
docker-compose exec cloudera hadoop fs -ls /tmp/horse_purchases
docker-compose exec cloudera hadoop fs -ls /tmp/guild_actions
```
External tables are then created to save data into the Hive metastore.

* sword_purchases 
* horse_purchases 
* guild_actions 

Data is then queried through a tool. In this example, presto is used for queries. 

Data is then saved in parquet for querying to give data scientists the ability to answer business questions. 

**Creating Testing Data:**

For unit testing, apache bench was leveraged to send batch records to create mock data into parquet and enable testing of reporting. Scenarios of testing data was coded into apache bench, this approach ensures if 10 values were created we can expect the 10 values in the reporting/analysis. A streaming script has been created called generate_data.sh file.

## Setting Up the Pipeline
Firstly, we will showcase how we built the pipeline. We have chosen to use Kafka, Hadoop and Spark to transport, transform and store the game events. We have defined the configuration for each of these in the **docker-compose.yml** file, below is a description of each part of this file.

### Zookeeper & Kafka
Zookeeper allows us to easily access and manage the Kafka instance. Kafka allows us to easily create a pipeline that we can publish game events to, which can then be consumed by Spark.  

We have opened ports in Zookeeper which are referenced in the Kafka configuration. This allows us to create topics in Kafka via Zookeeper. We have also exposed ports in Kafka that allow us to publish and consume messages from the topics.

```bash
zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000
    expose:
      - "2181"
      - "2888"
      - "32181"
      - "3888"
    extra_hosts:
      - "moby:127.0.0.1"

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    expose:
      - "9092"
      - "29092"
    extra_hosts:
      - "moby:127.0.0.1"
```

### Hadoop
We have added Cloudera configuration to be able to use Hadoop with additional ports exposed to be able to connect to Hive.

```bash
  cloudera:
    image: midsw205/hadoop:0.0.2
    expose:
      - "8020" # nn
      - "8888" # hue
      - "9083" # hive thrift
      - "10000" # hive jdbc
      - "50070" # nn http
    ports:
      - "8888:8888"      
    extra_hosts:
      - "moby:127.0.0.1"
```

### Spark
We have set up Spark using the MIDS W205 Spark Python base image. We're specifying the dependency of this service on Cloudera and the Hadoop name node that Spark will use when writing to HDFS.    
In addition to this, we have exposed ports which allow us to connect to Spark from a notebook.

```bash
  spark:
    image: midsw205/spark-python:0.0.6
    stdin_open: true
    tty: true   
    volumes:
      - "~/w205:/w205"
    command: bash
    depends_on:
      - cloudera
    environment:
      HADOOP_NAMENODE: cloudera
      HIVE_THRIFTSERVER: cloudera:9083
    expose:
      - "8888"
      - "7000" #jupyter notebook      
    ports:
      - "7000:7000" # map instance:service port   
    extra_hosts:
      - "moby:127.0.0.1"
```

### Presto
We have added Presto configution as an alternative tool for data scientists to query the data.

```bash
  presto:
    image: midsw205/presto:0.0.1
    hostname: presto
    volumes:
      - "~/w205:/w205"
    expose:
      - "8080"
    environment:
      HIVE_THRIFTSERVER: cloudera:9083
    extra_hosts:
      - "moby:127.0.0.1"  
```

### MIDS Base Image
This is the base image that we will use in the container, it allows us to run bash commands, as well as using kafkacat to publish messages to Kafka. We're specifying the w205 volume so that we have access to files.  

```bash
  mids:
    image: midsw205/base:latest
    stdin_open: true
    tty: true
    expose:
      - "5000"
    ports:
      - "5000:5000"
    volumes:
      - "~/w205:/w205"
    extra_hosts:
      - "moby:127.0.0.1"
```

## Bring Up the Pipeline
To do this we run the following command:

```bash
docker-compose up -d
```

Then we create a events Kafka topic which we will publish all the game events to:
```bash
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```

## Interacting With the Pipeline

### Flask App

Our Flask app is built in the game_api.py file. The app is designed to connect with Kafka, so that any events generated by a user are captured in our Kafka topic called "events". The app consists of four different functions: 

1. `purchase_a_sword`: accepts color and quantity parameters 
2. `purchase_a_horse`: accepts speed, size, and quantity parameters
3. `guild`: accepts inputs to join or leave the guild
4. `default_response`: this does not accept any parameters and is run when the user runs a blank input

Each function is structured as a post API request. As an example, see the code below for the purchase_a_horse function. The function accepts parameters in the API call and are captured in a dictionary. Next, we wrote error handling code so the function only accepts valid user inputs. Finally, the dictionary is written to our Kafka topic, "events", which captures the user inputs from any of our four functions.

```python
@app.route("/purchase_a_horse/<speed>/<size>/<quantity>", methods=["POST"])
def purchase_a_horse(speed, size, quantity):
    """
    Inputs:
    - speed
    - size (small, medium, or large)
    - quantity
    """
    
    # collect user inputs
    purchase_horse_event = {
        "event_type": "purchase_horse", 
        "speed": speed, 
        "size": size, 
        "quantity": quantity}
    
    # error handling
    if purchase_horse_event['size'].lower() not in ['small', 'medium', 'large']:
        raise Exception("Please enter either 'small', 'medium' or 'large' for horse size")
        
    elif float(purchase_horse_event['speed']) < 0:
        raise Exception("Please enter a non-negative value for speed")
        
    elif float(purchase_horse_event['quantity']) < 0:
        raise Exception("Please enter a non-negative value for quantity")
        
    else:
        # clean inputs to collect only lower case values for consistency
        purchase_horse_event['size'] = purchase_horse_event['size'].lower()
        
        # log event to kafka
        log_to_kafka("events", purchase_horse_event)
        return "Horse Purchased!\n"
```


### Extracting Events

After the users have used the app and generated events stored in the Kafka topic, we utilized Pyspark to extract the events and land them in HDFS. We wrote three separate Python scripts to individually extract data for each event type (sword, horse, and guild). Each script consists of four main steps:   
1. Creating a Hive table 
2. Read the events from the Kafka topic as a stream
3. Filter them to extract events of a specific type
4. Write them to HDFS tables as a stream in 10 second batch intervals.   

We have three separate HDFS tables with their own unique schema that are specific to the event type.

As an example, the code below shows how we use Spark streaming to filter the horse events and write them to HDFS in parquet format. 

```python
horse_purchases = raw_events \
    .filter(is_horse_purchase(raw_events.value.cast('string'))) \
    .select(raw_events.value.cast('string').alias('raw_event'),
            raw_events.timestamp.cast('string'),
            from_json(raw_events.value.cast('string'),
                      purchase_horse_event_schema()).alias('json')) \
    .select('raw_event', 'timestamp', 'json.*')

sink = horse_purchases \
    .writeStream \
    .format("parquet") \
    .option("checkpointLocation", "/tmp/checkpoints_for_horse_purchases") \
    .option("path", "/tmp/horse_purchases") \
    .trigger(processingTime="10 seconds") \
    .start()  
        
sink.awaitTermination()
```

The following command will run the flask application:
```bash
docker-compose exec mids env FLASK_APP=/w205/w205-project3-sandbox/game_api.py flask run --host 0.0.0.0
```

### Starting the Stream
Running the following three commands will set up a stream for each of the event types. As new events come in, they are automatically read from Kafka, written to HDFS and available in Hive to be queried using Presto.

```bash
docker-compose exec spark spark-submit /w205/w205-project3-sandbox/write_sword_events.py
docker-compose exec spark spark-submit /w205/w205-project3-sandbox/write_horse_events.py
docker-compose exec spark spark-submit /w205/w205-project3-sandbox/write_guild_events.py

```

## Generating Data
Now that the Spark streaming scripts are waiting for data to be generated, we can generate events using Apache bench.

Use the `generate_data.sh` file to send standardized events in bulk via Apache Bench: 

```bash
sh generate_data.sh
```

<details>
    <summary>generate_data.sh</summary>

```bash
# Sword purchasing data. 
docker-compose exec mids ab -n 100 -m POST -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword/red/2

docker-compose exec mids ab -n 100 -m POST -H "Host: user2.comcast.com" http://localhost:5000/purchase_a_sword/red/3

docker-compose exec mids ab -n 500 -m POST -H "Host: user3.comcast.com" http://localhost:5000/purchase_a_sword/red/1

docker-compose exec mids ab -n 200 -m POST -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword/blue/1

docker-compose exec mids ab -n 200 -m POST -H "Host: user2.comcast.com" http://localhost:5000/purchase_a_sword/blue/2

docker-compose exec mids ab -n 10 -m POST -H "Host: user3.comcast.com" http://localhost:5000/purchase_a_sword/blue/3

docker-compose exec mids ab -n 300 -m POST -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword/green/1

docker-compose exec mids ab -n 100 -m POST -H "Host: user2.comcast.com" http://localhost:5000/purchase_a_sword/green/1

docker-compose exec mids ab -n 100 -m POST -H "Host: user3.comcast.com" http://localhost:5000/purchase_a_sword/green/1


# Horse Purchasing Data
docker-compose exec mids ab -n 100 -m POST -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_horse/1/small/1

docker-compose exec mids ab -n 500 -m POST -H "Host: user2.comcast.com" http://localhost:5000/purchase_a_horse/1/small/1

docker-compose exec mids ab -n 200 -m POST -H "Host: user3.comcast.com" http://localhost:5000/purchase_a_horse/1/small/1

docker-compose exec mids ab -n 200 -m POST -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_horse/1/medium/2

docker-compose exec mids ab -n 400 -m POST -H "Host: user2.comcast.com" http://localhost:5000/purchase_a_horse/1/medium/1

docker-compose exec mids ab -n 300 -m POST -H "Host: user3.comcast.com" http://localhost:5000/purchase_a_horse/1/medium/1

docker-compose exec mids ab -n 50 -m POST -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_horse/1/large/1

docker-compose exec mids ab -n 50 -m POST -H "Host: user2.comcast.com" http://localhost:5000/purchase_a_horse/1/large/1

docker-compose exec mids ab -n 500 -m POST -H "Host: user3.comcast.com" http://localhost:5000/purchase_a_horse/1/large/1


# Guild Activity Data
docker-compose exec mids ab -n 200 -m POST -H "Host: user1.comcast.com" http://localhost:5000/guild/join

docker-compose exec mids ab -n 100 -m POST -H "Host: user1.comcast.com" http://localhost:5000/guild/leave

```
</details>

<br>

## Analyzing the Events

Now that we have event data generated and it has successfully gone through the pipeline, we can use both Presto and PySpark to query it to help answer business questions.  

### Presto
Open Presto using the following command:
```bash
docker-compose exec presto presto --server presto:8080 --catalog hive --schema default
```

Check to see what tables are available in presto: 
```bash
presto:default> show tables;
```

Output:
```bash
     Table     
---------------
 guild_actions   
 horse_purchases 
 sword_purchases
(3 rows)
```

#### Example Queries 
Showing some useful fields in our swords table
```sql
SELECT color, quantity FROM sword_purchases LIMIT 5;
```

Output
```bash
 color | quantity 
-------+----------
 red   | 2        
 red   | 2        
 red   | 2        
 red   | 2        
 red   | 2        
(5 rows)
```

How many horses have been purchased by size? 
```sql
SELECT
  size AS horse_size,
  SUM(CAST(quantity as INTEGER)) AS num_horses
FROM horse_purchases
GROUP BY 1;
```

Output:
```bash
 horse_size | num_horses 
------------+------------
 small      |        800 
 medium     |       1100 
 large      |        600 
```

### PySpark
As an alternative to Presto, we can also use PySpark to run queries directly in a Jupyter notebook.
```py
sword_purchases = spark.read.parquet('/tmp/sword_purchases')
sword_purchases.registerTempTable('sword_purchases')
```
# How many sword were purchased in the last year?

```py
spark.sql('SELECT \
    year, \
    sum(num_swords_purchaed) as num_swords_purchaed \
    FROM( \
        SELECT \
            YEAR(timestamp) as year, \
            CAST(quantity AS INTEGER) AS num_swords_purchaed \
        FROM sword_purchases \
    ) \
GROUP BY year').show()
```
```
+----+-------------------+
|year|num_swords_purchaed|
+----+-------------------+
|2021|               2160|
+----+-------------------+

```