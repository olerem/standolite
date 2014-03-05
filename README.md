standolite --- программное обеспечение для организации работы
по отладке ПО для встраиваемых систем. standolite обеспечивает
как можно более полный удалённый доступ к аппаратному
обеспечению встраиваемой системы (платы) для нескольких разработчиков.

Краткое описание технологии удалённой работы с платами на основе
опыта работы над bareboх.

Для работы над barebox я покупал плату (как правило в составе
готового устройства), обеспечивал её питанием и
 подключался к ней по RS232/UART. При этом держать плату рядом
с собой мне было как правило неудобно --- распотрошённое устройство
занимает сравнительно много места, хрупко, а кроме того, для
того, чтобы включить/выключить питание и нажать RESET надо
к нему тянуться собственными руками --- всё это создаёт много
проблем.

Для решения проблем я перешёл к методике удалённой работы.
Для каждой платы собирается *стенд*, т.е.

  * выделяется место где никто не ходит, есть питание и охлаждение;
  * выделяется *инструментальная ЭВМ* --- проще говоря ПК под управлением
    Debian Linux;
  * обеспечивается возможность удалённого включения/выключения
    питания платы, в крайнем случае подача сигнала RESET;
    для этой цели используются доступные устройства, например, USB-реле;
  * плату подключается к инструментальной ЭВМ по интерфейсам RS232/UART и, если
    есть, Ethernet и EJTAG;
  * если у платы имеется экран/лампочки, то для наблюдения используется
    web-камера.

При создании стендов преследуются две основные цели:
  * создать площадку для дистанционного обучения разработчиков встроенного ПО;
  * ускорить развитие barebox, Linux, openocd и другого ПО для отдельных плат.

# Приложение 1. Описание стенда на базе D-Link DIR-320 #

D-Link DIR-320 (далее просто DIR-320) представляет
собой недорогой беспроводный маршрутизатор для
подключения квартиры или небольшого офиса к сети Internet.

Технические детали DIR-320:

  * http://wiki.openwrt.org/toh/d-link/dir-320
  * http://www.dlink.ru/ru/products/5/786.html

Стенд позволяет работать с D-Link DIR-320
полностью удалённо. Стенд работает круглосуточно.

В стенде используется DIR-320 со штатной прошивкой.

