# Лабораторная работа 10 - Настройка протокола OSPFv2 для одной области
### Топология

![](lab10_1.png)

### Таблица адресации

| Устройство  | Интерфейс   |  IP-адрес            | Маска подсети       | 
|-------------|-------------|----------------------|---------------------|
| R1          | G0/0/1      | 10.53.0.1            |  255.255.255.0      |
|             | Loop1       | 172.16.1.1           |  255.255.255.0      |
| R2          | G0/0/1      | 10.53.0.2            |  255.255.255.0      |
|             | Loop1       | 192.168.1.1          |  255.255.255.0      |
  
### Цели
Часть 1. Создание сети и настройка основных параметров устройства  
Часть 2. Настройка и проверка базовой работы протокола  OSPFv2 для одной области  
Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области 

### Решение
### Часть 1. Создание сети и настройка основных параметров устройства
Базовая настройка для R1:

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
banner motd # NO ENTER!!! #
end
write memory

```

Базовая настройка для R2:

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
banner motd # NO ENTER!!! #
end
write memory
```
Базовая настройка для S1:

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
banner motd # NO ENTER!!! #
end
write memory

```

Базовая настройка для S2:

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
banner motd # NO ENTER!!! #
end
write memory
```

### Часть 2. Настройка и проверка базовой работы протокола OSPFv2 для одной области
### Шаг 1. Адресация интерфейсов и базовый OSPFv2
R1 - интерфейсы + OSPF
```
en
conf t
interface g0/0/1
ip address 10.53.0.1 255.255.255.0
no shutdown
interface loopback 1
ip address 172.16.1.1 255.255.255.0
no shutdown
exit
router ospf 56
router-id 1.1.1.1
network 10.53.0.0 0.0.0.255 area 0
exit
end
write memory
```

R2 - интерфейсы + OSPF
```
en
conf t
interface g0/0/1
ip address 10.53.0.2 255.255.255.0
no shutdown
interface loopback 1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
router ospf 56
router-id 2.2.2.2
network 10.53.0.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.255 area 0
exit
end
write memory
```
### Часть 3. Оптимизация и проверка конфигурации OSPFv2 для одной области
### Шаг 1. Оптимизация OSPFv2

R1 — приоритет, таймеры, маршрут по умолчанию, point-to-point, bandwidth

```
en
conf t
interface g0/0/1
ip ospf priority 50
ip ospf hello-interval 30
exit
ip route 0.0.0.0 0.0.0.0 loopback 1
router ospf 56
default-information originate
exit
interface loopback 1
ip ospf network point-to-point
exit
router ospf 56
auto-cost reference-bandwidth 10000
exit
end
write 
```
R2 — таймеры, point-to-point для Loopback1, passive-interface, bandwidth
```
en
conf t
interface g0/0/1
ip ospf hello-interval 30
exit
interface loopback 1
ip ospf network point-to-point
ip ospf passive-interface
exit
router ospf 56
passive-interface loopback 1
auto-cost reference-bandwidth 10000
exit
end
write 
```
### Шаг 2. Убедитесь, что оптимизация OSPFv2 реализовалась.

На R1:

```
show ip ospf interface g0/0/1
show ip route ospf
```
1. Приоритет интерфейса установлен равным 50, а временные интервалы — Hello 30, Dead 120, тип сети по умолчанию — Broadcast. 
2. Сеть R2 Loopback1 присутствует в таблице маршрутизации. Маска теперь составляет 24 бита, в отличие от 32 битов, ранее объявленных.

На R2:

```
show ip route ospf
ping 172.16.1.1
```
Единственная информация о маршруте OSPF - распространяемый по умолчанию маршрут R1.  
Ping до адреса интерфейса R1 Loopback 1 из R2 - успешен  

Почему стоимость OSPF для маршрута по умолчанию отличается от стоимости OSPF в R1 для сети 192.168.1.0/24?
20 у маршрута по умолчанию — это фиксированное число для внешних маршрутов, а у сети 192.168.1.0/24 — результат сложения по пути. Поэтому они разные. 
