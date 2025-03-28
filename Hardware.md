## Описание платы.  
Структурно, схему коммуникации хоста на плате K1 Creality и микроконтролеров можно описать так:

![](/images/pcb_overview.jpg "MCU communication") 

От микропроцессора к интересующей нас периферии отходят три порта -   
- ttyS7 ( к нему подключается микроконтроллер на основной плате, USART1 пины PA2 и PA3 )
- ttyS1 ( к нему подключается RS232 приемопередатчик SP32222 и далее на плату печатающей головки )
- ttyS9 (к нему подключается RS232 приемопередатчик SP32222 и далее на плату тензодатчиков )  
- сигнал включения вторичного напряжения 5В  

Зеленым и красным на рисунке обозначены логические уровни сигналов - 3.3В и 12В соответственно. USB-UART конвертер и UART выходы одноплатников рассчитаны на 3.3В поэтому мы не можем напрямую подключить кабель от печатающей головы а вместо этого вынуждены подпаиваться к SP3222E. Наша задача - отключить все три ( или два, если не планируется в перспективе заниматься экспериментами с тензодатчиками ) порта, отключить питание Wi-Fi модуля ( чтобы не мешал и не забивал сеть ) и подключить к освободившимся портам USB_UART конвертер.   
Кроме этого крайне желательно для стабильности отключить сигнал EN от процессора.

## 1. Подключение основного микроконтроллера 

Для удобства отключения микроконтроллера от хоста, производитель предусмотрел на плате резисторы с нулевым сопротивлением после включенных последовательно с пинами резисторов на 20 Ом. Таким образом отпаяв два резистора с нулевым сопротивлением и подпаявшись к контактным площадкам мы подключимся к портам через токоограничивающие резисторы.

![](/images/mcu_resistors.jpg "MCU resistors")  

Красным отмечены резисторы. которые нужно удалить. Зеленым отмечены точки подключения для USB-UART конвертера. TX UART 1 конвертера подключаем к RX микроконтроллера и, соответственно, RX UART 1 конвертера к TX микроконтроллера. В итоге выглядеть это должно как-то так:

![](/images/mcu_connection.jpg "MCU connection")  

## 2. Подключение печатающей головы и платы тензодатчиков

Также как и для порта ttyS7, для портов ttyS1 и ttyS9 производитель предусмотрел резисторы с нулевым сопротивлением с обратной стороны печатной платы :

![](/images/sp3222_resistors.jpg "SP3222E resistors")  

Нужно отпаять резисторы, отмеченные красным. Далее припаять к микросхеме SP3222E провода от конвертера. Будем припаиватся к пинам с ttl логическим уровнем cогласно даташиту:

![](/images/SP3222E_datasheet.jpg "SP3222E datasheet")  

T1IN и R1OUT - порт печатающей головы, T2IN и R2OUT - порт платы тензодатчиков.
Подключаем TX UART2 к T1IN, RX UART2 к R1IN и TX UART3 к T1IN, RX UART3 к R1IN


![](/images/SP3222E_connection.jpg "SP3222E connection")  

## 3. Отключение сигнала EN

Для возможности перезагрузки микроконтроллеров, на плате предусмотрен транзистор Q5, соединяющий цепь 5V1 и 5V2 вторичного источников питания. В какой-то момент у меня процессор стал включать и выключать это питание с частотой в несколько десятков герц, пришлось внести изменения в схему. 
Для возможности сохранения возможности подключения к микроконтроллеру SWD программатора нужно изменить схему так, чтобы при подключении программатора Q5 оставался закрытым, но при подаче основного питания открывался. Для этого я подобрал и установил резистор между базой тразистора Q6 и 5V1. Номинал резистора 900К. Кроме этого нужно удалить резистор R39

![](/images/EN_mod.jpg "ENABLE mod")

Если подключаться к микроконтроллеру программатором не требуется то можно обойтись одной перемычкой между коллектором и эмиттером транзистора Q6. В этом варианте отпаивать резистор R39 не требуется.

![](/images/Q6_short.jpg "Q6_short")

## 4. Отключение WiFi модуля.  

Для отключения модуля нужно отпаять дроссель в цепи питания. 

![](/images/WiFi_module.jpg "WiFi")  

## 5. Распиновка Bluepill.  

Распиновка приведена в [источнике](https://github.com/r2axz/bluepill-serial-monster), приведу тут нужную нам часть:

| Signal |   Direction   |     UART1     |     UART2     |     UART3     |
|:-------|:-------------:|:--------------|:--------------|:--------------|
|   RX   |      IN       |      PA10     |      PA3      |      PB11     |
|   TX   |      OUT      |      PA9      |      PA2      |      PB10     |

Подключаем все порты. Я подключил UART1 к основному микроконтроллеру, UART2 к плате тензодатчиков и UART3 к плате печатающей головы. Конечно, порядок может быть любой. нужно только прописать порядок портов в конфиге.  

GND я подключил от удобно для меня расположенной AMS1117 

![](/images/Bluepill_GND.jpg "GND connection") 

## 6. Подключение хоста к питанию.

Я подключил хост через DC-DC преобразователь. К преобразователю припаял кабель с USB type C разъемом для подключения к одноплатнику. Разумеется можно подключить иначе, например просто припаяв провода к одноплатнику или использовать гребенку. 

![](/images/DC-DC.jpg "DC-DC")

## 7 Подключение штатной WiFi антенны.

Я переиспользовал штатную антенну, просто отрезав кабель и соединив его с коннектором одноплатника. Конечно такое подключение увеличивает КСВ, вызывает переотражение сигнала в месте пайки, но никакого ухудшения качества связи с принтером я не заметил. 

![](/images/WiFi_antenna.jpg "WiFi antenna")

На этом с апаратной частью работа закончена, можно переходить к [программной](Software.md)