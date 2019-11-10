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

**[Resolution]**
- check current 
    ```
    [root@localhost ~]# sysctl -a|grep vm.max_map_count
    vm.max_map_count = 65530
    ```
- modify
    ```
    [root@localhost ~]# sysctl -w vm.max_map_count=262144
    vm.max_map_count = 262144
    ```
- to overwrite permenently, add to last rom
    ```
    [root@localhost ~]# vim /etc/sysctl.conf   
    vm.max_map_count=262144
    ```
    ```
    [root@localhost ~]# sysctl -p
    vm.max_map_count = 262144
    ```
- cancel the last docker compose
    ```
    [root@localhost ~]# docker-compose down -v
    Stopping cerebro ... done
    Stopping kibana7 ... done
    Removing es7_02  ... done
    Removing es7_01  ... done
    Removing cerebro ... done
    Removing kibana7 ... done
    Removing network root_es7net
    Removing volume root_es7data1
    Removing volume root_es7data2
    ```
- `docker-compose up` again


-----

# 2. Logstash 
https://artifacts.elastic.co/downloads/logstash/logstash-7.1.0.tar.gz , version7.1.0 needs to match es version

- `sudo ./logstash -f logstash.conf`, must add sudo, or else error
  ```
  [ERROR][logstash.javapipeline    ] A plugin had an unrecoverable error. Will restart this plugin.
  Error: Permission denied - Permission denied
  ```

-----

# 3. Analyzer tokenizer - IK Analysis for Elasticsearch


1. install plugin for **EVERY** ElasticSearch docker node

    - `docker exec -it DOCKER_ID /bin/bash`
        ```
        [root@localhost ~]# docker exec -it ae02ee75b357 /bin/bash

        [root@ae02ee75b357 elasticsearch]# ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.0/elasticsearch-analysis-ik-7.1.0.zip
        -> Downloading https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.1.0/elasticsearch-analysis-ik-7.1.0.zip
        [=================================================] 100%?? 
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        @     WARNING: plugin requires additional permissions     @
        @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
        * java.net.SocketPermission * connect,resolve
        See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
        for descriptions of what these permissions allow and the associated risks.

        Continue with installation? [y/N]y
        -> Installed analysis-ik
        ```
    - Restart **EVERY** ElasticSearch `docker restart DOCKER_ID`
        ```
        [root@localhost ~]# docker restart ae02ee75b357
        ```
2. Demo tokenizer
    - ik_smart
        ```json
        POST _analyze
        {
          "analyzer": "ik_smart",
          "text": "中华人民共和国国歌"
        }
        ```
        ```json
        {
          "tokens" : [
            {
              "token" : "中华人民共和国",
              "start_offset" : 0,
              "end_offset" : 7,
              "type" : "CN_WORD",
              "position" : 0
            },
            {
              "token" : "国歌",
              "start_offset" : 7,
              "end_offset" : 9,
              "type" : "CN_WORD",
              "position" : 1
            }
          ]
        }
        ```
    - ik_max_word
        ```json
        POST _analyze
        {
          "analyzer": "ik_max_word",
          "text": "中华人民共和国国歌"
        }
        ```
        ```json
        {
         "tokens" : [
            {
              "token" : "中华人民共和国",
              "start_offset" : 0,
              "end_offset" : 7,
              "type" : "CN_WORD",
              "position" : 0
            },
            {
              "token" : "中华人民",
              "start_offset" : 0,
              "end_offset" : 4,
              "type" : "CN_WORD",
              "position" : 1
            },
            {
              "token" : "中华",
              "start_offset" : 0,
              "end_offset" : 2,
              "type" : "CN_WORD",
              "position" : 2
            },
            {
              "token" : "华人",
              "start_offset" : 1,
              "end_offset" : 3,
              "type" : "CN_WORD",
              "position" : 3
            },
            {
              "token" : "人民共和国",
              "start_offset" : 2,
              "end_offset" : 7,
              "type" : "CN_WORD",
              "position" : 4
            },
            {
              "token" : "人民",
              "start_offset" : 2,
              "end_offset" : 4,
              "type" : "CN_WORD",
              "position" : 5
            },
            {
              "token" : "共和国",
              "start_offset" : 4,
              "end_offset" : 7,
              "type" : "CN_WORD",
              "position" : 6
            },
            {
              "token" : "共和",
              "start_offset" : 4,
              "end_offset" : 6,
              "type" : "CN_WORD",
              "position" : 7
            },
            {
              "token" : "国",
              "start_offset" : 6,
              "end_offset" : 7,
              "type" : "CN_CHAR",
              "position" : 8
            },
            {
              "token" : "国歌",
              "start_offset" : 7,
              "end_offset" : 9,
              "type" : "CN_WORD",
              "position" : 9
            }
          ]
        }
        ```
# 4. icu_analyzer
- Innstall
```
[root@bf89bfd7e1a7 elasticsearch]# ./bin/elasticsearch-plugin install analysis-icu
-> Downloading analysis-icu from elastic
[=================================================] 100%?? 
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.bouncycastle.jcajce.provider.drbg.DRBG (file:/usr/share/elasticsearch/lib/tools/plugin-cli/bcprov-jdk15on-1.61.jar) to constructor sun.security.provider.Sun()
WARNING: Please consider reporting this to the maintainers of org.bouncycastle.jcajce.provider.drbg.DRBG
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
-> Installed analysis-icu
```
```
[root@localhost ~]# docker-compose restart
Restarting cerebro ... done
Restarting es7_02  ... done
Restarting es7_01  ... done
Restarting kibana7 ... done
```
- Demo
```
POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "中华人民共和国国歌"
}
```

```
{
  "tokens" : [
    {
      "token" : "中华",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "人民",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "共和国",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "国歌",
      "start_offset" : 7,
      "end_offset" : 9,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    }
  ]
}
```
