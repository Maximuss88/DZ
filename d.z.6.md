������ 1  
��������� docker ��������� ������� PostgreSQL (������ 12) c 2 volume, � ������� ����� ������������ ������ �� � ������.  
��������� ������������ ������� ��� docker-compose ��������.  


������ ����� PostgreSQL  
docker pull postgres  
�������� ��� ���������� (�������� � ��� �������)  
docker run --name postgres1 -e POSTGRES_PASSWORD=111 -d postgres  
docker run --name postgres2 -e POSTGRES_PASSWORD=123 -d postgres  
����� ������ � ������ ���� � ������� � ��� ��� ����������  
docker network create postgres-connect  
docker network connect postgres-connect postgres1  
docker network connect postgres-connect postgres2  

**********  
������ 2  
� �� �� ������ 1:  
    �������� ������������ test-admin-user � �� test_db  
    � �� test_db �������� ������� orders � clients (��e��������� ������ ����)  
    ������������ ���������� �� ��� �������� ������������ test-admin-user �� ������� �� test_db  
    �������� ������������ test-simple-user  
    ������������ ������������ test-simple-user ����� �� SELECT/INSERT/UPDATE/DELETE ������ ������ �� test_db  
������� orders:  
    id (serial primary key)  
    ������������ (string)  
    ���� (integer)  
������� clients:  
    id (serial primary key)  
    ������� (string)  
    ������ ���������� (string, index)  
    ����� (foreign key orders)  
���������:  
    �������� ������ �� ����� ���������� ������� ����,  
    �������� ������ (describe)  
    SQL-������ ��� ������ ������ ������������� � ������� ��� ��������� test_db  
    ������ ������������� � ������� ��� ��������� test_db  


������ �� (\list ��� SELECT datname FROM pg_database)  
postgres=# SELECT datname FROM pg_database;  
  datname  
-----------  
 postgres  
 test_db  
 template1  
 template0  
(4 rows)  

�������� ������ (\d+ tablename)  
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

������� ��� ��������� ������������� � ����  
postgres=# SELECT table_catalog, table_schema, table_name, privilege_type FROM information_schema.table_privileges WHERE grantee = 'test_admin_user';  
 table_catalog | table_schema | table_name | privilege_type  
---------------+--------------+------------+----------------  
(0 rows)  
������, ������ ����� �������������� ��� ���������� ������ ���������� � �� ������������.  

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
������ 3  
��������� SQL ��������� - ��������� ������� ���������� ��������� �������:  
������� orders  
������������ 	����  
������� 	10  
������� 	3000  
�����   	500  
������� 	7000  
������  	4000  
������� clients  
��� 	               ������ ����������  
������ ���� �������� 	USA  
������ ���� �������� 	Canada  
������ ��������� ��� 	Japan  
����� ������ ��� 	Russia  
Ritchie Blackmore 	Russia  
��������� SQL ���������:  
    ��������� ���������� ������� ��� ������ �������  
    ��������� � ������:  
        �������  
        ���������� �� ����������.  


� ������� �������� ������� (INSERT INTO orders (name, price) VALUES ('�������', 10); � �.�.) �������� ��� �������  

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
������ 4  
����� ������������� �� ������� clients ������ �������� ������ �� ������� orders.  
��������� foreign keys ������� ������ �� ������, �������� �������:  
���             	�����  
������ ���� �������� 	�����  
������ ���� �������� 	�������  
������ ��������� ��� 	������  
��������� SQL-������� ��� ���������� ������ ��������.  
��������� SQL-������ ��� ������ ���� �������������, ������� ��������� �����, � ����� ����� ������� �������.  
�������� - ����������� ��������� UPDATE.  


��� � �����, ����� ������� ������������� �������, ��������� � � ������� UPDATE, � ����� ������ ������� �� �� �������������:  
create table table3 (fio varchar(50) not null, zakaz varchar(30) not null);  
UPDATE table3 SET fio = (SELECT name_2 FROM clients WHERE id = 1 AND id = 2 AND id = 3);  
UPDATE table3 SET zakaz = (SELECT name FROM orders WHERE id = 3 AND id = 4 AND id = 5);  

postgres=# SELECT fio FROM table3;  
         fio          
----------------------  
 ������ ���� ��������  
 ������ ���� ��������  
 ������ ��������� ���  
(3 rows)  

**********  
������ 5  
�������� ������ ���������� �� ���������� ������� ������ ���� ������������� �� ������ 4 (��������� ��������� EXPLAIN).  
��������� ������������ ��������� � ��������� ��� ������ ���������� ��������.  


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

EXPLAIN ��������� �������� ������ � ���������� ���������� - ����� ����������, ���-�� ����������� ����� � �.�.  
COSTS ������� "���������" ������� - ��������� �������� �� �������.  

**********  
������ 6  
�������� ����� �� test_db � ��������� ��� � volume, ��������������� ��� ������� (��. ������ 1).  
���������� ��������� � PostgreSQL (�� �� �������� volumes).  
��������� ����� ������ ��������� � PostgreSQL.  
������������ �� test_db � ����� ����������.  
��������� ������ ��������, ������� �� ��������� ��� ������ ������ � ��������������.  


������� �� psql � ������ ����� �� � ������� pg_dump ��-��� root:  
pg_dump -U postgres test_db > /tmp/backup  

������������ ���� ������ �� ������ ��������� � ��������������� (����� ���� ������� ������������ � ������ ��):  
psql -U postgres test_db < /tmp/backup  
