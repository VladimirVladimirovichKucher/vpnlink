# VPNLink

VPNLink - корпоративный SSL VPN сервер для организации удалённого доступа к внутренней сети. Поддерживает одновременную работу множества пользователей.

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

## Установка

### Готовые сборки

Скачайте файл anylink-deploy.tar.gz из раздела Releases.

### Сборка из исходников

Требуется Docker.
```shell
git clone https://github.com/VladimirVladimirovichKucher/vpnlink.git
cd vpnlink
bash build_web.sh
bash build.sh
cd anylink-deploy
sudo ./anylink
```

Панель управления: https://host:8800 (логин: admin, пароль: 123456)

## Конфигурация
```shell
./anylink -h
./anylink tool -p 123456
./anylink tool -s
./anylink tool -d
```

### Базы данных

| Тип БД   | Строка подключения |
|----------|--------------------|
| sqlite3  | ./conf/anylink.db |
| mysql    | user:password@tcp(127.0.0.1:3306)/anylink?charset=utf8mb4 |
| postgres | postgres://user:password@localhost/anylink?sslmode=verify-full |

Пример конфигурации: server/conf/server-sample.toml

## Настройка сети

### Зависимости

CentOS: yum install iptables iproute

Ubuntu/Debian: apt-get install iptables iproute2

### Режим tun (рекомендуется)

1. Включить форвардинг: sysctl -w net.ipv4.ip_forward=1
2. NAT настраивается автоматически
3. Подключайтесь с помощью клиента AnyConnect

## Docker
```bash
docker run -itd --name vpnlink --privileged -p 443:443 -p 8800:8800 -p 443:443/udp --restart=always cherts/anylink
```

С пользовательской конфигурацией:
```bash
docker run -itd --name vpnlink --privileged -p 443:443 -p 8800:8800 -p 443:443/udp -v /home/myconf:/app/conf --restart=always cherts/anylink
```

### Docker Compose
```bash
cd deploy
docker-compose up
```

### Kubernetes
```bash
cd deploy
kubectl apply -f deployment.yaml
```

## Обновление

1. Сделайте резервную копию директории conf и базы данных
2. Остановите сервис
3. Замените бинарник anylink на новую версию
4. Запустите сервис

## Поддерживаемые клиенты

- AnyConnect Secure Client (Windows/macOS/Linux/Android/iOS)
- OpenConnect (Windows/macOS/Linux)

## Лицензия

Проект распространяется под лицензией AGPL-3.0. Полный текст лицензии в файле LICENSE.