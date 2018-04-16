# Part 1 - Creating a Docker Container for Kafka

## About the Serie
This is a series of 3 articles on **Docker**, **Microservices** and **BigData**, I will post one article per week, every monday, started at 15-Apr-2018.
In this small series I will demonstrate the use of the docker with bigdata constructing a feeder of cryptocoins quotations.

## About this Article
In this first article we will create a container docker with Kafka, for those who do not know Kafka it is a kind of queue manager (to simplify the analogy).
which works with data streams, being a fundamental piece for bigdata working on the data input of the fast data layer, and basically has the following capabilities:

## Getting to know Kafka

Capabilities of Kafka:
- Publisher and Subscriber for data stream, similar to a queue manager;
- Store data stream durably with fault tolerance;
- Process stream data as it occurs;

Because it is scalable, resilient, fault tolerant, it is now used in the largest companies working with bigdata, such as Spotify, as well as being part of the most widely used distributions for bigdata such as Cloudera and Hortonworks.

To meet these capabilities it has the following features:
- Work on clusters from one or more servers that can be expanded to multiple datacenters;
- The Kafka cluster stores the stream of data in categories known as topics;
- Each message internally consists of a key, a value and a timestamp;

It is basically divided into the following blocks or APIs:
- Producer: that allows the publication of message in the topics;
- Consumer: that allows to consume the messages of the topics;
- Stream Processor: which allows the streaming of data from one or more input streams, processing them and producing a stream of output data on one or more topics;
- Connectors: allows you to create and run reusable producers or consumers that connect Kfaka topics to one or more applications;

