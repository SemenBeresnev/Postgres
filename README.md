# ДЗ урок 3. Докер

Буду использовать виртуальную машину с установленным Ubuntu 20.04 с прошлого урока.

## Установка Docker Engine.
Запустим перечнень команд
```
sudo apt update
```
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```
```
sudo apt update
```
```
sudo apt install docker-ce
```
Проверю, что все прошло хорошо
```
semen@progresssql:~$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-07-14 05:27:42 UTC; 2min 44s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 25075 (dockerd)
      Tasks: 7
     Memory: 30.0M
     CGroup: /system.slice/docker.service
             └─25075 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
## Развертывание контейнера PostgreSQL
Создадим каталог и убедимся в его наличии
```
mkdir /var/lib/postgres
root@progresssql:/var/lib/postgresql# cd ..
root@progresssql:/var/lib# ls
AccountsService  command-not-found  dpkg             landscape  PackageKit  postgresql  systemd                  udisks2              usb_modeswitch
apport           containerd         fwupd            logrotate  pam         private     tpm                      unattended-upgrades  usbutils
apt              dbus               git              man-db     plymouth    python      ubuntu-advantage         update-manager       vim
boltd            dhcp               grub             misc       polkit-1    snapd       ubuntu-release-upgrader  update-notifier
cloud            docker             initramfs-tools  os-prober  postgres    sudo        ucf                      upower
```
Создадим docker-network
```
root@progresssql:/var/lib# docker network create pg-net
325120f952c57e28c8b7133bb2df6f4db709b367b93186d79d642127913fc645
```
Развернем контейнер с PostgreSQL
```
sudo docker run --name pg --network pg-net -e POSTGRES_PASSWORD=123 -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14.4

```
Убедимся, что все хорошо
```
root@progresssql:/var/lib# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
postgres     14.4      1133a9cdc367   44 hours ago   376MB
```
А оказалось что не все хорошо. Порт 5432 оказался занят postgres-ом 12 созданным на предыдущем уроке.
Пришлось удалить контейнер
```
root@progresssql:/var/lib# docker stop pg
pg
root@progresssql:/var/lib# docker rm pg
pg
```
И создать заново на пору 5434
```
sudo docker run --name pg --network pg-net -e POSTGRES_PASSWORD=123 -d -p 5434:5434 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14.4
```
Подключимся к докеру
```
root@progresssql:/var/lib# docker exec -it pg psql -U postgres -w postgres
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=#
```
Вот теперь все хорошо.
## Развертывание контейнера с клиентом
```
root@progresssql:/var/lib# docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg -U postgres
Unable to find image 'postgres:14' locally
14: Pulling from library/postgres
Digest: sha256:3e2eba0a6efbeb396e086c332c5a85be06997d2cf573d34794764625f405df4e
Status: Downloaded newer image for postgres:14
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=#
```
Все удачно. Создадим табличку с данными.
```
postgres=# create table fuckingfuck (a int, b int)
postgres-# ;
CREATE TABLE
postgres=# insert into fuckingfuck values (1, 2), (3, 4);
INSERT 0 2
postgres=# select * from fuckingfuck;
 a | b
---+---
 1 | 2
 3 | 4
(2 rows)
```
## Подключение к контейнеру с Postgres извне
С другой машины (test20) подключились к контейнеру. Проверили таблицу, добавили строчку.
```
root@test20:~# psql -p 5434 -U postgres -h 192.168.3.50 -d postgres -W
Password:
psql (12.11 (Ubuntu 12.11-0ubuntu0.20.04.1), server 14.4 (Debian 14.4-1.pgdg110+1))
WARNING: psql major version 12, server major version 14.
         Some psql features might not work.
Type "help" for help.

postgres=# select * from fuckingfuck;
 a | b
---+---
 1 | 2
 3 | 4
(2 rows)

postgres=# insert into fuckingfuck values (5, 6);
INSERT 0 1
postgres=#
```
## Пересоздание контейнера
```
root@progresssql:~# docker stop pg
pg
root@progresssql:~# docker rm pg
pg
root@progresssql:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@progresssql:~# sudo docker run --name pg --network pg-net -e POSTGRES_PASSWORD=123 -d -p 5434:5434 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
1bc410d5faa5740905f2ffa873df520993d8a7a4c080b0b35745ea54dd6e8f94
root@progresssql:~# docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
1bc410d5faa5   postgres:14   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:5434->5434/tcp, :::5434->5434/tcp   pg
```
Пересоздали и все хорошо.
##  Проверка сохранности данных
Подключимся к нашему новому контейнеру и проверим, что данные в наличии.
```
postgres=# root@progresssql:/var/lib# docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg -U postgres
Password for user postgres:
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=# select * from fuckingfuck;
 a | b
---+---
 1 | 2
 3 | 4
 5 | 6
(3 rows)
```
Данные остались на месте. И те что занесли при первом подключении и те что занесли с удаленной машины.
