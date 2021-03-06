# Лабораторная работа: развертывание коммутируемой сети с резервными каналами.
# Задание:
- Базовые настройки сетевых устройств
- Определение корневого моста и состояния портов
- Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
- Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета порта

# Решение:

 # Схема лабораторного стенда:
 ![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/%D0%A1%D1%85%D0%B5%D0%BC%D0%B0%20%D0%BB%D0%B0%D0%B1%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80%D0%BD%D0%BE%D0%B3%D0%BE%20%D1%81%D1%82%D0%B5%D0%BD%D0%B4%D0%B0%20STP.png)

# Таблица адресации:
| Устройство | Интерфейс  |   IP адрес   | Маска подсети |
| :----------|:----------:| :-----------:|:-------------:|
| S1         | VLAN 1     | 192.168.1.1  | 255.255.255.0 |
| S2         | VLAN 1     | 192.168.1.2  | 255.255.255.0 |
| S3         | VLAN 1     | 192.168.1.3  | 255.255.255.0 |

# 1. Базовая конфигурация устройств.
Сделаем основные настройки коммутаторов S1, S2, S3:
```
conf t
 #hostname S1
 #no ip domain lookup
 #enable secret class
 #line console 0
 #password cisco
 #login
 #logging synchronous 
 #line vty 0 15
 #password cisco
 #login
 #logging synchronous 
 #exit
 service password-encryption 
 banner motd $ This is a secure system. Authorized access only. $
 end
 ```
Конфигурации находятся по ссылкам: [S1](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/config%20S1), [S2](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/config%20S2), [S3](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/config%20S3).

# 2. Определение корневого моста и состояния портов.
На всех трех коммутаторах введем команду show spanning-tree, чтобы определить корневой мост. Из предоставленных данных видно, что корневым мостом является коммутатор S1.
![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/root%20bridge.png)
На схеме ниже определены состояния всех подключенных портов.
![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/%D0%A1%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F%20%D0%BF%D0%BE%D1%80%D1%82%D0%BE%D0%B2.png)

# 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов.
С помощью команды show spanning-tree на коммутаторе S3 определим корневой и заблокированный порт:
![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/S3%20STP_1.png)
Порт Fa0/4 является корневым, порт  Fa0/2 - заблокированными. Далее с помощью команды spanning-tree vlan 1 cost 18 изменим стоимость порта Fa0/4:
```
int fa0/4
spanning-tree vlan 1 cost 18
```
После этого заметим, что топология протокола перестроилась и порт Fa0/2 стал назначенным. Так же заметно, что на коммутаторе S2 стал заблокированным порт Fa0/4. Так произошло, потому что из сегмента сети между S2 и S3 теперь стоимость пути до S1 меньше через S3.
![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/S3%20STP_2.png)
![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/S3%20STP_3.png)
Удаление записи стоимости порта Fa0/4:
```
int fa0/4
no spanning-tree vlan 1 cost 18
```

# 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета порта.
После активации портов Fa0/1 и Fa0/3, протокол STP перестроил топологию. Теперь на коммутаторе S2 порта Fa0/1 стал корневым, а Fa0/2 - заблокирован.
![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/S3%20STP_4.png)
На коммутаторе S3 порт Fa0/3 стал корневым, а Fa0/4 - заблокирован.
![](https://github.com/dmitriyklimenkov/LAB2-STP/blob/main/S3%20STP_5.png)
Так произошло, потому что протокол выбрал порт с наименьшим значение - корневым.



