# Лабораторная работа 2. Просмотр таблицы MAC-адресов коммутатора 
### Топология
![](lab2_1.png)
### Таблица адресации
| Устройство  | Интерфейс |  IP-адрес / префикс  |  
|-------------|-----------|----------------------|
| S1          | VLAN1     | 192.168.1.11         | 
|             |           | 255.255.255.0        |
| S2          | VLAN1     | 192.168.1.12         | 
|             |           | 255.255.255.0        |
| PC-A        | NIC       | 192.168.1.1          | 
|             |           | 255.255.255.0        |
| PC-B        | NIC       | 192.168.1.2          | 
|             |           | 255.255.255.0        |
## Задачи:
#### Часть 1. Создание и настройка сети
#### Часть 2. Изучение таблицы МАС-адресов коммутатора
## Решение:
### Часть 1. Создание и настройка сети
#### Шаг 1 подключить сеть в соответствии с топологией
![](lab2_2.png)
#### Шаг 2 настройка узлов ПК
![](lab2_3.png)

Для PC2. IPv4 Address 192.168.1.2/255.255.255.0
#### Шаг 3 выполнить инициализацию и перезагрузку коммутаторов 
```
Switch> enable 
Switch# write erase
Switch# reload
```
#### Шаг 4 настройка базовых параметров каждого коммутатора
```
Switch> enable 
Switch# conf t
Switch(config) interface vlan 1
Switch(config) ip address 192.168.1.11 255.255.255.0
Switch(config) no shutdown
Switch(config) no ip domain-lookup
Switch(config) hostname S1
S1(config) service password-encryption
S1(config) enable secret class
S1(config) banner motd # NO ENTER !!! #
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# logging synchronous
S1(config-line)# exit
S1 (config)# line vty 0 6
S1 (config-line)# password cisco
S1 (config-line)# login
S1 (config-line)# transport input telnet
S1# write memory
```
Удостовериться, что связь с маршрутизатором S1 налажена: PC1 > ping 192.168.1.11 

```
Switch> enable 
Switch# conf t
Switch(config) interface vlan 1
Switch(config) ip address 192.168.1.12 255.255.255.0
Switch(config) no shutdown
Switch(config) no ip domain-lookup
Switch(config) hostname S2
S2(config) service password-encryption
S2(config) enable secret class
S2(config) banner motd # NO ENTER !!! #
S2(config)# line console 0
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# logging synchronous
S2(config-line)# exit
S2(config)# line vty 0 6
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# transport input telnet
S2# write memory
```
Удостовериться, что связь с маршрутизатором S2 налажена: PC2 > ping 192.168.1.12 , а также, что связь PC1 с PC2 налажена: PC1 > ping 192.162.1.2

### Часть 2. Изучение таблицы МАС-адресов коммутатора
#### Шаг 1 Запишите МАС-адреса сетевых устройств.
```
PC1, PC2 > ipconfig /all.
```
MAC-адрес компьютера PC-1: 000C.CFD9.7333;
MAC-адрес компьютера PC-2: 0060.7032.3824
```
S1, S2 # show mac address-table
```
МАС-адрес коммутатора S1 Fast Ethernet 0/1: 0001.c9d7.1101;
МАС-адрес коммутатора S2 Fast Ethernet 0/1: 00e0.f9e3.1b01

#### Шаг 2 Просмотрите таблицу МАС-адресов коммутатора.
```
S2 # show mac address-table
```
*Записаны ли в таблице МАС-адресов какие-либо МАС-адреса? Нет, имеется mac-адрес только самого коммутатора, так как коммуникация в сети не происходила.*
*Какие МАС-адреса записаны в таблице? С какими портами коммутатора они сопоставлены и каким устройствам принадлежат? Игнорируйте МАС-адреса, сопоставленные с центральным процессором. В таблице записан только один MAC-адрес самого устройва, который сопоставлен с 1 портом устройства. *
