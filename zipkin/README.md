# Zipkin（9411）

Zipkin 服务，在使用 Spring Boot 2.x 版本后，官方就不推荐自行定制编译了，反而是直接提供了编译好的 jar 包来给我们使用

本项目使用 2.20 版本，采用 RabbitMQ 收集，包含两种存储方式 MySQL / Elasticsearch，具体启动脚本详见 [start_zipkin_server.sh](https://github.com/micyo202/lion/blob/master/zipkin/start_zipkin.sh)

### 官网
[https://zipkin.io](https://zipkin.io)

### 下载地址
下载最新版本请在阿里云仓库[https://maven.aliyun.com/mvn/search](https://maven.aliyun.com/mvn/search)搜索关键字zipkin-server，找到zipkin-server-*exec.jar下载对应版本即可

下载**zipkin-server-2.12.9-exec.jar**及之前版本地址
[https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/](https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/)

### 本地部署

```shell script
java -jar zipkin-server-*exec.jar
```

### Docker部署

```shell script
docker pull openzipkin/zipkin:tagname
```

```shell script
docker run -d \
        -p 9411:9411 \
        -e RABBIT_ADDRESSES=localhost:5672 \
        -e RABBIT_USER=guest \
        -e RABBIT_PASSWORD=guest \
        -e STORAGE_TYPE=mysql \
        -e MYSQL_HOST=localhost \
        -e MYSQL_USER=root \
        -e MYSQL_PASS=root \
        -e MYSQL_DB=zipkin \
        --name zipkin \
        --restart always \
        openzipkin/zipkin:tagname
```

### Mysql 存储 schema DDL
[https://github.com/openzipkin/zipkin/blob/master/zipkin-storage/mysql-v1/src/main/resources/mysql.sql](https://github.com/openzipkin/zipkin/blob/master/zipkin-storage/mysql-v1/src/main/resources/mysql.sql)

### 启动参数说明
[https://github.com/openzipkin/zipkin/tree/master/zipkin-server](https://github.com/openzipkin/zipkin/tree/master/zipkin-server)

### 详细参数查看
[https://github.com/openzipkin/zipkin/blob/master/zipkin-server/src/main/resources/zipkin-server-shared.yml](https://github.com/openzipkin/zipkin/blob/master/zipkin-server/src/main/resources/zipkin-server-shared.yml)

### MySQL 查看 schema DML
```sql
SELECT lower(concat(CASE trace_id_high
                        WHEN '0' THEN ''
                        ELSE hex(trace_id_high)
                    END,hex(trace_id))) AS trace_id,
       lower(hex(parent_id)) as parent_id,
       lower(hex(id)) as span_id,
       name,
       from_unixtime(start_ts/1000000) as timestamp
FROM zipkin_spans
where (start_ts/1000000) > UNIX_TIMESTAMP(now()) - 5 * 60;
```