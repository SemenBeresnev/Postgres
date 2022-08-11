# Postgres
Для экспериментов буду пользоваться созданным на предыдущих уроках кластером Postgres 14.  
Начнем-с  
## Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
Подключились и выполнили:
```
postgres=# ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM
postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)
```
Хотел найти файл postgres.auto.conf и не нашел его в дирректории где лежит postgres.conf. Пришел в недоумение.
Но потом нашел его в директории /mnt/data/main - примонтированный на прошых уроках диск с данными.
Проверим deadlock_timeout.
```
postgres=# SHOW deadlock_timeout;
 deadlock_timeout
------------------
 1s
```
А нужно 200 ms.
Переконфигурим
```
postgres=# ALTER SYSTEM SET deadlock_timeout = 200;
ALTER SYSTEM
postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# SHOW deadlock_timeout;
 deadlock_timeout
------------------
 200ms
```
Подготовим табличку для экспериментов
```
postgres=# create table locks (a int, b int);
CREATE TABLE
postgres=# insert into locks values (1, 2), (3, 4);
INSERT 0 2
postgres=# select * from locks;
 1 | 2
 3 | 4
```
Замутим "долгую" блокировку в сессии 1
```
postgres=# begin;
BEGIN
postgres=*# update locks set b = 11 where a = 1;
UPDATE 1
```
В сессии 2 сядем на блокировку
```
postgres=# update locks set b = 12 where a = 1;
```
В сессии 1 завершим транзакцию.
```
postgres=*# commit;
COMMIT
```
Сразу же заврешилось и выполнение команды с сессии 2.
```
postgres=# update locks set b = 12 where a = 1;
UPDATE 1
postgres=#
```
Видимо в файле 
```
2022-08-11 05:12:45.047 UTC [1876] postgres@postgres LOG:  process 1876 still waiting for ShareLock on transaction 7829554 after 200.066 ms
2022-08-11 05:12:45.047 UTC [1876] postgres@postgres DETAIL:  Process holding the lock: 1733. Wait queue: 1876.
2022-08-11 05:12:45.047 UTC [1876] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "locks"
2022-08-11 05:12:45.047 UTC [1876] postgres@postgres STATEMENT:  update locks set b = 12 where a = 1;
2022-08-11 05:12:57.411 UTC [1876] postgres@postgres LOG:  process 1876 acquired ShareLock on transaction 7829554 after 12564.673 ms
2022-08-11 05:12:57.411 UTC [1876] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "locks"
```
Ну, видимо, можно в этом как-то ковыряться ...
## Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.
Создадим табличку accounts и представления accounts_v и locks_v как в лекции.
Немножко изменим locks_v что бы видеть блокировки ключа accounts_pkey
```
CREATE VIEW locks_v AS
SELECT pid,
       locktype,
       CASE locktype
         WHEN 'relation' THEN relation::regclass::text
         WHEN 'transactionid' THEN transactionid::text
         WHEN 'tuple' THEN relation::regclass::text||':'||tuple::text
       END AS lockid,
       mode,
       granted
FROM pg_locks
WHERE locktype in ('relation','transactionid','tuple')
AND (locktype != 'relation' OR relation = 'accounts'::regclass or relation = 'accounts_pkey'::regclass);
```



## Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
Попробуйте воспроизвести такую ситуацию.
