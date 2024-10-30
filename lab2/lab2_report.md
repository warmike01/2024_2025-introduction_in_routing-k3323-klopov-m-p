- University: [ITMO University](https://itmo.ru/ru/)
- Faculty: [FICT](https://fict.itmo.ru)
- Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)
- Year: 2024/2025
- Group: K3323
- Author: Klopov Mikhail Pavlovich
- Lab: Lab2
- Date of create: 27.10.2024
- Date of finished: 29.10.2024

# Отчет по лабораторной работе №2 "Эмуляция распределенной корпоративной сети связи, настройка статической маршрутизации между филиалами"

## Ход работы

Сначала была задана топология сети:


Файл *lab2.yml*:
```
name: lab_2

topology:
 nodes:
  Router_Berlin:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.10
   startup-config: /home/warmike01/routing/lab2/configs/berlin_config.cfg   
  Router_Frankfurt:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.20
   startup-config: /home/warmike01/routing/lab2/configs/frankfurt_config.cfg   
  Router_Moscow:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.30
   startup-config: /home/warmike01/routing/lab2/configs/moscow_config.cfg
  PC1_Berlin:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.110
  PC1_Frankfurt:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.120 
  PC1_Moscow:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.130  
 links:
   - endpoints: ["Router_Berlin:eth1", "Router_Frankfurt:eth1"]
   - endpoints: ["Router_Moscow:eth2", "Router_Frankfurt:eth2"] 
   - endpoints: ["Router_Moscow:eth1", "Router_Berlin:eth2"] 
   - endpoints: ["Router_Berlin:eth3", "PC1_Berlin:eth1"] 
   - endpoints: ["Router_Frankfurt:eth3", "PC1_Frankfurt:eth1"]          
   - endpoints: ["Router_Moscow:eth3", "PC1_Moscow:eth1"] 
mgmt:
 network: static
 ipv4-subnet: 192.168.2.0/24
```
Вывод команды *containerlab graph -t lab2.yml*:

![Screenshot from 2024-10-30 00-02-45](https://github.com/user-attachments/assets/cc4fcfa9-d76f-46b2-9ec5-9a9fa986a55b)

Как и в прошлой работе, вся настройка маршрутизаторов была выполнена в конфигурационных файлах. В каждом из них одни и те же действия, меняются только IP-адреса.

Файл *configs/berlin_config.cfg*:
```
/ip pool
add name=berlin_pool ranges=10.10.10.128-10.10.10.254
/ip dhcp-server
add address-pool=berlin_pool disabled=no interface=ether4 name=dhcp-berlin
/ip address
add address=10.10.10.129/25 interface=ether4 network=10.10.10.128
add address=2.2.2.2/24 interface=ether2 network=2.2.2.0
add address=3.3.3.3/24 interface=ether3 network=3.3.3.0
/ip dhcp-server network
add address=10.10.10.128/25 gateway=10.10.10.129
/ip route
add dst-address=4.4.4.0/24 gateway=2.2.2.3
add dst-address=10.10.20.128/25 gateway=2.2.2.3
add dst-address=10.10.30.128/25 gateway=3.3.3.2
```

Файл *configs/frankfurt_config.cfg*:
```
/ip pool
add name=frankfurt_pool ranges=10.10.20.128-10.10.20.254
/ip dhcp-server
add address-pool=frankfurt_pool disabled=no interface=ether4 name=dhcp-frankfurt
/ip address
add address=10.10.20.129/25 interface=ether4 network=10.10.20.128
add address=2.2.2.3/24 interface=ether2 network=2.2.2.0
add address=4.4.4.3/24 interface=ether3 network=4.4.4.0
/ip dhcp-server network
add address=10.10.20.128/25 gateway=10.10.20.129
/ip route
add dst-address=10.10.10.128/25 gateway=2.2.2.2
add dst-address=3.3.3.0/24 gateway=2.2.2.2
add dst-address=10.10.30.128/25 gateway=4.4.4.2
```

Файл *configs/moscow_config.cfg*:
```
/ip pool
add name=moscow_pool ranges=10.10.30.128-10.10.30.254
/ip dhcp-server
add address-pool=moscow_pool disabled=no interface=ether4 name=dhcp-moscow
/ip address
add address=10.10.30.129/25 interface=ether4 network=10.10.30.128
add address=3.3.3.2/24 interface=ether2 network=3.3.3.0
add address=4.4.4.2/24 interface=ether3 network=4.4.4.0
/ip dhcp-server network
add address=10.10.30.128/25 gateway=10.10.30.129
/ip route
add dst-address=2.2.2.0/24 gateway=4.4.4.3
add dst-address=10.10.10.128/25 gateway=4.4.4.3
add dst-address=10.10.20.128/25 gateway=3.3.3.3
```

Здесь необходимо более подробное объяснение. Первым делом объявляем IP-адрес для интерфейса, выходящего к компьютеру, пул адресов для DHCP-сервера и собственно сам DHCP-сервер. Потом объявляются IP-адреса для интерфейсов роутеров, связанных друг с другом - 2.2.2.2/2.2.2.3 для связи Берлина и Франкфурта, 3.3.3.2/3.3.3.3 для связи Берлина и Москвы, 4.4.4.2/4.4.4.3 для связи Берлина и Франкфурта. Затем создаются маршруты к сетям компьютеров других городов и к интерфейсам роутеров, до которых нет прямой связи.

Остаётся только настроить компьютеры. Первая команда *apt update && apt install -y iputils-ping && apt install -y iproute2 && apt install -y udhcpc && udhcpc -i eth1* одинакова для всех компьютеров. Она устанавливает утилиту *iproute2*, в числе прочего позволяющую настраивать маршруты, утилиту *ping* и DHCP-клиент и сразу же запрашивает у роутера IP-адрес:

![Screenshot from 2024-10-30 03-22-12](https://github.com/user-attachments/assets/01e4e613-d461-4b7d-8647-57d70dee85fb)

Вторая команда устанавливает default gateway для компьютера, в неё вставляется IP соответствующего интерфейса роутера. Например для компьютера во Франкфурте она будет иметь вид *ip route change default via 10.10.20.129 dev eth1*. После её ввода на всех компьютерах они могут пинговать друг друга (и роутеры):

![Screenshot from 2024-10-30 03-27-03](https://github.com/user-attachments/assets/dc718dcc-5883-4fc4-85b9-725d3dc82bca)

Готово! Лабораторная работа выполнена.
