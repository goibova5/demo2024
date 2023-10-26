Задание 1
------------------
1. Выполните базовую настройку всех устройств:
   
  	a. Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian

    b. Присвоить имена в соответствии с топологией

    c. Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости 
    отредактируйте таблицу.
  
    d. Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.

    e. Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.


Таблица №1
-------------
|Имя устройства |Интерфейс|    IPv4      |Маска/префикс |    Шлюз     |
|---------------|---------|--------------|--------------|-------------|
|               |  ens192 |192.168.0.1   |  /27        |10.12.24.254 |
|   ISP         |  ens224 |192.168.0.2   |  /27         |             |                              
                |  ens225 |192.168.0.30  |  /27         |             |
|   HQ-R        |  ens192 |192.168.0.161 |  /30         |192.168.0.162|
|               |  ens224 |192.223.23.124   |  /25         |             |
|   BR-R        |  ens192 |192.168.0.2   |  /30         |192.168.0.161|
|               |  ens224 |192.168.0129  |  /27         |             |
|   HQ-SRV      |  ens192 |192.168.0.126 |  /25         |192.168.0.1  |
|   BR-SRV      |  ens192 |192.168.0.158 |  /27         |192.168.0.129|
