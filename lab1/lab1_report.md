- University: [ITMO University](https://itmo.ru/ru/)
- Faculty: [FICT](https://fict.itmo.ru)
- Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)
- Year: 2024/2025
- Group: K3323
- Author: Klopov Mikhail Pavlovich
- Lab: Lab1
- Date of create: 11.09.2023
- Date of finished: 23.10.2024

# Отчет по лабораторной работе №1 "Установка ContainerLab и развёртывание тестовой сети связи"

## Ход работы

Сначала на компьютер с Ubuntu LTS 20.04 был установлен Containerlab.

Затем с помощью YAML-файла была создана топология сети/ 

Файл *lab1.yml*:
```
name: lab_1

topology:
 nodes:
  Router1:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.254
   startup-config: /home/warmike01/routing/lab1/configs/router_config.cfg
  L3_Switch1:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   startup-config: /home/warmike01/routing/lab1/configs/switch1_config.cfg
   mgmt-ipv4: 192.168.2.10
  L3_Switch2:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.20
   startup-config: /home/warmike01/routing/lab1/configs/switch2_config.cfg
  L3_Switch3:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.30
   startup-config: /home/warmike01/routing/lab1/configs/switch3_config.cfg
  PC1:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.100
  PC2:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.101
 links:
   - endpoints: ["Router1:eth1", "L3_Switch1:eth1"]
   - endpoints: ["L3_Switch2:eth1", "L3_Switch1:eth2"]
   - endpoints: ["L3_Switch3:eth1", "L3_Switch1:eth3"]
   - endpoints: ["PC1:eth1", "L3_Switch2:eth2"]
   - endpoints: ["PC2:eth1", "L3_Switch3:eth2"]

mgmt:
 network: static
 ipv4-subnet: 192.168.2.0/24
```
Вывод команды *containerlab graph -t lab1.yml*:

![Screenshot from 2024-10-23 23-05-27](https://github.com/user-attachments/assets/7a46ad62-3ea1-412a-839c-c3b5345833c0)

Затем были пробы и ошибки. Много проб и много ошибок. Но в итоге удалось создать конфигурации роутера и коммутаторов, при которых они не требуют ручной настройки.
