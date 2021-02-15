Чтобы узнать о том, разрешён ли запуск сервиса при загрузке системы (enabled/disabled), нужно использовать комманду:

$ sudo systemctl list-unit-files | grep -i <service_name>
