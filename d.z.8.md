https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql  

***Задача 1  
Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.  
Подключитесь к БД PostgreSQL используя psql.  
Воспользуйтесь командой \? для вывода подсказки по имеющимся в psql управляющим командам.  
Найдите и приведите управляющие команды для:  
    вывода списка БД  
    подключения к БД  
    вывода списка таблиц  
    вывода описания содержимого таблиц  
    выхода из psql***  

<<<<<<< HEAD
Поднял контейнер c PostgreSQL, подключился к нему (docker exec -it 76fc79fbac69 bash) и к БД (psql -U postgres), нашел команды:  
1. \l[+]   [PATTERN]      list databases  
2. \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}   connect to new database (currently "postgres")  
3. \d[S+]                 list tables, views, and sequences  
4. \d[S+]  NAME           describe table, view, sequence, or index  
5. \q                     quit psql  
=======
    Поднял контейнер и подключился, нашел команды:  
    1. \l[+]   [PATTERN]      list databases  
    2. \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}   connect to new database (currently "postgres")  
    3. \d[S+]                 list tables, views, and sequences  
    4. \d[S+]  NAME           describe table, view, sequence, or index  
    5. \q                     quit psql  
>>>>>>> 419e473a712fb2aef66340dd5d2e3215bc8e6bde
    
**********    
Задача 2  
Используя psql создайте БД test_database.  
Изучите бэкап БД.  
Восстановите бэкап БД в test_database.  
Перейдите в управляющую консоль psql внутри контейнера.  
Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.  
Используя таблицу pg_stats, найдите столбец таблицы orders с наибольшим средним значением размера элементов в байтах.  
Приведите в ответе команду, которую вы использовали для вычисления и полученный результат.  


Бэкап успешно восстановил  
root@76fc79fbac69:/# psql -U postgres test_database < /tmp/test_dump3.sql  
SET  
SET  
SET  
SET  
SET  
 set_config  
------------  
 (1 row)  
SET  
SET  
SET  
SET  
SET  
SET  
CREATE TABLE  
ALTER TABLE  
CREATE SEQUENCE  
ALTER TABLE  
ALTER SEQUENCE  
ALTER TABLE  
COPY 8  
 setval  
--------  
      8  
(1 row)  
ALTER TABLE  


Затем сделал анализ таблицы (очень длинный, привёл только начало)  
postgres=# \c test_database  
You are now connected to database "test_database" as user "postgres".  
test_database=# ANALYZE VERBOSE test_database;  
ERROR:  relation "test_database" does not exist  
test_database=# ANALYZE VERBOSE;  
INFO:  analyzing "public.orders"  
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows  
INFO:  analyzing "pg_catalog.pg_type"  
INFO:  "pg_type": scanned 10 of 10 pages, containing 414 live rows and 0 dead rows; 414 rows in sample, 414 estimated total rows  
INFO:  analyzing "pg_catalog.pg_foreign_table"  
INFO:  "pg_foreign_table": scanned 0 of 0 pages, containing 0 live rows and 0 dead rows; 0 rows in sample, 0 estimated total rows  
INFO:  analyzing "pg_catalog.pg_authid"  
INFO:  "pg_authid": scanned 1 of 1 pages, containing 11 live rows and 0 dead rows; 11 rows in sample, 11 estimated total rows  
...........  


Таблица pg_stats содержит информацию о других таблицах, запросим для orders:  

postgres=# \c test_database 
You are now connected to database "test_database" as user "postgres".

test_database=# select avg_width from pg_stats where tablename = 'orders';
 avg_width 
-----------
         4
        16
         4
(3 rows)

Соответственно столбец с наибольшим средним значением элемента - второй, то есть title, и действительно:

test_database=# select * from orders;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  2 | My little database   |   500
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  6 | WAL never lies       |   900
  7 | Me and my bash-pet   |   499
  8 | Dbiezdmin            |   501
(8 rows)

**********  
Задача 3  
Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).  
Предложите SQL-транзакцию для проведения данной операции.  
Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?  

Для старой таблицы оставим (price>499), просто переименуем таблицу в orders_1, и создадим новую:  
CREATE TABLE orders_2 PARTITION OF orders_1 FOR VALUES IN (0..499);  
ALTER TABLE orders_1 DETACH PARTITION orders_2;  

Думаю да, можно создать заранее несколько секций для разных диапазонов значений.  

**********  
Задача 4  
Используя утилиту pg_dump создайте бекап БД test_database.  
Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца title для таблиц test_database?  


test_database=# \q  
root@76fc79fbac69:/# pg_dump -U postgres test_database > /tmp/test_database.sql  
root@76fc79fbac69:/# ls -la /tmp  
total 180  
drwxrwxrwt 1 root root     88 Aug 21 15:11 .  
drwxr-xr-x 1 root root     51 Aug 21 13:19 ..  
-rw-r--r-- 1 root root    541 Aug 18 19:17 backup  
-rw-r--r-- 1 root root   2082 Aug 21 15:11 test_database.sql  
-rw-rw-r-- 1 1000 1000   2083 Aug 21 13:18 test_dump3.sql  
-rw-rw-r-- 1 1000 1000 171427 Aug 20 16:29 test_dump.sql  

<<<<<<< HEAD

Столбец title есть только в таблице orders:

test_database=# select * from orders;
 id |        title         | price 
----+----------------------+-------
  1 | War and peace        |   100
  2 | My little database   |   500
  3 | Adventure psql time  |   300
  4 | Server gravity falls |   300
  5 | Log gossips          |   123
  6 | WAL never lies       |   900
  7 | Me and my bash-pet   |   499
  8 | Dbiezdmin            |   501
(8 rows)

Добавить уникальность значения столбца существующей таблицы можно так:
ALTER TABLE orders ADD CONSTRAINT title_uniq UNIQUE (title);
=======
Думаю, если таблица секционирована, то можно создать primary индекс для каждой секции, а если не секционирована, то проблем быть не должно.  
>>>>>>> 419e473a712fb2aef66340dd5d2e3215bc8e6bde
