Используется для удаления или перевода символов. Уберём двоеточие с конца строки:

$ echo "hello:" | tr -d ":" | cat

вывод: hello