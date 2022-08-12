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
Немножко изменим locks_v что бы видеть блокировки ключа accounts_pkey. Если честно, пока не знаю, а надо оно мне... Но пока так.
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
Из трех psql-ов проапдейтили первую строчку в accounts. Locks_v вернул такую картину:
```
postgres=# select * from locks_v;
 pid  |   locktype    |    lockid     |       mode       | granted
------+---------------+---------------+------------------+---------
 1938 | relation      | accounts_pkey | RowExclusiveLock | t
 1938 | relation      | accounts      | RowExclusiveLock | t
 1832 | relation      | accounts_pkey | RowExclusiveLock | t
 1832 | relation      | accounts      | RowExclusiveLock | t
 1724 | relation      | accounts_pkey | RowExclusiveLock | t
 1724 | relation      | accounts      | RowExclusiveLock | t
 1724 | transactionid | 7829573       | ExclusiveLock    | t
 1938 | tuple         | accounts:1    | ExclusiveLock    | f
 1832 | transactionid | 7829574       | ExclusiveLock    | t
 1832 | tuple         | accounts:1    | ExclusiveLock    | t
 1938 | transactionid | 7829575       | ExclusiveLock    | t
 1832 | transactionid | 7829573       | ShareLock        | f
(12 rows)
```
Что видим:
1. В деле учавствуют три процесса (1724, 1832 и 1938) со своими транзакциями (7829573, 7829574 и 7829575 соответственно).
2. Кроме блокировки строчки самой таблицы, блокируется и primary key. Видимо для того, что бы изменить его после завершения транзакции. Ну что бы он указывал уже на новую строчку.
3. Потоки 1832 и 1938 выстроились в очередь для занятия блокировки. Потому locktype и tuple.
4. Почему так проставились значения в поле grant я не понял.
## Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
Произвел взаимоблокировку транзакций. В логе видим следующую картинку
```
2022-08-12 05:51:41.839 UTC [1724] postgres@postgres LOG:  process 1724 still waiting for ShareLock on transaction 7829577 after 200.055 ms
2022-08-12 05:51:41.839 UTC [1724] postgres@postgres DETAIL:  Process holding the lock: 1832. Wait queue: 1724.
2022-08-12 05:51:41.839 UTC [1724] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "accounts"
2022-08-12 05:51:41.839 UTC [1724] postgres@postgres STATEMENT:  update accounts set amount = amount + 10 where acc_no = 3;
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres LOG:  process 1832 detected deadlock while waiting for ShareLock on transaction 7829576 after 200.050 ms
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres DETAIL:  Process holding the lock: 1724. Wait queue: .
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres CONTEXT:  while updating tuple (0,2) in relation "accounts"
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres STATEMENT:  update accounts set amount = amount + 10 where acc_no = 2;
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres ERROR:  deadlock detected
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres DETAIL:  Process 1832 waits for ShareLock on transaction 7829576; blocked by process 1724.
        Process 1724 waits for ShareLock on transaction 7829577; blocked by process 1832.
        Process 1832: update accounts set amount = amount + 10 where acc_no = 2;
        Process 1724: update accounts set amount = amount + 10 where acc_no = 3;
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres HINT:  See server log for query details.
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres CONTEXT:  while updating tuple (0,2) in relation "accounts"
2022-08-12 05:51:58.253 UTC [1832] postgres@postgres STATEMENT:  update accounts set amount = amount + 10 where acc_no = 2;
2022-08-12 05:51:58.253 UTC [1724] postgres@postgres LOG:  process 1724 acquired ShareLock on transaction 7829577 after 16614.096 ms
2022-08-12 05:51:58.253 UTC [1724] postgres@postgres CONTEXT:  while updating tuple (0,3) in relation "accounts"
2022-08-12 05:51:58.253 UTC [1724] postgres@postgres STATEMENT:  update accounts set amount = amount + 10 where acc_no = 3;
```
Видно:
1.deadlock был.
2.какие процессы принимали участие в возникновении deadlock-а.
3.какие запросы они пытались выполнить.
4.ситуация вполне прозрачная.
## Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга? Попробуйте воспроизвести такую ситуацию.
Такая ситуация может возникунть если один процес начнет менять таблицу снизу вверх, а другой сверху вниз практически одновременно.
Как легко создать такую ситуацию я не смог придумать. Order by в update-ах нет. Запускать update через 2 view-ки с разным sort order...
Кроме того мы еще не проходимл сложные sql скрипты в postgres, а они точно понядобяться, что бы запустить два запроса одновременно.

