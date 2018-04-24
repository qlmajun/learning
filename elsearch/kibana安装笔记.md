### kibana安装笔记 ###
---

### 下载和解压文件

```
tar -zxvf kibana-5.6.0-linux-x86_64
```

### 修改配置文件

config/kibana.yml

```
server.port: 5601
server.host: "192.168.23.135"
elasticsearch.url: "http://192.168.23.135:9200"
```

### 启动kibana

```
./kibana
```
