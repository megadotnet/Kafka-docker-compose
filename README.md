# Kafka基于docker-compose单结点部署SASL_PLAINTEXT

## 背景
Kafka是一个分布式流处理平台，由LinkedIn开发并开源，如今在多个行业中都有广泛的应用。以下是Kafka的当前用途以及行业应用的详细描述：

**Kafka的用途**：

1. **消息队列**：Kafka最常见的用途之一是作为高性能的消息队列，用于解耦应用程序的各个部分之间的通信。通过将消息发送到Kafka主题，不同的应用程序可以异步地交换数据，并且可以根据需要调整消费速率和处理能力。这种解耦方式使得系统更加灵活，易于扩展和维护。
2. **日志收集和分析**：Kafka也被广泛用于收集和存储大规模系统产生的日志数据。生产者将日志消息发送到Kafka主题，然后可以使用消费者实时消费和分析这些日志数据，进行监控、报警、分析和故障排查等操作。
3. **事件驱动架构**：Kafka提供了一种可靠的事件流平台，用于构建事件驱动架构（EDA）。通过将事件发送到Kafka主题，并使用流处理工具（如Kafka Streams），可以实现事件的实时处理和响应。
4. **实时数据处理**：Kafka可以构建实时数据处理系统，接收并传输大量实时数据，与多种流处理框架如Apache Storm、Apache Flink结合，实现高吞吐量和低延迟处理。

**Kafka的行业应用**：

1. **金融行业**：Kafka在金融行业中的应用非常广泛，特别是在处理实时交易数据、风控分析以及市场数据分发等方面。金融机构可以利用Kafka实现高效的数据传输和实时分析，从而快速响应市场变化并做出决策。
2. **电商行业**：在电商领域，Kafka被用于处理订单数据、用户行为数据以及商品推荐等场景。通过实时收集和分析这些数据，电商企业可以优化用户体验、提升转化率并实现精准营销。
3. **物联网行业**：随着物联网设备的普及，Kafka在物联网领域的应用也日益增多。它可以接收并处理各种传感器数据，如温度、湿度和气压等，实现数据的实时分析和监控。
4. **日志管理**：Kafka支持集中式日志收集，将生成的各类日志数据集中存储，以便后续进行实时或离线的分析。通过将数据发送到主题，可实现高效的处理。
5. **数据备份与复制**：Kafka提供数据备份与复制的机制，能将数据复制到多个Kafka集群，确保高可用性和容错性，尤其适用于需要数据可靠性和持久性的应用如分布式数据库和文件系统。

SASL PLAINTEXT是一种认证机制，用于Kafka集群中保护数据传输的安全性。它基于SASL（Simple Authentication and Security Layer）框架，使用明文密码进行身份验证。尽管在传输过程中，SASL协议会对数据进行加密，但由于客户端发送的用户名和密码是未加密的，因此，SASL PLAINTEXT被认为是一种不太安全的认证方式。然而，它仍然被广泛采用，尤其在测试环境中，或对于配置和运维成本要求较小的小型公司中的Kafka集群。

SASL是一种用来扩充C/S验证模式的认证机制，它规范了Client和Server之间的应答过程以及传输内容的编码方法。SASL验证架构决定了服务器如何存储客户端身份证书以及如何核验客户端提供的密码。一旦客户端通过验证，服务器端就能确定用户的身份，并据此决定用户具有的权限。

在Kafka中，SASL PLAINTEXT认证通常与SSL加密搭配使用，以提高安全性。在使用SASL PLAINTEXT认证时，需要对Kafka服务器和客户端进行相应的配置。例如，在Kafka服务器的配置文件中，需要设置监听地址、协议类型以及认证机制等参数。而在Kafka客户端上，则需要配置认证信息，如用户名和密码等。

请注意，尽管SASL PLAINTEXT在某些场景下可能适用，但在对安全性要求较高的生产环境中，建议使用更为安全的认证方式，如SASL/GSSAPI（Kerberos）或SASL/SCRAM等。这些认证方式提供了更强的密码加密和动态更新机制，能够更好地保护Kafka集群中的数据安全。

## 方案
密码生成器

https://www.avast.com/zh-cn/random-password-generator

两个关键核心配置文件

