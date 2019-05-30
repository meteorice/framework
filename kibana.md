---
description: kibana 7.x   docker
---

# kibana docker搭建

## 编写docker-compose

```yaml

version: "3"
services:
  kibana:
    container_name: kibana
    image: 'kibana:7.1.0'
    privileged: true
    cap_add:
      - SYS_PTRACE
    # environment:
      # XPACK_MONITORING_ENABLED: xpack.monitoring.enabled
    ulimits:
      memlock:
        soft: -1
        hard: -1
    # command: /home/elasticsearch/bin/elasticsearch
    volumes:
      - ./data:/usr/share/kibana/data
      - ./kibana.yml:/usr/share/kibana/config/kibana.yml
      - ./logs:/usr/share/kibana/logs:rw
    ports:
      - '5601:5601/tcp'

networks:
  default:
    external:
      name: elasticsearch_esnet

```

## 配置kibana.yml

```yaml

elasticsearch.hosts: http://es1:9200
# elasticsearch.username: "user"
# elasticsearch.password: "pass"
elasticsearch.requestTimeout: 30000

server.host: "0"

path.data: /usr/share/kibana/data

logging.dest: /usr/share/kibana/logs/kinbana.log

server.name: kibana

logging.timezone: Asia/Shanghai

```

## 运行docker-compose up -d

## 打开浏览器 127.0.0.1:5601