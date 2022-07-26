# Postgres
Для экспериментов буду пользоваться созданным на предыдущих уроках кластером Postgres 14
Ну и дальше по пунктам
1 создайте новый кластер PostgresSQL 13 (на выбор - GCE, CloudSQL)  
2 зайдите в созданный кластер под пользователем postgres  
3 создайте новую базу данных testdb  
```
postgres=# create database testdb;
CREATE DATABASE
postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(4 rows)
```
4 зайдите в созданную базу данных под пользователем postgres
```
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=#
```
5 создайте новую схему testnm  
6 создайте новую таблицу t1 с одной колонкой c1 типа integer  
7 вставьте строку со значением c1=1  
```
testdb=# CREATE SCHEMA testnm;
CREATE SCHEMA
testdb=# CREATE TABLE t1(c1 integer);
CREATE TABLE
testdb=# INSERT INTO t1 values(1);
INSERT 0 1
testdb=# select * from t1;
 c1
----
  1
(1 row)
```
8 создайте новую роль readonly  
9 дайте новой роли право на подключение к базе данных testdb  
10 дайте новой роли право на использование схемы testnm  
11 дайте новой роли право на select для всех таблиц схемы testnm  
```
testdb=# CREATE role readonly;
CREATE ROLE
testdb=# GRANT CONNECT ON DATABASE TESTDB TO READONLY;
GRANT
testdb=# GRANT CONNECT ON DATABASE TESTDB1 TO READONLY;
ERROR:  database "testdb1" does not exist
testdb=# GRANT USAGE ON SCHEMA TESTNM TO READONLY;
GRANT
testdb=# GRANT SELECT ON ALL TABLES IN SCHEMA TESTNM TO READONLY;
GRANT
```
*Попробовал, что будет если задать имя БД большими буквами. Postgres понял, но ответил маленькими.  
Попробовал, что будет если задать неправильное имя. Вдруг не ругаеться. Ругается.*  
12 создайте пользователя testread с паролем test123  
13 дайте роль readonly пользователю testread  
14 зайдите под пользователем testread в базу данных testdb  
```
testdb=# CREATE USER TESTREAD WITH PASSWORD 'TEST123';
CREATE ROLE
testdb=# GRANT READONLY TO TESTREAD;
GRANT ROLE
testdb=# \C TESTDB TESTREAD
Title is "TESTDB".
\C: extra argument "TESTREAD" ignored
testdb=# \c testdb testread
connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "testread"
Previous connection kept
```
*Оказывается не все case insensitive. Команды psql нужно писать маленькими буквами.  
Переподключимся к базе уже под новым пользователем  *
```
postgres@postgresql:~$ psql -h 127.0.0.1 -U testread -d testdb -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=>
```
15 сделайте select * from t1;
```
testdb=> SELECT * FROM t1;
ERROR:  permission denied for table t1
```
***По-любому проблемма в схеме.** Поэтому не плодим схем в рабочих приложениях без конкретной причины.*  
16 получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)  
17 напишите что именно произошло в тексте домашнего задания  
18 у вас есть идеи почему? ведь права то дали?  
19 посмотрите на список таблиц  
```
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)
```
20 подсказка в шпаргалке под пунктом 20  
21 а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)  
22 вернитесь в базу данных testdb под пользователем postgres  
```
testdb=> \c testdb postgres
Password for user postgres:
connection to server at "127.0.0.1", port 5432 failed: fe_sendauth: no password supplied
Previous connection kept
```
*Ага. Пришлось*
```
testdb=> \q
postgres@postgresql:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=#
```
23 удалите таблицу t1  
24 создайте ее заново но уже с явным указанием имени схемы testnm  
25 вставьте строку со значением c1=1  
```
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# drop table t1;
DROP TABLE
testdb=# create table testnm.t1(c integer);
CREATE TABLE
testdb=# insert into testnm.t1 values (1);
INSERT 0 1
```
26 зайдите под пользователем testread в базу данных testdb
```
testdb=# \q
postgres@postgresql:~$ psql -h 127.0.0.1 -U testread -d testdb -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=>
```
27 сделайте select * from testnm.t1;
```
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
```
28 получилось?  
29 есть идеи почему? если нет - смотрите шпаргалку  
30 как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку  
*Понятно. Нужно сделать, что то типа прав на роль db_datareader в MSSQL-е.  
Выходим, заходим, меняем базу, задаем привелегии по умолчанию, выходим, заходим.*  
```
testdb=> \q
postgres@postgresql:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# ALTER DEFAULT PRIVILEGES IN SCHEMA TESTNM GRANT SELECT ON TABLES TO READONLY;
ERROR:  schema "testnm" does not exist
postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# ALTER DEFAULT PRIVILEGES IN SCHEMA TESTNM GRANT SELECT ON TABLES TO READONLY;
ALTER DEFAULT PRIVILEGES
testdb=#
testdb=# \q
postgres@postgresql:~$ psql -h 127.0.0.1 -U testread -d testdb -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=>
```
31 сделайте select * from testnm.t1;
```
testdb=> select * from testnm.t1;
ERROR:  permission denied for table t1
```
*Вот ведь гадость. DEFAULT PRIVILEGES не полная аналогия с db_datareader.  
Выходим, заходим, меняем базу, задаим тупо GRANT, выходим, заходим.*  
32 получилось?
33 есть идеи почему? если нет - смотрите шпаргалку
```
testdb=> \q
postgres@postgresql:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# grant select on testnm.t1 to readonly;
GRANT
testdb=# \q
postgres@postgresql:~$ psql -h 127.0.0.1 -U testread -d testdb -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> select * from testnm.t1;
 c
---
 1
(1 row)
```
Одно радует, что подобная ситуация навряд ли часто происходит в жизни.
31 сделайте select * from testnm.t1;
32 получилось?
33 ура!
34 теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
35 а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
```
testdb=> create table t2(c1 integer);
CREATE TABLE
testdb=> insert into t2 values (2);
INSERT 0 1
testdb=> select * from t2;
 c1
----
  2
(1 row)
```
Ну понятно, что табличка создалась в public, где есть права у всех.
```
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t2   | table | testread
(1 row)
```
Ага. Точно.
36 есть идеи как убрать эти права? если нет - смотрите шпаргалку
37 если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
Опять перезайдем под postgres-ом и изымем права на создание объектов в public
```
testdb=> \q
postgres@postgresql:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# REVOKE CREATE on SCHEMA public FROM public;
REVOKE
postgres=# REVOKE ALL ON DATABASE testdb FROM public;
REVOKE
postgres=# \q
postgres@postgresql:~$ psql -h 127.0.0.1 -U testread -d testdb -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> create table t3(c1 integer);
CREATE TABLE
testdb=> \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t2   | table | testread
 public | t3   | table | testread
(2 rows)
```
Опять не сработало. А почему, а потому что я забрал права на public в базе public, а не в базе testdb.
Эта музыка будет вечной...
Делаем все еще раз не забывая сменить базу
```
testdb=> \q
postgres@postgresql:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=# REVOKE CREATE on SCHEMA public FROM public;
REVOKE
testdb=# REVOKE CREATE on SCHEMA public FROM public;
REVOKE
testdb=# REVOKE ALL ON DATABASE testdb FROM public;
REVOKE
testdb=# \q
postgres@postgresql:~$ psql -h 127.0.0.1 -U testread -d testdb -W
Password:
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

testdb=> CREATE TABLE t4(c1 integer);
ERROR:  permission denied for schema public
LINE 1: CREATE TABLE t4(c1 integer);
                     ^
testdb=> insert into t2 values (6);
INSERT 0 1
testdb=> select * from t2;
 c1
----
  2
  6
(2 rows)
```
По ходу из-за описки выясняется, что REVOKE одних и тех же прав можно выполнять сколько хочешь раз без ошибки. Видимо с GRANT так же.
38 теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
39 расскажите что получилось и почему
Ну CREATE теперь не проходит, а insert проходит. Ведь я же owner таблички. И кто мне запретит?
 
