# Лабораторная работа 11 - Настройка и проверка расширенных списков контроля доступа.
### Топология

![](lab11_1.png)

### Таблица адресации

| Устройство | Интерфейс | IP-адрес | Маска подсети | Шлюз по умолчанию |
|------------|-----------|----------|---------------|-------------------|
| R1         | G0/0/1    | —        | —             | —                 |
| R1         | G0/0/1.20 | 10.20.0.1| 255.255.255.0 | —                 |
| R1         | G0/0/1.30 | 10.30.0.1| 255.255.255.0 | —                 |
| R1         | G0/0/1.40 | 10.40.0.1| 255.255.255.0 | —                 |
| R1         | G0/0/1.1000| —       | —             | —                 |
| R1         | Loopback1 | 172.16.1.1| 255.255.255.0| —                 |
| R2         | G0/0/1    | 10.20.0.4| 255.255.255.0 | —                 |
| S1         | VLAN 20   | 10.20.0.2| 255.255.255.0 | 10.20.0.1         |
| S2         | VLAN 20   | 10.20.0.3| 255.255.255.0 | 10.20.0.1         |
| PC-A       | NIC       | 10.30.0.10| 255.255.255.0| 10.30.0.1         |
| PC-B       | NIC       | 10.40.0.10| 255.255.255.0| 10.40.0.1         |

### Таблица VLAN

| VLAN | Имя | Назначенный интерфейс |
|------|-----|-----------------------|
| 20   | Management | S2: F0/5 |
| 30   | Operations | S1: F0/6 |
| 40   | Sales | S2: F0/18 |
| 999  | ParkingLot | S1: F0/2-4, F0/7-24, G0/1-2<br>S2: F0/2-4, F0/6-17, F0/19-24, G0/1-2 |
| 1000 | Собственная | — |

### Задачи
Часть 1. Создание сети и настройка основных параметров устройства
Часть 2. Настройка и проверка списков расширенного контроля доступа

### Решение
### 1.Создание сети и настройка основных параметров устройства
Базовая настройка R1

```
en
conf t
hostname R1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
service password-encryption
banner motd # NO ENTER! #
end
write memory
```


Базовая настройка R2
```
en
conf t
hostname R2
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
service password-encryption
banner motd # NO ENTER! #
end
write memory
```

Базовая настройка S1
```
en
conf t
hostname S1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
service password-encryption
banner motd # NO ENTER! #
end
write memory
```

Базовая настройка S2
```
en
conf t
hostname S2
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
service password-encryption
banner motd # NO ENTER! #
end
write memory

```

### Часть 2 Настройка сетей VLAN на коммутаторах.
S1 — VLAN + Management + ParkingLot

```
configure terminal
vlan 20
 name Management
vlan 30
 name Operations
vlan 40
 name Sales
vlan 999
 name ParkingLot
vlan 1000
 name Native
exit
interface vlan 20
 ip address 10.20.0.2 255.255.255.0
 no shutdown
exit
ip default-gateway 10.20.0.1
interface range f0/2-4, f0/7-24, g0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
exit
interface f0/6
 switchport mode access
 switchport access vlan 30
 no shutdown
exit
interface f0/5
 switchport mode trunk
 no shutdown
exit
interface f0/1
 switchport mode trunk
 no shutdown
exit
end
write memory
```

S2 — VLAN + Management + ParkingLot
```
configure terminal
vlan 20
 name Management
vlan 30
 name Operations
vlan 40
 name Sales
vlan 999
 name ParkingLot
vlan 1000
 name Native
exit
interface vlan 20
 ip address 10.20.0.3 255.255.255.0
 no shutdown
exit
ip default-gateway 10.20.0.1
interface range f0/2-4, f0/6-17, f0/19-24, g0/1-2
 switchport mode access
 switchport access vlan 999
 shutdown
exit
interface f0/18
 switchport mode access
 switchport access vlan 40
 no shutdown
exit
interface f0/5
 switchport mode access
 switchport access vlan 20
 no shutdown
exit
interface f0/1
 switchport mode trunk
 no shutdown
exit
end
write memory
```
