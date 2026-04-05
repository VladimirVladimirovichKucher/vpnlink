# AnyLink

AnyLink - корпоративный SSL VPN сервер для организации удалённого доступа к внутренней сети. Поддерживает одновременную работу множества пользователей.

## Возможности

- TLS/TCP и DTLS/UDP туннели
- Совместимость с клиентами AnyConnect и OpenConnect
- Режимы работы: tun, tap, macvtap
- Поддержка Proxy Protocol v1 и v2
- Группы пользователей и групповые политики
- Многопользовательский режим с индивидуальными политиками
- Двухфакторная аутентификация (TOTP)
- Аутентификация через RADIUS и LDAP
- Автоматическая синхронизация пользователей LDAP
- Ограничение скорости трафика и сжатие трафика
- Панель администрирования через веб-интерфейс
- Управление правами доступа
- Аудит активности пользователей и IP-доступа
- Динамический раздельный туннель по доменным именам
- Автоотключение при простое
- Поддержка самоподписанных сертификатов
- Резолв внутренних доменов через внутренний DNS
- Защита от брутфорса (IP BAN)
- Работа Docker в непривилегированном режиме
- Динамический профиль Cisco (адрес сервера подставляется автоматически)

## Скриншоты

![Главная](doc/screenshot/home.jpg)

## Быстрый старт (Docker Compose)

Самый простой способ запустить AnyLink - через Docker Compose.

### 1. Клонируйте репозиторий
```shell
git clone https://github.com/VladimirVladimirovichKucher/vpnlink.git
cd vpnlink
```

### 2. Подготовьте конфигурацию

Создайте папку conf в корне проекта и скопируйте туда пример конфигурации:
```shell
mkdir -p conf
cp server/conf/server-sample.toml conf/server.toml
cp server/conf/profile.xml conf/profile.xml
```

### 3. Отредактируйте conf/server.toml

Откройте файл conf/server.toml и измените следующие параметры:

**Обязательные изменения:**

| Параметр | Значение по умолчанию | Что поменять |
|----------|----------------------|--------------|
| issuer | "XXX company VPN" | Название вашей компании, например "My Company VPN" |
| admin_pass | хэш от 123456 | Сгенерируйте новый: docker run --rm cherts/anylink tool -p ВашПароль |
| jwt_secret | abcdef.0123456789.abcdef | Сгенерируйте новый: docker run --rm cherts/anylink tool -s |
| cert_file | /etc/anylink/vpn_cert.pem | Путь к SSL сертификату: /app/conf/vpn_cert.pem |
| cert_key | /etc/anylink/vpn_cert.key | Путь к ключу: /app/conf/vpn_cert.key |
| db_source | /var/lib/anylink/anylink.db | Путь к базе: /app/conf/anylink.db |
| profile | /etc/anylink/profile.xml | Путь к профилю: /app/conf/profile.xml |
| files_path | /var/lib/anylink/files | Путь к файлам: /app/conf/files |

**Сетевые настройки (менять при необходимости):**

| Параметр | По умолчанию | Описание |
|----------|-------------|----------|
| ipv4_cidr | 192.168.90.0/24 | Подсеть для VPN клиентов |
| ipv4_gateway | 192.168.90.1 | Шлюз VPN подсети |
| ipv4_start | 192.168.90.100 | Начало пула IP адресов |
| ipv4_end | 192.168.90.200 | Конец пула IP адресов |
| ipv4_master | eth0 | Имя сетевой карты сервера |
| default_domain | example.com | Домен для DNS поиска клиентов |
| max_client | 200 | Максимум одновременных подключений |
| max_user_client | 3 | Максимум подключений на одного пользователя |

### 4. Добавьте SSL сертификат

Положите файлы сертификата в папку conf:
```shell
cp /path/to/your/cert.pem conf/vpn_cert.pem
cp /path/to/your/key.key conf/vpn_cert.key
```

Для тестирования можно использовать самоподписанный сертификат:
```shell
openssl req -x509 -newkey rsa:2048 -keyout conf/vpn_cert.key -out conf/vpn_cert.pem -days 365 -nodes -subj "/CN=vpn.example.com"
```

### 5. Запустите
```shell
docker-compose up -d
```

### 6. Проверьте работу

- Панель управления: https://IP-сервера:8800
- Логин: admin
- Пароль: 123456 (или тот что вы задали)
- VPN подключение: IP-сервера:443

### 7. Подключение клиента

Установите Cisco AnyConnect Secure Client и введите адрес вашего сервера (IP или домен). В профиле Cisco автоматически подставится правильный адрес сервера.

