# ДЗ урок 3. Докер

Бубу использовать виртуальную машину с утсановленным Ubuntu 20.04 с прошлого урока.

Установим на нем Docker Engine перечнем команд
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



сделать в GCE инстанс с Ubuntu 20.04 или развернуть докер любым удобным способом
• поставить на нем Docker Engine
• сделать каталог /var/lib/postgres
• развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
• развернуть контейнер с клиентом postgres
• подключится из контейнера с клиентом к контейнеру с сервером и сделать
таблицу с парой строк
• подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/места установки докера
• удалить контейнер с сервером
• создать его заново
• подключится снова из контейнера с клиентом к контейнеру с сервером
• проверить, что данные остались на месте
• оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами
