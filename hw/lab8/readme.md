# Лабораторная работа 8. Реализация DHCPv4 
### Топология
![](lab8_1.png)

### Таблица адресации
| Устройство  | Интерфейс   |  IPv6-адрес          | Маска подсети.      | Шлюз по умолчанию |
|-------------|-------------|----------------------|---------------------|-------------------|
| R1          | G0/0/0      | 10.0.0.1             | 255.255.255.252     | -                 |
|             | G0/0/1      | -                    | -                   |                   |
|             | G0/0/1.100  | 192.168.1.1          | 255.255.255.192     |                   |
|             | G0/0/1.200  | 192.168.1.65         | 255.255.255.224     |                   |
|             | G0/0/1.1000 | -                    | -                   |                   |
| R2          | G0/0        | 10.0.0.2             | 255.255.255.252     | -                 |
|             | G0/0/1      | 192.168.1.97         | 255.255.255.240     |                   |
| S1          | VLAN200     | 192.168.1.66         | 255.255.255.224     | 192.168.1.65      |
| S2          | VLAN1       | 192.168.1.98         | 255.255.255.240     | 192.168.1.97      | 
| PC-A        | NIC         | DHCP                 | DHCP                | DHCP              |
| PC-B        | NIC         | DHCP                 | DHCP                | DHCP              |


### Таблица VLAN

| VLAN        | Имя         |  Назначенный интерфейс | 
|-------------|-------------|------------------------|
| 1           | Нет         | S2: F0/18              | 
| 100         | Клиенты     | S1: F0/6               |  
| 200         | Управление  | S1: VLA200             | 
| 999         | Parking_lot | S1: F0/1-4, F0/7-24, G0/1-2 | 
| 1000        | Собственная | -                      | 

### Задачи 
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1
#### Часть 3. Настройка и проверка DHCP-ретрансляции на R2

### Решение
#### Часть 1.	Создание сети и настройка основных параметров устройства
#### Шаг 1. Создание схемы адресации
Подсеть A: 192.168.1.0/26, первый IP для R1 G0/0/1.100 — 192.168.1.1.   
Подсеть B: 192.168.1.64/27, первый IP для R1 G0/0/1.200 — 192.168.1.65; второй IP для S1 VLAN 200 — 192.168.1.66; шлюз по умолчанию S1 — 192.168.1.65.   
Подсеть C: 192.168.1.96/28, первый IP для R2 G0/0/1 — 192.168.1.97.   
#### Шаг 2. Создайте сеть согласно топологии
![](lab8_2.png)

### Шаг 3. Произведите базовую настройку маршрутизаторов.

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
R1(config)# exit
R1# clock set 12:00:00 25 Oct 2026
R1# write
```
Для маршрутизатора R2 выполнена аналогичная настройка.

### Шаг 4.	Настройка маршрутизации между сетями VLAN на маршрутизаторе R1
a.	Активируйте интерфейс G0/0/1 на маршрутизаторе.
```
enable
conf t
interfece g0/0/1
no sh
```
b.	Настройте подинтерфейсы для каждой VLAN в соответствии с требованиями таблицы IP-адресации. Все субинтерфейсы используют инкапсуляцию 802.1Q и назначаются первый полезный адрес из вычисленного пула IP-адресов. Убедитесь, что подинтерфейсу для native VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.
```
enable
conf t

interface g0/0/1.100
 description Client VLAN 100 - Subnet A
 encapsulation dot1Q 100
 ip address 192.168.1.1 255.255.255.192
 no shutdown
 exit

interface g0/0/1.200
 description Management VLAN 200 - Subnet B
 encapsulation dot1Q 200
 ip address 192.168.1.65 255.255.255.224
 no shutdown
 exit

interface g0/0/1.1000
 description Native VLAN 1000
 encapsulation dot1Q 1000 native
 no shutdown
 exit

write
```
c.	Убедитесь, что вспомогательные интерфейсы работают.
```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.100192.168.1.1     YES manual up                    up 
GigabitEthernet0/0/1.200192.168.1.65    YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 

R1#show ip route connected
 C   192.168.1.0/26  is directly connected, GigabitEthernet0/0/1.100
 C   192.168.1.64/27  is directly connected, GigabitEthernet0/0/1.200
```
### Шаг 5. Настройка интерфейсов R2 и статической маршрутизации на R1 и R2
a.	Настройте G0/0/1 на R2 с первым IP-адресом подсети C, рассчитанным ранее.
```
enable
conf t
interface g0/0/1
 description Client network
 ip address 192.168.1.97 255.255.255.240
 no shutdown
```

b.	Настройте интерфейс G0/0/0 для каждого маршрутизатора на основе приведенной выше таблицы IP-адресации.
Для R2:
```
en
conf t
interface g0/0/0
 ip address 10.0.0.2 255.255.255.252
 no sh
