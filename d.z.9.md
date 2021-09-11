***Задача 1***
***В этом задании вы потренируетесь в:***
    ***установке elasticsearch***
    ***первоначальном конфигурировании elastcisearch***
    ***запуске elasticsearch в docker***
***Используя докер образ centos:7 как базовый и документацию по установке и запуску Elastcisearch:***
    ***составьте Dockerfile-манифест для elasticsearch***
    ***соберите docker-образ и сделайте push в ваш docker.io репозиторий***
    ***запустите контейнер из получившегося образа и выполните запрос пути / c хост-машины***
***Требования к elasticsearch.yml:***
    ***данные path должны сохраняться в /var/lib***
    ***имя ноды должно быть netology_test***
***В ответе приведите:***
    ***текст Dockerfile манифеста***
    ***ссылку на образ в репозитории dockerhub***
    ***ответ elasticsearch на запрос пути / в json виде***


    Создал докерфайл (из-за того, что elasticsearch не запускается из-под root, и того, что данные path должны сохраняться в /var/lib, придется дать всем полные         права на директорию (как временное решение, на настоящем сервере можно задать гибкие права через ACL)):
    FROM centos:7
    LABEL maintainer "Maks Plaksin"
    WORKDIR /tmp
    RUN yum install -y wget
    RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.14.1-linux-x86_64.tar.gz
    CMD gunzip elasticsearch-7.14.1-linux-x86_64.tar.gz
    CMD tar -xf elasticsearch-7.14.1-linux-x86_64.tar
    CMD mv elasticsearch-7.14.1 /var
    WORKDIR /var/elasticsearch-7.14.1
    EXPOSE 9020
    CMD groupadd elastic
    CMD useradd -g elastic elastic
    CMD chown elastic:elastic -R /var/elasticsearch-7.14.1/
    CMD chmod 777 /var/lib/
    CMD su elastic
    ENTRYPOINT ["/var/elasticsearch-7.14.1/bin/elasticsearch"]

    Затем создал контейнер из докерфайла:
    [max@max_centos 111]$ docker build -t elastic1 -f dockerfile3 .
    Sending build context to Docker daemon   2.56kB
    Step 1/16 : FROM centos:7
    7: Pulling from library/centos
    2d473b07cdd5: Pull complete 
    ...........
    ...........
     ---> Running in 3b1e5e3b8b9d
    Removing intermediate container 3b1e5e3b8b9d
     ---> a35fe3a30cc6
    Successfully built a35fe3a30cc6
    Successfully tagged elastic1:latest

    Загрузил новый образ в докерхаб:
    [max@max_centos 111]$ docker login -u maximuss88
    [max@max_centos 111]$ docker tag elastic1 maximuss88/test:elastic1
    [max@max_centos 111]$ docker push maximuss88/test:elastic1

    https://hub.docker.com/layers/166159901/maximuss88/test/elastic1/images/sha256-c0ae1cb1fc0d9fbbf699fee6aa8e742c53a4cc0ee510590e7a3bac527dab9b95?context=repo&tab=layers

    Потом запустил контейнер, подключился, отредактировал elasticsearch.yml, перезапустил elasticsearch:
    [max@max_centos ~]$ docker exec -it a34d52e7fb66 /bin/bash

    [root@a34d52e7fb66 elasticsearch-7.14.1]# nano /var/elasticsearch-7.14.1/config/elasticsearch.yml
    Раскомментировал и изменил
    node.name: netology_test
    path.data: /var/lib

    И ответ на запрос / (curl http://127.0.0.1:9200):
    {
      "name" : "netology_test",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "dapi0qUFRr-_QkQgWL6mwQ",
      "version" : {
        "number" : "7.14.1",
        "build_flavor" : "default",
        "build_type" : "tar",
        "build_hash" : "66b55ebfa59c92c15db3f69a335d500018b3331e",
        "build_date" : "2021-08-26T09:01:05.390870785Z",
        "build_snapshot" : false,
        "lucene_version" : "8.9.0",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"

**********
***Задача 2
В этом задании вы научитесь:
    создавать и удалять индексы
    изучать состояние кластера
    обосновывать причину деградации доступности данных
Ознакомтесь с документацией и добавьте в elasticsearch 3 индекса, в соответствии со таблицей:
Имя 	Количество реплик 	Количество шард
ind-1   	0 	        1
ind-2 	        1 	        2
ind-3    	2 	        4
Получите список индексов и их статусов, используя API и приведите в ответе на задание.
Получите состояние кластера elasticsearch, используя API.
Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?
Удалите все индексы.***


    Запрашиваем статус кластера:
    [root@e17499370674 /]# curl http://127.0.0.1:9200/_cluster/health?pretty
    {
      "cluster_name" : "elasticsearch",
      "status" : "green",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 1,
      "active_shards" : 1,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 0,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 100.0
    }

    Теперь создадим индексы:
    [root@e17499370674 /]# curl -X PUT "127.0.0.1:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
    > {
    >   "settings": {
    >     "number_of_shards": 1,
    >     "number_of_replicas": 0
    >   }
    > }
    > '
    {
      "acknowledged" : true,
      "shards_acknowledged" : true,
      "index" : "ind-1"
    }

    Затем аналогично ind-2 и ind-3.

    Запросим статус индексов, шардов и еще раз статус кластера:
    [root@e17499370674 /]# curl http://127.0.0.1:9200/_cat/indices?pretty
    green  open .geoip_databases TlSPwsqmSvS7ldGhk454Fw 1 0 42 40 42.3mb 42.3mb
    green  open ind-1            2RMKwlnuRLOJRL1P2lc9Mw 1 0  0  0   208b   208b
    yellow open ind-3            HLgC_Dg7Sk-uBfoFJCLxhQ 4 2  0  0   832b   832b
    yellow open ind-2            RFw7HRJLQxWj6Ca2hNQ6NQ 2 1  0  0   416b   416b

    [root@e17499370674 /]# curl http://127.0.0.1:9200/_cat/shards?pretty
    .geoip_databases 0 p STARTED    42 42.3mb 127.0.0.1 netology_test
    ind-3            2 p STARTED     0   208b 127.0.0.1 netology_test
    ind-3            2 r UNASSIGNED                     
    ind-3            2 r UNASSIGNED                     
    ind-3            3 p STARTED     0   208b 127.0.0.1 netology_test
    ind-3            3 r UNASSIGNED                     
    ind-3            3 r UNASSIGNED                     
    ind-3            1 p STARTED     0   208b 127.0.0.1 netology_test
    ind-3            1 r UNASSIGNED                     
    ind-3            1 r UNASSIGNED                     
    ind-3            0 p STARTED     0   208b 127.0.0.1 netology_test
    ind-3            0 r UNASSIGNED                     
    ind-3            0 r UNASSIGNED                     
    ind-1            0 p STARTED     0   208b 127.0.0.1 netology_test
    ind-2            1 p STARTED     0   208b 127.0.0.1 netology_test
    ind-2            1 r UNASSIGNED                     
    ind-2            0 p STARTED     0   208b 127.0.0.1 netology_test
    ind-2            0 r UNASSIGNED

    [root@e17499370674 /]# curl http://127.0.0.1:9200/_cluster/health?pretty
    {
      "cluster_name" : "elasticsearch",
      "status" : "yellow",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 8,
      "active_shards" : 8,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 10,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 44.44444444444444
    }

    Статус кластера и индексов ind-2 и ind-3 определён как "yellow" потому, что primary шарды в состоянии STARTED, а реплики в состоянии UNASSIGNED (а в индексе ind-1 нет реплик).

    И наконец, удалим все созданные индексы:
    [root@e17499370674 /]# curl -X DELETE "127.0.0.1:9200/ind-1?pretty"
    {
      "acknowledged" : true
    }
    ........

    [root@e17499370674 /]# curl http://127.0.0.1:9200/_cat/indices?pretty
    green open .geoip_databases TlSPwsqmSvS7ldGhk454Fw 1 0 42 40 42.3mb 42.3mb
 
**********
Задача 3
В данном задании вы научитесь:
    создавать бэкапы данных
    восстанавливать индексы из бэкапов
Создайте директорию {путь до корневой директории с elasticsearch в образе}/snapshots.
Используя API зарегистрируйте данную директорию как snapshot repository c именем netology_backup.
Приведите в ответе запрос API и результат вызова API для создания репозитория.
Создайте индекс test с 0 реплик и 1 шардом и приведите в ответе список индексов.
Создайте snapshot состояния кластера elasticsearch.
Приведите в ответе список файлов в директории со snapshotами.
Удалите индекс test и создайте индекс test-2. Приведите в ответе список индексов.
Восстановите состояние кластера elasticsearch из snapshot, созданного ранее.
Приведите в ответе запрос к API восстановления и итоговый список индексов.


Создал каталог /var/elasticsearch-7.14.1/snapshots/, добавил в конфиг elasticsearch.yml строку (path.repo: /var/elasticsearch-7.14.1/snapshots/), перезапустил elasticsearch и зарегистрировал snapshot repository:
[root@e17499370674 elasticsearch-7.14.1]# curl -X PUT "127.0.0.1:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
> {
>   "type": "fs",
>   "settings": {
>     "location": "my_backup_location"
>   }
> }
> '
{
  "acknowledged" : true
}

Затем создаём индекс test и выводим список индексов:
[root@e17499370674 elasticsearch-7.14.1]# curl -X PUT "127.0.0.1:9200/test?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "number_of_shards": 1,
>     "number_of_replicas": 0
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test"
}

[root@e17499370674 elasticsearch-7.14.1]# curl http://127.0.0.1:9200/_cat/indices?pretty
green open .geoip_databases TlSPwsqmSvS7ldGhk454Fw 1 0 42 40 42.3mb 42.3mb
green open test             7T1y4CT0TZSrN--tNuMbiQ 1 0  0  0   208b   208b

Теперь создаём snapshot:
[root@e17499370674 elasticsearch-7.14.1]# curl -X PUT "127.0.0.1:9200/_snapshot/netology_backup/snapshot_1?wait_for_completion=true&pretty"
{
  "snapshot" : {
    "snapshot" : "snapshot_1",
    "uuid" : "cuCeYR8gRfKZ27pOfs95rQ",
    "repository" : "netology_backup",
    "version_id" : 7140199,
    "version" : "7.14.1",
    "indices" : [
      "test",
      ".geoip_databases"
    ],
............

[root@e17499370674 my_backup_location]# ls -la
total 40
drwxrwxr-x 3 elastic elastic   134 Sep 11 12:20 .
drwxrwxr-x 3 elastic elastic    32 Sep 11 12:15 ..
-rw-rw-r-- 1 elastic elastic   828 Sep 11 12:20 index-0
-rw-rw-r-- 1 elastic elastic     8 Sep 11 12:20 index.latest
drwxrwxr-x 4 elastic elastic    66 Sep 11 12:20 indices
-rw-rw-r-- 1 elastic elastic 27659 Sep 11 12:20 meta-cuCeYR8gRfKZ27pOfs95rQ.dat
-rw-rw-r-- 1 elastic elastic   437 Sep 11 12:20 snap-cuCeYR8gRfKZ27pOfs95rQ.dat

Удаляем первый индекс и создаём второй:
[root@e17499370674 my_backup_location]# curl -X DELETE "127.0.0.1:9200/test?pretty"
{
  "acknowledged" : true
}

[root@e17499370674 my_backup_location]# curl -X PUT "127.0.0.1:9200/test-2?pretty" -H 'Content-Type: application/json' -d'
> {
>   "settings": {
>     "number_of_shards": 2,
>     "number_of_replicas": 0
>   }
> }
> '
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test-2"
}

[root@e17499370674 my_backup_location]# curl http://127.0.0.1:9200/_cat/indices?pretty
green open .geoip_databases TlSPwsqmSvS7ldGhk454Fw 1 0 42 40 42.3mb 42.3mb
green open test-2           3z8izElVTHSBCrNnhzt9qg 2 0  0  0   416b   416b

И теперь восстанавливаем кластер из бэкапа (здесь возникла неожиданная проблема - ошибка восстановления .geoip_databases, остановить или удалить этот системный индекс непросто и это не рекомендуется, поэтому указал при восстановлении конкретно индекс test):
[root@e17499370674 my_backup_location]# curl -X POST "127.0.0.1:9200/_snapshot/netology_backup/snapshot_1/_restore?pretty" -H 'Content-Type: application/json' -d'
> {
>   "indices": "test"
> }
> '
{
  "accepted" : true
}

Индекс успешно восстановлен:
[root@e17499370674 my_backup_location]# curl http://127.0.0.1:9200/_cat/indices?pretty
green open .geoip_databases TlSPwsqmSvS7ldGhk454Fw 1 0 42 40 42.3mb 42.3mb
green open test-2           3z8izElVTHSBCrNnhzt9qg 2 0  0  0   416b   416b
green open test             1SBq1TKmR8e0CcQNBTU3Pg 1 0  0  0   208b   208b

