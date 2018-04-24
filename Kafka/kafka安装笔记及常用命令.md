### kafka安装笔记及常用命令 ###
----
### kafka安装

#### 1、下载代码
下载版本kafka_2.10-0.10.2.0并且解压

```
tar -zxvf kafka_2.10-0.10.2.0.tgz
```

#### 2、启动zookeeper服务

运行kafka需要使用Zookeeper，所以你需要先启动Zookeeper，如果你没有Zookeeper，你可以使用kafka自带打包和配置好的Zookeeper。

```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

也可以使用自己搭建的zookeeper服务,更改config/server.properties文件。

```
zookeeper.connect=192.168.23.129:2181/kafka
```

#### 3、启动kafka服务

```
 bin/kafka-server-start.sh config/server.properties &
```

### kafka常用命令

#### 1、创建topic

```
./kafka-topics.sh --create --zookeeper 172.20.13.62:2181 --replication-factor 1 --topic test --partitions 1
```

#### 2、查看topic列表

```
./kafka-topics.sh --list --zookeeper 172.20.13.62:2181
```

#### 3、生产一条消息

```
./kafka-console-producer.sh --broker-list 172.20.13.62:9092 --topic test
```

#### 4、查看topic中的消息

```
./kafka-console-consumer.sh --zookeeper 172.20.13.62:2181 --from-beginning --topic test
```
