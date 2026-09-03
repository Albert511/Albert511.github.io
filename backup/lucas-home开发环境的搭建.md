# lucas-home开发环境的搭建

lucas-home

auth.http.auth_req.url

http://192.168.2.90/lucas/firmware/device/vertify

web.hook.url

http://192.168.2.90/lucas/firmware/device/connect/notify



## 1、部署流程

### 1、kafka部署

采用 docker-compose 方式部署

> ps：默认 `bridge` 网络没有 Compose 服务名的自动 DNS 解析功能，不能自动解析 ZooKeeper 容器 IP。因此改 默认 `bridge` 网络 为 自定义 bridge 网络

```
mkdir -p /service/kafka-zk/{kafka_log,zk_data}
```

```
[root@localhost /service/kafka-zk]# cat docker-compose.yml 
services:
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    environment:
      TZ: Asia/Shanghai
    ports:
      - 2181:2181
    volumes:
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
      - ./zk_data:/opt/zookeeper-3.4.13/data
    restart: always

  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    ports:
      - 9092:9092
    depends_on:
      - zookeeper
    environment:
      KAFKA_HEAP_OPTS: "-Xmx512M -Xms512M"
      KAFKA_OPTS: "-Duser.timezone=Asia/Shanghai"
      KAFKA_ADVERTISED_HOST_NAME: 192.168.2.90
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LOG_DIRS: /kafka/logs
    volumes:
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - ./kafka_log:/kafka/logs
    restart: always
```

### 2、emqx部署

#### 1、部署

EMQX 4.x 版本的容器初始化、插件机制 和 持久化目录设计比 5.x 版本更老。改用 docker run 方式部署

```
# 创建配置目录
mkdir -p /service/emqx/{data,etc,log}
cd /service/emqx

# 创建一个临时容器。加载核心配置文件
docker create --name emqx-tmp emqx/emqx:latest
docker cp emqx-tmp:/opt/emqx/data/. /service/emqx/data/
docker cp emqx-tmp:/opt/emqx/etc/.  /service/emqx/etc/
docker cp emqx-tmp:/opt/emqx/log/.  /service/emqx/log/
docker rm emqx-tmp

# 修复目录权限
# 1000 是 EMQX 默认使用的用户 UID
chown -R 1000:1000 \
  /service/emqx/data \
  /service/emqx/etc \
  /service/emqx/log

chmod -R u+rwX \
  /service/emqx/data \
  /service/emqx/etc \
  /service/emqx/log
  
# 确认默认插件清单不是空的
cat /service/emqx/data/loaded_plugins
```

```
# 启动 emqx 服务
docker run -d \
  --name emqx \
  --hostname emqx.local \
  --restart always \
  -e 'EMQX_NODE_NAME=emqx@emqx.local' \
  -p 1883:1883 \
  -p 8083:8083 \
  -p 8084:8084 \
  -p 8883:8883 \
  -p 18083:18083 \
  -v /service/emqx/data:/opt/emqx/data \
  -v /service/emqx/etc:/opt/emqx/etc \
  -v /service/emqx/log:/opt/emqx/log \
  emqx/emqx:latest
  
# 查看启动状态
docker logs --tail=100 emqx

# 访问 dashboard：http://服务器IP:18083/
# 默认登录账号及密码（admin、public）
```

#### 2、配置更改

1.mqtt入网配置

需要修改plugin的emqx_auth_http.conf文件（注意大小写）：

auth.http.auth_req.url的值为设备入网时的鉴权接口；

```
auth.http.auth_req.url = http:xxx/device/vertify
auth.http.auth_req.method = post
auth.http.auth_req.headers.content_type = application/json
auth.http.auth_req.params = clientId=%c,username=%u,password=%P
```

网页端需要启动对应的emqx http认证方式

2.客户端自动订阅

需要修改emqx.conf文件

```
module.subscription.1.topic=iot/ota/firmware/light/%c/upgrade
module.subscription.2.topic=iot/time/light/%c/set
module.subscription.3.topic=iot/device/light/%c/state/query
module.subscription.4.topic=iot/device/light/%c/command
module.subscription.5.topic=iot/device/light/%c/wifi/query
module.subscription.6.topic=iot/ota/firmware/light/%c/package
```

网页端需要启动模块下的emqx_mod_subscription

3.修改ACL订阅规则

启用插件emqx_web_hook，修改配置 emqx_web_hook.conf文件；

web.hook.url的值为项目的回调地址；

