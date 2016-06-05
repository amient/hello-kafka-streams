This is an equivalent of [hello-samza](https://samza.apache.org/startup/hello-samza/0.10/) project which transforms wikipedia irc channel events into wikipdia-raw topic and does some basic stateful aggregation of stats over this record stream. It uses only Apache Kafka components, namely Kafka Connect and Kafka Streams. 

You can find a walk-through tutorial for this demo and more background about the APIs used in [this  post](http://www.confluent.io/blog/hello-world-kafka-connect-kafka-streams) at Confluent's blog. 
 
# Quick Start

## Prerequisites

The demo uses Java 1.8 features so you'll need the correct jdk to run it.

Some of the features of Kafka used in this demo are only available since the 0.10.x release.

If you're using Confluent Platform Alpha1 tech.preview you need to switch to the [cp branch of this demo](https://github.com/amient/hello-kafka-streams/tree/cp).

## Setup local environment 

The master branch of this demo uses 0.10.x features of Apache Kafka so all you need to do is clone and install kafka 
0.10.0.0 into your local maven (you'll need to have `gradle` command already installed):
 
    $ git clone https://github.com/apache/kafka.git $KAFKA_HOME
    $ cd $KAFKA_HOME
    $ git checkout 0.10.0.0
    $ gradle
    $ ./gradlew -PscalaVersion=2.11 jar 
    $
    $ # IF YOU ARE USING WINDOWS, USE `.bat` IN PLACE OF `.sh` FOR THE LAUNCH SCRIPTS BELLOW:
    $ export SCALA_VERSION="2.11.8"; export SCALA_BINARY_VERSION="2.11";
    $ ./bin/zookeeper-server-start.sh ./config/zookeeper.properties &
    $ ./bin/kafka-server-start.sh ./config/server.properties

Initialize topics: if you're already running a local Zookeeper and Kafka and you have topic auto-create enabled on the 
broker you can skip the following setup, just note that if your default partitions number is 1 you will only be able 
to run a single instance demo.

    $ # IF YOU ARE USING WINDOWS, USE `.bat` IN PLACE OF `.sh` FOR THE LAUNCH SCRIPTS BELLOW:
    $ ./bin/kafka-topics.sh --zookeeper localhost --create --topic wikipedia-raw --replication-factor 1 --partitions 4
    $ ./bin/kafka-topics.sh --zookeeper localhost --create --topic wikipedia-parsed --replication-factor 1 --partitions 4
    $ ./bin/kafka-topics.sh --zookeeper localhost --create --topic wikipedia-streams-wikipedia-edits-by-user-changelog \
                            --replication-factor 1 --partitions 4

## Build and start the executable demo app

In another terminal `cd` into the directory where you have cloned the `hello-kafka-streams` project and use provided
gradle wrapper to create executable jar.

    $ ./gradlew build

Use the following command to start an instance of the integrated demo topology.

    ./build/scripts/hello-kafka-streams          # hello-kafka-streams.bat IF YOU ARE USING WINDOWS

If you have created topics more than 1 partition you can run the command above again in another terminal 
to see how the re-balance mechanism will scale both the embedded connect workers as well as the streams processors.

You should see changelog for the each user's cummulative number of edits being printed into the standard out. 


## Digging in

For inspecting the underlying intermediate topics, in yet another terminal `cd` into kafka home directory 
and run some console consumer commands:

    $ cd $KAFKA_HOME
    $ # IF YOU ARE USING WINDOWS, USE `.bat` IN PLACE OF `.sh` FOR THE LAUNCH SCRIPTS BELLOW:
    $ ./bin/kafka-console-consumer.sh --zookeeper localhost --topic wikipedia-raw
    $ ./bin/kafka-console-consumer.sh --zookeeper localhost --topic wikipedia-parsed

See [WikipediaStreamDemo.java](src/main/java/io/amient/examples/wikipedia/WikipediaStreamDemo.java) for more details about the actual Topology.
