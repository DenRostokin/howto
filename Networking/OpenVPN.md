Установка и настройка OpenVPN-сервера. После обновления всех пакетов установим следующие:

\# apt-get install openvpn easy-rsa

CLI-утилита easy-rsa необходима для простой генерации RSA-ключей. Эта утилита разработана самими OpenVPN-разработчиками.

Переходим в /usr/share/doc/openvpn/examples/sample-config-files. Теперь распакуем server.conf.gz в /etc/openvpn/server.conf:

\# sudo gunzip -c server.conf.gz > /etc/openvpn/server.conf

Переходим в /etc/openvpn и отрываем для редактирования файл server.conf. Первое, что нужно сделать, - это обновить параметры Диффи-Хэллмана, если по умолчанию используется значение 1024. Нужно установить 2048, максимально возможное значение. Строка с данной настройкой начинается с dh. *В моём случае сразу было установлено значение 2048.

Далее необходимо раскомментировать строку redirect-gateway, чтобы весь трафик клиента, включая DNS-запросы, шёл через настраиваемый VPN. Затем необходимо указать адреса DNS-серверов, на которые будут делаться запросы от VPN-сервера. *Блок с данными настройками находится под блоком с redirect-gateway. Расскомментируем обе строки.

Находим блок с nobody, чтобы отнять root-права у фонового процесса OpenVPN после инициализации. Расскомментируем обе строки. Сохраняем файл server.conf и переходим к настройке Firewall'а.

Во-первых, разрешаем передачу трафика. Чтобы проверить, разрешена ли передача трафика, откроем файл:

\# cat /proc/sys/net/ipv4/ip_forward

Если выводится 0, то передача трафика запрещена. Для её разрешения выполним комманду:

\# echo 1 > /proc/sys/net/ipv4/ip_forward

Но данная настройка не сохранится после перезагрузки сервера. Для сохранения этой настройки нужно отредактировать файл /etc/sysctl.conf. Нужно раскомментировать строку net.ipv4.ip_forward=1. Сохраняем, выходим.

Теперь необходимо настроить Firewall. Будем работать не с iptables, т.к. он очень сложен, а заиспользуем утилиту ufw (uncomplicated firewall). Для начала нужно её установить.

\# apt-get install ufw

Прежде чем включать firewall, необходимо разрешить подключение по ssh, иначе наше соединение отвалится.

\# ufw allow ssh

Затем разрешим подключение к дефолтному порту openvpn по udp, т.к. этот протокол был указан в server.conf.

\# ufw allow 1194/udp

Открываем файл с конфигурациями ufw: /etc/default/ufw. Меняем DEFAULT_FORWARD_POLICY с DROP на ACCEPT. Сохраняем файл и выходим.

Теперь нам нужно разрешить NAT. Открываем файл /etc/ufw/before.rules. Добавим в этот файл правила для разрешения NAT:

*nat
:POSTROUTING ACCEPT [0.0]
-A POSTROUTING -s <VPN_SUBNET_ADDRESS> -o <VPN_SERVER_INTERFASE_NAME> -j MASQUERADE
COMMIT

VPN_SUBNET_ADDRESS можно посмотреть в /etc/openvpn/server.conf. В моём случае - это 10.8.0.0/24. Например, можно указать так: -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

Сохраняем, выходим. После этого выполняем комманду:

\# ufw enable

На этом настройка firewall'а завершена. Переходим к сертификатам PKI (Public Key Infrastructure)!

Переходим в директорию /usr/share/easy-rsa/. Скопируем эту директорию в /etc/openvpn:

\# cp -r /usr/share/easy-rsa/ /etc/openvpn

В директории /etc/openvpn/easy-rsa/ есть файл vars.example. Копируем его в /etc/openvpn/easy-rsa/vars. Открываем его для редактирования. Раскомментируем и меняем поля EASYRSA_REQ_COUNTRY, ...CITY, и т.д. Сохраняем и выходим. Теперь можно генерировать ключи!

Переходим в /etc/openvpn/easy-rsa/. Сначала запускаем комманду:

\# ./easyrsa init-pki

