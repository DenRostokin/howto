LVM (Logical Volume Manager) - менеджер логических томов - позволяет создавать логические разделы на диске, вместо настоящих. Работает на уровне ядра системы и позволяет абстрагироваться от уровня железа. Упрощает работу с накопителями, т.к. разрешает изменять размеры разделов, их количество и т.д.

Структура LVM состоит из трех слоев:

    Физический том (один или несколько), Physical Volume (PV)
    Группа физических томов, Volume Group (VG)
    Логический том, который и будет доступен программам, Logical Volume (LV)

Для работы с LVM необходимо установить пакет lvm2.
