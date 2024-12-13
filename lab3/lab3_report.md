- University: [ITMO University](https://itmo.ru/ru/)
- Faculty: [FICT](https://fict.itmo.ru)
- Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)
- Year: 2024/2025
- Group: K3323
- Author: Klopov Mikhail Pavlovich
- Lab: Lab3
- Date of create: 11.10.2024
- Date of finished: 27.11.2024

# Отчет по лабораторной работе №2 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

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
