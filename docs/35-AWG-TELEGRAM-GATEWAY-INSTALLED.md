# AWG Telegram Gateway Installed

Дата подтверждения: 2026-07-22

## Назначение

На VPS `185.237.186.95` установлен отдельный входящий AmneziaWG-интерфейс для клиентских устройств.

Схема:

```text
устройство
→ AmneziaWG UDP 443
→ awg-client (10.77.0.1/24)
→ nftables forwarding/NAT
→ awg-egress
→ Telegram
```

## Подтверждённое состояние

- `awg-client.service`: active, enabled
- `awg-client-firewall.service`: active, enabled
- `awg-egress`: active, enabled
- Teleproxy: active, autostart disabled
- nginx: active
- `net.ipv4.ip_forward = 1`
- входящий интерфейс: `awg-client`
- адрес сервера внутри туннеля: `10.77.0.1/24`
- адрес первого клиента: `10.77.0.2/32`
- входящий порт: UDP `443`
- Teleproxy продолжает слушать TCP `443`
- UDP `443` и TCP `443` работают одновременно
- перезагрузка VPS не выполнялась

## Маршрутизация клиента

Через входящий VPN разрешены только IPv4-сети Telegram:

```text
91.105.192.0/23
91.108.4.0/22
91.108.56.0/22
149.154.160.0/20
```

Остальной интернет-трафик клиентского устройства через этот VPN не направляется.

Маршруты Telegram на сервере подтверждённо выходят через `awg-egress`.

## Firewall и NAT

Созданы отдельные nftables-таблицы:

```text
inet awg_client_filter
ip awg_client_nat
```

Правила:

- разрешают входящий UDP `443` на `eth0`;
- разрешают forwarding из `awg-client` в `awg-egress` только к сетям Telegram;
- разрешают обратный established/related трафик;
- блокируют остальной forwarding из `awg-client`;
- выполняют MASQUERADE в `awg-egress` только для сетей Telegram.

## Созданные файлы

```text
/etc/amnezia/amneziawg/awg-client.conf
/root/prx-tg-awg.conf
/root/prx-tg-awg-info.txt
/etc/sysctl.d/99-awg-client-forwarding.conf
/etc/nftables-awg-client.nft
/usr/local/sbin/awg-client-firewall
/etc/systemd/system/awg-client-firewall.service
/etc/systemd/system/awg-client.service
/root/awg_client_install_v3.log
/root/awg_client_install_v3_report.txt
```

Все приватные конфигурации имеют права `0600`.

## Клиентская конфигурация

Файл для первого устройства:

```text
/root/prx-tg-awg.conf
```

Его нужно скачать по SFTP и импортировать в нативное приложение AmneziaWG.

Один клиентский файл использовать только на одном устройстве. Для каждого следующего устройства необходимо создавать отдельную пару ключей и отдельный peer.

## Проверенный результат установки

```text
awg-client service: active
awg-client autostart: enabled
firewall service: active
firewall autostart: enabled
IPv4 forwarding: 1
UDP 443: listening
TCP 443: Teleproxy listening
Telegram-only forwarding and NAT: VERIFIED
```

## Резервная копия

```text
/root/awg-client-v3-backup-20260722T153621Z
```

Секреты, приватные ключи и содержимое клиентской конфигурации в GitHub не сохранялись.
