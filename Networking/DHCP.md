Установка DHCP-сервера на Ubuntu:

$ sudo apt-get install isc-dhcp-server

Чтобы сконфигурировать интерфейсы, на которых DHCP-сервер дожен обрабатывать запросы, нужно редактировать файл:

/etc/default/isc-dhcp-server

Добавим в него следующее:

INTERFACESv4: "enp0s8"

Основным файлом конфигурации DHCP-сервера является:

/etc/dhcp/dhcp.conf

Добавим в него следующее:

subnet 192.168.1.0 netmask 255.255.255.0 {
        option routers                  192.168.1.1;
        option subnet-mask              255.255.255.0;
        option domain-search            "tecmint.lan";
        option domain-name-servers      8.8.8.8;
        range   192.168.1.2   192.168.1.254;
}

Теперь нужно запустить и разрешить сервис isc-dhcp-server:

$ sudo systemctl start isc-dhcp-server
$ sudo systemctl enable isc-dhcp-server

Настройка dhcp-сервера завершена!

Для того, чтобы на клиенте запросить IP-адрес у DHCP-сервера, к сети которого подключёт клиент, например, интерфейсом enp0s3, нужно использовать утилиту dhclient, которая может быть установлена из официальных репозиториев:

$ sudo dhclient enp0s3

После выполнения этой комманды, интерфейсу enp0s3 будет присвоен адрес в сети, к которой он подлючён. Проверить это можно коммандой:

$ ip -br a

##############################################
Ресурсы:

1. https://www.tecmint.com/install-dhcp-server-client-on-centos-ubuntu/