```plaintext
##====================================================================
## WebHook
##====================================================================

## Webhook URL
##
## Value: String
web.hook.url = http://xxx/connect/notify

## HTTP Headers
##
## Example:
## 1. web.hook.headers.content-type = application/json
## 2. web.hook.headers.accept = *
##
## Value: String
web.hook.headers.content-type = application/json

## The encoding format of the payload field in the HTTP body
## The payload field only appears in the on_message_publish and on_message_delivered actions
##
## Value: plain | base64 | base62
web.hook.body.encoding_of_payload_field = plain

##--------------------------------------------------------------------
## PEM format file of CA's
##
## Value: File
## web.hook.ssl.cacertfile  = <PEM format file of CA's>

## Certificate file to use, PEM format assumed
##
## Value: File
## web.hook.ssl.certfile = <Certificate file to use>

## Private key file to use, PEM format assumed
##
## Value: File
## web.hook.ssl.keyfile = <Private key file to use>

## Turn on peer certificate verification
##
## Value: true | false
## web.hook.ssl.verify = false

## If not specified, the server's names returned in server's certificate is validated against
## what's provided `web.hook.url` config's host part.
## Setting to 'disable' will make EMQ X ignore unmatched server names.
## If set with a host name, the server's names returned in server's certificate is validated
## against this value.
##
## Value: String | disable
## web.hook.ssl.server_name_indication = disable

## Connection process pool size
##
## Value: Number
web.hook.pool_size = 32

## Whether to enable HTTP Pipelining
##
## See: https://en.wikipedia.org/wiki/HTTP_pipelining
web.hook.enable_pipelining = true

##--------------------------------------------------------------------
## Hook Rules
## These configuration items represent a list of events should be forwarded
##
## Format:
##   web.hook.rule.<HookName>.<No> = <Spec>
#web.hook.rule.client.connect.1       = {"action": "on_client_connect"}
#web.hook.rule.client.connack.1       = {"action": "on_client_connack"}
web.hook.rule.client.connected.1     = {"action": "on_client_connected"}
web.hook.rule.client.disconnected.1  = {"action": "on_client_disconnected"}
#web.hook.rule.client.subscribe.1     = {"action": "on_client_subscribe"}
#web.hook.rule.client.unsubscribe.1   = {"action": "on_client_unsubscribe"}
#web.hook.rule.session.subscribed.1   = {"action": "on_session_subscribed"}
#web.hook.rule.session.unsubscribed.1 = {"action": "on_session_unsubscribed"}
#web.hook.rule.session.terminated.1   = {"action": "on_session_terminated"}
#web.hook.rule.message.publish.1      = {"action": "on_message_publish"}
#web.hook.rule.message.delivered.1    = {"action": "on_message_delivered"}
#web.hook.rule.message.acked.1        = {"action": "on_message_acked"}
```



### 3、redis部署

仅挂载 data目录 到 redis 容器下，继续使用 docker run 方式部署

```
mkdir -p /service/redis/data
```

```
docker run -d \
  --name redis \
  --restart always \
  -p 6379:6379 \
  -v /service/redis/data:/data \
  redis:5.0 \
  redis-server \
  --appendonly yes \
  --requirepass 'neewerrds123'
```

```sh
docker logs --tail=50 redis
# 修改redis密码
docker exec redis redis-cli -a 'neewerrds123' ping
```



### 4、xxl-job部署

采用脚本方式运行 xxl-job 服务

```
# 目录文件路径
/service/xxl-job
├── Dockerfile
├── logs/
├── startup.sh
├── xxl-job-20260306.jar
└── xxl-job-admin-2.4.2-SNAPSHOT.jar

# 拷贝 xxl-job 项目的测试环境jar包、Dockerfile、startup.sh到指定目录
```

直接运行 startup.sh 脚本



### 5、lucas-home

采用脚本方式运行 lucas-home 服务

```
# 目录文件路径
/service/lucas-home
├── Dockerfile
├── geoip/
├── lib/
├── lucas-home-0.0.1-SNAPSHOT.jar
└── startup.sh

# 拷贝 lucas-home 项目的测试环境jar包、Dockerfile、startup.sh、lib/等到指定目录
```

直接运行 startup.sh 脚本



### 6、nginx部署

采用原生方式部署 Nginx 服务；

部署好后增加neewer.conf配置文件；

```
[root@localhost /etc/nginx]# cat conf.d/neewer.conf 
upstream lucashomeservice {
    server 127.0.0.1:8080;
    server 127.0.0.1:8086;
}

server {
    listen 80;

    location /lucashome {
        proxy_pass  http://lucashomeservice;
        client_max_body_size 100m;
        proxy_buffer_size  128k;
        proxy_buffers   32 32k;
        proxy_busy_buffers_size 128k;

        proxy_set_header  Host        $host;
        proxy_set_header  X-Real-IP   $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header  Accept-Encoding "";
    }

    location /xxl-job-admin {
        proxy_pass  http://127.0.0.1:8081;
        client_max_body_size 100m;
        proxy_buffer_size  128k;
        proxy_buffers   32 32k;
        proxy_busy_buffers_size 128k;

        proxy_set_header  Host        $host;
        proxy_set_header  X-Real-IP   $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header  Accept-Encoding "";
    }
}
```

修改nginx.conf文件，增加include /etc/nginx/conf.d/*.conf;

修改 `nginx.conf` 后必须重新加载配置才能生效，通常不需要重启 Nginx。

先检查配置语法：sudo nginx -t

显示 `syntax is ok` 和 `test is successful` 后，平滑重新加载：sudo systemctl reload nginx

也可以使用：sudo nginx -s reload

验证服务状态：sudo systemctl status nginx



## 2、项目配置修改

### 1、redis配置

修改redis连接地址跟密码；

### 2、xxl-job配置

修改连接地址，用户名，密码，group，以及appname的值；

### 3、mqtt服务

修改ip跟externIp的值；

### 4、kafka服务

修改kafka的连接地址；

### 5、micro微社区服务

修改微服务的连接地址；