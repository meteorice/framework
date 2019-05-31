---
    x-pack 配置 elasticsearch security kibana security
---

# 启用xpack

## elasticsearch security

1. 在elasticsearch.yml 增加安全配置
``` yaml
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12

#head插件能够访问
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
```
2. 生成认证文件，执行以下命令 把生成的`elastic-certificates.p12`复制到各个节点的config目录(如果是docker集群复制到映射的宿主机配置目录)
 > bin/elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""
3. 启动集群，这时候还是可以直接访问。要等设置完密码才会阻止不安全访问
4. 登录master节点执行修改密码command

```bash
veoot@1b37d5948a20 elasticsearch]# ./bin/elasticsearch-setup-passwords interacti 
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user. 
You will be prompted to enter passwords as the process progresses.  
Please confirm that you would like to continue [y/N]y       
Enter password for [elastic]: 
Reenter password for [elastic]:                
Enter password for [apm_system]: 
Reenter password for [apm_system]:
Enter password for [kibana]:                             
Reenter password for [kibana]:                          
Enter password for [logstash_system]:                      
Reenter password for [logstash_system]:                     
Enter password for [beats_system]:                         
Reenter password for [beats_system]:                      
Enter password for [remote_monitoring_user]:       
passwords must be at least [6] characters long Try again.   
Enter password for [remote_monitoring_user]:     
Reenter password for [remote_monitoring_user]:  
Changed password for user [apm_system]                        
Changed password for user [kibana]                         
Changed password for user [logstash_system]   
Changed password for user [beats_system]                    
Changed password for user [remote_monitoring_user]             
Changed password for user [elastic]                      
[root@1b37d5948a20 elasticsearch]#   
```
5. 把master的配置目录中`elasticsearch.keystore`文件，复制到其他各个节点，并重启
6. 打开浏览器访问，这时候需要你输入用户和密码

## kibana security

修改配置文件`kibana.yml`，把用户名和密码的注释去掉，用户是:kibana，密码是你刚刚用`elasticsearch-setup-passwords`设置的密码，然后重启kibana

```YAML
elasticsearch.username: "kibana"
elasticsearch.password: "kibana"
```

## 用户

* elastic 用户是superuser角色
* logstash_system用户是给Logstash使用的
* Filebeat会使用beats_system用户向Elasticsearch发送监控数据



