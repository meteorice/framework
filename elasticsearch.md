---
description: elasticsearch 7.x   docker
---

# Elasticsearch docker集群搭建

## 要注意问题

> jvm堆最大最小要直接配置成相等,比如: -Xms512m -Xmx512m 
>
> 映射的文件夹要 chmod 777

## 编写docker-compose

### 上docker-compose.yml

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

## 检查是否集群建立成功

* 打开浏览器，[http://192.168.1.208:9201/\_cat/nodes](http://192.168.1.208:9201/_cat/nodes) ，星号既是master



  ```text
  172.28.0.2 24 33 0 0.02 0.20 0.22 mdi - node1
  172.28.0.3 24 33 1 0.02 0.20 0.22 mdi * node2
  ```



## 遇到的检查问题

* max virtual memory areas vm.max\_map\_count \[65530\] is too low, increase to at least \[262144\]



  1. 切换到root用户修改配置sysctl.conf

     `vi /etc/sysctl.conf`

  2. 添加下面配置

     `vm.max_map_count=655360`

  3. 并执行命令

     `sysctl -p`

  4. 重新启动elasticsearch

* 失败信息如下

  因为没有配置端口号，是es7自适应寻找正确端口所以日志中会出现此类错误

```java
failed to join {node1}{pPjTF86WSRKFil3Rt3eisg}{Ww9e2XQ-Qdip3xFXWyCtWQ}{172.28.0.2}{172.28.0.2:9300}{ml.machine_memory=135189749760, xpack.installed=true, ml.max_open_jobs=20} with JoinRequest{sourceNode={node1}{pPjTF86WSRKFil3Rt3eisg}{Ww9e2XQ-Qdip3xFXWyCtWQ}{172.28.0.2}{172.28.0.2:9300}{ml.machine_memory=135189749760, xpack.installed=true, ml.max_open_jobs=20}, optionalJoin=Optional[Join{term=1, lastAcceptedTerm=0, lastAcceptedVersion=0, sourceNode={node1}{pPjTF86WSRKFil3Rt3eisg}{Ww9e2XQ-Qdip3xFXWyCtWQ}{172.28.0.2}{172.28.0.2:9300}{ml.machine_memory=135189749760, xpack.installed=true, ml.max_open_jobs=20}, targetNode={node1}{pPjTF86WSRKFil3Rt3eisg}{Ww9e2XQ-Qdip3xFXWyCtWQ}{172.28.0.2}{172.28.0.2:9300}{ml.machine_memory=135189749760, xpack.installed=true, ml.max_open_jobs=20}}]}" , 
"stacktrace": ["org.elasticsearch.transport.RemoteTransportException: [node1][172.28.0.2:9300][internal:cluster/coordination/join]", 
"Caused by: org.elasticsearch.cluster.coordination.CoordinationStateRejectedException: became follower", 
"at org.elasticsearch.cluster.coordination.JoinHelper$CandidateJoinAccumulator.lambda$close$3(JoinHelper.java:476) [elasticsearch-7.1.0.jar:7.1.0]", 
"at java.util.HashMap$Values.forEach(HashMap.java:976) [?:?]", 
"at org.elasticsearch.cluster.coordination.JoinHelper$CandidateJoinAccumulator.close(JoinHelper.java:476) [elasticsearch-7.1.0.jar:7.1.0]",
"at org.elasticsearch.cluster.coordination.Coordinator.becomeFollower(Coordinator.java:600) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.cluster.coordination.Coordinator.onFollowerCheckRequest(Coordinator.java:237) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.cluster.coordination.FollowersChecker$2.doRun(FollowersChecker.java:187) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingAbstractRunnable.doRun(ThreadContext.java:751) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:37) [elasticsearch-7.1.0.jar:7.1.0]", 
"at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128) [?:?]", 
"at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628) [?:?]", 
"at java.lang.Thread.run(Thread.java:835) [?:?]"] }```

```java
failed to join {node1}{pPjTF86WSRKFil3Rt3eisg}{Ww9e2XQ-Qdip3xFXWyCtWQ}{172.28.0.2}{172.28.0.2:9300}{ml.machine_memory=135189749760, ml.max_open_jobs=20, xpack.installed=true} with JoinRequest{sourceNode={node2}{o9cl5VZ2R5OFScm6J_6Uxg}{IkRvRaHLSEecQ338oADlKA}{172.28.0.3}{172.28.0.3:9300}{ml.machine_memory=135189749760, xpack.installed=true, ml.max_open_jobs=20}, optionalJoin=Optional[Join{term=1, lastAcceptedTerm=0, lastAcceptedVersion=0, sourceNode={node2}{o9cl5VZ2R5OFScm6J_6Uxg}{IkRvRaHLSEecQ338oADlKA}{172.28.0.3}{172.28.0.3:9300}{ml.machine_memory=135189749760, xpack.installed=true, ml.max_open_jobs=20}, targetNode={node1}{pPjTF86WSRKFil3Rt3eisg}{Ww9e2XQ-Qdip3xFXWyCtWQ}{172.28.0.2}{172.28.0.2:9300}{ml.machine_memory=135189749760, ml.max_open_jobs=20, xpack.installed=true}}]}" , 
"stacktrace": ["org.elasticsearch.transport.NodeNotConnectedException: [node1][172.28.0.2:9300] Node not connected", 
"at org.elasticsearch.transport.ConnectionManager.getConnection(ConnectionManager.java:151) ~[elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.transport.TransportService.getConnection(TransportService.java:558) ~[elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.transport.TransportService.sendRequest(TransportService.java:530) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.cluster.coordination.JoinHelper.sendJoinRequest(JoinHelper.java:278) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.cluster.coordination.JoinHelper.sendJoinRequest(JoinHelper.java:211) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.cluster.coordination.JoinHelper.lambda$new$2(JoinHelper.java:135) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.xpack.security.transport.SecurityServerTransportInterceptor$ProfileSecuredRequestHandler$1.doRun(SecurityServerTransportInterceptor.java:251) [x-pack-security-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:37) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.xpack.security.transport.SecurityServerTransportInterceptor$ProfileSecuredRequestHandler.messageReceived(SecurityServerTransportInterceptor.java:309) [x-pack-security-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.transport.RequestHandlerRegistry.processMessageReceived(RequestHandlerRegistry.java:63) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.transport.TcpTransport$RequestHandler.doRun(TcpTransport.java:1077) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.common.util.concurrent.ThreadContext$ContextPreservingAbstractRunnable.doRun(ThreadContext.java:751) [elasticsearch-7.1.0.jar:7.1.0]", 
"at org.elasticsearch.common.util.concurrent.AbstractRunnable.run(AbstractRunnable.java:37) [elasticsearch-7.1.0.jar:7.1.0]", 
"at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128) [?:?]", 
"at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628) [?:?]", 
"at java.lang.Thread.run(Thread.java:835) [?:?]"] }
```

