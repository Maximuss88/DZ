###Задача 1  
**Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.**  
**Приведите получившуюся команду или docker-compose манифест.**  


Скачал образ PostgreSQL  
docker pull postgres  
Запустил два контейнера (основной и для бэкапов)  
docker run --name postgres1 -e POSTGRES_PASSWORD=111 -d postgres  
docker run --name postgres2 -e POSTGRES_PASSWORD=123 -d postgres  
Затем создал в докере сеть и добавил в нее оба контейнера  
docker network create postgres-connect  
docker network connect postgres-connect postgres1  
docker network connect postgres-connect postgres2  

**********  
Задача 2  
В БД из задачи 1:  
    создайте пользователя test-admin-user и БД test_db  
    в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)  
    предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db  
    создайте пользователя test-simple-user  
    предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db  
Таблица orders:  
    id (serial primary key)  
    наименование (string)  
    цена (integer)  
Таблица clients:  
    id (serial primary key)  
    фамилия (string)  
    страна проживания (string, index)  
    заказ (foreign key orders)  
Приведите:  
    итоговый список БД после выполнения пунктов выше,  
    описание таблиц (describe)  
    SQL-запрос для выдачи списка пользователей с правами над таблицами test_db  
    список пользователей с правами над таблицами test_db  

    Список БД (\list или SELECT datname FROM pg_database)  
    postgres=# SELECT datname FROM pg_database;  
    datname  
    -----------  
 postgres  
 test_db  
 template1  
 template0  
(4 rows)  

Описание таблиц (\d+ tablename)  
postgres=# \d+ orders;  
                                                       Table "public.orders"  
 Column |         Type          | Collation | Nullable |              Default               | Storage  | Stats target | Description  
--------+-----------------------+-----------+----------+------------------------------------+----------+--------------+-------------  
 id     | integer               |           | not null | nextval('orders_id_seq'::regclass) | plain    |              |  
 name   | character varying(50) |           | not null |                                    | extended |              |  
 price  | integer               |           | not null |                                    | plain    |              |  
Indexes:  
    "orders_pkey" PRIMARY KEY, btree (id)  
Referenced by:  
    TABLE "clients" CONSTRAINT "clients_id_order_fkey" FOREIGN KEY (id_order) REFERENCES orders(id)  
Access method: heap  

postgres=# \d+ clients;  
                                                        Table "public.clients"  
  Column  |         Type          | Collation | Nullable |               Default               | Storage  | Stats target | Description  
----------+-----------------------+-----------+----------+-------------------------------------+----------+--------------+-------------  
 id       | integer               |           | not null | nextval('clients_id_seq'::regclass) | plain    |              |  
 name_2   | character varying(50) |           | not null |                                     | extended |              |  
 country  | character varying(50) |           |          |                                     | extended |              |  
 id_order | integer               |           |          |                                     | plain    |              |  
Indexes:  
    "clients_pkey" PRIMARY KEY, btree (id)  
Foreign-key constraints:  
    "clients_id_order_fkey" FOREIGN KEY (id_order) REFERENCES orders(id)  
Access method: heap  

Селекты для просмотра пользователей и прав  
postgres=# SELECT table_catalog, table_schema, table_name, privilege_type FROM information_schema.table_privileges WHERE grantee = 'test_admin_user';  
 table_catalog | table_schema | table_name | privilege_type  
---------------+--------------+------------+----------------  
(0 rows)  
Видимо, полные права воспринимаются как отсутствие особых разрешений и не показываются.  

postgres=# SELECT table_catalog, table_schema, table_name, privilege_type FROM information_schema.table_privileges WHERE grantee = 'test_simple_user';  
 table_catalog | table_schema | table_name | privilege_type  
