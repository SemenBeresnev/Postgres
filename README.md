# ДЗ урок 3. Докер

Буду использовать виртуальную машину с установленным Ubuntu 20.04 с прошлого урока.

##Установка Docker Engine.
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
## Развертывание PostgreSQL
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
Развернем контейнер с PostgreSQL
```
docker container run -d --name=pg -p 5432:5432 -e POSTGRES_PASSWORD=secret -e PGDATA=/var/lib/postgres -v /var/lib/postgres:/var/lib/postgres postgres:14.4
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
docker container run -d --name=pg -p 5434:5434 -e POSTGRES_PASSWORD=secret -e PGDATA=/var/lib/postgres -v /var/lib/postgres:/var/lib/postgres postgres:14.4
```
Подключимся к докеру
```
root@progresssql:/var/lib# docker exec -it pg psql -U postgres -w postgres
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

postgres=#
```
Вот теперь все хорошо.


• развернуть контейнер с клиентом postgres
• подключится из контейнера с клиентом к контейнеру с сервером и сделать
таблицу с парой строк
• подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/места установки докера
• удалить контейнер с сервером
• создать его заново
• подключится снова из контейнера с клиентом к контейнеру с сервером
• проверить, что данные остались на месте
• оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами
