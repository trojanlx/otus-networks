#  Развертывание коммутируемой сети с резервными каналами

###  Задание:

  1. Создание сети и настройка основных параметров устройства;
  2. Выбор корневого моста;
  3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов;
  4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов;
  
### 1 Карта сети:

![](netmap2.jpg)


Проверьте способность компьютеров обмениваться эхо-запросами
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
### 2. Транк 
```
S2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      8

Port        Vlans allowed on trunk
Et0/0       3-4,8

Port        Vlans allowed and active in management domain
Et0/0       3-4,8

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       3-4,8
```

### 3.	Включите порты F0/2 и F0/4 на всех коммутаторах. Отключив остальные порты.
```
S1#show interfaces status

Port      Name               Status       Vlan       Duplex  Speed Type
Et0/0                        disabled     trunk        auto   auto unknown
Et0/1                        connected    trunk        auto   auto unknown
Et0/2                        disabled     trunk        auto   auto unknown
Et0/3                        connected    trunk        auto   auto unknown
```


### 4 Отобразите данные протокола spanning-tree.
```
S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr 
```
```
S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr 
```
```
S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr
Et0/3               Root FWD 100       128.4    Shr 
```
Корневым мостом в моем случае стал коммутатор S1 т.к. имеет наименьший mac aabb.cc00.1000

![](spanning-tree1.jpg)

### 5	Измените стоимость порта

Меняем на S3
```
S3(config)#interface ethernet 0/3
S3(config-if)#spanning-tree cost 18
```
Смотрим изменения S3
```
S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        18
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg LRN 100       128.2    Shr
Et0/3               Root FWD 18        128.4    Shr 
```
Смотрим изменения S2
Порт E0/3 перешел в режим Altn
```
S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Altn BLK 100       128.4    Shr 
```
Протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе в соостветстии с изменившейся стоимостью порта.

b. Пинг с ПК-A на ПК-B
```
PC-A> ping 192.168.4.3

84 bytes from 192.168.4.3 icmp_seq=1 ttl=63 time=2.068 ms
```

c. Пинг с ПК-A на S2
```
PC-A> ping 192.168.3.12
84 bytes from 192.168.3.12 icmp_seq=1 ttl=255 time=0.455 ms
```

Шаг 2: Выполните следующий тест с помощью PC-B.

Из командной строки на PC-B отправьте команду tracert по адресу PC-A.
```
PC-B> trace 192.168.3.3

trace to 192.168.3.3, 8 hops max, press Ctrl+C to stop

 1   192.168.4.1   0.972 ms  0.574 ms  0.539 ms
 
 2   *192.168.3.3   0.741 ms (ICMP type:3, code:3, Destination port unreachable)
``` 
Вопрос: Какие промежуточные IP-адреса показаны в результатах?

Ответ: 192.168.4.1 - ip адресс виртуального интерфейса G0/0/1.4 на R1 для VLAN 4
