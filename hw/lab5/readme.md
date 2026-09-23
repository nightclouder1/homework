# Лабораторная работа 5. Доступ к сетевым устройствам по протоколу SSH
### Топология
![](lab5_1.png)
### Таблица адресации
| Устройство  | Интерфейс |  IPv6-адрес          | Маска подсети.      | Шлюз по умолчанию |
|-------------|-----------|----------------------|---------------------|-------------------|
| R1          | G0/0/0    | 192.168.1.1          | 255.255.255.0       | -                 |
| S1          | VLAN 1    | 192.168.1.11         | 255.255.255.0       | 192.168.1.1       |
| PC-A        | NIC       | 192.168.1.3          | 255.255.255.0       | 192.168.1.1       |

### Задачи:
#### Часть 1. Настройка основных параметров устройства
#### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH
#### Часть 3. Настройка коммутатора для доступа по протоколу SSH
#### Часть 4. SSH через интерфейс командной строки (CLI) коммутатора

### Решение:
#### Часть 1. Настройка основных параметров устройства

Настройка коммутатора:
```
Switch> enable
Switch# conf t
Switch(config)# hostname S1
S1(config)# no ip domain-lookup
S1(config)# enable secret class
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
S1(config)# line vty 0 4
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
S1(config)# service password-encryption
S1(config)# banner motd # NO ENTER !!! #
S1# write 
```
Настройка маршрутизатора: 

```
Router> enable
Router# conf t
Router(config)# hostname R1
R1(config)# no ip domain-lookup
R1(config)# enable secret class
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit
R1(config)# service password-encryption
R1(config)# banner motd # NO ENTER !!! #
R1(config)# interface gigabitEthernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config)# exit
R1# write
```
Настройка компьютера произведена относительно таблицы адресации

```
PC > ping 192.168.1.1
Pinging 192.168.1.1 with 32 bytes of data:
Reply from 192.168.1.1: bytes=32 time=2ms TTL=255
```

