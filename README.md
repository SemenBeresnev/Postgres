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
##выполнить pgbench -i postgres
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
запустить pgbench -c8 -P 60 -T 3600 -U postgres postgres
```
postgres@postgresql:~$  pgbench -c8 -P 60 -T 3600 -U postgres postgres
pgbench (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 944.5 tps, lat 8.467 ms stddev 2.130
progress: 120.0 s, 942.7 tps, lat 8.486 ms stddev 1.840
...

```
дать отработать до конца
зафиксировать среднее значение tps в последней ⅙ части работы
а дальше настроить autovacuum максимально эффективно
так чтобы получить максимально ровное значение tps на горизонте часа
