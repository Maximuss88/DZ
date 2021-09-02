https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql  

***Задача 1  
Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.  
Изучите бэкап БД и восстановитесь из него.  
Перейдите в управляющую консоль mysql внутри контейнера.  
Используя команду \h получите список управляющих команд.  
Найдите команду для выдачи статуса БД и приведите в ответе из ее вывода версию сервера БД.  
Подключитесь к восстановленной БД и получите список таблиц из этой БД.  
Приведите в ответе количество записей с price > 300.  
В следующих заданиях мы будем продолжать работу с данным контейнером.***  

Скачал нужную версию, запустил и подключился:  

[max@max_centos docker]$ docker search mysql  
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED  
mysql                             MySQL is a widely used, open-source relation…   11342     [OK]  
.................  

[max@max_centos docker]$ docker pull mysql  

[max@max_centos docker]$ docker inspect mysql  
..................  
"MYSQL_MAJOR=8.0",  
                "MYSQL_VERSION=8.0.26-1debian10"  
...............  

[max@max_centos ~]$ docker run --name MySQL -e MYSQL_ROOT_PASSWORD=password123 -d mysql  
ed75c33e2d60685c8677ded13d8557738824ade61c6a08567881cdbd59579723  

[max@max_centos ~]$ docker exec -it ed75c33e2d60 /bin/bash  
root@ed75c33e2d60:/#  

Затем скачал файл бэкапа, закинул в контейнер (docker cp /tmp/test_dump.sql ed75c33e2d60:/tmp/) и развернул:  
mysql> CREATE DATABASE test_dump;  
Query OK, 1 row affected (0.01 sec)  
root@ed75c33e2d60:/# mysql -u root -p test_dump < /tmp/test_dump.sql  

Команда для выдачи статуса БД:  
root@ed75c33e2d60:/# mysql -u root -p  
Enter password:  
Welcome to the MySQL monitor.  Commands end with ; or \g.  
................  
    mysql> \s  
    --------------  
    mysql  Ver 8.0.26 for Linux on x86_64 (MySQL Community Server - GPL)  

    Connection id:          13  
    Current database:  
    Current user:           root@localhost  
    SSL:                    Not in use  
    Current pager:          stdout  
    Using outfile:          ''  
    Using delimiter:        ;  
    Server version:         8.0.26 MySQL Community Server - GPL  
    Protocol version:       10  
    Connection:             Localhost via UNIX socket  
    Server characterset:    utf8mb4  
    Db     characterset:    utf8mb4  
    Client characterset:    latin1  
    Conn.  characterset:    latin1  
    UNIX socket:            /var/run/mysqld/mysqld.sock  
    Binary data as:         Hexadecimal  
    Uptime:                 30 min 31 sec  

    Threads: 2  Questions: 81  Slow queries: 0  Opens: 158  Flush tables: 3  Open tables: 75  Queries per second avg: 0.044  
    --------------  

Затем подключился к БД и выполнил запросы:  

mysql> USE test_dump;  
Reading table information for completion of table and column names  
You can turn off this feature to get a quicker startup with -A  
Database changed  

mysql> SHOW TABLES;  
+---------------------+  
| Tables_in_test_dump |  
+---------------------+  
| orders              |  
+---------------------+  
1 row in set (0.00 sec)  

mysql> SELECT * FROM orders WHERE price > 300;  
+----+----------------+-------+  
| id | title          | price |  
+----+----------------+-------+  
|  2 | My little pony |   500 |  
+----+----------------+-------+  
1 row in set (0.00 sec)  

Или сразу так:  
mysql> SELECT count(*) FROM orders WHERE price > 300;  
+----------+  
| count(*) |  
+----------+  
|        1 |  
+----------+  
1 row in set (0.00 sec)  

**********  
***Задача 2  
Создайте пользователя test в БД c паролем test-pass, используя:  
    плагин авторизации mysql_native_password  
    срок истечения пароля - 180 дней  
    количество попыток авторизации - 3  
    максимальное количество запросов в час - 100  
    аттрибуты пользователя:  
        Фамилия "Pretty"  
        Имя "James"  
Предоставьте привелегии пользователю test на операции SELECT базы test_db.  
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю test и приведите в ответе к задаче.***  

