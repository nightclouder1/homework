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
### Часть 2 Ручная настройка IPv6-адресов
#### Шаг 1. Назначьте IPv6-адреса интерфейсам Ethernet на R1
```
R1(config)# interface gigabitethernet 0/0
R1(config-if)# ipv6 address 2001:db8:acad:a::1/64
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface gigabitethernet 0/1
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
R1(config-if)# no shutdown
R1(config-if)# exit
R1# write
```
