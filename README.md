# Postgres
Для экспериментов буду пользоваться созданным на предыдущих уроках кластером Postgres 14.  
Начнем-с  
##применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Задали настройки в файле postgresql.conf
```
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 6553kB
min_wal_size = 4GB
max_wal_size = 16GB
```  
## выполнить pgbench -i postgres
```
postgres@postgresql:~$ pgbench -i postgres
dropping old tables...
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.09 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.37 s (drop tables 0.01 s, create tables 0.00 s, client-side generate 0.17 s, vacuum 0.10 s, primary keys 0.10 s).
```  
## запустить pgbench -c8 -P 60 -T 3600 -U postgres postgres
```
postgres@postgresql:~$  pgbench -c8 -P 60 -T 3600 -U postgres postgres
pgbench (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 944.5 tps, lat 8.467 ms stddev 2.130
progress: 120.0 s, 942.7 tps, lat 8.486 ms stddev 1.840
...
progress: 3600.0 s, 914.4 tps, lat 8.749 ms stddev 8.336
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 3600 s
number of transactions actually processed: 2504741
latency average = 11.497 ms
latency stddev = 13.747 ms
initial connection time = 13.053 ms
tps = 695.759413 (without initial connection time)
```
Заметим, что tpc менялось от 257 до 983. Скажем так неровненько...

## настроить autovacuum максимально эффективно
Воспользуемся сервисом тюнинга по адресу https://pgtune.leopard.in.ua/. Учтем, что у нас диск SSD, а не HDD.
Рекомендовано изменить два параметра
```
random_page_cost = 1.1
effective_io_concurrency = 200
```
Внесем исправления в conf, перезапустим postgres, пересоздадим таблицы и запустим бенсмарк заново.
```
postgres@postgresql:~$  pgbench -c8 -P 60 -T 3600 -U postgres postgres
pgbench (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 918.4 tps, lat 8.707 ms stddev 4.554
progress: 120.0 s, 970.2 tps, lat 8.245 ms stddev 1.695
...
progress: 3540.0 s, 459.9 tps, lat 17.404 ms stddev 9.071
progress: 3600.0 s, 475.1 tps, lat 16.836 ms stddev 7.785
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 3600 s
number of transactions actually processed: 2257979
latency average = 12.754 ms
latency stddev = 15.367 ms
initial connection time = 13.809 ms
tps = 627.213654 (without initial connection time)
```
tpc менялось от 222 до 978.
Все стало еще хуже. Дурацкий сайт https://pgtune.leopard.in.ua/.
## Настройка autovacuum
Разремил все настройки группы AUTOVACUUM, файла postresql.conf и хотя значание по умолчанию для autovacuum = on резльтат не замедлил сказаться.
```
postgres@postgresql:~$  pgbench -c8 -P 60 -T 3600 -U postgres postgres
pgbench (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 747.1 tps, lat 10.703 ms stddev 21.723
...
progress: 3600.0 s, 913.1 tps, lat 8.773 ms stddev 8.571
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 3600 s
number of transactions actually processed: 2965261
latency average = 9.712 ms
latency stddev = 11.995 ms
initial connection time = 15.445 ms
tps = 823.681036 (without initial connection time)
```
Средний tps вырос с 627 до 823. Разброс тоже серьезно изменился.
Было 222 - 978, а стало 304 - 981. Значительно более ровный результат.
Видно, что настройки меняют картину, причем заметно.
Но как их крутить, пока не ясно...
Думаю, что когда возникнет непосредственная задача буду знать куда лезть и что крутить.
