# HIVE-PRESTODB TUTORIAL
The idea of this repo is to provide some simple step by step guide to set up an isolated test/dev version Hive & PrestoDB locally and start playing around with it :-).
This set up is totally NOT recommended for production workloads.

Note: As I mentioned in [References](#References) this work is based in the containers build by [Big Data Europe](https://github.com/big-data-europe)

## Set up Hive & PrestoDB locally (isolated)
Because our purpose is just to experiment, to accelerate the set up we could use docker images (to be accurate, docker-compose).
First of all we need to clone this repo:
```
git@github.com:jordicenzano/hive-presto-tutorial.git
cd hive-presto-tutorial
```

The first step is to pull the docker images and start the cluster based on those images.
To do that we'll execute the following command **from the root of this project**(*):
```
docker-compose up 
```
(*) We are assuming that you have docker installed and configured, if not take a look to this [guide](https://docs.docker.com/install/overview/)

## Getting some data to analyze
In our example we'll be analyzing the following file [temp-data.csv](temp-data.csv), here you can see a sample:
```
2018-01-01T01:00:00,52
2018-01-01T02:00:00,50
2018-01-01T03:00:00,48
2018-01-01T04:00:00,48
2018-01-01T05:00:00,45
...
```
That file contains the temperature measured every hour from a weather station for all 2018.

## Analyzing data
The first step to use is to create the table with the data to analyze, we are going to use Hive for that, execute the following command to open shell into a cluster machine:
```
docker-compose exec hive-server bash
```
Copy the file to HDFS:
```
# hdfs dfs -mkdir /input
# hdfs dfs -put /host_shared/data/temp-data.csv /input/temp-data.csv
```
Connect the JDBC driver:
```
# /opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
```
Finally create the table from the data in the file:
```
> CREATE EXTERNAL TABLE horlytemp(time STRING, temp INT) COMMENT 'temperature from csv file' ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE location '/input';
```
We can check if the data is accessible doing:
```
> select * from horlytemp;
```
We should see the list of temperatures.

### Querying in presto
Now lets execute a presto client:
```
cd presto_client
./presto.jar --server localhost:8080 --catalog hive --schema default
```
Finally from that shell we can query that table, for instance:
```
# select * from horlytemp where temp > 80;
```
We should see something like: 
```
       time         | temp
---------------------+------
 2018-05-15T17:00:00 |   81
 2018-06-17T10:00:00 |   81
 2018-06-17T11:00:00 |   84
 2018-06-17T12:00:00 |   86
 2018-06-17T13:00:00 |   86
```

### Clean up
Stop the docker containers by doing:
```
docker-compose down
```

## References
- Hive-presto docker compose: https://github.com/big-data-europe/docker-hive
- Hadoop docker compose: https://github.com/big-data-europe/docker-hadoop