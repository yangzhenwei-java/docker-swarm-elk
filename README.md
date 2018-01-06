
## 准备三个节点 elk1、elk2、elk3

```
1. docker-machine create  --engine-registry-mirror=https://7os77co8.mirror.aliyuncs.com \
  --driver generic \
  --generic-ip-address=10.33.80.122  \
  --generic-ssh-key ~/.ssh/id_rsa \
   elk1

2. docker-machine create  --engine-registry-mirror=https://7os77co8.mirror.aliyuncs.com \
  --driver generic \
  --generic-ip-address=10.33.80.123	\
  --generic-ssh-key ~/.ssh/id_rsa \
   elk2

3.  docker-machine create  --engine-registry-mirror=https://7os77co8.mirror.aliyuncs.com \
  --driver generic \
  --generic-ip-address=10.33.80.124  \
  --generic-ssh-key ~/.ssh/id_rsa \
   elk3
```

## 准备elk环境

```
1. docker-machine ssh elk1 "sysctl -w vm.max_map_count=262144"

2. docker-machine ssh elk1 "echo 'vm.max_map_count=262144' >> /etc/sysctl.conf"

3. docker-machine ssh elk1 "echo    '*  soft nproc 65536
                                  *  hard nproc 65536
                                  *  soft nofile 65536
                                  *  hard nofile 65536
                                  *  soft memlock unlimited
                                  *  hard memlock unlimited' >> /etc/security/limits.conf"
                                  
4. docker-machine ssh elk1 "sudo swapoff -a" （永久关闭交换内存则需要修改/etc/fstab）

5. docker-machine ssh elk2、elk3也执行上述步骤。
```
 
## 准备swarm环境

```
1. docker-machine ssh elk1  "docker swarm init"

2. docker-machine ssh elk2  "docker swarm join --token SWMTKN-1-************************ IP(elk1的IP)"

3. docker-machine ssh elk3  "docker swarm join --token SWMTKN-1-************************ IP(elk1的IP)"

4. docker-machine ssh elk1 "firewall-cmd --permanent --zone=public --add-port=5601/tcp  --add-port=9200/tcp  --add-port=5044/tcp --add-port=7946/tcp  --add-port=7946/udp  --add-port=4789/udp  --add-port=4789/tcp --permanent  && firewall-cmd --reload" 

5. docker-machine ssh elk2 "firewall-cmd --permanent --zone=public --add-port=5601/tcp  --add-port=9200/tcp  --add-port=5044/tcp --add-port=7946/tcp  --add-port=7946/udp  --add-port=4789/udp  --add-port=4789/tcp --permanent  && firewall-cmd --reload" 

6. docker-machine ssh elk3 "firewall-cmd --permanent --zone=public --add-port=5601/tcp  --add-port=9200/tcp  --add-port=5044/tcp --add-port=7946/tcp  --add-port=7946/udp  --add-port=4789/udp  --add-port=4789/tcp --permanent  && firewall-cmd --reload" 
备注:5601是kibana端口、9200是elasticsearch的端口、5044是beat接收日志的端口、7946节点通信、4789覆盖网络流量
```


## 准备启动

```
1. eval $(docker-machine env elk1)

2.  docker node update --label-add logstash=logstash elk2 && docker node update --label-add logstash=logstash elk3 
备注： 有logstash标签的节点才会接受logstash服务

3. docker node update --label-add elasticsearch=elasticsearch elk1 && \
   docker node update --label-add elasticsearch=elasticsearch elk2 && \
   docker node update --label-add elasticsearch=elasticsearch elk3
备注： 有elasticsearch标签的节点才会接受elasticsearch服务 

4. git clone https://github.com/yangzhenwei-java/docker-swarm-elk.git

5. cd docker-swarm-elk && docker stack deploy -c docker-compose.yml elk

6. docker run  --name filebeat -v /root/logs:/usr/share/filebeat/logs  registry.cn-beijing.aliyuncs.com/yangzhenwei/filebeat:6.1.0
备注：当前docker-machine的默认控制节点是elk1, 执行eval $(docker-machine env --unset)去除控制
```

## 备注：

```
    1. registry.cn-beijing.aliyuncs.com/yangzhenwei/elasticsearch 只是为了加速下载镜像,
        没有做别的改动。官方的镜像如docker.elastic.co/elasticsearch/elasticsearch:6.1.0 下载太慢了。
    
    2. 收集tomcat的日志格式如下
    <pattern> %d^|^%contextName^|^%X{requestId}^|^%level^|^%c{1.}^|^%t^|^%m%n</pattern>
    
    3. 收集nginx的日志格式如下
        log_format json '{"datetime":"$time_local",'
                        '"clientip":"$remote_addr",'
                        '"ident":"$remote_user",'
                        '"http_host":"$host",'
                        '"method":"$request_method",'
                        '"url":"$uri",'
                        '"status":$status,'
                        '"bytes":$body_bytes_sent,'
                        '"referrer":"$http_referer",'
                        '"agent":"$http_user_agent",'
                        '"x_forwarded_for":"$http_x_forwarded_for",'
                        '"appName":"$app_name",'
                        '"upstream_addr":"$upstream_addr",'
                        '"upstream_response_time":"$upstream_response_time",'
                        '"request_time":$request_time}';
```
### 作者邮箱 : sxlc_yzw@163.com
    
### 作者QQ : 646473942