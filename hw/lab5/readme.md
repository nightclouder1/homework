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
#### Часть 2. Настройка маршрутизатор для доступа по протоколу SSH
#### Шаг 1. Настройка аутентификации устройств 
#### Шаг 2. Создайте ключ шифрования с указанием его длины

```
R1(config)# ip domain-name Router
R1(config)# crypto key generate rsa
How many bits in the modulus [512]: 1024
R1(config)# ip ssh version 2 
```
#### Шаг 3. Активируйте протокол SSH на линиях VTY
```
R1(config)# username admin privilege 15 secret Cisco
R1(config)# line vty 0 4
R1(config)# login local
R1(config)# transport input ssh
R1(config)# transport input telnet
R# write
```
#### Часть 3. Настройка коммутатора для доступа по протоколу SSH
```
S1(config)# ip domain-name Router
S1(config)# crypto key generate rsa
How many bits in the modulus [512]: 1024
S1(config)# ip ssh version 2 
S1(config)# username admin privilege 15 secret Cisco
S1(config)# line vty 0 4
S1(config)# login local
S1(config)# transport input ssh
S1(config)# transport input telnet
S1(config-if)# ip address 192.168.1.11 255.255.255.0
S1(config-if)# no shutdown
S1# write
```
#### Часть 4. Настройка протокола SSH с использованием интерфейса командной строки 
```
PC > ssh -l admin 192.168.1.1
```
Как предоставить доступ к сетевому устройству нескольким пользователям, у каждого из которых есть собственное имя пользователя? Внести в базу данных коммутатора/маршрутизатора всех пользователей с персональными логинами и паролями.
