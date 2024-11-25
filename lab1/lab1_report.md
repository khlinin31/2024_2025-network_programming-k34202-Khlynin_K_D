**Лабораторная работа №1 "Установка CHR и Ansible, настройка VPN"**

**University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2024/2025  
Group: 34202  
Author: Хлынин Кирилл Дмитриевич: Lab1  
Date of create: 06.11.2024  
Date of finished: 06.11.2024**  

**Описание**

Данная работа предусматривает обучение развертыванию виртуальных машин (VM) и системы контроля конфигураций Ansible а также организации собственных VPN серверов.

**Цель работы**

Целью данной работы является развертывание виртуальной машины на базе платформы Microsoft Azure с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox

**Ход работы**

**По гайду от [Mikrotik](https://wiki.mikrotik.com/wiki/Manual:CHR_VirtualBox_installation) (<https://wiki.mikrotik.com/Wiki/Manual:CHR_VirtualBox_installation>) была установлена виртуальная машина на RouterOS в VirtualBox:**

![image](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.001.png)

Далее, был куплен сервер на хостинге VDSina для выполнения второй части работы:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.002.png)

На систему сервера был установлен python3 и Ansible:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.003.png)

Далее, был установлен Wireguard, созданы публичный и приватный ключ, а также файл конфигурации wg0.conf:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.004.png)

После чего интерфейс был поднят с помощью команды wg-quick up и systemctl start <wg-quick@wg0.service>:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.005.png)

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.006.png)

Далее, со стороны клиента в WinBox был настроен интерфейс Wireguard, в котором был указан приватный ключ и порт доступа от интерфейса:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.007.png)


![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.008.png)

**Проверка работоспособности**

- Пинг с клиента на сервер:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.009.png)

- Доступ в интернет на клиенте:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.010.png)

- Пинг с сервера на клиент:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.011.png)

- Доступ в интернет на сервере:

![](Aspose.Words.3b1fac52-b9df-4a6e-b9cb-297ff4c901f5.012.png)

**Вывод**

В результате работы была развернута виртуальная машина для роутера Mikrotik, которая соединяется Wireguard VPN туннелем с удаленным сервером в облаке.
