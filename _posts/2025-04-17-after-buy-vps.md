---
layout: post
title: Базовая настройка VPS
categories: vps
---
Небольшая заметка-шпаргалка о минимальных действиях после покупки VPS.

1. Подключаемся
```
ssh root@<ip>
```
2. Обновляем пакеты и ставим sudo
```
apt update && apt upgarde && apt install sudo
```
3. Меняем пароль
```
passwd
```
4. Создаем нового пользователя
```
adduser <username>
```
5. Добавляем его в sudo
```
/usr/sbin/usermod -aG sudo <username>
```
6. Выходим
```
exit
```
7. Копируем SSH-ключ для нового пользователя
```
ssh-copy-id <username>@<ip>
```
8. Подключаемся
```
ssh <username>@<ip>
```
9. Редактируем настройки SSH-сервера
```
sudo nano /etc/ssh/sshd_config
Снимаем коментарии (#) и меняем дефолтный порт, запрещаем доступ по паролю и доступ от root
Port <port>
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
Проверяем наличе файлов в /etc/ssh/sshd_config.d/ иногда, там сюрприз от хостера
```
10. Перезапускаем SSH
```
sudo systemctl retsart sshd
```
11. Устанавливаем фаервол
```
sudo apt install ufw -y
```
12. Разрешаем нужные порты
```
sudo ufw allow <port>
```
13. Включаем фаервол
```
sudo ufw enable
```
14. Проверяем статус
```
sudo ufw status
```