# Программные изменения, прошивка.

## Предварительные требования:

Устанавливаем на одноплатник хоста ОС и с помощью [KIAUH](https://github.com/dw-0/kiauh) (или любым другим удобным для вас способом) устанавливаем нужные сервисы - klipper, moonraker, fluidd или mainsall. Для установки KIAUH вводим :

`sudo apt-get update && sudo apt-get install git -y`  
`cd ~ && git clone https://github.com/dw-0/kiauh.git`   
`./kiauh/kiauh.sh`

И далее в меню install устанавливаем нужные сервисы.

На этом этапе прописывать конфиг подключения к MCU не нужно, иначе нужные порты будут заняты и нужно будет вводить команду `sudo service klipper stop`

## 0. Установка прошивки bluepill-serial-monster



## 1. Прошивка бутлоадеров

Для замены бутлоадера нам потребуется скрипт mcu_utils.py из репозитория [cryoz](https://github.com/cryoz/k1_mcu_flasher) а вместо оригинального бутлоадера будем использовать форк katapult от [Arksine](https://github.com/Arksine/katapult) сделанный человеком с ником Fdl3 из [discord чата D3vil Design](https://discord.com/channels/1154500511777693819/1217873195424813177).


Клонируем нужные нам репозитории:  

`cd ~ && git clone https://github.com/cryoz/k1_mcu_flasher`  

Ищем порты к которым у нас подключены микроконтроллеры: 

`dmesg | grep tty`

Получаем вывод наподобие
```
[    0.000600] printk: console [tty1] enabled
[    1.716300] printk: console [ttyS0] disabled
[    1.716377] 5000000.serial: ttyS0 at MMIO 0x5000000 (irq = 285, base_baud = 1500000) is a 16550A
[    1.716712] printk: console [ttyS0] enabled
[   12.810369] systemd[1]: Created slice Slice /system/serial-getty.
[   13.964922] systemd[1]: Found device /dev/ttyS0.
[   14.283800] cdc_acm 4-1:1.0: ttyACM0: USB ACM device
[   14.285835] cdc_acm 4-1:1.2: ttyACM1: USB ACM device
[   14.287994] cdc_acm 4-1:1.4: ttyACM2: USB ACM device
[   14.627703] mtty_probe init device addr: 0x00000000eab98f5e
[   20.149292] mtty_open device success!
```
В моем случае UART1-3 конвертера определились в системе как ttyACM0-ttyACM2.  

В качестве замены неудобного оригинального бутлоадера (он неудобен тем что handshake нужно сделать в течение 15 секунд после включения микроконтроллера, в этот момент одноплатник еще не успеет загрузится, а значит для обновления прошивки нужно лезть в подвал принтера и вручную скидывать питание) мы будем использовать форк katapult который изменен таким образом, что может быть загружен через бутлоадер от creality. Кроме того он скомпилирован в deploy режиме, и после установки заменяет собой бутлоадер от creality.  

Скачиваем на одноплатник [файл прошивки](/binaries/deployer.bin)


Для прошивки katapult используем скрипт mcu_utils.py.

Переходим в папку со скриптом 

`cd ~/k1_mcu_flasher`

Перезагружаем материнскую плату (я просто снял на несколько секунд предохранитель с материнской платы и поставил обратно) и в течение 15 секунд успеваем запустить скрипт который переведет бутлоадер в режим ожидания прошивки:

`python3 mcu_util.py -c -i /dev/ttyACM2 -g -v`

Где `/dev/ttyACM2` наш порт к которому подключен микроконтроллер для прошивки. В моем случае ttyACM2 это порт микроконтроллера головы. 

Если все успешно, то получаем вывод в консоль:

```
send handshake
rcv data b'75'
handshake confirmed
send version request
rcv data b'6e6f7a305f3132305f4733302d6e6f7a305f3030335f303030e8'
version received! b'noz0_120_G30-noz0_003_000'
FW Version: noz0_120_G30-noz0_003_000
```
После этого запускаем скрипт прошивки deployer.bin

`python3 mcu_util.py -c -i /dev/ttyACM2 -v -u -f ~/deployer.bin`

Если все успешно то получаем вывод:

```
send handshake
send sectorsize request
rcv data b'02fd'
sector size received! 2
send update request
[flash_update] [1] rcv data b'758a'
[flash_update] [1] update request confirmed!
[flash_update] [2] rcv data b'758a'
[flash_update] [2] FW size confirmed!
[flash_update] rcv data b'758a'
[flash_update] chunk flashed
[flash_update] rcv data b'758a'
[flash_update] chunk flashed
[flash_update] rcv data b'20df'
[flash_update] [3] flash completed
Firmware updated successfully
send app_start request
rcv data b'758a'
app started!
App started

```

Проверяем, что katapult установился:

`python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -s`

Если все успешно то вывод :

```
Connecting to Serial Device /dev/ttyACM2, baud 230400
Attempting to connect to bootloader
Katapult Connected
Software Version: v0.0.1-91-fdl3
Protocol Version: 1.1.0
Block Size: 64 bytes
Application Start: 0x8002000
MCU type: stm32f103xe
Status Request Complete
```
Повторяем прошивку для основного микроконтроллера - опять перезагружаем материнскую плату и запускаем 

`python3 mcu_util.py -c -i /dev/ttyACM0 -g -v`

Затем 

`python3 mcu_util.py -c -i /dev/ttyACM0 -v -u -f ~/deployer.bin`

Теперь бутлоадеры прошиты, можно приступать к сборке и прошивке бинарников klipper. 

Перед прошивкой Katapult через st-link обязательно нужно сделать full chip erase
Обязательно нужно отключить klipper и перезагрузить (ну или в конфиге убрать /dev/tty)

## 3. Прошивка klipper

### 3.1 Плата печатающей головы

Переходим в папку с klipper и запускаем make menuconfig (или можно в KIAUH выбрать "Advanced" - "Build")

`cd ~/klipper`  
`make menuconfig`

Выбираем как на скриншоте:

![](/images/nozzle_mcu_menuconfig.jpg "toolhead menuconfig")

Выходим с сохранением настроек и запускаем сборку

`make`

Теперь прошиваем получившийся бинарник  

`python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -s`

Далее

`python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -f ~/klipper/out/klipper.bin`  

Вывод должен быть:

```
Connecting to Serial Device /dev/ttyACM2, baud 230400
Detected Klipper binary version v0.12.0-458-gd886c176, MCU: stm32f103xe
Attempting to connect to bootloader
Katapult Connected
Software Version: v0.0.1-91-fdl3
Protocol Version: 1.1.0
Block Size: 64 bytes
Application Start: 0x8002000
MCU type: stm32f103xe
Flashing '/home/orangepi/klipper/out/klipper.bin'...

[##################################################]

Write complete: 34 pages
Verifying (block count = 531)...

[##################################################]

Verification Complete: SHA = 9A0A75F1987338EE8D4E1A5736E793B1E63E8914
Programming Complete

```

### 3.2 Основной микроконтроллер

Повторяем все то же самое что и в предыдущем пункте, только выбираем нужный порт ( у меня ttyACM0) и настраиваем сборку согласно скриншоту

![](/images/main_mcu_menuconfig.jpg "mcu menuconfig")


## 4. Базовый минимальный конфиг

Для проверки можно использовать мой минимальный конфиг. Он подразумевает что у вас установлен CRTouch, модуль [Klippain](https://github.com/Frix-x/klippain-shaketune), замененные шкивы.

[Конфиг](/config/)