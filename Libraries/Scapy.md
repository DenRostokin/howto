Для создания пакетов используются функции из scapy.all. Например, для создания ARP-пакета используется функция scapy.all.ARP()

В функцию передаются аргументы, утанавливавющие различные значения в заголовке пакета. Все значения заголовков пакета можно посмотреть через метод show():

scapy.all.ARP().show()

Но более правильным методом будет зайти в консоль python и ввести:

import scapy.all as scapy
scapy.ls(scapy.ARP)

Пакеты можно комбинировать через операцию деления (/):

ether_packet = scapy.all.Ether(dst="ff:ff:ff:ff:ff:ff")
arp_packet = scapy.all.ARP(psrc="192.168.1.1")

packet = ether_packet/arp_packet

Мы создали пакет для запроса MAC-адреса хоста по его IP-адресу.

Теперь нужно передать этот пакет в сеть. Для этого используется функция:

[answered_list, unanswered_list] = scapy.all.srp(packet)

answered_list и unanswered_list - это списки пар. Каждая пара состоит из запроса и ответа на этот запрос. Например, чтобы извлечь запрашиваемый MAC-адрес, напишем:

response = answered_list[0] [1]

mac = response.hwsrc

Т.е. мы берём первую пару из списка, и берем второй элемент пары, т.е. ответ на запрос. Ответ response - это словарьс теми же ключами, что передаются в функцию ARP. Следовательно, чтобы взять MAC-адрес, нам нужно взять hwsrc.

Готово!!!

