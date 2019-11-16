В локальной сети есть 3 машины: клиент, роутер и сервер. Клиент и сервер имеют по одному внутреннему интерфейсу, а роутер - соответственно 2 для каждой из подсетей. Настройка интерфейсов описана в LocalNetwork.md.

Сконфигурируем роутер. Для этого отредактируем файл /etc/sysctl.conf. Нужно раскоментировать или добавить (если нет) строчку:

net.ipv4.ip_forward=1

Эти изменения будут применяться при каждой загрузке системы. Но чтобы изменения вступили в силу прямо сейчас, нужно перезагрузить роутер! Чтобы не перезагружать роутер, нужно выполнить ещё одну команду:

$ sudo echo 1 > /proc/sys/net/ipv4/ip_forward

Теперь форвардинг разрешён!