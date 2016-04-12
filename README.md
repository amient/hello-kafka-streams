This is an equivalent of [hello-samza](https://samza.apache.org/startup/hello-samza/0.10/) project which transforms wikipedia irc channel events into wikipdia-raw topic and does some basic stateful aggregation of stats over this record stream. It uses only Apache Kafka components, namely Kafka Connect and Kafka Streams. 

You can find a walk-through tutorial for this demo and more background about the APIs used in [this  post](http://www.confluent.io/blog/hello-world-kafka-connect-kafka-streams) at Confluent's blog. 
 
# Quick Start

## Prerequisites

The demo uses Java 1.8 features so you'll need the correct jdk to run it.

Some of the features of Kafka used in this demo are part of the upcoming 0.10.x release, the [master branch of this demo](https://github.com/amient/hello-kafka-streams)
is the most up-to-date with this forthcoming release.

If you're using Confluent Platform Alpha1 tech.preview you need to switch to the [cp branch of this demo](https://github.com/amient/hello-kafka-streams/tree/cp).

## Setup local environment 

Because confluent platform uses a mixture of Kafka 0.9.x and 0.10.x APIs, the code and dependencies differ slightly but
this is only due to "tech.preview" nature of their alpha1 release. The `gradle.properties` configuration on this branch 
assumes you have Confluent Platform version 2.1.0-alpha1 installed in `/opt/confluent-2.1.0-alpha1`, change the `flatDirs` 
repository property if otherwise.

    $ cd $CONFLUENT_PLATFORM_HOME
    $ ./bin/zookeeper-server-start ./etc/kafka/zookeeper.properties & 
    $ ./bin/kafka-server-start ./etc/kafka/server.properties

Initialize topics:

    $ ./bin/kafka-topics --zookeeper localhost --create --topic wikipedia-raw --replication-factor 1 --partitions 4
    $ ./bin/kafka-topics --zookeeper localhost --create --topic wikipedia-parsed --replication-factor 1 --partitions 4
    $ ./bin/kafka-topics --zookeeper localhost --create --topic wikipedia-streams-wikipedia-edits-by-user-changelog \
                            --replication-factor 1 --partitions 4


## Build and start the executable demo app

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
