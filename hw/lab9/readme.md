# Лабораторная работа 9 - Конфигурация безопасности коммутатора 
### Топология
![](lab9_1.png)

### Таблица адресации
| Устройство  | Интерфейс   |  IP-адрес            | Маска подсети       | 
|-------------|-------------|----------------------|---------------------|
| R1          | G0/0/0      | 192.168.10.1         |  255.255.255.0      |
|             | Loop0       | 10.10.1.1            |  255.255.255.0      |
| S1          | VLAN10      | 192.168.10.201       |  255.255.255.0      |
| S2          | VLAN10      | 192.168.10.202       |  255.255.255.0      |
| PC-A        | NIC         | DHCP                 |  255.255.255.0      |
| PC-B        | NIC         | DHCP                 |  255.255.255.0      |

### Задачи
### Часть 1. Настройка основного сетевого устройства
### Часть 2. Настройка сетей VLAN
### Часть 3: Настройки безопасности коммутатора.

### Решение
### Часть 1. Настройка основного сетевого устройства
### Шаг 1. Создайте сеть
![](lab9_2.png)

### Шаг 2. Настройте маршрутизатор R1.

a.	Загрузите следующий конфигурационный скрипт на R1.

```
enable
configure terminal
hostname R1
no ip domain lookup
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.10.201 192.168.10.202
ip dhcp relay information trust-all
ip dhcp pool Students
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 domain-name CCNA2.Lab-11.6.1
interface Loopback0
 ip address 10.10.1.1 255.255.255.0
interface GigabitEthernet0/0/1
 description Link to S1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
line con 0
 logging synchronous
 exec-timeout 0 0
```
b.	Проверьте текущую конфигурацию на R1, используя следующую команду:

```
R1# show ip interface brief
```
### Шаг 3. Настройка и проверка основных параметров коммутатора

Для S1:
```
enable
configure terminal
hostname S1
no ip domain-lookup
interface f0/5
 description Link to R1
 exit
interface f0/6
 description PC-A
 exit
interface f0/1
 description Trunk to S2
 exit
ip default-gateway 192.168.10.1
vlan 10
 name MANAGEMENT
 exit
interface vlan 10
 ip address 192.168.10.201 255.255.255.0
 no shutdown
 exit
interface f0/5
 switchport mode access
 switchport access vlan 10
 exit
interface f0/6
 switchport mode access
 switchport access vlan 10
 exit
interface f0/1
 switchport mode trunk
 switchport trunk native vlan 10
 switchport trunk allowed vlan 10
 exit
exit
write
```
Для S2:

```
enable
configure terminal
hostname S2
no ip domain-lookup
interface f0/1
 description Trunk to S1
 exit
interface f0/18
 description PC-B
 exit
ip default-gateway 192.168.10.1
vlan 10
 name MANAGEMENT
 exit
interface vlan 10
 ip address 192.168.10.202 255.255.255.0
 no shutdown
 exit
interface f0/1
 switchport mode trunk
 switchport trunk native vlan 10
 switchport trunk allowed vlan 10
 exit
interface f0/18
 switchport mode access
 switchport access vlan 10
 exit
exit
write

```

### Часть 2. Настройка сетей VLAN на коммутаторах.
### Шаг 1. Сконфигруриуйте VLAN 10.
### Шаг 2. Сконфигруриуйте SVI для VLAN 10.
### Шаг 3. Настройте VLAN 333 с именем Native на S1 и S2.
### Шаг 4. Настройте VLAN 999 с именем ParkingLot на S1 и S2.
Для S1:

```
enable
configure terminal
vlan 10
 name Management
exit
vlan 333
 name Native
exit
vlan 999
 name ParkingLot
exit
interface vlan 10
 description Management SVI
 ip address 192.168.10.201 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.10.1
end
write 

```

Для S2:
```
enable
configure terminal
vlan 10
name Management
exit
vlan 333
name Native
exit
vlan 999
name ParkingLot
exit
interface vlan 10
description Management SVI
ip address 192.168.10.202 255.255.255.0
no shutdown
exit
ip default-gateway 192.168.10.1
end
write 
```

### Часть 3. Настройки безопасности коммутатора.
a.	Настройте все магистральные порты Fa0/1 на обоих коммутаторах для использования VLAN 333 в качестве native VLAN.

Для S1:
```
enable
configure terminal
interface FastEthernet0/1
description Link to S2
switchport mode trunk
switchport trunk native vlan 333
exit
end
```
Для S2:
```
enable
configure terminal
interface FastEthernet0/1
description Link to S1
switchport mode trunk
switchport trunk native vlan 333
exit
end
```
c.	Отключить согласование DTP F0/1 на S1 и S2. 

