После установки тайлового менеджера оказалось, что тачпад не понимает тап-события, а только клики. Через команду:

$ sudo libinput list-devices

удалось выяснить, что tap-to-click disabled. Сначала я думал, что после того, что удастся заставить тачпад понимать tap, это значение (tap-to-click) станет enabled. Но, видимо, это значение, зашитое в tachpad и оно не изменяется после настроек.

Чтобы настроить поведение tachpad нужно идти в /etc/X11/xorg.conf.d директорию. Там создаём файл 30-touchpad.conf со следующим содержимым:

Section "InputClass"
        Identifier "MyTouchpad"
        MatchIsTouchpad "on"
        Driver "libinput"
        Option "Tapping" "on"
EndSection

Сохраняем и перезагружаемся. Тачпад должен работать!
