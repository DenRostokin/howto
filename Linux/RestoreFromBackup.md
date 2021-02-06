Сначала нужно разбить диск на разделы и отформатировать его. Если на диске есть разделы, то можно удалить их командой cfdisk в интерактивном режиме.

$ sudo cfdisk /dev/sdb

Создадим на диске таблицу разделов GPT командой fdisk:

$ sudo fdisk /dev/sdb

Затем в интерактивном режиме вводим g, а затем w (для сохранения и выхода). Теперь разобьём диска на разделы командой cfdisk. Должны быть вот такие разделы:

1. /dev/sdb1 512M FAT32 EFI
2. /dev/sdb2 +100%

При создании раздела под UEFI необходимо указать тип раздела: EFI!

Форматируем /dev/sdb1 в FAT32, т.к. только эту файловую систему понимает UEFI:

$ sudo mkfs.vfat -F32 /dev/sdb1

Создаём криптоконтейнер на разделе /dev/sdb2 и открываем его:

$ sudo cryptsetup --type luks2 -c aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 3000 -y --use-random luksFormat /dev/sdb2
$ sudo cryptsetup luksOpen /dev/sdb2 usb

В /dev/mapper появился виртуальный диск usb.

Теперь создаём виртуальные разделы под систему посредством LVM. Сначала создадим виртуальный физический раздел:

$ sudo pvcreate /dev/mapper/usb

Посмотреть, действительно ли был создан физический том можно командой:

$ sudo pvscan

Для вывода более подробной информации по физическому тому используется команда:

$ sudo pvdisplay

Далее создаём виртуальную группу разделов:

$ sudo vgcreate vg0 /dev/mapper/usb

Посмотреть группу томов можно командой:

$ sudo vgdisplay

После создания виртуальной группы, она, по-умолчанию, становится активной. Для того, чтобы можно было закрыть LUKS-контейнер, необходимо будет деактивировать виртуальную группу. Делается это командой:

$ sudo vgchange -a n <volume_group_name>

Осталось создать виртуальные разделы. Создадим 2 раздела:

1. Раздел подкачки (swap) размером 8Гб
2. root-раздел подо всю систему

$ sudo lvcreate --size 8G vg0 --name swap
$ sudo lvcreate -l +100%FREE vg0 --name root

Посмотреть информацию по созданным виртуальным разделам можно командой:

$ sudo lvdisplay

Создаём файловые системы для этих виртуальных разделов:

$ sudo mkfs.ext4 /dev/mapper/vg0-root
$ sudo mkswap /dev/mapper/vg0-swap

Монтируем /dev/mapper/vg0-root в /mnt/usb:

$ sudo mount /dev/mapper/vg0-root /mnt/usb

Включаем раздел подкачки:

$ sudo swapon /dev/mapper/vg0-swap

Создаём директорию для загрузчика:

$ sudo mkdir /mnt/usb/boot

Монтируем /dev/sdb1 в /mnt/usb/boot:

$ sudo mount /dev/sdb1 /mnt/usb/boot

Далее копируем весь backup в примонтированный раздел. Путь backup примонтирован в /mnt/backup. Копируем через rsync:

$ sudo rsync --progress -av /mnt/backup/ /mnt/usb

ВАЖНО! в пути /mnt/backup/ обязательно нужен завершающий слэш, иначе в /mnt/usb создастся папка backup, в которой уже будут находиться системные файлы.

Чтобы корректно создать загрузочную запись, монтируем рабочие каталоги к нашему будующему root-каталогу. Каталоги /dev и /proc сейчас используются live-системой, поэтому используем параметр --bind, чтобы они были доступны сразу в 2-х местах:

$ sudo mount --bind /dev /mnt/usb/dev
$ sudo mount --bind /proc /mnt/usb/proc
$ sudo mount --bind /sys /mnt/usb/sys

Переключаемся в новую систему, использую chroot:

$ sudo chroot /mnt/usb

Редактируем файл /etc/fstab. Узнать текущие UUID разделов можно командой blkid:

$ sudo blkid

Теперь устанавливаем загрузчик systemd-boot:

\# bootctl --path=/boot install

Устанавливаем параметры загрузки:

\# echo 'default arch' >> /boot/loader/loader.conf
\# echo 'timeout 5' >> /boot/loader/loader.conf

Создаём файл arch.conf. Имя файла должно быть таким же, как и в параметре default arch в файле /boot/loader/loader.conf!

\# touch /boot/loader/entries/arch.conf

Добавляем в этот файл следующие строки:

\# echo 'title Arch Linux' >> /boot/loader/entries/arch.conf
\# echo 'linux /vmlinuz-linux' >> /boot/loader/entries/arch.conf
\# echo 'initrd /initramfs-linux.img' >> /boot/loader/entries/arch.conf
\# echo 'options cryptdevice=UUID=<UUID>:vg0 root=/dev/mapper/vg0-root resume=/dev/mapper/vg0-swap rw intel_pstate=no_hwp' >> /boot/loader/entries/arch.conf

Вместо <UUID> нужно указать UUID раздела жёсткого диска, на котором установлем криптоконтейнер, например, /dev/sdb2.

Выходим из chroot. Размонтируем /mnt/usb/dev, /mnt/usb/proc и /mnt/usb/sys. Затем размонтируем раздел /dev/sdb1 и размонтируем логический раздел /dev/mapper/vg0-root. Выключаем раздел подкачки /dev/mapper/vg0-swap:

$ sudo swapoff /dev/mapper/vg0-swap

После того, как все разделы размонтированы, деактивируем виртуальную группу:

$ sudo vgchange -a n vg0

Закрываем криптоконтейнер:

$ sudo cryptsetup luksClose usb

Перезагружаемся!