```

Для R1:
```
en
conf t
interface g0/0/0
 description Link to R2
 ip address 10.0.0.1 255.255.255.252
 no sh
```

c.	Настройте маршрут по умолчанию на каждом маршрутизаторе, указываемом на IP-адрес G0/0/0 на другом маршрутизаторе.
Для R1:
```
conf t
ip route 0.0.0.0 0.0.0.0 10.0.0.2
end
write
```
Для R2:
```
conf t
ip route 0.0.0.0 0.0.0.0 10.0.0.1
end
write
```

d.	Убедитесь, что статическая маршрутизация работает с помощью пинга до адреса G0/0/1 R2 от R1.

```
R1#ping 192.168.1.97

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```


e.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

на обоих маршрутизаторах ввести "write"

### Шаг 6.	Настройте базовые параметры каждого коммутатора.

```
enable
conf t
hostname S1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd # NO ENTER !!! #
exit
clock set 12:00:00 25 Oct 2026
write
```
Для S2 настройка выполнена аналогично.

### Шаг 7.	Создайте сети VLAN на коммутаторе S1.

a.	Создайте необходимые VLAN на коммутаторе 1 и присвойте им имена из приведенной выше таблицы.

```
enable
conf t
vlan 100
 name Клиенты
vlan 200
 name Управление
vlan 999
 name Parking_Lot
vlan 1000
 name Собственная
exit
end
write
```

b.	Настройте и активируйте интерфейс управления на S1 (VLAN 200), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того установите шлюз по умолчанию на S1.
```
enable
conf t
interface vlan 200
 description Management VLAN 200
 ip address 192.168.1.66 255.255.255.224
 no shutdown
exit
ip default-gateway 192.168.1.65
end
write
```

c.	Настройте и активируйте интерфейс управления на S2 (VLAN 1), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того, установите шлюз по умолчанию на S2
```
enable
conf t
interface vlan 1
 description Management VLAN 1
 ip address 192.168.1.98 255.255.255.240
 no shutdown
exit
ip default-gateway 192.168.1.97
end
write
```

d.	Назначьте все неиспользуемые порты S1 VLAN Parking_Lot, настройте их для статического режима доступа и административно деактивируйте их. На S2 административно деактивируйте все неиспользуемые порты.

Для S1:
```
enable
conf t
interface range f0/1-4, f0/7-24, g0/1-2
 description Parking_Lot 
 switchport mode access
 switchport access vlan 999
 shutdown
exit
end
write
```

Для S2:
```
enable
conf t
interface range f0/1-4, f0/6-17, f0/19-24, g0/1-2
 shutdown
exit
end
write
```
### Шаг 8.	Назначьте сети VLAN соответствующим интерфейсам коммутатора.

Для S1:

```
enable
conf t
interface f0/6
 description PC-A - Client VLAN
 switchport mode access
 switchport access vlan 100
 no shutdown
exit
end
write
```

Для S2:
```
enable
conf t
interface f0/18
 description PC-B - Client VLAN
 switchport mode access
 switchport access vlan 1
 no shutdown
exit
end
write
```

b.	Убедитесь, что VLAN назначены на правильные интерфейсы.
```
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5
100  VLAN0100                         active    Fa0/6
200  VLAN0200                         active    
999  Parking_Lot                      active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 VLAN1000                         active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

Почему интерфейс F0/5 указан в VLAN 1? Потому что F0/5 является транковым интерфейсом

### Шаг 9.	Вручную настройте интерфейс S1 F0/5 в качестве транка 802.1Q.

```
enable
conf t
# a. Принудительно создаем trunk
interface f0/5
 switchport mode trunk
# b. Устанавливаем native VLAN 1000
 switchport trunk native vlan 1000
# c. Разрешаем прохождение VLAN 100, 200 и 1000
 switchport trunk allowed vlan 100,200,1000
```

 Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP? Адрес из пула VLAN200

### Часть 2.	Настройка и проверка двух серверов DHCPv4 на R1
### Шаг 1.	Настройте R1 с пулами DHCPv4 для двух поддерживаемых подсетей. Ниже приведен только пул DHCP для подсети A
```
enable
conf t
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp excluded-address 192.168.1.97 192.168.1.101
ip dhcp pool VLAN100_Pool
 network 192.168.1.0 255.255.255.192
 default-router 192.168.1.1
 domain-name CCNA-lab.com
 lease 2 12 30
 exit
ip dhcp pool R2_Client_LAN
 network 192.168.1.96 255.255.255.240
 default-router 192.168.1.97
 domain-name CCNA-lab.com
 lease 2 12 30
 exit
end
write
```
### Шаг 3.	Проверка конфигурации сервера DHCPv4

```
show ip dhcp pool
show ip dhcp bindings
show ip dhcp server statistics 
```
