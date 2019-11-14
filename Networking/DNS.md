За разрешение имён на сервере (без NetworkManager) отвечает сервис systemd-resolved.

Конфиги DNS: /etc/resolv.conf, /etc/systemd/resolved.conf. За управление systemd-resolved отвечает утилита resolvectl (или resolvconf).

Для систем с NetworkManager можно использовать nmcli для просмотра текущего DNS-сервера:

$ nmcli dev show | grep DNS

При прямом редактировании файла /etc/resolv.conf данные не сохраняются, а перезаписываются NetworkManager или Systemd-Networkd.
<!-- Для разрешения доменных имён используется утилита resolvectl (или resolvconf). -->

Я смог настроить DNS следующим образом. Для утилиты netplan я создал yaml-конфиг для одного из интерфейсов и указал для него DNS-сервер:

nameservers:
  addresses: [8.8.8.8]

После этого выполнил:

$ sudo netplan try

Разрешение имён заработало!
