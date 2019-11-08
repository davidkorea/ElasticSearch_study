# ElasticSearch_study


# 1. Install by docker-compose

### Issue1: elasticsearch start failed
1. `docker-compose up` with docker-compose.yaml file
2. but when `docker ps` find no elaticsearch container, only kibana and cerebro
3. exec `docker ps -a` find elasticsearch conbtainer is EXITED status
    ```
    [root@localhost ~]# docker ps -a
    CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS                       PORTS                    NAMES
    6826362e507e        docker.elastic.co/elasticsearch/elasticsearch:7.1.0   "/usr/local/bin/dock…"   11 minutes ago      Exited (78) 10 minutes ago                            es7_02
    e35bb96c6da7        docker.elastic.co/elasticsearch/elasticsearch:7.1.0   "/usr/local/bin/dock…"   11 minutes ago      Exited (78) 10 minutes ago                            es7_01
    e557763bd936        lmenezes/cerebro:0.8.3                                "/opt/cerebro/bin/ce…"   11 minutes ago      Up 11 minutes                0.0.0.0:9000->9000/tcp   cerebro
    c9a24cc182ef        docker.elastic.co/kibana/kibana:7.1.0                 "/usr/local/bin/kiba…"   11 minutes ago      Up 11 minutes                0.0.0.0:5601->5601/tcp   kibana7
    ```
4. check EXITED containber logs by `docker logs 6826362e507e`, find that:
    ```
    ERROR: [1] bootstrap checks failed
    [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    ```










# 2. Logstash 
https://artifacts.elastic.co/downloads/logstash/logstash-7.1.0.tar.gz , version7.1.0 needs to match es version

- `sudo ./logstash -f logstash.conf`, must add sudo, or else error
  ```
  [ERROR][logstash.javapipeline    ] A plugin had an unrecoverable error. Will restart this plugin.
  Error: Permission denied - Permission denied
  ```
