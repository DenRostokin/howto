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

##############################################
Ресурсы:

1. https://www.tecmint.com/install-dhcp-server-client-on-centos-ubuntu/
