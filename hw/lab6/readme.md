# Лабораторная работа 6. Внедрение маршрутизации между виртуальными локальными сетями 
### Топология
![](lab6_3.png)

### Таблица адресации
| Устройство  | Интерфейс   |  IPv6-адрес          | Маска подсети.      | Шлюз по умолчанию |
|-------------|-------------|----------------------|---------------------|-------------------|
| R1          | G0/0/1.10   | 192.168.10.1         | 255.255.255.0       | -                 |
|             | G0/0/1.20   | 192.168.20.1         | 255.255.255.0       | -                 |
|             | G0/0/1.30   | 192.168.30.1         | 255.255.255.0       | -                 |
|             | G0/0/1.1000 | -                    | -                   | -                 |
| S1          | VLAN10      | 192.168.10.11        | 255.255.255.0       | 192.168.10.1      |
| S2          | VLAN10      | 192.168.10.12        | 255.255.255.0       | 192.168.10.1      | 
| PC-A        | NIC         | 192.168.20.3         | 255.255.255.0       | 192.168.20.1      |
| PC-B        | NIC         | 192.168.30.3         | 255.255.255.0       | 192.168.30.1      |


### Таблица VLAN

| VLAN        | Имя         |  Назначенный интерфейс | 
|-------------|-------------|------------------------|
| 10          | Управление  | S1: VLAN10 S2: VLAN10  | 
| 20          | Sales       | S1: F0/6               |  
| 30          | Opertions   | S2: F0/18              | 
| 999         | Parking_lot | С1: F0/2-4, F0/7-24, G0/1-2 С2: F0/2-17, F0/19-24, G0/1-2                      | 
| 1000        | Собственная | -                      | 

### Задачи:
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Создание сетей VLAN и назначение портов коммутатора
#### Часть 3. Настройка транка 802.1Q между коммутаторами.
#### Часть 4. Настройка маршрутизации между сетями VLAN
#### Часть 5. Проверка, что маршрутизация между VLAN работает

### Решение:
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Шаг 1. Создайте сеть согласно топологии
![](lab6_2.png)

#### Шаг 2. Настройте базовые параметры для маршрутизатора
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
R1# clock set 14:30:00 28 Sep 2026
R1# write
```
#### Шаг 3. Настройте базовые параметры каждого коммутатора.
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
S1# clock set 14:30:00 28 Sep 2026
S1# write 
```
Аналогичная настройка для Switch2

#### Шаг 4. Настройте узлы ПК
Настройка узлов ПК выполнена согласно топологии:
![](lab6_4.png)
![](lab6_5.png)