При использовании самоподписанного сертификата снимите галочку "Block connections to untrusted servers" в настройках клиента.

## Генерация паролей и ключей
```shell
# Генерация хэша пароля администратора
docker run --rm cherts/anylink tool -p ВашНовыйПароль

# Генерация JWT секрета
docker run --rm cherts/anylink tool -s

# Генерация OTP ключа (для двухфакторной аутентификации)
docker run --rm cherts/anylink tool -o

# Просмотр всех параметров конфигурации
docker run --rm cherts/anylink tool -d
```

## Конфигурация базы данных

| Тип БД   | Строка подключения |
|----------|--------------------|
| sqlite3  | /app/conf/anylink.db |
| mysql    | user:password@tcp(127.0.0.1:3306)/anylink?charset=utf8mb4 |
| postgres | postgres://user:password@localhost/anylink?sslmode=verify-full |

## Установка без Docker

### Зависимости

CentOS: yum install iptables iproute

Ubuntu/Debian: apt-get install iptables iproute2

### Сборка из исходников

Требуется Docker для компиляции.
```shell
git clone https://github.com/VladimirVladimirovichKucher/vpnlink.git
cd vpnlink
bash build_web.sh
bash build.sh
cd anylink-deploy
sudo ./anylink
```

### Настройка сети (режим tun)
```shell
# Включить форвардинг
sysctl -w net.ipv4.ip_forward=1

# Для постоянного сохранения добавьте в /etc/sysctl.conf:
# net.ipv4.ip_forward = 1
```

NAT настраивается автоматически. При необходимости вручную:
```shell
# Замените eth0 на имя вашей сетевой карты
iptables -t nat -A POSTROUTING -s 192.168.90.0/24 -o eth0 -j MASQUERADE
```

### Установка через Systemd

1. Скопируйте бинарник: cp anylink-deploy/anylink /usr/sbin/anylink && chmod +x /usr/sbin/anylink
2. Создайте директорию конфигурации: mkdir -p /etc/anylink
3. Скопируйте конфиги: cp anylink-deploy/conf/* /etc/anylink/
4. Создайте рабочую директорию: mkdir -p /var/lib/anylink/files
5. Скопируйте файлы: cp anylink-deploy/conf/files/* /var/lib/anylink/files/
6. Установите systemd сервис:
```shell
# Ubuntu/Debian
cp deploy/anylink.service /lib/systemd/system/
# CentOS
cp deploy/anylink.service /usr/lib/systemd/system/

systemctl daemon-reload
systemctl start anylink
systemctl enable anylink
```

## Docker (ручной запуск)
```bash
# Запуск с монтированием конфигурации
docker run -itd --name anylink --privileged \
    -p 443:443 -p 8800:8800 -p 443:443/udp \
    -v /home/myconf:/app/conf \
    --restart=always \
    cherts/anylink

# Непривилегированный режим
docker run -itd --name anylink \
    -p 443:443 -p 8800:8800 -p 443:443/udp \
    -v /dev/net/tun:/dev/net/tun --cap-add=NET_ADMIN \
    --restart=always \
    cherts/anylink
```

## Kubernetes
```bash
cd deploy
kubectl apply -f deployment.yaml
```

## Обновление

1. Сделайте резервную копию директории conf и базы данных
2. Остановите сервис: docker-compose down (или systemctl stop anylink)
3. Обновите образ: docker-compose pull (или замените бинарник)
4. Запустите: docker-compose up -d (или systemctl start anylink)

## Порты

| Порт | Протокол | Назначение |
|------|----------|------------|
| 443 | TCP | VPN подключение (TLS) |
| 443 | UDP | VPN подключение (DTLS) |
| 8800 | TCP | Панель администрирования |

## Поддерживаемые клиенты

- AnyConnect Secure Client (Windows/macOS/Linux/Android/iOS)
- OpenConnect (Windows/macOS/Linux)

## Поддерживаемые ОС (сервер)

CentOS 7/8, Ubuntu 18/20/22/24/26, Debian 10/11/12/13

## Дополнительные скриншоты

![system.jpg](doc/screenshot/system.jpg)
![setting.jpg](doc/screenshot/setting.jpg)
![other.jpg](doc/screenshot/other.jpg)
![audit.jpg](doc/screenshot/audit.jpg)
![users.jpg](doc/screenshot/users.jpg)
![ip_map.jpg](doc/screenshot/ip_map.jpg)
![group.jpg](doc/screenshot/group.jpg)

## Лицензия

Проект распространяется под лицензией AGPL-3.0. Полный текст лицензии в файле LICENSE.