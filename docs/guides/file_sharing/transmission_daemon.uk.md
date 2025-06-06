---
title: Передача BitTorrent Seedbox
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.4
tags:
  - передача файлів
---

## Вступ

BitTorrent не потребує представлення, але якщо ви не знаєте, BitTorrent — це одноранговий протокол обміну файлами. BitTorrent покладається на те, що декілька однорангових пристроїв завантажують (завантажують) запитуваний файл до вас, але ви також повертаєтеся до майбутніх завантажувачів.

Transmission — це популярний клієнт BitTorrent із відкритим вихідним кодом із кількома інтерфейсами та серверами. Тут ви встановите бекенд безголового «демона».

У сучасному світі, орієнтованому на мобільні пристрої, доцільніше запускати Transmission як безголовий сервер, ніж безпосередньо на ноутбуці чи настільному комп’ютері. Таким чином, ви можете завантажувати файли 24 години на добу, 7 днів на тиждень, у той час як акумулятор мобільного пристрою не витрачається під час завантаження.

## Встановлення

Щоб встановити Transmission, спочатку потрібно встановити EPEL:

```bash
dnf install -y epel-release
```

Потім встановіть Transmission:

```bash
dnf install -y transmission-daemon
```

## Перше налаштування

На відміну від більшості демонов Linux, Transmission налаштовує конфігурацію під час першого запуску, тому запускайте та зупиняйте Transmission за допомогою:

```bash
systemctl start transmission-daemon
systemctl stop transmission-daemon
```

Після цих кроків у вас буде файл конфігурації. Найкраще було б зупинити Transmission, оскільки ви не можете редагувати файл конфігурації під час роботи.

## Конфігурація

Налаштування передачі:

```bash
cd /var/lib/transmission/.config/transmission-daemon
vi settings.json
```

Перейдіть до запису JSON `"peer-port"` і, якщо потрібно, замініть стандартний порт на потрібний порт:

```bash
    "peer-port": 51413,
```

Тут автор змінює його на `12345`:

```bash
    "peer-port": 12345,
```

Згодом перейдіть до запису JSON «rpc-password» і змініть пароль:

```bash
    "rpc-password": "{9cfaaade11d56c8e82bfc23b696fa373fb20c10e4U2NXY3.",
```

Введіть тут свій простий текстовий пароль. Якщо безпека викликає занепокоєння, зауважте, що Transmission зашифрує пароль під час наступного перезапуску.

Якщо ви хочете дозволити доступ з інших IP-адрес, перейдіть до запису «rpc-whitelist»:

```bash
    "rpc-whitelist": "127.0.0.1,::1",
```

Наприклад, якщо ви хочете дозволити доступ до свого робочого столу за IP-адресою `192.168.1.100`, ви можете додати його до значення, розділеного комами:

```bash
    "rpc-whitelist": "127.0.0.1,::1,192.168.1.100",
```

Якщо вам не потрібен білий список IP-адрес, ви можете вимкнути його, встановивши `"rpc-whitelist-enable"` на `false`:

```bash
    "rpc-whitelist-enabled": false,
```

Після завершення налаштування запустіть і ввімкніть Transmission:

```bash
systemctl enable --now transmission-daemon
```

## Конфігурація брандмауера та мережі

Згодом вам потрібно буде дозволити відповідні порти `12345` (для BitTorrent) і `9091` (для панелі керування Transmission) у нашому брандмауері:

```bash
firewall-cmd --permanent --zone=public --add-port=12345/tcp
firewall-cmd --permanent --zone=public --add-port=9091/tcp
firewall-cmd --runtime-to-permanent
```

Якщо ви не перебуваєте за маршрутизатором із підтримкою NAT-PMP або UPnP або підключені без NAT, вам потрібно перенаправити порт BitTorrent («12345» у нашому прикладі). Кожен роутер різний, але як приклад на авторському роутері MikroTik:

```bash
/ip firewall nat add action=dst-nat chain=dstnat dst-port=12345 in-interface=ether1 protocol=tcp to-addresses=SERVER_IP to-ports=12345
```

Замініть `SERVER_IP` на IP-адресу сервера, на якому запущено Transmission.

## Тестування трансмісії

Перейдіть до IP-адреси, на якій працює наш сервер передачі. Як приклад, ви можете завантажити торрент дистрибутива Linux, наприклад Ubuntu:

![Our Transmission downloading Ubuntu](../images/transmission.png)

## Висновок

BitTorrent був розроблений на початку 2000-х, коли більшість людей підключалися до Інтернету через настільний ПК. Хоча запускати BitTorrent на ноутбуці чи телефоні непрактично, запустити його на безголовому сервері через Transmission ідеально. Таким чином ви можете створювати файли 24/7, але наші завантаження завжди будуть доступними.
