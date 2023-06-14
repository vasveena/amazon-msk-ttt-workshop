# **Client Migration and Cutover**

We will perform the following steps in this section: 

1. Consumer Migration
* Terminate consumer reading from source cluster.
* Start consumer reading from destination MSK cluster.

2. Producer Migration
* Terminate producer sending to source cluster.
* Start producer sending to destination MSK cluster.

3. Stop MM2 and Kafka Connect

## **Consumer Migration**

Again, we will continue where we last left off and assume you are still connected to KafkaClientInstance1. (The linux prompt should include the instance name.)

1. Terminate the Consumer

* We need the **pid** of the Consumer. If you don't remember it, you can issue the following command to obtain the **pid**:

```
jps -l | grep Consumer
```

* And then, stop the Consumer with the following command:
```
kill <pid>
```
2. Tail the consumer log file to check if it exited properly and note down the last processed Global Seq no and the last offset. Ctrl-C to exit from the tail command.
```
tail -100 /tmp/consumer.log | grep Global
tail -100 /tmp/consumer.log | grep "Last Offsets"

```
3. Update Consumer connection configuration to use Destination cluster. 

```
cd /tmp/kafka
sed -i -e "s/BOOTSTRAP_SERVERS_CONFIG=/BOOTSTRAP_SERVERS_CONFIG=$brokersmskdest/g" consumer.properties_dest

```

4. Backup log and Run the Consumer against the destination Amazon MSK cluster.

``` 
cp /tmp/consumer.log /tmp/consumer.log_src
java $EXTRA_ARGS -jar KafkaClickstreamConsumer-1.0-SNAPSHOT.jar -t ExampleTopic -pfp /tmp/kafka/consumer.properties_dest -nt 3 -rf 10800 -src msksource -iam -gsr -gsrr $region  > /tmp/consumer.log 2>&1 &

```

5. Check the consumer log to make sure it is consuming messages.

```
tail -f /tmp/consumer.log

```
6. Check to make sure that the consumer started consuming from the last offset synced by the **MirrorCheckpointConnector**.

```
grep 'Consumer position:' /tmp/consumer.log
```

*At this point the consumer is migrated to the destination Amazon MSK cluster.*

## **Producer Migration**

1. Terminate the Producer

Again, we need the **pid** that you had recorded earlier when starting the producer. If you don't remember it, run the following command:
``` 
jps -l | grep KafkaClickstreamClient

```
and then:

``` 
kill <pid>

```

2. Tail the producer log file to check if it exited properly. When exiting, the producer records the last **Global Seq No**. Ctrl-C to exit.

``` 
tail -f /tmp/producer.log

```
3. Update Producer configuration

``` 
sed -i -e "s/BOOTSTRAP_SERVERS_CONFIG=/BOOTSTRAP_SERVERS_CONFIG=$brokersmskdest/g" producer.properties_msk_dest
export EXTRA_ARGS=-javaagent:/home/ec2-user/prometheus/jmx_prometheus_javaagent-0.13.0.jar=3800:/home/ec2-user/prometheus/kafka-producer-consumer.yml

```

4. Now, run the Producer against the destination Amazon MSK cluster

``` 
mv /tmp/producer.log /tmp/producer.log_src
java $EXTRA_ARGS -jar KafkaClickstreamClient-1.0-SNAPSHOT.jar -t ExampleTopic -pfp /tmp/kafka/producer.properties_msk_dest -nt 8 -rf 10800 -nle -gsr -iam -gsrr $region -gar -gcs $schema_compatibility > /tmp/producer.log 2>&1 &

```

5. Tail the producer log file. Ctrl-C to exit.

``` 
tail -f /tmp/producer.log

```

6. Run the following command to check if the consumer picked up the new partitions for the newly created ExampleTopic in the destination Amazon MSK cluster.

``` 
grep "Consumer assignment" /tmp/consumer.log

```
* Check the consumer.log file to see if the consumer is reading messages from the newly created ExampleTopic in the destination Amazon MSK cluster.

``` 
tail -f /tmp/consumer.log

```

*At this point the producer is migrated to the destination Amazon MSK cluster.*

## **Stop MirrorMaker 2 and Kafka Connect**

At this point, we have successfully migrated from our Source to Destination cluster, and we can stop Kafka Connect (and this will stop the MirrorMaker 2 connectors).

* Run the following commands:

``` 
sudo systemctl stop kafka-connect
sudo systemctl status kafka-connect

```