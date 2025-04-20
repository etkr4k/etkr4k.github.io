---
layout: post
title: Настройка Cloud-Init в Proxmox
categories: pve
---
Домашняя лаборатория предполагает частое разворачивание новых виртуальных машин и если бы в мире был чемпионат speedrun по установки Debian - я однозначно бы победил. В общем, надоело мне пользователей создавать, sudo ставить, dvd из source.list убирать и прочие радости голой системы. Была мысль сделать свой образ, но прибивать гвоздями некоторые вещи не хотелось. С pve я знаком давно, но многие вещи до сих пор не знаю. Пришло время разобраться с Cloud-init.

#### План действий такой:
1. Создать виртуалку
2. Поднастроить ее
3. Сделать ее шаблоном
4. Клонировать n-количество раз применяя нужные настройки

Создаем виртуалку
```
qm create 9001 --name debian-cloud --memory 2048 --net0 virtio,bridge=vmbr0
```
Добавляем обычный диск и ISO
```
qm set 9001 --scsi0 local-lvm:disk-0
qm set 9001 --cdrom local:iso/<iso.img>
```
Запускаем ВМ
```
qm start 9001
```
Дальше устанавливаем систему. Разрешаем SSH, систему не шифруем.
После загрузки устанавливаем cloud-init (команда выполняется на ВМ)
```
apt update
apt install -y cloud-init
```
Выключаем виртуальную машину и добавляем диск cloud-init
```
qm set 9001 --ide2 local-lvm:cloudinit
```
Указываем основной диск как загрузочный
```
qm set 9001 --boot c --bootdisk scsi0
```

#### Дальше у нас есть два варианта:
Первый, использовать GUI настройки cloud-init (мало настроек)
Второй, использовать yaml сниппет

#### В первом случай переходим в GUI 
Во вкладке ВМ в раздел Cloud-Init и настраиваем нужные параметры. Жмем "Regenerate Image"
Далее очищаем cloud-init и выключаем ВМ(команда выполняется на ВМ)
```
cloud-init clean
sudo poweroff
```
Правой кнопкой мыши кликаем по нашей ВМ и выбираем Convert to Template.
Теперь наша ВМ стала шаблоном.
Еще раз кликаем правой кнопкой по ВМ и выбираем "Clone"
Указываем имя и VM ID
Выбираем тип клонирования:
- Linked Clone — быстрее и экономит место.
- Full Clone — копия шаблона (больше, но независимая).
Нажимаем "Clone" и запускаем. 

#### Во втором случае мы создаем Snippet.
Первым делом проверяем что snippet включен в настройках хранилища.
Datacenter → Storage → local, в поле Content должна быть галочка напротив Snippets.
Далее создаем шаблон
```
nano /var/lib/vz/snippets/user-data.yaml
```
В моем случае примерная конфигурация выглядит так:
```
#cloud-config
hostname: testvm
fqdn: testvm.local

users:
  - name: username
    groups: sudo
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh-authorized-keys:
      - ssh-rsa AAAAB3...

packages:
  - htop
  - curl
  - git

runcmd:
  - echo "Hello from Cloud-Init" > /home/devuser/hello.txt
  - systemctl restart ssh


final_message: "Cloud-Init setup complite!"
```
После создания сниппета, клонируем ВМ и прицепляем к ней сниппет
```
qm set 9002 --cicustom "user=local:snippets/user-data.yaml"
```
Остается зайти во вкладку cloud-init и нажать "Regenerate Image"
Запускаем и подключаемся

> Важный момент: если вы используете .yaml файл для настройки - оставьте значения cloud-init в gui на дефолтных значениях, иначе настройки будут применяться сразу из двух мест.