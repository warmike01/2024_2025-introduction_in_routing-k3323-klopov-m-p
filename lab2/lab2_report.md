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
