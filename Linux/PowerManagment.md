Для управления питанием используется интерфейс ACPI. Это модули ядра, которые предоставляют специальные функции и добавляют информацию в /proc и /sys. Эта информация может парсится такими демонами, как acpid для подписки на события.

В системах с systemd уже имеется служба, работающая с ACPI - это systemd-logind. Чтобы проверить, запущена ли она, выполним команду:

$ sudo systemctl status systemd-logind

Если эта служба запущена, но не нужно устанавливать что-то ещё, т.к. ото может привести к конфликтам!

Конфиги systemd-logind находятся в /etc/systemd/

Но systemd не умеет обрабатывать события батареи. Поэтому, если мы хотим установить утилиты, которые уменьшают потребление батареи (например, Laptop Mode Tools), то acpid необходим!

Установим acpid

$ sudo pacman -S acpid

Запустим и разрешим сервис:

$ sudo systemctl start acpid
$ sudo systemctl enable acpid

Когда происходит событие, acpid запускает скрипт для обработки этого события: /etc/acpi/handler.sh.

Для управления частотами процессора используется утилита cpupower:

$ sudo pacman -S cpupower

Его конфигурационный файл: /etc/default/cpupower. Он считывается скриптом /usr/lib/systemd/scripts/cpupower, который, в свою очередь, запускается службой cpupower.service. Запустим и разрешим этот сервис:

$ sudo systemctl enable cpupower
$ sudo systemctl start cpupower

В завершение, установим Laptop Mode Tools из AUR. Их основной задачей является разрешить режим Laptop Mode в ядре Linux.

$ yay laptop-mode-tools

В KDE вместо laptop-mode-tools используется TLP, который является конфликтом. Поэтому нужно выбрать что-то одно.

Разрешим и запустим laptop-mode:

$ sudo systemctl enable laptop-mode
$ sudo systemctl start laptop-mode
