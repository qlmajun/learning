### elsearch安装笔记 ###
---

### 下载压缩包，解压压缩文件

```
# tar zxvf elasticsearch-5.6.0.tar.gz  
# mv elasticsearch-5.6.0 /usr/local/elasticsearch  
```

### 配置ES参数

```
# vi /usr/local/elasticsearch/config/elasticsearch.yml  
# 注意冒号后有空格  
http.port: 9200  
node.name: node-1  
cluster.name: es_cluster  
network.host: 192.168.23.135  
bootstrap.memory_lock: false  
path.data: /usr/local/elasticsearch/data  
path.logs: /usr/local/elasticsearch/logs  
```

### 配置足够内存

```
# vi /usr/local/elasticsearch/config/jvm.options  
-Xms512M  
-Xmx512M
```

### 添加独立用户,并启动服务

```
# useradd elsearch -g elsearch  
# chown -R elsearch:elsearch /usr/local/elasticsearch  
# su elsearch  
# cd /usr/local/elasticsearch/bin  
# ./elasticsearch
```

### 启动可能错误

错误一:

```
ERROR: bootstrap checks failed  
max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]  
max number of threads [1024] for user [elsearch] likely too low, increase to at least [2048]  
max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```

解决办法:修改配置  

```
# vi /etc/security/limits.conf      #永久设置  
elsearch               soft    nofile           65536  
elsearch               hard    nofile           65536  
elsearch               soft    nproc            2048  
elsearch               hard    nproc            2048  

# ulimit -n 65536           #临时设置，重启无效  
# ulimit -u 2048  
# ulimit -a  

# vi /etc/sysctl.conf           #永久设置  
vm.max_map_count=262144  

# sysctl -w vm.max_map_count=262144 #临时设置，重启无效  
# sysctl -a | grep "vm.max_map_count"
```

错误二:

```
Java HotSpot(TM) Server VM warning: INFO: os::commit_memory(0xcc000000, 469762048, 0) failed; error='Cannot allocate memory' (errno=12)  
#  
# There is insufficient memory for the Java Runtime Environment to continue.  
# Native memory allocation (mmap) failed to map 469762048 bytes for committing reserved memory.  
# An error report file with more information is saved as:  
# /usr/local/elasticsearch/bin/hs_err_pid7598.log  
```

解决办法：修改足够的内存  

```
# vi /usr/local/elasticsearch/config/jvm.options  
-Xms512M  
-Xmx512M
```

### 后台启动ES

```
./bin/elasticsearch -d
```
