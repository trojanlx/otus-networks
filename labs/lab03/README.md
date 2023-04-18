#  Лабораторная работа — реализация DHCPv4

###  Задание:

  1. Создание сети и настройка основных параметров устройства;
  2. Настройка и проверка двух серверов DHCPv4 на маршрутизаторе R1;
  3. Настройка и проверка ретрансляции DHCP на маршрутизаторе R2;
  4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов;
  
### 1. Cоздание сети и настройка основных параметров устройства:

![](netmap2.jpg)

 Проверьте связь

```
S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/6 ms
S2#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```
### 2.	Определение корневого моста
