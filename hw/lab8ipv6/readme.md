# Лабораторная работа 8. Реализация DHCPv4 
### Топология
![](lab8v6_1.png)
### Таблица адресации
| Устройство  | Интерфейс   |  IPv6-адрес          | 
|-------------|-------------|----------------------|
| R1          | G0/0/0      | 2001:db8:acad:2::1/64| 
|             |             | fe80::1              |
|             | G0/0/1      | 2001:db8:acad:1::1/64|
|             |             | fe80::1              |
| R2          | G0/0/0      | 2001:db8:acad:2::2/64| 
|             |             | fe80::2              | 
|             | G0/0/1      | 2001:db8:acad:3::1/64| 
|             |             | fe80::1              |
| PC-A        | NIC         | DHCP                 | 
| PC-B        | NIC         | DHCP                 |

### Задачи
### Часть 1. Создание сети и настройка основных параметров устройства
### Часть 2. Проверка назначения адреса SLAAC от R1
### Часть 3. Настройка и проверка сервера DHCPv6 без гражданства на R1
### Часть 4. Настройка и проверка состояния DHCPv6 сервера на R1
### Часть 5. Настройка и проверка DHCPv6 Relay на R2

### Решение
### Часть 1. Создание сети и настройка основных параметров устройства
### Шаг 1. Произведите базовую настройку маршрутизаторов

Настройка для R1:

```
en
conf t
hostname R1
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
banner motd # NO ENTER!!! #
ipv6 unicast-routing
exit
write
```

Настройка для R2 выполнена аналогично.

### Шаг 2. Настройка интерфейсов и маршрутизации для обоих маршрутизаторов

Настройка для R1:

```
en
conf t
int g0/0/0
 description Link to R2
 ipv6 address 2001:db8:acad:2::1/64
 ipv6 address fe80::1 link-local
 no sh
exit
int g0/0/1
 description LAN to PC-A
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 address fe80::1 link-local
 no sh
exit
ipv6 route ::/0 2001:db8:acad:2::2
exit
write

Настройка для R2:

```
en
conf t
interface g0/0/0
 description Link to R1
 ipv6 address 2001:db8:acad:2::2/64
 ipv6 address fe80::2 link-local
 no sh
exit
interface g0/0/1
 description LAN to PC-B
 ipv6 address 2001:db8:acad:3::1/64
 ipv6 address fe80::1 link-local
 no sh
exit
ipv6 route ::/0 2001:db8:acad:2::1
exit
write
```

### Часть 2. Проверка назначения адреса SLAAC от R1

```
PC-A > ipconfig

IPv6 Address. . . . . . . . . . . : 2001:db8:acad:1:5c43:ee7c:2959:da68
Link-local IPv6 Address . . . . . : fe80::5c43:ee7c:2959:da68%6
Default Gateway . . . . . . . . . : fe80::1%6
```
Откуда взялась часть адреса с идентификатором хоста? Сгенерирована автоматически

### Часть 3. Настройка и проверка сервера DHCPv6 без состояния на R1

Шаг 1. Изучите конфигурацию PC-A

text
ipconfig /all
Обратите внимание: основной DNS-суффикс отсутствует, DNS-серверы — адреса «локального сайта anycast» (fec0:0:0:ffff::1%1 и т.д.).

Шаг 2. Настройте R1 для предоставления DHCPv6 без состояния для PC-A

a. Создайте пул DHCPv6 на R1:

text
enable
conf t
ipv6 dhcp pool R1-STATELESS
 dns-server 2001:db8:acad::254
 domain-name STATELESS.com
exit
b. Настройте интерфейс G0/0/1 на R1:

text
interface g0/0/1
 ipv6 nd other-config-flag
 ipv6 dhcp server R1-STATELESS
exit
write
c. Перезапустите PC-A.

d. Проверьте вывод ipconfig /all на PC-A:

text
Connection-specific DNS Suffix . : STATELESS.com
DNS Servers . . . . . . . . . . . : 2001:db8:acad::254
DNS Suffix Search List. . . . . . : STATELESS.com
e. Проверьте подключение с PC-A:

text
ping 2001:db8:acad:3::1

