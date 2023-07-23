---
layout: post
title: Лучший карманный роутер
---
# Заметка о настройке WireGuard на VPS
```
apt-get update && apt-get upgrade -y
```
Скачиваем скрипт для установки:
```
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
```
Выдаем права на выполнение:
```
chmod +x wireguard-install.sh
```
Запускаем скрипт и следуем настройкам
```
./wireguard-install.sh
```
Заходим на [github](https://github.com/ngoduykhanh/wireguard-ui/), скачиваем последний релиз, распаковываем и закидываем по sftp на сервер и выдаем права на исполнение
```
chmod +x wireguard-ui
```
Запускаем:
```
./wireguard-ui
```
Настраиваем подсеть, порт, прописываем PostUp и PostDown
```
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
```
Перезапускаем сервер