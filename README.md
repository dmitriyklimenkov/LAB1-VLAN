# Лабораторная работа: конфигурация VLAN и настройка Inter-VLAN Routing методом Router-on-a-Stick
# Задание:
- Базовые настройки сетевых устройств
- Настроить VLAN на коммутаторах
- Настроить транковые каналы
- Настроить Inter-VLAN Routing

 # Решение:
 
 # Схема лабораторного стенда:
![](https://github.com/dmitriyklimenkov/LAB1-VLAN/blob/main/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D0%BB%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D0%BE%D0%B3%D0%BE%20%D1%81%D1%82%D0%B5%D0%BD%D0%B4%D0%B0%20VLAN.png)
 
# Таблица адресации:
| Устройство | Интерфейс  |   IP адрес   | Маска подсети | Шлюз по умолчанию |
| :----------|:----------:| :-----------:|:-------------:| -----------------:|
| R1         | G0/0/1.3   | 192.168.3.1  | 255.255.255.0 |                   |
|            | G0/0/1.4   | 192.168.4.1  | 255.255.255.0 |                   | 
|            | G0/0/1.8   |              |               |                   | 
| S1         | VLAN3      | 192.168.3.11 | 255.255.255.0 | 192.168.3.1       |
| S2         | VLAN3      | 192.168.3.12 | 255.255.255.0 | 192.168.3.1       |
| PC-A       | NIC        | 192.168.3.3  | 255.255.255.0 | 192.168.3.1       |
| PC-B       | NIC        | 192.168.4.3  | 255.255.255.0 | 192.168.4.1       |

# Таблица VLAN:
|     VLAN      | Имя | Интерфейс |
| :------------ |:---------------:| -----:|
| 3      | Management        | S1: VLAN3 S2: VLAN3 S1: Fa0/6 |
| 4      | Operations        | S2: Fa0/18 |
| 7      | ParkingLot        | S1: F0/2-4, F0/7-24, G0/1-2 S2: F0/2-17, F0/19-24, G0/1-2 |
| 8      | Native            |               |


# 1. Конфигурация S1.
- основные настройки коммутатора S1:
```conf t
 hostname S1
 no ip domain lookup
 enable secret class
 line console 0
 password cisco
 login
 exit
 line vty 0 15
 password cisco
 login
 exit
 service password-encryption 
 banner motd $ This is a secure system. Authorized access only. $
 exit
 ```
- настройка транковых портов Fa0/1 и Fa0/5:
```int fa0/1
 des TO_S2
 switchport mode trunk
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4
 int fa0/5
 switchport mode trunk
 switchport trunk allowed vlan 3,4
 switchport trunk native vlan 8
 ```
- настройка порта доступа Fa0/6:
```int fa0/6
 switchport mode access
 switchport access vlan 3
 ```
Файл конфигурации [здесь](https://github.com/dmitriyklimenkov/LAB1-VLAN/blob/main/S1).
 
# 2. Конфигурация S2.
Сделаем основные настройки коммутатора S2, затем настроим порт Fa0/1 как транковый и Fa0/18 как порт доступа. Файл конфигурации [здесь](https://github.com/dmitriyklimenkov/LAB1-VLAN/blob/main/S2).

# 3. Конфигурация R1.
- основные настройки маршрутизатора R1:
```conf t
 hostname R1
 no ip domain lookup
 enable secret class
 line console 0
 password cisco
 login
 exit

 line vty 0 15
 password cisco
 login
 exit
 service password-encryption
 exit
 copy run start
 clock set 20:45:00 29 september 2020
 int gi0/0/1
 no shut
 exit
 banner motd $ This is a secure system. Authorized access only $
 ```
- настройка и активация сабинтерфейсов для каждого VLAN:
 ```int gi0/0/1.3
 des Management
 encapsulation dot1q 3
 ip address 192.168.3.1 255.255.255.0
 int gi0/0/1.4
 des Operations
 encapsulation dot1q 4
 ip address 192.168.4.1 255.255.255.0
 ```
Файл конфигурации [здесь](https://github.com/dmitriyklimenkov/LAB1-VLAN/blob/main/R1).

# 4. Проверка связности.
С PC-B отправим пинги на шлюз по умолчанию и на PC-A
![](https://github.com/dmitriyklimenkov/LAB1-VLAN/blob/main/%D0%A1%D0%B2%D1%8F%D0%B7%D0%BD%D0%BE%D1%81%D1%82%D1%8C1.png) ![](https://github.com/dmitriyklimenkov/LAB1-VLAN/blob/main/%D0%A1%D0%B2%D1%8F%D0%B7%D0%BD%D0%BE%D1%81%D1%82%D1%8C2.png)
Видно, что пинги проходят, из этого делаем вывод, что все настроено верно.
