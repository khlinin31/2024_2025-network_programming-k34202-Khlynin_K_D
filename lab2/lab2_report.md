University: [ITMO University](https://itmo.ru/ru/)

Faculty: [FICT](https://fict.itmo.ru)

Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)

Year: 2024/2025

Group: K34202

Author: Хлынин Кирилл Дмитриевич

Lab: Lab2

Date of create: 18.11.2024

Date of finished: 18.11.2024

# Лабораторная работа №2 "Развертывание дополнительного CHR, первый сценарий Ansible"

# Описание
В данной лабораторной работе вы на практике ознакомитесь с системой управления конфигурацией Ansible, использующаяся для автоматизации настройки и развертывания программного обеспечения.

## Цель работы
С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

## Ход выполнения работы

1. В VirtualBox был скопирован первый CHR. На новом роутере был выбран также сетевой мост

![1](./assets/1.jpg)

2. ---

![2](./assets/2.jpg)

3. На сервере в конфиг wierguard была добавлена информация о новом клиенте с адресом 10.66.66.3. Результат проверки соединения vpn сервера со вторым CHR:

![3](./assets/3.jpg)

4. В файле [inventory.yaml](./assets/inventory.yaml) была указана информация об узлах, на которых будет производена настройка

```yaml
all:
  hosts:
    chr1:
      ansible_host: 10.66.66.2
      ansible_user: admin
      ansible_password: admin
      ansible_connection: ansible.netcommon.network_cli
      ansible_network_os: community.routeros.routeros
      ansible_ssh_private_key_file: ~/.ssh/id_rsa
    chr2:
      ansible_host: 10.66.66.3
      ansible_user: admin
      ansible_password: admin
      ansible_connection: ansible.netcommon.network_cli
      ansible_network_os: community.routeros.routeros
      ansible_ssh_private_key_file: ~/.ssh/id_rsa

````

5. Для настройки устройств был создан Ansible плейбук [playbook.yaml](./assets/playbook.yaml), который меняет пароль для админа, настраивает NTP и OSPF и выводит итоговый конфиг устройства:

```yaml
---
    - name: CHR configuration
      hosts: all
      become: yes
      tasks:
        - name: Change user password
          community.routeros.command:
            commands:
              - /user set admin password={{ansible_password}}
    
        - name: Configure NTP
          community.routeros.command:
            commands:
              - /system ntp client set enabled=yes mode=unicast
              - /system ntp client servers add address=194.190.168.1
    
        - name: Configure OSPF
          community.routeros.command:
            commands:
              - /interface bridge add name=loopback
              - /ip address add address={{ router_id }} interface=loopback
              - /routing ospf area add instance=default name=backbone
              - /routing ospf instance add disabled=no name=default
              - /routing ospf instance set default router-id={{ router_id }}
              - /routing ospf interface-template add area=backbone interfaces=ether1 type=ptp
          vars:
            router_id: "{% if ansible_host == '10.66.66.2' %}1.1.1.1{% elif ansible_host == '10.66.66.3' %}2.2.2.2{% else %}3.3.3.3{% endif %}"
    
        - name: Get OSPF topology details
          community.routeros.command:
            commands:
              - /routing ospf neighbor print detail
          register: ospf_topology
    
        - name: Print OSPF topology
          debug:
            var: ospf_topology.stdout_lines
    
        - name: Get full config
          community.routeros.command:
            commands:
              - /export
          register: full_config
    
        - name: Print full config
          debug:
            var: full_config.stdout_lines
```


6. В результате выполнения плейбука были получены данные по OSPF топологии:

```bash
TASK [Print OSPF topology] ******************************************************************************************************************************************************************
ok: [chr1] => {
    "ospf_topology.stdout_lines": [
        [
            "Flags: V - virtual; D - dynamic ",
            " 0  D instance=default area=backbone address=172.20.10.2 router-id=2.2.2.2 ",
            "      state=\"Full\" state-changes=6 adjacency=2m27s timeout=33s"
        ]
    ]
}
ok: [chr2] => {
    "ospf_topology.stdout_lines": [
        [
            "Flags: V - virtual; D - dynamic ",
            " 0  D instance=default area=backbone address=172.20.10.5 router-id=1.1.1.1 ",
            "      state=\"Full\" state-changes=5 adjacency=2m27s timeout=33s"
        ]
    ]
}
```

7. Также полный конфиг устройств:

```bash
TASK [Print full config] ********************************************************************************************************************************************************************
ok: [chr1] => {
    "full_config.stdout_lines": [
        [
            "# 2024-10-18 17:40:47 by RouterOS 7.15.3",
            "# software id = ",
            "#",
            "/disk",
            "set slot1 media-interface=none media-sharing=no slot=slot1",
            "set slot2 media-interface=none media-sharing=no slot=slot2",
            "set slot3 media-interface=none media-sharing=no slot=slot3",
            "set slot4 media-interface=none media-sharing=no slot=slot4",
            "set slot5 media-interface=none media-sharing=no slot=slot5",
            "set slot6 media-interface=none media-sharing=no slot=slot6",
            "set slot7 media-interface=none media-sharing=no slot=slot7",
            "set slot8 media-interface=none media-sharing=no slot=slot8",
            "set slot9 media-interface=none media-sharing=no slot=slot9",
            "/interface bridge",
            "add name=loopback",
            "/interface wireguard",
            "add listen-port=49734 mtu=1420 name=wg-client",
            "/routing ospf instance",
            "add disabled=no name=default router-id=1.1.1.1",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.66.66.0/24 endpoint-address=172.20.10.8 endpoint-port=\\",
            "    49734 interface=wg-client name=peer1 public-key=\\",
            "    \"	VhKOpIk0T71Ofa00ftlcwjWpat9oDscLBvBkxFgIi4=\"",
            "/ip address",
            "add address=10.66.66.2/24 interface=wg-client network=10.66.66.0",
            "add address=1.1.1.1 interface=loopback network=1.1.1.1",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip firewall filter",
            "add action=accept chain=input dst-port=49734 protocol=udp src-address=\\",
            "    172.20.10.5",
            "/ip route",
            "add disabled=no distance=1 dst-address=10.66.66.1/32 gateway=wg-client \\",
            "    routing-table=main scope=30 suppress-hw-offload=no target-scope=10",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=194.190.168.1"
        ]
    ]
}
ok: [chr2] => {
    "full_config.stdout_lines": [
        [
            "# 2024-10-18 17:40:47 by RouterOS 7.16.1",
            "# software id = ",
            "#",
            "/interface bridge",
            "add name=loopback",
            "/interface wireguard",
            "add listen-port=49734 mtu=1420 name=wg-client",
            "/routing ospf instance",
            "add disabled=no name=default router-id=2.2.2.2",
            "/routing ospf area",
            "add disabled=no instance=default name=backbone",
            "/interface wireguard peers",
            "add allowed-address=10.66.66.0/24 endpoint-address=172.20.10.8 endpoint-port=\\",
            "    49734 interface=wg-client name=peer1 public-key=\\",
            "    \"	VhKOpIk0T71Ofa00ftlcwjWpat9oDscLBvBkxFgIi4=\"",
            "/ip address",
            "add address=10.66.66.3/24 interface=wg-client network=10.66.66.0",
            "add address=2.2.2.2 interface=loopback network=2.2.2.2",
            "/ip dhcp-client",
            "add interface=ether1",
            "/ip route",
            "add disabled=no dst-address=10.66.66.1/32 gateway=wg-client routing-table=\\",
            "    main suppress-hw-offload=no",
            "/routing ospf interface-template",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "add area=backbone disabled=no interfaces=ether1 type=ptp",
            "/system note",
            "set show-at-login=no",
            "/system ntp client",
            "set enabled=yes",
            "/system ntp client servers",
            "add address=194.190.168.1"
        ]
    ]
}

```
## Схема связи

![2](./assets/7.jpg)

