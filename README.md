# ДЗ по уроку 6 Физический уровень Postgres
## Подготовка виртуальной машины
Для выполнения ДЗ буду использовать виртуальную машину, развернутую на первом задании.
```
semen@postgresql:~$ sudo -u postgres pg_lsclusters
[sudo] password for semen:
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
```
Все в порядке.
## Создание данных в Postgres
```
semen@postgresql:~$ sudo su - postgres
postgres@postgresql:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# create table test (c1 text);
CREATE TABLE
postgres=# insert into test values ('some data');
INSERT 0 1
postgres=# select * from test;
    c1
-----------
 some data
(1 row)
```
Остановим Postgres
```
semen@postgresql:~$ sudo -u postgres pg_ctlcluster 14 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@14-main
semen@postgresql:~$ sudo -i
root@postgresql:~# pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
root@postgresql:~#
```
Почему-то Ubuntu сказала, что лучше останавливать кластер через systemctl, но кластер остановила.
## Подключим новый диск
Диск для виртуальной машины создали и подвязали через Hyper-V.

```root@postgresql:~# fdisk -l``` - узнали имя раздела созданного диска - ```/dev/sdb```.
```
root@postgresql:~# mkdir /mnt/data
root@postgresql:~# mkfs.ext4 -L postgresdata /dev/sdb
root@postgresql:~# mount -o defaults /dev/sdb /mnt/data
```
Создали дирректорию, отформатировали раздел и смонтировали раздел к дирректории
```
root@postgresql:~# nano /etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-Sr2uHOla77LLBEtewcGfWakrQTUmBYpDmBQg5yUULgYRSxCjN55FzK30OZv31IgX / ext4 defaults 0 1
/dev/sdb /mnt/data ext4 defaults 0 2
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/96886e89-b6b7-4653-8daa-b45ba17b93b8 /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
```
Настроили диск для автоматического монтирования при загрузке


создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
перенесите содержимое /var/lib/postgres/14 в /mnt/data - mv /var/lib/postgresql/14 /mnt/data
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
напишите получилось или нет и почему
задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
напишите что и почему поменяли
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
напишите получилось или нет и почему
зайдите через через psql и проверьте содержимое ранее созданной таблицы
