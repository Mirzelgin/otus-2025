# Настройка и проверка расширенных списков контроля доступа
## Исходные данные

> [!NOTE]
> Построенная топология отличается от приведённой в методичке в части нумерации портов. Связано это с тем что данная работа выполняется в эмуляторе сети EVE-NG и нумерация портов устройств отличается от таковой в Cisco Packet Tracer

### Топология

![](img/Screenshot%202026-03-30%20at%2010-43-46%20EVE%20Topology.png)

### Таблица адресации
| Устройство | Интерфейс | IP-адрес       | Маска подсети | Шлюз по умолчанию |
|------------|-----------|----------------|---------------|-------------------|
| R1         | e0/0      | -              | -             | -                 |
| R1         | e0/0.20   | 10.20.0.1      | 255.255.255.0 |                   |
| R1         | e0/0.30   | 10.30.0.1      | 255.255.255.0 |                   |
| R1         | e0/0.40   | 10.40.0.1      | 255.255.255.0 |                   |
| R1         | e0/0.1000 | -              | 255.255.255.0 |                   |
| R1         | Loopback1 | 172.16.1.1     | 255.255.255.0 |                   |
| R2         | e0/0      | 10.20.0.4      | 255.255.255.0 | -                 |
| S1         | VLAN 20   | 10.20.0.2      | 255.255.255.0 | 10.20.0.1         |
| S2         | VLAN 20   | 10.20.0.3      | 255.255.255.0 | 10.20.0.1         |
| PC-A       | eth0      | 10.30.0.10     | 255.255.255.0 | 10.30.0.1         |
| PC-B       | eth0      | 10.40.0.10     | 255.255.255.0 | 10.40.0.1         |


### Таблица VLAN
| VLAN | Имя         | Назначенный интерфейс  |
|------|-------------|------------------------|
| 20   | Management  | S2: e0/1               |
| 30   | Operations  | S1: e0/2               |
| 40   | Sales       | S2: e0/2               |
| 999  | ParkingLot  | S1: e0/3 </br>S2: e0/3 |
| 1000 | Native      |  —                     |

## Задачи
- [Создание сети и настройка основных параметров устройства](#создание-сети-и-настройка-основных-параметров-устройства)
- [Настройка и проверка списков расширенного контроля доступа](#настройка-и-проверка-списков-расширенного-контроля-доступа)

## Создание сети и настройка основных параметров устройства
Выполним базовую настройку сетевых устройств. 

### VLAN
Настроим на коммутаторах VLAN и назначим соответствующим портам

```
vlan 20
 name Management
vlan 30
 name Operations
vlan 40
 name Sales
vlan 999
 name ParkingLog
vlan 1000
 name Native
```

**S1:**

```
int e0/2
 switchport mode access
 switchport access vlan 30
int e0/3
 switchport mode access
 switchport access vlan 999
 shutdown
int vlan 20
 ip addr 10.20.0.2 255.255.255.0
 no sh
!
ip route 0.0.0.0 0.0.0.0 10.20.0.1
```

**S2:**

```
int e0/1
 switchport mode access
 switchport access vlan 20
int e0/2
 switchport mode access
 switchport access vlan 40
int e0/3
 switchport mode access
 switchport access vlan 999
 shutdown
int vlan 20
 ip addr 10.20.0.3 255.255.255.0
 no sh
!
ip route 0.0.0.0 0.0.0.0 10.20.0.1
```

### Магистральные каналы
Настроим транк между коммутаторами. Так как они соединены одноименными портами конфигурация будет следующей

```
int e0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40,1000
 switchport trunk native vlan 1000
```

И настраиваем транк на коммутаторе **S1** к маршрутизатору с теми же параметрами

### Маршрутизация
#### R1
Настроим подинтерфейсы на маршрутизаторе **R1**, VLAN и IP-адреса

```
int e0/0.20
 description Management
 encapsulation dot1Q 20
 ip address 10.20.0.1 255.255.255.0

int e0/0.30
 description Operations
 encapsulation dot1Q 30
 ip address 10.30.0.1 255.255.255.0

int e0/0.40
 description Sales
 encapsulation dot1Q 40
 ip address 10.40.0.1 255.255.255.0

int e0/0.1000
 description Native
 encapsulation dot1Q 1000 native
```

Настроим loopback имитирующий подключение к интернету

```
int lo 1
 ip addr 172.16.1.1 255.255.255.0
```

Проверим конфигурацию подинтерфейсов
 
![](img/2026-04-16_21-39-39.png)

#### R2
Выполним настройку **R2**

```
ip route 0.0.0.0 0.0.0.0 10.20.0.1
!
int e0/0
 ip addr 10.20.0.4 255.255.255.0
 no sh
```

### Удалённый доступ
Настроим доступ по ssh, а также доступ через web

```
ip domain-name ccna-lab.com
ip ssh version 2
crypto key generate rsa general-keys modulus 1024
username SSHadmin privilege 15 secret $cisco123!
!
line vty 0 4
 transport input ssh
 login local
```

На **R1** активируем web сервер

```
ip http secure-server
ip http authentication local
```

## Настройка и проверка списков расширенного контроля доступа
Списком `SALES_ACL` мы выполним следующие политики:
- Сеть Sales не может использовать SSH в сети Management (но в  другие сети SSH разрешен)
- Сеть Sales не имеет доступа к IP-адресам в сети Management с помощью любого веб-протокола. Сеть Sales также не имеет доступа к интерфейсам R1 кроме Loopback1 с помощью любого веб-протокола. Разрешён весь другой веб-трафик
- Сеть Sales не может отправлять эхо-запросы ICMP в сети Operations или Management. Разрешены эхо-запросы ICMP к другим адресатам

```
ip access-list extended SALES_ACL
 remark BLOCK SSH,WEB TO MANAGEMENT
 deny tcp any 10.20.0.0 0.0.0.255 eq 22 80 443
 remark BLOCK WEB TO ROUTER IP
 deny tcp any host 10.30.0.1 eq 80 443
 deny tcp any host 10.40.0.1 eq 80 443
 remark BLOCK ICMP TO OPERATIONS
 deny icmp any 10.30.0.1 0.0.0.255 echo
 remark BLOCK ICMP TO MANAGEMENT
 deny icmp any 10.20.0.1 0.0.0.255 echo
 permit ip any any
```

Применим данный список на интерфейс `e0/0.40` по направлению `in`

```
int e0/0.40
 ip access-group SALES_ACL in
```

Списком `OPERATIONS_ACL` выполним следующие политики:
- Cеть Operations  не может отправлять ICMP эхозапросы в сеть Sales. Разрешены эхо-запросы ICMP к другим адресатам

```
ip access-list extended OPERATIONS_ACL
 remark BLOCK ICMP FROM OPERATIONS TO SALES 
 deny icmp any 10.40.0.1 0.0.0.255 echo
 permit ip any any
```

Применим данный список на интерфейс `e0/0.30` по направлению `in`

```
int e0/0.30
 ip access-group OPERATIONS_ACL in
```

Проведём различные тесты и посмотрим счетчики ACL

![](img/2026-04-22_22-51-26.png)

По результата тестов списки отрабатывают корректно, доступ к интерфейсу loopback1 маршрутизатора **R1** сохранён

![](img/2026-04-22_22-52-58.png)

![](img/2026-04-22_22-56-04.png)