server_jaas.conf
```
Client {
     org.apache.zookeeper.server.auth.DigestLoginModule required
     username = "admin"
     password = "RvB9SuYrJMPR";
};
Server {
     org.apache.zookeeper.server.auth.DigestLoginModule required
     username = "admin"
     password = "RvB9SuYrJMPR"
     user_super = "RvB9SuYrJMPR"
     user_admin = "RvB9SuYrJMPR";
};
KafkaServer {
     org.apache.kafka.common.security.plain.PlainLoginModule required
     username = "admin"
     password = "RvB9SuYrJMPR"
     user_admin = "RvB9SuYrJMPR";
};
KafkaClient {
     org.apache.kafka.common.security.plain.PlainLoginModule required
     username = "admin"
     password = "RvB9SuYrJMPR";
};
```


zoo.cfg
```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/opt/zookeeper-3.4.13/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
autopurge.purgeInterval=1

## 开启SASl关键配置
authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider

requireClientAuthScheme=sasl

jaasLoginRenew=3600000
zookeeper.sasl.client=true
```


对以下字符串base64加密,https://www.base64decode.org/ 实现简单encode
```
sasl.mechanism: PLAIN
       security.protocol: SASL_PLAINTEXT
       sasl.jaas.config: org.apache.kafka.common.security.scram.ScramLoginModule required username='admin' password='RvB9SuYrJMPR';
```


kafdrop
**obsidiandynamics/kafdrop是一个Kafka开源可视化工具**，它提供了Web UI界面，用于查看Kafka主题和浏览消费者组。通过这个工具，用户可以显示代理、主题、分区、消费者等信息，并且可以预览topic消息。Kafdrop支持Windows平台环境，并且几乎不需要配置，只需通过简单的命令即可运行。在Docker环境下，用户可以搜索并拉取obsidiandynamics/kafdrop镜像，然后通过一系列配置命令启动Kafdrop服务。启动后，用户可以通过访问指定的Web页面来查看和管理Kafka的相关信息。

因此，obsidiandynamics/kafdrop的主要作用是为用户提供一个直观、便捷的方式来查看和管理Kafka集群的状态和数据，从而方便用户对Kafka进行监控和维护。



最后我们的docker-compose.yaml文件是这样的：
```
version: '3.8'
services:
   zookeeper:
     image: wurstmeister/zookeeper
     volumes:
        - ./data/zookeeper:/data
        - ./config:/opt/zookeeper-3.4.13/conf
        - ./config:/opt/zookeeper-3.4.13/secrets
     container_name: zookeeper
     environment:
       ZOOKEEPER_CLIENT_PORT: 2181
       ZOOKEEPER_TICK_TIME: 2000
       SERVER_JVMFLAGS: -Djava.security.auth.login.config=/opt/zookeeper-3.4.13/secrets/server_jaas.conf
     ports:
       - 2181:2181
     restart: always
   kafka:
     image: wurstmeister/kafka
     container_name: kafka
     depends_on:
       - zookeeper
     ports:
       - 9092:9092
     volumes:
       - ./data/kafka:/kafka
       - ./config:/opt/kafka/secrets
     environment:
       KAFKA_BROKER_ID: 0
       KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
       KAFKA_LISTENERS: INTERNAL://:9093,EXTERNAL://:9092
       KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka:9093,EXTERNAL://172.18.0.155:9092
       KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:SASL_PLAINTEXT,EXTERNAL:SASL_PLAINTEXT
       ALLOW_PLAINTEXT_LISTENER: 'yes'
       KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
       KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
       KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
       KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
       KAFKA_OPTS: -Djava.security.auth.login.config=/opt/kafka/secrets/server_jaas.conf
     restart: always
  ##  kafdrop 监控kafka的Ui工具
   kafdrop:
     image: obsidiandynamics/kafdrop
     restart: always
     ports:
        - "19001:9000"
     environment:
        KAFKA_BROKERCONNECT: "kafka:9093"
     ## 如kafka开启了sasl认证后以下 sasl认证链接是必要的，下面的事经过base64加密后的结果
        KAFKA_PROPERTIES: c2FzbC5tZWNoYW5pc206IFBMQUlOCiAgICAgIHNlY3VyaXR5LnByb3RvY29sOiBTQVNMX1BMQUlOVEVYVAogICAgICBzYXNsLmphYXMuY29uZmlnOiBvcmcuYXBhY2hlLmthZmthLmNvbW1vbi5zZWN1cml0eS5zY3JhbS5TY3JhbUxvZ2luTW9kdWxlIHJlcXVpcmVkIHVzZXJuYW1lPSdhZG1pbicgcGFzc3dvcmQ9J1J2QjlTdVlySk1QUic7
     depends_on:
       - zookeeper
       - kafka
     cpus: '1'
     mem_limit: 1024m
     container_name: kafdrop
```