# 1. Create an EC2 instance in AWS

#### **_Also generate ssh key which can be used to connect to EC2 from terminal._**

# 2. Connect to instance

### **Save the generated key somewhere in local machine and cd to that directory.**

```sh
cd path_to_key
ssh -i "bigdata-datascience-project.pem" ec2-user@ec2-3-238-16-255.compute-1.amazonaws.com
```

> **_NOTE:_** If you are getting permission error :
>
> #### Mac or Linux
>
> ```sh
> chmod 400 bigdata-datascience-project.pem
> ```
>
> #### Windows
>
> ```powershell
> icacls .\bigdata-datascience-project.pem /reset
> icacls .\bigdata-datascience-project.pem /grant:r "$($env:username):(r)"
> icacls .\bigdata-datascience-project.pem /inheritance:r
> ```

# 3. Download Kafka to instance

```sh
wget https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz
tar -xvf kafka_2.13-3.4.0.tgz
```

# 4. Check if Java is available

```sh
java -version

#if not install java
sudo yum install java-1.8.0-openjdk
java -version

#Change directory to kafka
cd kafka_2.13-3.4.0
```

# 5. Start Zoo-keeper:

```sh
bin/zookeeper-server-start.sh config/zookeeper.properties
```

# 6. Start Kafka-server:

```sh
#Duplicate the session & enter in a new console --
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
cd kafka_2.13-3.4.0
bin/kafka-server-start.sh config/server.properties
```

### **Right now, it is pointing to private server , change _server.properties_ so that it can run in public IP.**

```sh
sudo nano config/server.properties
#change ADVERTISED_LISTENERS to public ip of the EC2 instance, eg:
advertised_listeners=PLAIN_TEXT//3.238.16.255:9092
```

# 7. Restart Zoo-keeper and Kafka-server

# 8. Create the topic:

```sh
#Duplicate the session & enter in a new console
#Here, I have created topic named 'Sales' with singe partition and single replication factor: By default it will spin 3 replication
cd kafka_2.13-3.4.0
bin/kafka-topics.sh --create --topic Sales --bootstrap-server 3.238.16.255:9092 --replication-factor 1 --partitions 1
```

# 9. Start Producer:

```sh
bin/kafka-console-producer.sh --topic Sales --bootstrap-server 3.238.16.255:9092
```

# 10. Start Consumer:

```sh
#Duplicate the session & enter in a new console --
cd kafka_2.13-3.4.0
bin/kafka-console-consumer.sh --topic Sales --bootstrap-server 3.238.16.255:9092
```

Now you can write message to Kafka broker/server from producer and it will be reflected to consumer since we have one consumer connected.

> **_Note :_**
> In kafka, basically one partitioned can only be allocated to one consumer since Kafka is built to support both the feature like queue and pub/sub.
>
> If another consumer have to access the data from same partition, it has to use another consumer group.
