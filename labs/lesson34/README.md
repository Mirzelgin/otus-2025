# Настройка NAT для IPv4
## Исходные данные

> [!NOTE]
> Построенная топология отличается от приведённой в методичке в части нумерации портов. Связано это с тем что данная работа выполняется в эмуляторе сети EVE-NG и нумерация портов устройств отличается от таковой в Cisco Packet Tracer

### Топология

![](img/Screenshot%202026-04-08%20at%2020-02-27%20EVE%20Topology.png)

### Таблица адресации

| Устройство | Интерфейс | IP-адрес        | Маска подсети   |
|------------|-----------|-----------------|-----------------|
| R1         | e0/0      | 209.165.200.230 | 255.255.255.248 |
| R1         | e0/1      | 192.168.1.1     | 255.255.255.0   |
| R2         | e0/0      | 209.165.200.225 | 255.255.255.248 |
| R2         | Lo1       | 209.165.200.1   | 255.255.255.224 |
| S1         | VLAN 1    | 192.168.1.11    | 255.255.255.0   |
| S2         | VLAN 1    | 192.168.1.12    | 255.255.255.0   |
| PC-A       | eth0      | 192.168.1.2     | 255.255.255.0   |
| PC-B       | eth0      | 192.168.1.3     | 255.255.255.0   |

## Задачи
- [Создание сети и настройка основных параметров устройства](#создание-сети-и-настройка-основных-параметров-устройства)
- [Настройка и проверка NAT для IPv4](#настройка-и-проверка-nat-для-ipv4)
- [Настройка и проверка PAT для IPv4](#настройка-и-проверка-pat-для-ipv4)
- [Настройка и проверка статического NAT для IPv4](#настройка-и-проверка-статического-nat-для-ipv4)

## Создание сети и настройка основных параметров устройства
### Произведём базовую настройку маршрутизаторов

**R1:**

```
en
conf t
hostname R1
no ip domain lookup
service password-encryption
enable secret class
!
line con 0
 password cisco
 login
 logging synchronous
!
line vty 0 4
 password cisco
 login
!
banner motd #
Unauthorized access is strictly prohibited!#
!
int e0/0
 ip addr 209.165.200.230 255.255.255.248
 no shutdown
!
int e0/1
 ip addr 192.168.1.1 255.255.255.0
 no shutdown
```

**R2:**

```
en
conf t
hostname R2
no ip domain lookup
service password-encryption
enable secret class
!
line con 0
 password cisco
 login
 logging synchronous
!
line vty 0 4
 password cisco
 login
!
banner motd #
Unauthorized access is strictly prohibited!#
!
int e0/0
 ip addr 209.165.200.225 255.255.255.248
 no shutdown
!
int lo1
 ip addr 209.165.200.1 255.255.255.224
!
ip route 0.0.0.0 0.0.0.0 209.165.200.230
```

### Произведём базовую настройку коммутаторов

**S1:**

```
en
conf t
hostname S1
no ip domain lookup
service password-encryption
enable secret class
!
line con 0
 password cisco
 login
 logging synchronous
!
line vty 0 4
 password cisco
 login
!
banner motd #
Unauthorized access is strictly prohibited!#
!
int vlan 1
 ip addr 192.168.1.11 255.255.255.0
 no shutdown
!
int e0/3
 shutdown
```

**S2:**

```
en
conf t
hostname S2
no ip domain lookup
service password-encryption
enable secret class
!
line con 0
 password cisco
 login
 logging synchronous
!
line vty 0 4
 password cisco
 login
!
banner motd #
Unauthorized access is strictly prohibited!#
!
int vlan 1
 ip addr 192.168.1.12 255.255.255.0
 no shutdown
!
int ra e0/2-3
 shutdown
```

## Настройка и проверка NAT для IPv4
## Настройка и проверка PAT для IPv4
## Настройка и проверка статического NAT для IPv4