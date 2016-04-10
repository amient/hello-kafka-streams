This is an equivalent of [hello-samza](https://samza.apache.org/startup/hello-samza/0.10/) project which transforms wikipedia irc channel events into wikipdia-raw topic and does some basic stateful aggregation of stats over this record stream. It uses only Apache Kafka components, namely Kafka Connect and Kafka Streams. 

You can fina a walk-through tutorial for this demo and more backtround about the APIs used here in [this  post](http://www.confluent.io/blog/hello-world-kafka-connect-kafka-streams) at Confluent's blog. 
 
# Quick Start

## Prerequisites

The demo uses Java 1.8 features so you'll need the correct jdk to run it.

Some of the features of Kafka used in this demo are part of the upcoming 0.10.x release. If you're using Confluent Platform
you can use the tech preview version described [here](http://www.confluent.io/developer#streamspreview) and change
the `kafkaVersion` in the `gradle.properties` appropriately. I have used a local installation of Apache Kafka trunk, 
which at the time of writing this demo corresponded to version 0.10.1.0-SNAPSHOT, compiled with scala version 2.11.

If you're already running a local Zookeeper and Kafka and you have topic auto-create enabled on the broker you can 
skip the following setup, just note that if your default partitions number is 1 you will only be able to run a single instance
demo.
 
    $ cd $KAFKA_HOME
    $ export SCALA_VERSION="2.11.8"; export SCALA_BINARY_VERSION="2.11";
    $ ./bin/zookeeper-server-start.sh ./config/zookeeper.properties & ./bin/kafka-server-start.sh ./config/server.properties

    $ ./bin/kafka-topics.sh --zookeeper localhost --create --topic wikipedia-raw --replication-factor 1 --partitions 4
    $ ./bin/kafka-topics.sh --zookeeper localhost --create --topic wikipedia-parsed --replication-factor 1 --partitions 4
    $ ./bin/kafka-topics.sh --zookeeper localhost --create --topic wikipedia-streams-wikipedia-edits-by-user-changelog \
                            --replication-factor 1 --partitions 4

## Build the executable demo app

In another terminal `cd` into the directory where you have cloned the `hello-kafka-streams` project and use provided
gradle wrapper to create executable jar.

    $ ./gradlew build

Use the following command to start an instance of the integrated demo topology.

    ./build/scripts/hello-kafka-streams

If you have created topics more than 1 partition you can run the command above again in another terminal 
to see how the re-balance mechanism will scale both the embedded connect workers as well as the streams processors.

You should see changelog for the each user's cummulative number of edits being printed into the standard out. 


## Digging in

For inspecting the underlying intermediate topics, in yet another terminal `cd` into kafka home directory 
and run some console consumer commands:

    $ cd $KAFKA_HOME
    $ ./bin/kafka-console-consumer.sh --zookeeper localhost --topic wikipedia-raw
    $ ./bin/kafka-console-consumer.sh --zookeeper localhost --topic wikipedia-parsed

See [WikipediaStreamDemo.java](src/main/java/io/amient/examples/wikipedia/WikipediaStreamDemo.java) for more details about the actual Topology.
