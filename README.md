Задание 1
------------------
1. Выполните базовую настройку всех устройств:
   
  	a. Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian

    b. Присвоить имена в соответствии с топологией

    c. Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости 
    отредактируйте таблицу.
  
    d. Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.

    e. Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.





   ![image](https://github.com/goibova5/demo2024/assets/148867942/ea83f36c-9a0c-45a7-a77c-b9a490b0fd6d)

 

Таблица №1

|Имя устройства |Интерфейс|    IPv4      |Маска/префикс |    Шлюз     |
|---------------|---------|--------------|--------------|-------------|
|               |  ens192 |10.12.15.5    |  /24         |10.12.200.200|
|   ISP         |  ens224 |192.168.0.162 |  /30         |             |                              
|               |  ens256 |192.168.0.166 |  /30         |             |
|   HQ-R        |  ens192 |192.168.0.1   |  /25         |             |
|               |  ens224 |192.168.0.161 |  /30         |192.168.0.162|
|   BR-R        |  ens192 |192.168.0.129 |  /27         |             |
|               |  ens224 |192.168.0.165 |  /30         |192.168.0.166|
|   HQ-SRV      |  ens192 |192.168.0.2   |  /25         |192.168.0.1  |
|   BR-SRV      |  ens192 |192.168.0.130 |  /27         |192.168.0.129|



## Задание 1.1 

Чтоб узнать интерфейсы виртуальной машины, воспользовалась командой:
 
 ```
ip a
```

После нужно зайти в конфигурационный файл интерфейсов:
```
nano /etc/network/interfaces
```
Затем нужно изменить их так, как показано в таблице адресации


ISP
````
auto ens192
iface ens192 inet static
address 10.10.201.100
netmask 255.255.255.0
gateway 10.10.201.254

auto ens224
iface ens224 inet static
address 192.168.0.165
netmask 255.255.255.252

auto ens256
iface ens225 inet static
address 192.168.0.161
netmask 255.255.255.252
````

Сохраняем настройки комбинацией клавиш:

```
Ctrl+s
````
Выходим с помощю комбинацией клаивш:
```
Ctrl+x
```

Далее нужно перезакгрузить сеть
```
systemctl restart networking
````

Аналогично выполняем настройку и на других машинах.

BR-R
```
auto ens192
iface ens192 inet static
address 192.168.0.129
netmask 255.255.255.224


auto ens224
iface ens224 inet static
address 192.168.0.162
netmask 255.255.255.252
gateway 192.168.0.161
```

HQ-R
```
auto ens192
iface ens192 inet static
address 192.168.0.1
netmask 255.255.255.128


auto ens224
iface ens224 inet static
address 192.168.0.166
netmask 255.255.255.252
gateway 192.168.0.165
```

BR-SRV
```
auto ens192
iface ens192 inet static
address 192.168.0.130
netmask 255.255.255.224
gateway 192.168.0.129
```

HQ-SRV

```
auto ens192
iface ens192 inet static
address 192.168.0.2
netmask 255.255.255.128
gateway 192.168.0.1
```
Задание 1.2
-------
В данном задание нужно настроить внутреннюю динамическую маршрутизацию по средствам FRR. Выбрать и обосновать выбор протокола динамической маршрутизации из расчёта, что в дальнейшем сеть будет масштабироваться.

Сперва нужно установить пакеотов frr:
```
apt-get install frr
```

Проверим состояние frr:
```
nano /etc/frr/daemons
```

OSPFD=NO надо поменять на:
```
ospfd=YES
```

Переагружаем службу frr:
```
systemctl restart frr
```
Заходим в среду роутера с посощью командой:
```
vtysh
```

Теперь пропишем адреса ближайших сетевых устройств (BR-R,HQ-R) в ospf
```
conf t
router ospf
   net 192.168.0./30 area 0
   net 192.168.0.164/30 area 0
sh ip ospf neighbor
```
Так ж нв устройстве включаем пересылку пакетов, командоой:
```
vtysh
conf t
ip forwarding
```
Далее настроиваем frr на HQ-R и BR-R




Задание 1.3
----------

Настройка автоматического распределения IP-адресов на роутере HQ-R. У сервера должен быть зарезервирован адрес.

  Сперва на HQ-R надо загрузить dhcp:

```
apt install isc-dhcp-server
```

Далее нужно войти в конфигурацию dhcp:

```
nano /etc/default/isc-dhcp-server
```

И указать интерфейс в сторону сети, куда будут идти IP-адреса:
 
 ```INTERFACESV4="ens224"```

Затем надо настроить раздачу IP-адрессов:
```
nnao /etc/dhcp/dhcpd.conf
```
Далее вписываем ...

```
subnet 192.168.0.0 netmask 255.255.255.0 {
range 192.168.0.4 192.168.0.125;
option routers 192.168.0.1;
}
```
Теперь перезагружаем свой сервер:
```
systemctl restart isc-dhcp-server.servise
```
Вкдючаем службу dhcp:
```
systemctl enable isc-dhcp-server
```

## Затем переходим на HQ-SRV

Далее заходим в настройки интерфейсов:
```
nano /etc/network/interfaces
```
Интерфейс ens192 меняем с статического адреса на dhcp
```
# The primary network interface
auto ens192
iface ens192 inet dhcp
#address 192.168.0.1
#netmask 255.255.255.128
#gateway 192.168.0.2
#dns-nameservers 8.8.8.8
```
Послее перезагружаем сестевой интерфейс:
```
systemctl restart networking
```
Теперь проверим выданный IP-адресс с dhcp-сервера:
```
ip a
```
И там ддолжны получить:
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:9e:f2:c7 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 192.168.0.4/24 brd 192.168.0.255 scope global dynamic ens192
       valid_lft 30633sec preferred_lft 30633sec
    inet6 fe80::20c:29ff:fe9e:f2c7/64 scope link
       valid_lft forever preferred_lft forever

Задание 1.4
-----
Настройка локальных учетных записей на всех устройствах в соответствии с таблицей
| Учетная запись| Пароль| Примечание       |
|:-------------:|:-----:|:----------------:|
| Admin         | 22198 |ISP; HQ-R; HQ-SRV |
|Branch admin   | 22198 |BR-R; BR-SRV      |
|Network admin  | 22198 | HQ-R; BR-R; BR-SRV|


Для начала зашла на HQ-SRV, и прописала команду, добавляющая пользователя:
```
useradd Admin
```
Затем задал пароль для пользователя Admin, командой:
```
passwd Admin
```
Чтобы просмотреть список созданных пользователей, нужно прописать команду:
```
nano /etc/passwd
```
После всех действий, должна появиться такая строка:
```
Admin:x:1001:1001::/home/Admin:/bin/sh
```
### Все те же действия я выполнила на HQ-R, ISP, BR-R, BR-SRV

Задание 1.5
---------
Измерение пропускной способности сети между узлами HQ-R-ISP по средствам утилиты iperf3

Для начало нужно устновить утилиту на устройствах HQ-SRV и BR-SRV, с помощью команды:
```
apt install iperf3
```

Затем на компьютере выступающем в роли сервера (```BR-SRV```) нужно прописать команду:
```
iperf3 -s
```

(По стандарту принимающий сервер открыт по порту 5021, если нужно принять данные по другому порту, следует написать)
```
iperf3 -s -p 5022
```
В тот же момент на клиентской стороне (```HQ-SRV```), нужно написать IP-адрес принимающего сервера и его порт:
```
iperf3 -c 192.168.0.130 -p 5022
```

Такой результат должен быть на ```BR-SRV```:
```
root@debian:~# iperf3 -s -p 5022
-----------------------------------------------------------
Server listening on 5022
-----------------------------------------------------------
Accepted connection from 192.168.0.130, port 40598
[  5] local 192.168.0.129 port 5022 connected to 192.168.0.130 port 40604
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec   302 MBytes  2.54 Gbits/sec
[  5]   1.00-2.00   sec   613 MBytes  5.14 Gbits/sec
[  5]   2.00-3.00   sec   544 MBytes  4.57 Gbits/sec
[  5]   3.00-4.00   sec   560 MBytes  4.70 Gbits/sec
[  5]   4.00-5.00   sec   445 MBytes  3.73 Gbits/sec
[  5]   5.00-6.00   sec   414 MBytes  3.47 Gbits/sec
[  5]   6.00-7.00   sec   494 MBytes  4.15 Gbits/sec
[  5]   7.00-8.00   sec   407 MBytes  3.41 Gbits/sec
[  5]   8.00-9.00   sec   274 MBytes  2.29 Gbits/sec
[  5]   9.00-10.00  sec   316 MBytes  2.65 Gbits/sec
[  5]  10.00-10.05  sec  12.6 MBytes  2.08 Gbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-10.05  sec  4.28 GBytes  3.66 Gbits/sec                  receiver
```
И такой ж на  ```HQ-SRV``` :
```
# root@debian:~# iperf3 -c 192.168.0.129 -p 5022
Connecting to host 192.168.0.129, port 5022
[  5] local 192.168.0.4 port 40604 connected to 192.168.0.129 port 5022
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   324 MBytes  2.72 Gbits/sec  209   1.48 MBytes
[  5]   1.00-2.00   sec   621 MBytes  5.21 Gbits/sec    1   1.39 MBytes
[  5]   2.00-3.00   sec   542 MBytes  4.55 Gbits/sec    0   1.64 MBytes
[  5]   3.00-4.00   sec   558 MBytes  4.68 Gbits/sec   14   1.41 MBytes
[  5]   4.00-5.00   sec   445 MBytes  3.73 Gbits/sec   28   1.25 MBytes
[  5]   5.00-6.00   sec   424 MBytes  3.55 Gbits/sec    0   1.47 MBytes
[  5]   6.00-7.00   sec   484 MBytes  4.06 Gbits/sec   58   1.33 MBytes
[  5]   7.00-8.00   sec   400 MBytes  3.36 Gbits/sec   14   1.14 MBytes
[  5]   8.00-9.00   sec   274 MBytes  2.30 Gbits/sec    0   1.30 MBytes
[  5]   9.00-10.00  sec   314 MBytes  2.63 Gbits/sec    3   1.04 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  4.28 GBytes  3.68 Gbits/sec  327             sender
[  5]   0.00-10.05  sec  4.28 GBytes  3.66 Gbits
```

Задание 1.6
------------
Составление backup скриптов для сохранения конфигурации сетевых устройств, а именно HQ-R BR-R

Для начало на устройстве ```HQ-R``` установим RSYNC:
```
apt install rsync -y
```
Потом нужно создать каталог для хранения наших бэкапов:
```
mkdir /etc/networkbacup
```
После нужно зайти в раздел ```crontab```, он нужен для выполнения задач по расписанию:
```
crontab -e
```
Нам предложат выбрать текстовый редактор, выбираем nano, после чего вписываем туда скрипт:
```
0 0 * * * rsync -avzh /etc/frr/frr.conf /etc/networkbackup
```
В скрипте вместо нулей нужно прописать точное время старта скрипта, первый ноль отвечает за ```минуты```, второй за ```часы```
Что бы узнать точное время нужно прописать команду:
```
date
```
По итогу у нас получится:
```
41 6 * * * rsync -avzh /etc/frr/frr.conf /etc/networkbackup
```
Что бы проверить скрипт, нужно зайти в наш созданный каталог:
```
cd /etc/networkbackup
```
Теперь там нужно прописать:
```
ls -a
```
После выйдет:
```
.  ..  frr.conf
```
Далее все тоже самое нужно проделать на ```BR-R```

Задание 1.7
----------
Настройка подключения по SSH для удалённого конфигурирования устройства HQ-SRV по порту 2222. Стоит учитывать, что необходимо перенаправить трафик на этот порт по средствам конролирования трафика
