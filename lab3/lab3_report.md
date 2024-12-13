- University: [ITMO University](https://itmo.ru/ru/)
- Faculty: [FICT](https://fict.itmo.ru)
- Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)
- Year: 2024/2025
- Group: K3323
- Author: Klopov Mikhail Pavlovich
- Lab: Lab3
- Date of create: 11.11.2024
- Date of finished: 27.11.2024

# Отчет по лабораторной работе №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

## Ход работы

Сначала была задана топология сети:

Файл lab3.yml:
```
name: lab_3

topology:
 nodes:
  Router_Petersburg:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.10
   startup-config: /home/warmike01/routing/lab3/configs/petersburg_config.cfg   
  Router_Helsinki:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.20
   startup-config: /home/warmike01/routing/lab3/configs/helsinki_config.cfg    
  Router_Moscow:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.30
   startup-config: /home/warmike01/routing/lab3/configs/moscow_config.cfg     
  Router_Slovenia:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.40
   startup-config: /home/warmike01/routing/lab3/configs/slovenia_config.cfg         
  Router_London:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.50
   startup-config: /home/warmike01/routing/lab3/configs/london_config.cfg   
  Router_New_York:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.60
   startup-config: /home/warmike01/routing/lab3/configs/newyork_config.cfg       
  PC1_Petersburg:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.110  
  SGI_Prism:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.120     
 links:
   - endpoints: ["Router_Petersburg:eth1", "Router_Moscow:eth1"]
   - endpoints: ["Router_Petersburg:eth2", "Router_Helsinki:eth2"]
   - endpoints: ["Router_Slovenia:eth2", "Router_Moscow:eth2"]
   - endpoints: ["Router_Slovenia:eth3", "Router_Helsinki:eth3"]
   - endpoints: ["Router_London:eth1", "Router_Helsinki:eth1"]
   - endpoints: ["Router_Slovenia:eth1", "Router_New_York:eth1"]
   - endpoints: ["Router_London:eth2", "Router_New_York:eth2"]
   - endpoints: ["Router_Petersburg:eth3", "PC1_Petersburg:eth1"]
   - endpoints: ["Router_New_York:eth3", "SGI_Prism:eth1"]
   
   

mgmt:
 network: static
 ipv4-subnet: 192.168.2.0/24
```
Вывод команды *containerlab graph -t lab3.yml*:

![Карта сети](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab3/Screenshot%20from%202024-12-13%2016-04-11.png)

Большая часть настройки маршрутизаторов была выполнена в конфигурационных файлах. Однако, OSPF почему-то не работает при объявлнии в конфигах, поэтому пришлось вго настроить на всех роутерах с помощью WinBox:

![Настройка OSPF](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab3/Screenshot%20from%202024-12-11%2023-54-51.png)

Через конфиги на всех маршрутизаторах были настроены IP-адреса на интерфейсах и MPLS. На конечных пунктах сети (в Санкт-Петербурге и Нью-Йорке) были также настроены VPLS-туннель и DHCP-сервер. Конфигурации роутеров:

Санкт-Петербург (petersburg_config.cfg):

```
/interface bridge
add name=loopback
add name=vpn
/ip pool
add name=petersburg_pool ranges=10.0.0.128-10.0.0.190
/ip dhcp-server
add address-pool=petersburg_pool disabled=no interface=ether4 name=dhcp-petersburg
/ip dhcp-server network
add address=10.0.0.128/26 gateway=10.0.0.129
/ip address
add address=10.0.0.129/26 interface=vpn network=10.0.0.128
add address=5.5.5.5/30 interface=ether2
add address=5.5.5.9/30 interface=ether3 
add address=10.0.0.2/32 interface=loopback
/mpls ldp
set enabled=yes lsr-id=10.0.0.2 transport-address=10.0.0.2
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/interface vpls add disabled=no name=l2vpn remote-peer=10.0.0.7 cisco-style=yes cisco-style-id=5
/interface bridge port
add bridge=vpn interface=ether4
add bridge=vpn interface=l2vpn
```

Москва (moscow_config.cfg):

```
/interface bridge
add name=loopback
/ip address
add address=5.5.5.6/30 interface=ether2 
add address=5.5.5.13/30 interface=ether3
add address=10.0.0.3/32 interface=loopback
/mpls ldp
set enabled=yes lsr-id=10.0.0.3 transport-address=10.0.0.3
/mpls ldp interface
add interface=ether2
add interface=ether3
```
Хельсинки (helsinki_config.cfg):

```
/interface bridge
add name=loopback
/ip address
add address=5.5.5.21/30 interface=ether2
add address=5.5.5.10/30 interface=ether3
add address=5.5.5.17/30 interface=ether4 
add address=10.0.0.4/32 interface=loopback
/mpls ldp
set enabled=yes lsr-id=10.0.0.4 transport-address=10.0.0.4
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
```
Любляна (slovenia_config.cfg):

```
/interface bridge
add name=loopback
/ip address
add address=5.5.5.25/30 interface=ether2
add address=5.5.5.14/30 interface=ether3
add address=5.5.5.18/30 interface=ether4 
add address=10.0.0.5/32 interface=loopback
/mpls ldp
set enabled=yes lsr-id=10.0.0.5 transport-address=10.0.0.5
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
```
Лондон (london_config.cfg):

```
/interface bridge
add name=loopback
/ip address
add address=5.5.5.22/30 interface=ether2
add address=5.5.5.29/30 interface=ether3
add address=10.0.0.6/32 interface=loopback
/mpls ldp
set enabled=yes lsr-id=10.0.0.6 transport-address=10.0.0.6
/mpls ldp interface
add interface=ether2
add interface=ether3
```
Нью-Йорк (newyork_config.cfg):

```
/interface bridge
add name=loopback
add name=vpn
/ip pool
add name=newyork_pool ranges=10.0.0.193-10.0.0.254
/ip dhcp-server
add address-pool=newyork_pool disabled=no interface=ether4 name=dhcp-newyork
/ip dhcp-server network
add address=10.0.0.192/26 gateway=10.0.0.193
/ip address
add address=10.0.0.193/26 interface=vpn network=10.0.0.192
add address=5.5.5.26/30 interface=ether2
add address=5.5.5.30/30 interface=ether3
add address=10.0.0.7/32 interface=loopback
/mpls ldp
set enabled=yes lsr-id=10.0.0.7 transport-address=10.0.0.7
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/interface vpls add disabled=no name=l2vpn remote-peer=10.0.0.2 cisco-style=yes cisco-style-id=5
/interface bridge port
add bridge=vpn interface=ether4
add bridge=vpn interface=l2vpn
```
