После установки PostgreSQL локально из репозиториев Linux, необходимо перед запуском сервера создать кластер баз данных. Это производится командой:

$ sudo -u postgres initdb --locale en_US.UTF-8 -D /var/lib/postgres/data

Теперь директорией кластера является /var/lib/postgres/data.

После этого можно запустить сервер командой:

$ sudo systemctl start postgresql

То же самое можно сделать командой:

$ pg_ctl -D /var/lib/postgres/data -l logfile start

но у меня нет каких-то разрешений для записи логов и команда фэйлится. А с systemctl всё работает!
