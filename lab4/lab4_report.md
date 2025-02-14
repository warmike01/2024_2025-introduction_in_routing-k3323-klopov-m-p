- University: [ITMO University](https://itmo.ru/ru/)
- Faculty: [FICT](https://fict.itmo.ru)
- Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)
- Year: 2024/2025
- Group: K3323
- Author: Klopov Mikhail Pavlovich
- Lab: Lab4
- Date of create: 14.01.2025
- Date of finished: 23.01.2025

# Отчет по лабораторной работе №4 "Эмуляция распределенной корпоративной сети связи, настройка iBGP, организация L3VPN, VPLS"

## Ход работы

Сначала была задана топология сети:

Файл lab4.yml:

```
name: lab_4

topology:
 nodes:
  Router_Petersburg:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.10
   startup-config: /home/warmike01/routing/lab4/configs/petersburg_config.cfg   
  Router_Helsinki:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.20
   startup-config: /home/warmike01/routing/lab4/configs/helsinki_config.cfg      
  Router_Slovenia:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.30
   startup-config: /home/warmike01/routing/lab4/configs/slovenia_config.cfg   
  Router_SVL:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.40
   startup-config: /home/warmike01/routing/lab4/configs/svl_config.cfg         
  Router_London:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.50
   startup-config: /home/warmike01/routing/lab4/configs/london_config.cfg   
  Router_New_York:
   kind: vr-mikrotik_ros
   image: vrnetlab/vr-routeros:6.47.9
   mgmt-ipv4: 192.168.2.60
   startup-config: /home/warmike01/routing/lab4/configs/newyork_config.cfg       
  PC1_SPB:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.110  
  PC2_NYC:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.120 
  PC3_SVL:
   kind: linux
   image: ubuntu:focal
   mgmt-ipv4: 192.168.2.130     
 links:
   - endpoints: ["Router_Petersburg:eth1", "Router_Helsinki:eth1"]
   - endpoints: ["Router_London:eth2", "Router_Helsinki:eth2"]
   - endpoints: ["Router_London:eth1", "Router_New_York:eth1"]
   - endpoints: ["Router_Slovenia:eth1", "Router_Helsinki:eth3"]
   - endpoints: ["Router_Slovenia:eth2", "Router_London:eth3"]
   - endpoints: ["Router_Slovenia:eth3", "Router_SVL:eth1"]
   - endpoints: ["Router_Petersburg:eth2", "PC1_SPB:eth1"]
   - endpoints: ["Router_New_York:eth3", "PC2_NYC:eth1"]
   - endpoints: ["Router_SVL:eth2", "PC3_SVL:eth1"]
   

   
   

mgmt:
 network: static
 ipv4-subnet: 192.168.2.0/24
```

Вывод команды *containerlab graph -t lab4.yml*:

![Карта сети](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab4/Screenshot%20from%202025-01-23%2005-19-36.png)



Базовая настройка маршрутизаторов была выполнена в конфигурационных файлах (ознакомиться с ними можно в папке проекта), а OSPF и BGP настроены с помощью WinBox. 

Сначала, как и в прошлой работе, на каждом роутере объявлен OSPF (пример для Хельсинки):

![OSPF](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab4/Screenshot%20from%202025-01-23%2016-45-26.png)

Затем на каждом роутере объявлен инстанс BGP:

![Инстанс BGP](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab4/Screenshot%20from%202025-01-23%2016-59-01.png)

На каждом роутере объявлены соседи:

![Соседи BGP](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab4/Screenshot%20from%202025-01-23%2005-05-36.png)

Наконец, на роутерах, подключённых к компьютерам (SPB, SVL, NYC) объявлен VRF (пример для Нью-Йорка):

![VRF](https://github.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/blob/master/lab4/Screenshot%20from%202025-01-23%2017-11-36.png)

В результате таблица маршрутизации в Нью-Йорке выглядит так:

![Таблица маршрутизации](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab4/Screenshot%20from%202025-01-23%2005-15-27.png)

Связь с сетью Санкт-Петербурга проходит: 

![Связь NYC-SPB](https://raw.githubusercontent.com/warmike01/2024_2025-introduction_in_routing-k3323-klopov-m-p/refs/heads/master/lab4/Screenshot%20from%202025-01-23%2005-17-11.png)
