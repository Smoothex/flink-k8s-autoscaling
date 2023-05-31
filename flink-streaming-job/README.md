
# Setup

## Start the Kafka cluster
After you have cloned the repository, open up a terminal and run the following command:
```
default_interface=$(ip ro sh | awk '/^default/{ print $5; exit }')
export DOCKER_HOST_IP=$(ip ad sh $default_interface | awk '/inet /{print $2}' | cut -d/ -f1)
[ -z "$DOCKER_HOST_IP" ] && (echo "No docker host ip address">&2; exit 1 )
echo "$DOCKER_HOST_IP"
```
This retrieves the IP address of the default interface on the Docker host. More on this [here](https://stackoverflow.com/questions/35418939/kafka-with-docker-dynamic-advertised-host-name).

Now the use the value of DOCKER_HOST_IP to modify some properties.

Navigate to the _docker-compose_ folder and edit line 14 of _docker-compose.yml_ with the value of DOCKER_HOST_IP:

```KAFKA_ADVERTISED_HOST_NAME: <DOCKER_HOST_IP>```

Navigate to the _producer_ folder and under _src/main/resources_, edit the _producer.properties_ file on line 6:

```kafka.brokerList=<DOCKER_HOST_IP>:9092```

Start the Kafka cluster by switching to the folder _docker-compose_ and running ```docker-compose up```. The Kafka UI should be available at localhost:8080
![image](https://github.com/Smoothex/flink-k8s-autoscaling/assets/79105432/51237546-d0ee-44e5-94fe-f44e0a2ded3d)


## Start the producer
Switch back to the root folder (in this case it's flink-streaming job, it is used for docker-compose, the other one handles the k8s deployment).

Run ```mvn clean && mvn package``` (this cleans the project build artifacts and dependencies, then compiles the source code, runs tests, and packages the project into an output artifact (e.g., JAR file) in the target directory).

Now, the folder target inside of the folder producer is created along with the needed .jar file.

Run ```java -jar producer/target/producer-1.0-SNAPSHOT.jar``` from the root directory. This should hopefully start the data generator, which will generate ad events to the Kafka topic specified in _producer.properties_. Then the Kafka UI should look like this:

![image](https://github.com/Smoothex/flink-k8s-autoscaling/assets/79105432/59166726-99e2-40bc-9406-9b12b3cf1cbd)

As you can see, the topic input has been created and the message count is growing.

The producer generates data based on the _advertising_3H_increased_load.csv_ file (also located in the _resources_ directory. The whole test lasts for about 3 hours, so you can just stop the data producer with Ctrl+C in the terminal.
