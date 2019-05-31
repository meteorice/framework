---
description: metricbeat 7.x   docker
---

# metricbeat 最简运行

## 配置 metricbeat.yml

```yaml

setup.kibana:
  host: "192.168.1.208:5601"

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["192.168.1.208:9201","192.168.1.208:9202"]

```

## 启动

* 去除 modules.d/*.disabled 中的 `disabled` 后缀
* ./metricbeat setup --dashboards   装入kibana dashboards
* 启动后台进程

```bash

nohup ./metricbeat -e -c metricbeat.yml -d "publish"  >> start.out 2>&1 &

```