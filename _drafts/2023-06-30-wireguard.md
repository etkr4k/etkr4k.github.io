---
layout: post
title: Настройка сервера Wireguard
---
Обновляемся
`apt-get update && apt-get upgrade -y`
Устанавливаем curl
`apt install curl`
Скачиваем скрипт для установки Wireguard
`curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh`
Делаем его исполняемым
`chmod +x wireguard-install.sh`
Запускаем
`./wireguard-install.sh`