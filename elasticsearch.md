# Elasticsearch docker 集群搭建

## 问题

> jvm堆最大最小要直接配置成相等,比如: -Xms512m -Xmx512m
> 映射的文件夹要 chmod 777 

## 上docker-compose.yml

```yaml

version: "3"
services:
  es3:
    container_name: es3
    image: 'elasticsearch:7.1.0'
    privileged: true
    cap_add:
      - SYS_PTRACE
    environment:
      - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    command:
      - /home/elasticsearch/bin/elasticsearch
    volumes:
      - ./data3:/usr/share/elasticsearch/data
      - ./node3.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./logs3:/usr/share/elasticsearch/logs:rw
    ports:
      - '9203:9200/tcp'
      - '9303:9300/tcp'
    networks:
      - esnet

  es1:
    container_name: es1
    image: 'elasticsearch:7.1.0'
    privileged: true
    cap_add:
      - SYS_PTRACE
    environment:
      - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    command:
      - /home/elasticsearch/bin/elasticsearch
    volumes:
      - ./data1:/usr/share/elasticsearch/data
      - ./node1.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./logs1:/usr/share/elasticsearch/logs:rw
    ports:
      - '9201:9200/tcp'
      - '9301:9300/tcp'
    networks:
      - esnet
  es2:
    image: elasticsearch:7.1.0
    container_name: es2
    environment:
      - "ES_JAVA_OPTS=-Xms1024m -Xmx1024m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data2:/usr/share/elasticsearch/data
      - ./node2.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - ./logs2:/usr/share/elasticsearch/logs:rw
    ports:
      - '9202:9200/tcp'
      - '9302:9300/tcp'
    networks:
      - esnet

networks:
  esnet:

```

### node1.yml

```yaml

bootstrap.memory_lock: true
cluster.name: "es-cluster"
node.name: node1
node.master: true
node.data: true
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
cluster.initial_master_nodes: ["node1","node2","node3"]
discovery.seed_hosts: ["es2","es3"]

path.logs: /usr/share/elasticsearch/logs

http.cors.enabled: true
http.cors.allow-origin: "*"

```

### node2.yml

```yaml

bootstrap.memory_lock: true
cluster.name: "es-cluster"
node.name: node2
node.master: true
node.data: true
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
cluster.initial_master_nodes: ["node1","node2","node3"]
discovery.seed_hosts: ["es1","es3"]

path.logs: /usr/share/elasticsearch/logs

http.cors.enabled: true
http.cors.allow-origin: "*"

```

### node3.yml

```yaml

bootstrap.memory_lock: true
cluster.name: "es-cluster"
node.name: node3
node.master: true
node.data: true
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
cluster.initial_master_nodes: ["node1","node2","node3"]
discovery.seed_hosts: ["es1","es2"]

path.logs: /usr/share/elasticsearch/logs

http.cors.enabled: true
http.cors.allow-origin: "*"


```