### Kfaka API´s 
![Kafka API´s](https://github.com/fnldesign/tech-articles/blob/series-bigdata-docker-microservices/00-kafka-apis.png)

   
For those who want to know a little better Kafka put below two references about it:
- [Kafka Site] (https://kafka.apache.org/)
- [Introduction to Kafka] (https://kafka.apache.org/intro)
- [Tutorial on Kafka TutorialPoint] (https://www.tutorialspoint.com/apache_kafka/index.htm)

## Docker
Docker is a platform that allows the creation, testing and deployment of applications through standardized images known as containers.

Each container runs on top of a physical or virtual machine, the container has everything the application needs including libraries, system tools, code and runtime or application executable code.

The purpose here in the article is not to deepen in docker, but we will use some basic commands to create and execute our application through containers.

To know more about docker:
- [Docker official website] (https://www.docker.com/)
- [Docker Overview] (https://www.docker.com/what-docker)
- [docker Official Documentation] (https://docs.docker.com/)
- [Docker on Amazon] (https://aws.amazon.com/docker/)
- [Basic Command Reference] (https://docs.docker.com/engine/reference/commandline/docker/)

## HandOn
Given this little introduction let's go the practical part. Let's download, start and run a docker container with a Kafka image, Spotfy distribution.
Soon after we will create a topic in Kafka and start a Producer to post messages and a Consumer to consume the messages sent.

Kafka offers several methods and apis to use, as well as command line tools that we will use here. See more in [Kfaka Clients] (https://cwiki.apache.org/confluence/display/KAFKA/Clients)

### Downloading and Starting the Container
The commands below will download and execute a container with Kafka and Zookeeper, mapped to standard ports 2181 (Zookeeper) and 9092 (Kafka).

```
docker pull spotify/kafka
docker run -d -p 2181:2181 -p 9092:9092 --env ADVERTISED_HOST=kafka --env ADVERTISED_PORT=9092 --name kafka spotify/kafka
```
Let's explain the commands

The first command (`` `docker pull spotify / kafka``) downloads the image of the Kafka container from Spotfy, while the second command (` `` docker run```) executes the container in the background `` `-d option `` `.

The `` `-p``` options define a mapping between the internal ports of the container and the external ones of the host operating system,` `` -env``` define the values ​​for environment variables and `` `-name` `` defines the host name of the container, then we need to add this host name to the local host file so that it can be viewed by the host and other processes and containers.

Let's put it all up and then see how to do it in the [Setting environment variables] (./ ddd) session.

`ADVERTISTED_HOST` has been set to` kafka`, this setting will allow other containers to run Producers and Consumers separately and independently.

Configuring `ADVERTISED_HOST` for` localhost`, `127.0.0.1`, or` 0.0.0.0` will work fine only if the Producers and Consumers are started inside the `kafka` container, or if you are using DockerForMac or DockerForWindows (as I) and you want to run the Producers and Consumers OSX or Windows respectively.

Even then, you should map the hostname of the container to an external ip host host, as explained in the [Setting Environment Variables] (./ ddd) section. Otherwise running the Producers and Consumers later on you will get a `kafka.common.LeaderNotAvailableException` error.

However, these use cases running with `localhost` are much less interesting, so let's start Producers and Consumers from other containers.

In a productive environment we should not use localhost, but rather create a network with docker this will also be explained later in the [Configuring a network for docker] (./ ddd) session.

We need to use an IP address or hostname for the `kafka` service to be accessed from another container. The IP address is not known before the container starts, so we have to choose a host name and I chose `kafka` in this example.

[Why am I useful the Spotify distribution?](https://github.com/spotify/docker-kafka#why)

### Creating a topic
```
docker exec kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

The `` `exec``` command executes a command in the container in which case we are called the shell script` `` kafka-topics.sh``` that handles the Kafka threads, the other parameters define which Zookeeper address, number of partitions, and the topic name.

exit:
```
Created topic "test".
```

### Listing the topics
```
docker exec kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-topics.sh --list --zookeeper localhost:2181
```

exit:
```
test
```

### Iniciando um Producer (em uma nova janela)
Starting a Producer (in a new window)
The command below will use the shell script `kafka-console-producer.sh` to start a Producer to send messages to Kafka that we created earlier.

This command will execute an instance of `spotfy \ kfaka` linked to the` kafka` service previously created, will start a Producer, and will wait to enter the separated messages or Streams by line break until you abort the process, which will destroy the container Producer.

In other words, starting this instance of the container you can send messages to the Kfaka topic just by entering the messages and sending them by entering enter.

```
docker run -it --rm --link kafka spotify/kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-console-producer.sh --broker-list kafka:9092 --topic test
```

#### Let's explain the command

The `docker run` command we already know and knows that it is used to execute a command inside a docker container, so let's get the news.
The `-i` parameter causes the docker to rotate in an iterative way, or allow commands to be entered differently from the` -q` option, since the `-t` option together with` -i` emulates a TTY terminal console , ie the two options together allow us to input commands to the Producer as in a standard terminal.

The `--rm` option causes the container to be automatically removed when closed.

Now the `--link kafka spotify / kafka` option causes this new instance to be related to the Kfaka server started in the previous steps.

In practical terms this set of commands starts a contain with a Kafka Producer that allows us to type messages for the `test`pecified topic and send the messages when we give` ENTER'.
To end the container press `CTRL + C` which stops the process and ends the container releasing the resources of the same.

Before trying the commands insert the `kafka` host associated with the IPs` 127.0.0.1` and `0.0.0.0` so that the` kafka.common.LeaderNotAvailableException` exception does not occur.

** Try it now **

### Starting a Consumer (in a new window)

To consume the messages sent by Producer created in the previous step we need a Consumer, through the command below, similarly to Producer we will start a Consumer in a new console window to check the arrival of the messages.

This command will start a new instance of the `spotify / kafka` container connected to the` kafka` service, start a Consumer and display the messages sent through the Producer to the `test` topic, similar to the Producer with the` -rm` Consumer will be finalized when pressing `CTRL + C`.

The Kfaka `--from-beginning` options cause Consumer to consume all messages of the topic from the beginning, then we will see how to consume the messages from where we stop with` offset`.
```
docker run -it --rm --link kafka spotify/kafka /opt/kafka_2.11-0.10.1.0/bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic test --from-beginning
```

**Try it now**

#### Sent and Receiving Messages
Send some messages from Producer and type `ENTER`. The messages should appear in the Consumer window.


### Next steps

As next steps in the next article we will implement a microservice that will obtain the quotations of crpyptocurrencies and will send to the kafka topics that we will create.

In the meantime, study the docker command line options as well as study Kafka, in this part we do not explore the partitioning and message grouping capabilities, nor the replication of messages given by the replication factor feature.

Taking a sample here of these indispensable and very important resources of Kafka I place the image below to have a vision of how Kafka is powerful, resilient and scalable.

#### Anatomy of a Topic
! [Anatomy of a Topic] (https://github.com/fnldesign/tech-articles/blob/series-bigdata-docker-microservices/01-anatomy-of-a-topic.png)

#### Group ID
! [Group Id] (https://github.com/fnldesign/tech-articles/blob/series-bigdata-docker-microservices/02-consumer-groups.png)

These features can be further explored in [Kafka Introduction] (https://kafka.apache.org/intro).

Until the next article !!

I look forward to suggestions, opinions, doubts and other contributions to expand and discuss these topics. That's All Folks !! : sunglasses: