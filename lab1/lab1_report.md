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

Затем с помощью YAML-файла была задана топология сети:

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

Во-первых, чтобы подключаться к микротикам по ssh, в файл */home/warmike01/.ssh/config* нужно было добавить

```
Host clab-*
        User root
        StrictHostKeyChecking no
        UserKnownHostsFile /dev/null 
```
Чтобы при перезапуске контейнеров не приходилось зачищать known_hosts, туда же добавлено

```
Host 192.168.*.*       
        StrictHostKeyChecking no
        UserKnownHostsFile /dev/null 
```

Теперь в контейнеры можно заходить по ssh, напримнер в роутер заходим командой *ssh admin@192.168.2.254* и вводим пароль admin.

![Screenshot from 2024-10-23 23-35-08](https://github.com/user-attachments/assets/0932b09c-3610-48a3-a73f-735ee94fed96)

Оттуда можно вводить команды, которые в итоге с помощью команды */export* были перенесены в конфиги, чтобы при перезапуске контейнера не вводить их вручную:

Файл *configs/router_config.cfg*:
```
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
add interface=ether2 name=vlan20 vlan-id=20
/ip pool
add name=vlan10_pool ranges=10.10.10.0-10.10.10.254
add name=vlan20_pool ranges=10.10.20.0-10.10.20.254
/ip dhcp-server
add address-pool=vlan10_pool disabled=no interface=vlan10 name=dhcp-vlan10
add address-pool=vlan20_pool disabled=no interface=vlan20 name=dhcp-vlan20
/ip address
add address=10.10.10.129/25 interface=vlan10 network=10.10.10.128
add address=10.10.20.129/25 interface=vlan20 network=10.10.20.128
add address=10.10.2.2/19 interface=ether2 network=10.10.0.0
/ip dhcp-server network
add address=10.10.10.128/25 gateway=10.10.10.129
add address=10.10.20.128/25 gateway=10.10.20.129
```

Краткое объяснение: создаём две VLAN, выдаём им пул IP-адресов и настраиваем на него DHCP-сервер, выдаём серверу IP-адреса в каждой из этих VLANов.

Файл *configs/switch1_config.cfg*:
```
/interface bridge
add name=bridge10
add name=bridge20
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
add interface=ether3 name=vlan11 vlan-id=10
add interface=ether2 name=vlan20 vlan-id=20
add interface=ether4 name=vlan21 vlan-id=20
/interface bridge port
add bridge=bridge10 interface=vlan10
add bridge=bridge10 interface=vlan11
add bridge=bridge20 interface=vlan21
add bridge=bridge20 interface=vlan20
/ip dhcp-client
add disabled=no interface=bridge10
add disabled=no interface=bridge20
```

Краткое объяснение: создаём два моста, добавляем на них VLANы (в каждой из которых два интерфейса - на роутер и на соответствующий коммутатор) и настраиваем на них DHCP-клиент.


Файл *configs/switch2_config.cfg*:
```
/interface bridge
add name=bridge10
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
/interface bridge port
add bridge=bridge10 interface=vlan10
add bridge=bridge10 interface=ether3
/ip dhcp-client
add disabled=no interface=bridge10
```

Файл *configs/switch3_config.cfg*:
```
/interface bridge
add name=bridge20
/interface vlan
add interface=ether2 name=vlan20 vlan-id=20
/interface bridge port
add bridge=bridge20 interface=vlan20
add bridge=bridge20 interface=ether3
/ip dhcp-client
add disabled=no interface=bridge20
```

Краткое объяснение: соединяем мостом соответствующую VLAN с портом, подключённым к ПК, настраиваем на него DHCP-клиент.

На интерфейсы коммутаторов уже выдаются IP-адреса, а для их выдачи на ПК необходимо установить DHCP-клиент. Для этого подключаемся к ПК командой *docker exec -it clab-lab_1-PC1 bash* и вводим:

```
apt update
apt install udhcpc
udhcpc -i eth1
```
IP-адрес получен:

![Screenshot from 2024-10-23 23-52-14](https://github.com/user-attachments/assets/b9843522-206f-4799-9d6d-afdff98516ac)

То же самое проделываем и со вторым ПК. Результат можем подтвердить через таблицу DHCP-сервера роутера командой */ip dhcp-server lease print*:

![Screenshot from 2024-10-23 23-54-02](https://github.com/user-attachments/assets/b8ffc58f-5fc9-44f0-bdbc-cd11a0a0e47a)

*ping* тоже отлично проходит:

![Screenshot from 2024-10-23 23-56-51](https://github.com/user-attachments/assets/13ee1da6-980d-4580-a948-5e1914398b49)


Готово! Лабораторная работа выполнена.