CREATE USER test;  

ALTER USER 'test'  
IDENTIFIED WITH mysql_native_password BY 'password123'  
PASSWORD EXPIRE INTERVAL 180 DAY  
FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 1  
WITH MAX_CONNECTIONS_PER_HOUR 100;  

GRANT SELECT ON test_dump.* TO 'test';  

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';  
+------+------+-----------+  
| USER | HOST | ATTRIBUTE |  
+------+------+-----------+  
| test | %    | NULL      |  
+------+------+-----------+  
1 row in set (0.00 sec)  

Не нашел других способов изменить атрибуты пользователя, кроме этого, однако недостаточно прав даже у root:  
mysql> UPDATE INFORMATION_SCHEMA.USER_ATTRIBUTES SET ATTRIBUTE='James Pretty' WHERE USER='test';  
ERROR 1044 (42000): Access denied for user 'root'@'localhost' to database 'information_schema'  

**********  
***Задача 3  
Установите профилирование SET profiling = 1. Изучите вывод профилирования команд SHOW PROFILES;.  
Исследуйте, какой engine используется в таблице БД test_db и приведите в ответе.  
Измените engine и приведите время выполнения и запрос на изменения из профайлера в ответе:  
    на MyISAM  
    на InnoDB***  

Устанавливаем профилирование и смотрим доступные движки:  
mysql> SET profiling = 1;  
Query OK, 0 rows affected, 1 warning (0.00 sec)  

    mysql> show engines;  
    +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+  
    | Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |  
    +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+  
    | FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |  
    | MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |  
    | InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |  
    | PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |  
    | MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |  
    | MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |  
    | BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |  
    | CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |  
    | ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |  
    +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+  
    9 rows in set (0.01 sec)  

    Узнаем, какой движок у нашей таблицы и проверим на нем запросы:  
    mysql> SHOW TABLE STATUS FROM test_dump LIKE 'orders';  
    Name    Engine  ...  
    orders  InnoDB  ...  
    .....  

mysql> SELECT * FROM orders;  

    mysql> SHOW PROFILES;  
    +----------+------------+------------------------------------------------+  
    | Query_ID | Duration   | Query                                          |  
    +----------+------------+------------------------------------------------+  
    |        1 | 0.00397125 | show engines                                   |  
    |        2 | 0.06046500 | SHOW TABLE STATUS FROM test_dump LIKE 'orders' |  
    |        3 | 0.00389125 | SELECT * FROM orders                           |  
    +----------+------------+------------------------------------------------+  
    3 rows in set, 1 warning (0.00 sec)  

    mysql> SHOW PROFILE FOR QUERY 3;  
    +--------------------------------+----------+  
    | Status                         | Duration |  
    +--------------------------------+----------+  
    | starting                       | 0.000160 |  
    | Executing hook on transaction  | 0.002958 |  
    | starting                       | 0.000039 |  
    | checking permissions           | 0.000048 |  
    | Opening tables                 | 0.000092 |  
    | init                           | 0.000028 |  
    | System lock                    | 0.000035 |  
    | optimizing                     | 0.000022 |  
    | statistics                     | 0.000048 |  
    | preparing                      | 0.000057 |  
    | executing                      | 0.000171 |  
    | end                            | 0.000024 |  
    | query end                      | 0.000019 |  
    | waiting for handler commit     | 0.000031 |  
    | closing tables                 | 0.000032 |  
    | freeing items                  | 0.000056 |  
    | cleaning up                    | 0.000073 |  
    +--------------------------------+----------+  
    17 rows in set, 1 warning (0.00 sec)  

Затем изменим его на MyISAM и проверим то же самое с ним:  
mysql> ALTER TABLE orders ENGINE = MyISAM;  
Query OK, 5 rows affected (0.12 sec)  
Records: 5  Duplicates: 0  Warnings: 0  

    mysql> SHOW TABLE STATUS FROM test_dump LIKE 'orders';  
    Name    Engine  ...  
    orders  MyISAM  ...  
    .....  

