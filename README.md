# Lambda architecture

![Alt text](diagram.png?raw=true "Lambda architecture")

![Alt text](dashboard.png?raw=true "Dashboard")

Read about the project [here](https://dzone.com/articles/lambda-architecture-how-to-build-a-big-data-pipeli)

Watch the videos demonstrating the project <a href="https://dzone.com/articles/lambda-architecturehow-to-build-a-big-data-pipelin" target="_blank">here</a>

Our Lambda project receives real-time IoT Data Events coming from Connected Vehicles, 
then ingested to Spark through Kafka. Using the Spark streaming API, we processed and analysed 
IoT data events and transformed them into vehicle information.
While simultaneously the data is also stored into HDFS for Batch processing. 
We performed a series of stateless and stateful transformation using Spark streaming API on 
streams and persisted them to Cassandra database tables. In order to get accurate views, 
we also perform a batch processing and generating a batch view into Cassandra.
We developed responsive web traffic monitoring dashboard using Spring Boot, 
SockJs and Bootstrap which get the views from the Cassandra database and push to the UI using web socket.


All component parts are dynamically managed using Docker, which means you don't need to worry 
about setting up your local environment, the only thing you need is to have Docker installed.

System stack:
- Java 11
- Maven
- ZooKeeper
- Kafka
- Cassandra
- Spark 3
- Docker
- HDFS


The streaming part of the project was done from iot-traffic-project [InfoQ](https://www.infoq.com/articles/traffic-data-monitoring-iot-kafka-and-spark-streaming)

<a href="https://www.buymeacoffee.com/apssouza"><img src="https://miro.medium.com/max/654/1*rQv8JgstmK0juxP-Kb4IGg.jpeg"></a>

## How to use
* `mvn package`
* `docker-compose -p lambda up`
* Wait all services be up and running, then...
* `./project-orchestrate.sh`
* Run realtime job `docker exec spark-master /spark/bin/spark-submit --class com.apssouza.iot.streaming.StreamingProcessor  --master spark://localhost:7077 /opt/spark-data/iot-spark-processor-1.0.0.jar`
* Access the Spark cluster http://localhost:8080
* Run the traffic producer `java -jar iot-kafka-producer/target/iot-kafka-producer-1.0.0.jar`
* Run the service layer (Web app) `java -jar iot-springboot-dashboard/target/iot-springboot-dashboard-1.0.0.jar` 
* Access the dashboard with the data http://localhost:3000/
* Run the batch job `docker exec spark-master /spark/bin/spark-submit --class com.apssouza.iot.batch.BatchProcessor  --master spark://localhost:7077 /opt/spark-data/iot-spark-processor-1.0.0.jar`
* Run the ML job `docker exec spark-master /spark/bin/spark-submit --class com.apssouza.iot.ml.SpeedPrediction  --master spark://localhost:7077 /opt/spark-data/iot-spark-processor-1.0.0.jar`

### Miscellaneous

#### Spark

#### Submit a job to master
- `mvn package`
- `spark-submit --class com.apssouza.iot.streaming.StreamingProcessor --master spark://spark-master:7077 iot-spark-processor/target/iot-spark-processor-1.0.0.jar`
Add `spark-master` to /etc/hosts pointing to localhost

#### GUI
http://localhost:8080 Master
http://localhost:8081 Slave


### HDFS

Commands https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/1/

Open a file - http://localhost:50070/webhdfs/v1/path/to/file/file.csv?op=open

Web file handle - https://hadoop.apache.org/docs/r1.0.4/webhdfs.html

#### Commands :
* `hdfs dfs -mkdir /user`
* `hdfs dfs -mkdir /user/lambda`
* `hdfs dfs -put localhost.csv /user/lambda/`
* Access the file http://localhost:9870/webhdfs/v1/user/lambda/localhost.csv?op=OPEN&namenoderpcaddress=namenode:8020&offset=0

#### Gui
http://localhost:9870
http://localhost:50075


### Kafka
* kafka-topics --create --topic iot-data-event --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:2181
* kafka-console-producer --request-required-acks 1 --broker-list kafka:9092 --topic iot-data-event
* kafka-console-consumer --bootstrap-server kafka:9092 --topic iot-data-event
* kafka-topics --list --zookeeper zookeeper:2181


### Cassandra
- Log in `docker exec -it cassandra-iot cqlsh --username cassandra --password cassandra`
- Access the keyspace `use TrafficKeySpace;`
- List data `SELECT * FROM TrafficKeySpace.Total_Traffic;`

## Please consider leaving a star if this project has helped you. 

