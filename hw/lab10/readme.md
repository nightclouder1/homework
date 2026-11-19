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
configure terminal
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
configure terminal
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