mysql> SELECT * FROM orders;  

    mysql> SHOW PROFILES;  
    +----------+------------+------------------------------------------------+  
    | Query_ID | Duration   | Query                                          |  
    +----------+------------+------------------------------------------------+  
    |        1 | 0.00397125 | show engines                                   |  
    |        2 | 0.06046500 | SHOW TABLE STATUS FROM test_dump LIKE 'orders' |  
    |        3 | 0.00389125 | SELECT * FROM orders                           |  
    |        4 | 0.12465500 | ALTER TABLE orders ENGINE = MyISAM             |  
    |        5 | 0.00462175 | SHOW TABLE STATUS FROM test_dump LIKE 'orders' |  
    |        6 | 0.00076250 | SELECT * FROM orders                           |  
    +----------+------------+------------------------------------------------+  
    6 rows in set, 1 warning (0.00 sec)  

    mysql> SHOW PROFILE FOR QUERY 6;  
    +--------------------------------+----------+  
    | Status                         | Duration |  
    +--------------------------------+----------+  
    | starting                       | 0.000129 |  
    | Executing hook on transaction  | 0.000023 |  
    | starting                       | 0.000025 |  
    | checking permissions           | 0.000021 |  
    | Opening tables                 | 0.000112 |  
    | init                           | 0.000023 |  
    | System lock                    | 0.000031 |  
    | optimizing                     | 0.000021 |  
    | statistics                     | 0.000038 |  
    | preparing                      | 0.000047 |  
    | executing                      | 0.000150 |  
    | end                            | 0.000022 |  
    | query end                      | 0.000022 |  
    | closing tables                 | 0.000026 |  
    | freeing items                  | 0.000040 |  
    | cleaning up                    | 0.000034 |  
    +--------------------------------+----------+  
    16 rows in set, 1 warning (0.00 sec)  

Таким образом, видим, что селекты на движке MyISAM выполняются быстрее.  

**********  
***Задача 4  
Изучите файл my.cnf в директории /etc/mysql.  
Измените его согласно ТЗ (движок InnoDB):  
    Скорость IO важнее сохранности данных  
    Нужна компрессия таблиц для экономии места на диске  
    Размер буффера с незакомиченными транзакциями 1 Мб  
    Буффер кеширования 30% от ОЗУ  
    Размер файла логов операций 100 Мб  
Приведите в ответе измененный файл my.cnf.***  

Вывод файла ниже, добавил в него следующие параметры:  

Скорость IO важнее сохранности данных (innodb_flush_log_at_trx_commit = 2)  
Нужна компрессия таблиц для экономии места на диске (innodb_file_per_table = 1)  
Размер буффера с незакомиченными транзакциями 1 Мб (innodb_log_buffer_size)  
Буффер кеширования 30% от ОЗУ (innodb_buffer_pool_size, у меня 4 Гб ОЗУ, значит выставим 1,3 Гб)  
Размер файла логов операций 100 Мб (innodb_log_file_size = 100 мб, на диске займет 200 мб, поскольку файлов логов всегда два)  

    root@ed75c33e2d60:/# cat /etc/mysql/my.cnf  
    # Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.  
    #  
    # This program is free software; you can redistribute it and/or modify  
    # it under the terms of the GNU General Public License as published by  
    # the Free Software Foundation; version 2 of the License.  
    #  
    # This program is distributed in the hope that it will be useful,  
    # but WITHOUT ANY WARRANTY; without even the implied warranty of  
    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the  
    # GNU General Public License for more details.  
    #  
    # You should have received a copy of the GNU General Public License  
    # along with this program; if not, write to the Free Software  
    # Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA  

    #  
    # The MySQL  Server configuration file.  
    #  
    # For explanations see  
    # http://dev.mysql.com/doc/mysql/en/server-system-variables.html  

    [mysqld]  
    pid-file        = /var/run/mysqld/mysqld.pid  
    socket          = /var/run/mysqld/mysqld.sock  
    datadir         = /var/lib/mysql  
    secure-file-priv= NULL  

    innodb_flush_log_at_trx_commit = 2  
    innodb_file_per_table          = 1  
    innodb_log_buffer_size         = 1M  
    innodb_buffer_pool_size        = 1300M  
    innodb_log_file_size           = 100M  

    # Custom config should go here  
    !includedir /etc/mysql/conf.d/  
