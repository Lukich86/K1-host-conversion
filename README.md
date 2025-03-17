
# K1-host-conversion

## Подключение одноплатника OrangePi (или любого другого) к K1.

Плюсы - чистый клиппер, возможность использовать хост с расширенным функционалом (памятью, наличием CAN шины, портов, свободных пинов и т.д.)

Минусы - не будет работать штатный дисплей, не будут работать тензодатчики.  

Требуемые детали:

1. Одноплатный компьютер. Я использовал Orange Pi Zero 3 1GB RAM

2. USB-TTL x2 конвертер (опционально).  
Я делал эту конверсию в расчете на то, что буду подключать к хосту три микроконтроллера - один на материнской плате, второй в печатающей голове и третий для тензометрических датчиков. Но в процессе я понял, что лично мне тензы не нужны и перешел на CR Touch. В таком случае если не подключать плату тензодатчиков нужно только два UART порта и если на на одноплатнике есть два UART с уровнем 3,3В то покупать конвертер не надо. Я использовал https://github.com/r2axz/bluepill-serial-monster прошивку для bluepill (нужна именно на STM32F103**C8**T6 а не на STM32F103**C6**T6 ) но можно купить готовый конвертер на FTDI чипе, например такой https://aliexpress.ru/item/1005006850550816.html.  
![](/images/USB_UART_converter_example.png "FTDI USB UART")
3. DC-DC преобразователь для питания одноплатника (при необходимости). Я использую Orange Pi c напряжением питания 5В и использую преобразователь для понижения 24В до 5В. Если ваша плата поддерживает питание от 24В то преобразователь не нужен. Я использовал преобразователь на LM2596 https://aliexpress.ru/item/1775868763.html  
![](/images/DC_DC.png "DC DC") 


## [ Аппаратные изменения, подключение хоста.](/Hardware.md)

## [ Программные изменения, прошивка.](/Hardware.md)



