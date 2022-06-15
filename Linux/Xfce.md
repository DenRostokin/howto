Для смены поведения клавиши CapsLock на Ctrl нужно отредактировать файл /etc/default/keyboard. Нужно добивить (или изменить) строку, чтобы она стала такой:

XKBOPTIONS="ctrl:nocaps"
