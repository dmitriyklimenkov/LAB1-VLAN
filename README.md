# Лабораторная работа: конфигурация VLAN и настройка Inter-VLAN Routing методом Router-on-a-Stick
# Задание:
- Базовые настройки сетевых устройств
- Настроить VLAN на коммутаторах
- Настроить транковые каналы
- Настроить Inter-VLAN Routing

 # Решение:
 
 #Схема лабораторного стенда:
![](https://github.com/dmitriyklimenkov/LAB1-VLAN/blob/main/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D0%BB%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D0%BE%D0%B3%D0%BE%20%D1%81%D1%82%D0%B5%D0%BD%D0%B4%D0%B0%20VLAN.png)
 
# Таблица адресации:
| Устройство | Интерфейс  |   IP адрес   | Маска подсети | Шлюз по умолчанию |
| :----------|:----------:| :-----------:||:------------:| |----------------:|
| R1         | G0/0/1.3   | 192.168.3.1  | 255.255.255.0 |                   |
|            | G0/0/1.4   | 192.168.4.1  | 255.255.255.0 |                   | 
|            | G0/0/1.8   |              |               |                   | 
| S1         | VLAN3      | 192.168.3.11 | 255.255.255.0 | 192.168.3.1       |
| S2         | VLAN3      | 192.168.3.12 | 255.255.255.0 | 192.168.3.1       |
| PC-A       | NIC        | 192.168.3.3  | 255.255.255.0 | 192.168.3.1       |
| PC-B       | NIC        | 192.168.4.3  | 255.255.255.0 | 192.168.4.1       |

#Таблица VLAN:
|     VLAN      | Имя | Интерфейс |
| :------------ |:---------------:| -----:|
| 3      | Management        | S1: VLAN3 S2: VLAN3 S1: Fa0/6 |
| 4      | Operations        | S2: Fa0/18 |
| 7      | ParkingLot        | S1: F0/2-4, F0/7-24, G0/1-2 S2: F0/2-17, F0/19-24, G0/1-2 |
| 8      | Native            |               |


 # Конфигурация S1:
 `<en
conf t
 hostname S1
 no ip domain lookup
 enable password class
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

 clock set 21:02:00 29 september 2020
 copy run start
 vlan 3
 name Management
 exit
 int vlan 3
 ip address 192.168.3.11 255.255.255.0
 no shut
 des Management
 exit
 ip default-gateway 192.168.3.1
 vlan 7
 name ParkingLot
 exit
 int range fa0/2-4
 des ParkingLot
 shut
 shutdown 
 exit
 int range fa0/2-4
 switch
 switchport mode access
 switch
 switchport access vlan 7
 int range fa0/7-24
 switchport mode access
 switchport access vlan 7
 shutdown
 des ParkingLot
 int range gi0/1-2
 switchport mode access
 switchport access vlan 7
 des ParkingLot
 shut
 int fa0/6
 switchport mode access
 switchport access vlan 3

 int fa0/1
 des TO_S2
 switchport mode trunk
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4
 int fa0/5
 switchport mode trunk
 switchport trunk allowed vlan 3,4
 switchport trunk native vlan 8
 copy run start>`
 
 # Конфигурация S2:
 `<en
conf t
 hostname S2
 no ip domain lookup
 enable password class
 line console 0
 password cisco
 login
 exit
 line vty 0 15
 password cisco
 login
 exit
 service password-encryption 
 banner motd $ This is secure system. Authorized access only. $
 exit 

 S2#clock set 21:05:00 29 september 2020
 vlan 3
 name Management
 exit
 int vlan 3
 

 ip address 192.168.3.12 255.255.255.0
 no shut
 des Management
 exit
 ip default-gateway 192.168.3.1
 vlan 7
 name ParkingLot
 exit
 int range fa0/2-17
 des ParkingLot
 shut
 switchport mode access
 switchport access vlan 7
 exit
 int range fa0/19-24
 switchport access vlan 7
 switchport mode access
 des ParkingLot
 shut
 exit
 int range gi0/1-2
 des ParkingLot
 switchport mode access
 switchport access vlan 7
 shut
 vlan 4
 name Operations
 exit
 int fa0/18
 switchport mode access
 switchport access vlan 4

 int fa0/1
 des TO_S1
 switchport mode trunk
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4
 copy run start>`
 
 # Конфигурация R1:
 `<en
conf t
 hostname R1
 no ip domain lookup
 enable password class
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
 
 int gi0/0/1.3
 des Management
 encapsulation dot1q 3
 ip address 192.168.3.1 255.255.255.0
 int gi0/0/1.4
 des Operations
 encapsulation dot1q 4
 ip address 192.168.4.1 255.255.255.0
 copy run start>` 
 
 
 