---------------+--------------+------------+----------------  
 postgres      | public       | orders     | INSERT  
 postgres      | public       | orders     | SELECT  
 postgres      | public       | orders     | UPDATE  
 postgres      | public       | orders     | DELETE  
 postgres      | public       | clients    | INSERT  
 postgres      | public       | clients    | SELECT  
 postgres      | public       | clients    | UPDATE  
 postgres      | public       | clients    | DELETE  

**********  
Задача 3  
Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:  
Таблица orders  
Наименование 	цена  
Шоколад 	10  
Принтер 	3000  
Книга   	500  
Монитор 	7000  
Гитара  	4000  
Таблица clients  
ФИО 	               Страна проживания  
Иванов Иван Иванович 	USA  
Петров Петр Петрович 	Canada  
Иоганн Себастьян Бах 	Japan  
Ронни Джеймс Дио 	Russia  
Ritchie Blackmore 	Russia  
Используя SQL синтаксис:  
    вычислите количество записей для каждой таблицы  
    приведите в ответе:  
        запросы  
        результаты их выполнения.  

С помощью операций вставки (INSERT INTO orders (name, price) VALUES ('Шоколад', 10); и т.д.) заполнил обе таблицы  

postgres=# SELECT COUNT(*) FROM orders;  
 count  
-------  
     5  
(1 row)  

postgres=# SELECT COUNT(*) FROM clients;  
 count  
-------  
     5  
(1 row)  

**********
Задача 4  
Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.  
Используя foreign keys свяжите записи из таблиц, согласно таблице:  
ФИО             	Заказ  
Иванов Иван Иванович 	Книга  
Петров Петр Петрович 	Монитор  
Иоганн Себастьян Бах 	Гитара  
Приведите SQL-запросы для выполнения данных операций.  
Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.  
Подсказк - используйте директиву UPDATE.  


Как я понял, нужно создать промежуточную таблицу, наполнить её с помощью UPDATE, а затем просто выбрать из неё пользователей:  
create table table3 (fio varchar(50) not null, zakaz varchar(30) not null);  
UPDATE table3 SET fio = (SELECT name_2 FROM clients WHERE id = 1 AND id = 2 AND id = 3);  
UPDATE table3 SET zakaz = (SELECT name FROM orders WHERE id = 3 AND id = 4 AND id = 5);  

postgres=# SELECT fio FROM table3;  
         fio          
----------------------  
 Иванов Иван Иванович  
 Петров Петр Петрович  
 Иоганн Себастьян Бах  
(3 rows)  

**********  
Задача 5  
Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).  
Приведите получившийся результат и объясните что значат полученные значения.  


postgres=# EXPLAIN SELECT fio FROM table3;  
                        QUERY PLAN                         
-----------------------------------------------------------  
 Seq Scan on table3  (cost=0.00..13.60 rows=360 width=118)  
(1 row)  

postgres=# EXPLAIN ANALYZE SELECT fio FROM table3;  
                                             QUERY PLAN                                              
-----------------------------------------------------------------------------------------------------  
 Seq Scan on table3  (cost=0.00..13.60 rows=360 width=118) (actual time=0.316..0.333 rows=3 loops=1)  
 Planning Time: 0.119 ms  
 Execution Time: 1.380 ms  
(3 rows)  

EXPLAIN выполняет заданный запрос и возвращает статистику - время выполнения, кол-во прочитанных строк и т.д.  
COSTS выводит "стоимость" запроса - ожидаемую нагрузку на ресурсы.  

**********  
Задача 6  
Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).  
Остановите контейнер с PostgreSQL (но не удаляйте volumes).  
Поднимите новый пустой контейнер с PostgreSQL.  
Восстановите БД test_db в новом контейнере.  
Приведите список операций, который вы применяли для бэкапа данных и восстановления.  


Выходим из psql и делаем бэкап БД с помощью pg_dump из-под root:  
pg_dump -U postgres test_db > /tmp/backup  

Перекидываем файл бэкапа во второй контейнер и восстанавливаем (перед этим создаем пользователя и пустую БД):  
psql -U postgres test_db < /tmp/backup  
