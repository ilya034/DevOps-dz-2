# Отчёт по домашней работе

## Проблема 1

### Что было обнаружено
* status 203/exec
* homework-app.service: Failed to locate executable /opt/linux-devops-homework/app.py: No such file or directory

### Как проводилась диагностика

```bash
sudo systemctl status homework-app
sudo journalctl -u homework-app
```

### В чём была причина

Неправильное название исполняемого файла (app.py, а не server.py)

### Что было изменено

```bash
sudo systemctl edit homework-app
```
Добавили переопределения:

``` 
[Service]
ExecStart=
ExecStart=/opt/linux-devops-homework/server.py
```

```bash
sudo systemctl restart homework-app
```

### Почему было выбрано это решение

Надо изменить название файла, проще всего переопределить ExecStart в самом сервисе

### Как проверялся результат

```bash
sudo systemctl status homework-app
sudo journalctl -u homework-app
```
Ошибка ~~пропала~~ изменилась

---

## Проблема 2

### Что было обнаружено
* systemd ExecStart is incorrect

### Как проводилась диагностика

```bash
./scripts/check.sh
```

### В чём была причина

Ожидается server.py в /opt/linux-devops-homework/*app*/server.py, а не /opt/linux-devops-homework/server.py

### Что было изменено

```bash
sudo mkdir /opt/linux-devops-homework/app
sudo mv /opt/linux-devops-homework/server.py /opt/linux-devops-homework/app
sudo systemctl edit homework-app
```

Добавили переопределения:
```
[Service]
ExecStart=
ExecStart=/opt/linux-devops-homework/*app*/server.py
```

```bash
sudo systemctl restart homework-app
```

### Почему было выбрано это решение

Надо изменить путь файла, проще всего переопределить ExecStart в сервисе + добавить переложить исполяемый файл в папку /opt/linux-devops-homework/app

### Как проверялся результат

```bash
./scripts/check.sh
```
Ошибка пропала

---

## Проблема 3

### Что было обнаружено
* PermissionError: [Errno 13] Permission denied: '/var/lib/linux-devops-homework/startup.log'

### Как проводилась диагностика

```bash
sudo systemctl status homework-app
sudo journalctl -u homework-app
sudo ls -lah /var/lib/linux-devops-homework
```

### В чём была причина

Скрипт не может открыть файл логов, т.к. нет доступа в папке /var/lib/linux-devops-homework, т.к. непраивльный владелец и права на linux-devops-homework/

### Что было изменено

```bash
sudo chown homework:homework /var/lib/linux-devops-homework
sudo chmod 750 /var/lib/linux-devops-homework
sudo sudo systemctl daemon-reload
sudo systemctl restart homework-app
```

### Почему было выбрано это решение

Чтобы скрипт мог создать файл в директории, ему нужны права на запись в директорию(*x*), 

ну и по заданию требуется homework:homework 750 на директории linux-devops-homework в /var/lib

### Как проверялся результат

```bash
sudo journalctl -u homework-app -fn 50
```
Ошибка пропала

---

## Проблема 4

### Что было обнаружено
*  TCP/8080 is reachable only through loopback

### Как проводилась диагностика

```bash
./scripts/check.sh
sudo cat /etc/linux-devops-homework/app.conf
```

### В чём была причина

Приложение доступно тольько через loopback адрес (127.0.0.1)

### Что было изменено

```bash
sudo vim /etc/linux-devops-homework/app.conf
```
Меняем 127.0.0.1 -> 0.0.0.0

### Почему было выбрано это решение

0.0.0.0 означает все доступные ipv4 интерфейсы

### Как проверялся результат

```bash
./scripts/check.sh
```
Ошибка пропала

---

## Итоговый ход диагностики

Проверили статус сервиса - не работает, т.к. не найден исполняемый файл (проблема 1)
После решил проверить check.sh - проблема 2 с неправильным ожидаемым путём файла
Потом проверяем статус сервиса - ошибка доступа к логам (Проблема 3)
После решил проверить check.sh - недоступность со всех интерфейсов (Проблема 4)

## Итоговая проверка

```bash
sudo ./scripts/check.sh
```

```text
Linux DevOps Homework Checker

[PASS] systemd ExecStart points to the application
[PASS] service runs as homework:homework
[PASS] state directory ownership and permissions are correct
[PASS] service is enabled
[PASS] service is active
[PASS] running process has the expected UID
[PASS] TCP/8080 listens on 0.0.0.0
[PASS] GET /health returns the expected response
[PASS] fixed systemd unit is saved in the repository
[PASS] fixed application configuration is saved in the repository

Result: 10 passed, 0 failed
```