Затем запускаем генерацию сертификата CA:

\# ./easyrsa build-ca

Будет предложено ввести passphrase. Нужно ввести надёжный пароль и сохранить его у себя для дальнейшей работы с центром сертификации. После этого будет создан сертификат.

Теперь нужно сформировать файл параметров Диффи-Хеллмана dh.pem, он также будет расположен в директории pki:

\# ./easyrsa gen-dh

На этом создание центра сертификации (CA) можно считать законченным.

Создание ключа и сертификата для сервера. Сначала нужно создать запрос на сертификат:

\# ./easyrsa gen-req vpn-server nopass

Для выпуска сертификата нужно выполнить:

\# ./easyrsa sign-req server vpn-server

Опция server обозначает выпуск сертификата для сервера. Также потребуется ввести пароль закрытого ключа центра сертификации.

Для создания ta ключа используем команду:

\# openvpn --genkey --secret pki/ta.key

Теперь скопируем необходимые сертификаты и ключи в конфигурационную директорию openvpn:

\# cp pki/private/vpn-server.key pki/issued/vpn-server.crt pki/ca.crt pki/dh.pem pki/ta.key /etc/openvpn

В завершение нужно указать данные ключи в файле /etc/openvpn/server.conf. Там всё просто: ищем нужную строку и указываем имя соответствующего файла. Закомментируем строку tls-auth ta.key, т.к. это не будет использоваться. Поэтому ta.key можно было не генерировать и не копировать.

Запускаем OpenVPN-сервер:

\# systemctl start openvpn

Смотрим статус:

\# systemctl status openvpn

Должно быть active!

Создадим сертификаты клиента. Для каждого клиента нужно создавать свой собственный сертификат! Создаем запрос на сертификат и сам сертификат:

\# ./easyrsa gen-req laptop nopass
\# ./easyrsa sign-req client laptop

Подтверждаем правильность данных и вводим пароль корневого сертификата.

В домашней директории на сервере создадим папку client:

\# mkdir ~/client

Переходим в /usr/share/doc/openvpn/examples/sample-config-files/. Копируем файл client.conf в созданную папку:

\# cp client.conf ~/client

Переименуем client.conf в ridlaptop.ovpn:

\# mv client.conf ridlaptop.ovpn

Копируем ключи клиента в ~/client:

\# cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/ta.key /etc/openvpn/easy-rsa/pki/private/ridlaptop.key /etc/openvpn/easy-rsa/pki/issued/ridlaptop.crt ~/client

При использовании tls, также копируем ta.key.

Открываем для редактирования файл ridlaptop.ovpn. В строке remote указываем IP-адрес сервера. Порт оставляем 1194. Затем переходим к строкам nobody. Раскомметируем их. В строке group нужно изменить nogroup на nobody. После этого переходим к блоку с SSL/TLS параметрами. Нужно ЗАКОММЕНТИРОВАТЬ строки ca, cert, key!

Добавим данные сертификата ca.crt в .ovpn файл:

\# echo "<ca>" >> ridlaptop.ovpn
\# cat ca.crt >> ridlaptop.ovpn
\# echo "</ca>" >> ridlaptop.ovpn

То же самое делаем для ridlaptop.crt:

\# echo "<cert>" >> ridlaptop.ovpn
\# cat ridlaptop.crt >> ridlaptop.ovpn
\# echo "</cert>" >> ridlaptop.ovpn

Для ridlaptop.key:

\# echo "<key>" >> ridlaptop.ovpn
\# cat ridlaptop.key >> ridlaptop.ovpn
\# echo "</key>" >> ridlaptop.ovpn

И для ta.key:
\# echo "<tls-auth>" >> ridlaptop.openvpn
\# cat ta.key >> ridlaptop.openvpn
\# echo "</tls-auth>" >> ridlaptop.openvpn

Файл .ovpn готов! Теперь его нужно передать на клиент посредством комманды scp. Предварительно файл ridlaptop.ovpn нужно скопировать в /home/admin и изменить владельца и группу на admin.

\# scp -i cert.pem admin@<ip_address>:ridlaptop.ovpn ~
