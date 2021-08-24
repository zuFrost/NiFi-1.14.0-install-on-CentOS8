# Установка NiFi 1.14.0 на CentOS 8
01 Параметры сервера
02 Установка
03 Запуск NiFi
04 Настройка
05 Тюнинг OS

## Параметры сервера
Установка по данной инструкции проводилась на разнообразные виртуальные машины в разных средах.
Минимальная конфигурация, на которой проверялась установка следующая:
- OS - CentOS8
- 2 vCPU
- RAM 2 ГБ
- HDD 20 ГБ

## 02 Установка
Основной мануал установки <https://nifi.apache.org/docs/nifi-docs/html/getting-started.html#downloading-and-installing-nifi>
Требования к OS - [System Requirements](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#system_requirements)
`подготовка системы`
```sh
$ sudo -i # заходим в root
# yum update -y
# yum install bash-completion wget -y
```
`Установка Java8 и добавление системной переменной $JAVA_HOME`
```sh
# dnf install java-1.8.0-openjdk-devel -y
# echo "export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el8_4.x86_64/jre" >> ~/.bash_profile
# source ~/.bash_profile # применение системной переменной
```
`проверяем, что Java установлена`
```sh
# java -version
# echo $JAVA_HOME
```
`ожидаемый вывод результа:`
```sh
> openjdk version "1.8.0_302"
> OpenJDK Runtime Environment (build 1.8.0_302-b08)
> OpenJDK 64-Bit Server VM (build 25.302-b08, mixed mode)

> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el8_4.x86_64/jre
```
`Отключаем selinux`
```sh
# vi /etc/selinux/config
```
строку SELINUX=enforcing заменяем на SELINUX=disabled

Результат должен выглядеть примерно так:

```txt
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
`перезагружаем машину`
```sh
# reboot
```
`После перезагрузки заходим под root`
```sh
$ sudo -i
```
Cоздаём директорию /opt/nifi-server куда будет установлена программа
```sh
# mkdir /opt/nifi-server
```
заходим в директорию /opt/nifi-server и скачиваем дистрибьютив, в нашем случае последняя версия - [1.14.0](https://www.apache.org/dyn/closer.lua?path=/nifi/1.14.0/nifi-1.14.0-bin.tar.gz)

после распаковки, для простоты переименовываем папку /opt/nifi-server/nifi-1.14.0 в /opt/nifi-server/nifi
удаляем архив, чтобы освободить место на диске (1.4Gb)
```sh
# cd /opt/nifi-server
# wget https://apache-mirror.rbc.ru/pub/apache/nifi/1.14.0/nifi-1.14.0-bin.tar.gz
# tar -xvf nifi-1.14.0-bin.tar.gz
# mv nifi-1.14.0 nifi
# rm -f nifi-1.14.0-bin.tar.gz
```
в результате путь к скрипту запуска должен быть следующим /opt/nifi-server/nifi/bin/nifi.sh

Установка закончена, первый запуск должен производиться перед настройкой nifi в файле /opt/nifi-server/nifi/conf/nifi.properties иначе сервер запускаться не будет.
## Запуск NiFi
Основная страница [How to install and start NiFi](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#how-to-install-and-start-nifi)

/opt/nifi-server/nifi/bin/nifi.sh можно запускать с следующими параметрами:
```sh
/opt/nifi-server/nifi/bin/nifi.sh <command>:
start: starts NiFi in the background
stop: stops NiFi that is running in the background
status: provides the current status of NiFi
run: runs NiFi in the foreground and waits for a Ctrl-C to initiate shutdown of NiFi
install: installs NiFi as a service that can then be controlled via
service nifi start
service nifi stop
service nifi status
```
`запуск NiFi`
```sh
# /opt/nifi-server/nifi/bin/nifi.sh start
```
ожидаемый вывод
```txt
Java home: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.302.b08-0.el8_4.x86_64/jre
NiFi home: /opt/nifi-server/nifi

Bootstrap Config File: /opt/nifi-server/nifi/conf/bootstrap.conf
```
Система стартует пару минут, в результате GUI доступен не будет, так как не все настройки еще произведены. 
Успешность запуска можно отслеживать по логам во второй сессии подключения.
```sh
# tail -f /opt/nifi-server/nifi/logs/nifi-app.log
```
Примерный вывод окончания загрузки:
```
2029-09-09 00:00:01,643 INFO [main] org.apache.nifi.web.server.JettyServer http://101.151.1.111:8080/nifi
2029-09-09 00:00:01,645 INFO [main] org.apache.nifi.BootstrapListener Successfully initiated communication with Bootstrap
2029-09-09 00:00:01,645 INFO [main] org.apache.nifi.NiFi Controller initialization took 16124837417 nanoseconds (16 seconds).
```
`После успешного запуска настраиваем запуск NiFi как службу`
```sh
# /opt/nifi-server/nifi/bin/nifi.sh stop
# /opt/nifi-server/nifi/bin/nifi.sh install
# systemctl daemon-reload
# systemctl start nifi
```
`перезагружаем машину`
```sh
# reboot
```
После перезагрузки виртуальной машины nifi должен запускаться автоматически
## 04 Настройка
Основная страница - [System Properties](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#system_properties)
`После перезагрузки заходим под root`
```sh
$ sudo -i
```
`изменение параметров nifi.properties`
```sh
$ vi /opt/nifi-server/nifi/conf/nifi.properties
```
в файле делаем следующие изменения:
**nifi.remote.input.secure=false**
**nifi.web.http.host=127.0.0.1**
**nifi.web.http.port=8080** (для удобства меняем порт на общепринятый, можно использовать другой, главное потом обращаться по указанному порту)
**nifi.web.http.network.interface.default=eth0** (eth0 интерфейс, на котором находится наш IP доступа)
очищаем значения следующих ключей
***nifi.web.https.host=***
***nifi.web.https.port=***
`настраиваем запуск NiFi как службу`
```sh
# systemctl restart nifi
```
После пары минут перезагрузки GUI NiFi без логина и пароля доступен по адресу http://<IP-адрес>:8080/nifi/ 
## 05 Тюнинг OS
Основная страница - [Сonfiguration best practices](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#configuration-best-practices)
```sh
# vi /etc/security/limits.conf
```
установить:
```
hard  nofile  50000
soft  nofile  50000
hard  nproc  10000
soft  nproc  10000
```
```sh
# vi /etc/security/limits.d/90-nproc.conf
```
установить:
```
soft  nproc  10000
```
```sh
# sudo sysctl -w net.ipv4.ip_local_port_range="10000 65000"
```
```sh
# vi /etc/sysctl.conf
```
установить:
**vm.swappiness = 0**
финальная перезагрузка
```sh
# reboot
```