Стенд состоит из управляющей ЭВМ (zoo) и нескольких адаптеров,
которые обеспечивают подключение к интерфейсам DIR-320:

  * через Ethernet DIR-320 подключён к zoo; на zoo имеется tftp-сервер;
  * через USB-UART адаптер обеспечивается доступ к последовательной
    консоли DIR-320; в частности становится возможно прервать
    базовый сценарий работы встроенного ПО (CFE) и вместо штатного
    ядра Linux 2.4.20 грузить через Ethernet своё ядро или barebox;
  * через USB-JTAG адаптер на базе ft2232 обеспечивается доступ
    к интерфейсу EJTAG BCM5354;
  * при помощи [KernelChip Laurent](http://www.kernelchip.ru/Laurent.php)
    обеспечивается управление питанием DIR-320 --- можно отключать и снова
    включать питание, обеспечивая таким образом полный сброс платы.

````
                       _________
                      | USB-JTAG|
               _______| adapter |__________
    __________|__     |_________|          |
   |             |                         |EJTAG 14 pin
   |             |                     ____|____
   |     zoo     |                    |         |
   |             |     _________      | D-Link  |
   |             |____| USB-UART|_____| DIR-320 |
   |_____________|    | adapter | UART|_________|
        |     |       |_________|     LAN1|    |
        |     |                           |    |управление
        |     |            _________      |    |питанием
        |     |           | Ethernet|     |    |
        |     |___________| switch2 |_____|    |
        |                 |_________|          |
        |                                      |
        |                  _________          _|________
        |                 | Ethernet|        |         |
        |_________________| switch1 |________| Laurent |
                          |_________|        |_________|

````

# Приложение 2. Инструкция по доступу к стенду D-Link DIR-320 #

Под <<стендом отладки ПО микропроцессорной платы>> будем
понимать инструментальную ЭВМ на базе x86, к которой
через необходимые преобразователи по интерфейсам RS232, UART,
Ethernet, EJTAG, GPIO, видеоинтерфейсам и другим подключается
микропроцессорная плата (как правило на базе микропроцессора
с архитектурой MIPS).

Основными пользователи подобных стендов являются разработчики
ПО, для продуктивной работы которых желательно обеспечить
удобный удалённый доступ к микропроцессорной плате:

  * для быстрой загрузки в ОЗУ платы разрабатываемого ПО;
  * для получения отладочной информации о запуске
    разрабатываемого ПО;
  * для включения/выключения платы, а также для подачи сигнала
    СБРОС на плату.

С точки зрения пользователя стендом является несколько
программ с текстовым интерфейсом, которые обеспечат

  * доступ к аппаратуре управления питанием и сбросом платы;
  * доступ к терминальной программе для взаимодействия с платой
    через последовательный интерфейс RS232 (UART);
  * доступ к консоли openocd (или другого ПО работы с JTAG);
  * передачу файлов на инструментальную ЭВМ, с которой имеется
    возможность загрузить данные в плату по RS232 (UART) или
    Ethernet.

Управление доступом к стендам осуществляется при помощи
доморощенного ПО standolite, созданного под впечатлением
gitolite. Подобно тому, как gitolite осуществляет разграничение
доступа нескольких пользователей к нескольким репозиториям git,
standolite осуществляет разграничение доступа нескольких пользователей
к нескольким стендам. Для доступа к стенду используется протокол ssh.

Для доступа к стенду пользователь должен предоставить свой открытый RSA ключ.

**Для доступа к стенду необходимо использовать терминал не менее 80x26 символов!**

Для проверки доступа необходимо подключиться по ssh к dir.tftpd.net
(порт 2022), пользователь standolite, используя для идентификации
тот секретный ключ, соответствующий которому открытый ключ был
отправлен администрации standolite.

````
$ ssh -p 2022 -i <id_rsa-key-file> standolite@<standolite host>
````

В случае успешного подключения на экран будет выдано
приветствие standolite:

````
hello <username>, this is standolite@<standolite host> running standolite
````

также будет выведена краткая шпаргалка по настройке конфигурационного
файла openssh ```.ssh/config```.

Для того, чтобы использовать для подключения к стенду при помощи openssh
более короткую строку, чем та, которая описана выше, следует занести
в ```.ssh/config``` запись вида

````
host standolite
        hostname <standolite host>
        port <standolite host ssh port>
        user standolite
        identityfile <id_rsa-key-file>
        ForwardX11 no
        RequestTTY force
````

В этом случае для проверки настроек подключения достаточно выполнить команду

````
$ ssh standolite
````

Команды доступа к стендам задаются так:

````
$ ssh standolite <имя команды> [ <аргументы команды> ]
````

## list ##

Показывает доступные пользователю стенды.

Пример:

````
$ ssh standolite list

hello antony, this is standolite@zoo running standolite

stand-dir-320
````

Таким образом в примере пользователю antony доступен только один
стенд: ```stand-dir-320```.

## attach <имя стенда> ##

 <<Присоединиться>> к стенду.

Пример:

````
$ ssh standolite attach stand-dir-320
````

После подключения к стенду пользователь подключается к сессии tmux, в качестве
текущего окна tmux выбирается окно с кратким руководством, как использовать
сессию tmux для доступа к ресурсам стенда.


# Приложение 3. Установка gitolite #

0. на инструментальной ЭВМ под управлением Debian Linux
установить по крайней мере tmux, dialog, openssh-server;

1. создать пользователя ```standolite```;

2. в домашнем каталоге пользователя ```standolite``` создать
каталог ```standolite```, в который поместить содержимое
данного репозитория; таким образом будет доступен стенд stand-dir-320;

3. создать файл ```~standolite/.ssh/authorized_keys```, куда
добавить ключи авторизованных пользователей (пример файла
```authorized_keys``` см. в ```ssh/authorized_keys```).
