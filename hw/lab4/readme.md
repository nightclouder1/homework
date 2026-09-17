# Лабораторная работа 4. Настройка IPv6-адресов на сетевых устройствах 
### Топология
![](lab4_1.png)
### Таблица адресации
| Устройство  | Интерфейс |  IPv6-адрес          | Link local IPv6     | Длина префикса | Шлюз по умолчанию |
|-------------|-----------|----------------------|---------------------|----------------|-------------------|
| R1          | G0/0/0    | 2001:db8:acad:a::1   |    fe80::1          | 64             | -                 |
|             | G0/0/1    | 2001:db8:acad:1::1   |    fe80::1          | 64             | -                 |
| S1          | VLAN 1    | 2001:db8:acad:1::b   |    fe80::1          | 64             | -                 |
| PC-A        | NIC       | 2001:db8:acad:1::3   |    SLAAC            | 64             | fe80::1           |
| PC-B        | NIC       | 2001:db8:acad:a::3   |    SLAAC            | 64             | fe80::1           |

### Задачи:
#### Часть 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора
#### Часть 2. Ручная настройка IPv6-адресов
#### Часть 3. Проверка сквозного соединения

### Решение:
### Часть 1 Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора
#### Шаг 1. Настройка маршрутизатора
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
R1# write

```
#### Шаг 2. Настройка коммутатора

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
S1(config)# exit
S1# write 
```
```
S1# configure terminal
S1(config)# sdm prefer dual-ipv4-and-ipv6 default
S1(config)# end
S1# reload
```
### Часть 2 Ручная настройка IPv6-адресов
#### Шаг 1. Назначьте IPv6-адреса интерфейсам Ethernet на R1
```
R1(config)# interface gigabitethernet 0/0
R1(config-if)# ipv6 address 2001:db8:acad:a::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface gigabitethernet 0/1
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config-if)# exit
R1# write
```
#### Шаг 2. Активируйте IPv6-маршрутизацию на R1
```
PC-B > ipconfig
```
Назначен ли индивидуальный IPv6-адрес сетевой интерфейсной карте (NIC) на PC-B? Да, у него есть индивидуальный link-local IPv6-адрес, который он сам себе создал из MAC-адреса при помощи технологии SLAAC.
```
R1(config)# > IPv6 unicast-routing
```
Почему PC-B получил глобальный префикс маршрутизации и идентификатор подсети, которые вы настроили на R1?  Потому что после активации ipv6 unicast-routing маршрутизатор R1 начал рассылать сообщения. PC-B, используя SLAAC сформировал свой IPv6-адрес.

#### Шаг 3. Назначьте IPv6-адреса интерфейсу управления (SVI) на S1.

```
S1(config)# interface vlan 1
S1(config-if)# ipv6 address 2001:db8:acad:1::b/64
S1(config-if)# ipv6 address fe80::b link-local
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# exit
```
#### Шаг 4. Назначьте компьютерам статические IPv6-адреса.

```
PC-B >   
   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::202:17FF:FE13:A538
   IPv6 Address....................: 2001:DB8:ACAD:A::3
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0

PC-A >
   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::260:47FF:FEE8:ACC2
   IPv6 Address....................: 2001:DB8:ACAD:1::3
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

### Часть 3. Проверка сквозного подключения

```
PC-A > ping FE80:1 #Маршрутизатор. 
PC-A > ping 2001:DB8:ACAD:1::B #Коммутатор. 
PC-A > ping FE80::B #Коммутатор.
```
```
Tracing route to 2001:DB8:ACAD:A::3 over a maximum of 30 hops:    

  1   0 ms      0 ms      0 ms      2001:DB8:ACAD:1::1 #R1.   
  2   0 ms      0 ms      0 ms      2001:DB8:ACAD:A::3 #PC-B.   

Trace complete.  
```


```
PC-B > ping 2001:db8:acad:a::3 #PC-B. 
PC-B > ping 2001:db8:acad:a::1 #R1. 
```
1.	Почему обоим интерфейсам Ethernet на R1 можно назначить один и тот же локальный адрес канала — FE80::1? Потому что это link-local адрес для двух отдельных сетей, одна для сети PC-A, вторая для PC-B.  
2.	Какой идентификатор подсети в индивидуальном IPv6-адресе 2001:db8:acad::aaaa:1234/64? Идентификатор подсети: 0000
16 бит 2001 : 16 бит 0db8 : 16 бит acad : 16 бит 0000 : 0000 : 0000 : aaaa : 1234 /64
id подсети 4 октет : 0000
