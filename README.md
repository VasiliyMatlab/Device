# Device
Система с множественным управлением (СМУ).


- [Состав системы и приницы работы](#Info)
  - [Общая структура](#Structure)
    - [ВПО](#VPO)
    - [Табло](#Tablo)
    - [ВПИ](#VPI)
    - [Панель](#Panel)
  - [Информационный обмен](#Exchange)
    - [ВПИ-ВПО](#VPI-VPO)
    - [ВПО-ВПИ](#VPO-VPI)
    - [ВПО-Табло](#VPO-Tablo)
    - [Табло-ВПО](#Tablo-VPO)
    - [ВПО-Панель](#VPO-Panel)
    - [Панель-ВПО](#Panel-VPO)
- [Требования](#Requirements)
- [Скачивание и сборка](#Building)
- [Использование](#Usage)


<a name="Info"/><h3>Состав системы и приницы работы</h3>
<a name="Structure"/><h4>Общая структура</h4>
СМУ состоит из трех физических объектов: Устройства (осноной объект), ЭВМ и Внешних абонентов. Программное обеспечение СМУ состоит из четрех взаимозависимых компонентов: внешнего пользовательского интерфеса (ВПИ), встраиваемого программного обеспечения (ВПО), передней Панели Устройства, и Табло. Связи между компонентами, а также языки программирования и библиотеки, с помощью которых они реализованы, показаны на Рисунке 1.
<p align="center"><img src="images/structure.jpg" width="600"/></p>
<p align="center"><em>Рисунок 1</em></p>
Рассмотрим подробнее каждый из компонентов.


<a name="VPO"/><h5>ВПО</h5>
ВПО (встраиваемое прораммное обеспечение) - основной компонент встраиваемой программы Устройства и СМУ, является главным связующим звеном всего СМУ. К основным задачам и функциям работы ВПО можно отнести:
1. Управление [Табло](#Tablo) с помощью сетевых интерфейсов.
2. Обеспечение связи с передней [Панелью](#Panel) Устройства.
3. Сетевое взаимодействие с [ВПИ](#VPI).
4. Автономная работа без средств управления (ВПИ и Панель).
5. Отработка различных режимов и состояний работы Устройства.

Устройство, в состав которого входит ВПО, может находиться в одном из двух состояний: ОТКЛ/ВКЛ (отключено/включено). В состоянии ОТКЛ ВПО никак не взаимодействует с другими компонентами, за исключением отработки команды "Включить" от ВПИ или передней Панели. В состоянии ВКЛ ВПО выполняет свои целевые функции в полном объеме.  
А также в одном из следующих режимов: ЛУ/ВУ (локальное управление/внешнее управление). В состоянии ЛУ ВПО отрабатывает команды, полученные от передней Панели, и игнорирует команды, пришедшие от ВПИ. В состоянии ВУ ВПО отрабатывает команды, полученные от ВПИ, и игнорирует команды, пришедшие от Панели. Нахождение Устройства в состоянии ЛУ/ВУ регулируется переключателем (далее по тексту - <em>тумблер</em>) на передней Панели.  
Граф режимов и состояний Устройства изображен на Рисунке 2. На Рисунке 2 сплошные стрелки обозначают команды, получаемые от передней Панели; штрих-пунктирные - команды от ВПИ; пунктирные - команды от ВПИ или от Панели.
<p align="center"><img src="images/modes_and_states.jpg" width="600"/></p>
<p align="center"><em>Рисунок 2</em></p>
Архитектура программного компонента ВПО представлена на Рисунке 3. Она состоит из четырех потоков: Main Thread (отвечает за взаимодействие с другими потоками и вычисление своего состояния и режимов работы), VPI Thread (отвечает за сетевое взаимодействие с ВПИ), Panel Thread (отвечает за взаимодействие с Панелью), Tablo Thread (отвечает за сетевое взаимодействие с Табло). Блочные стрелки показывают взаимодействие между потоками на основе шины (множества переменных и сигналов); тонкие стрелки обозначают пересылку исключительно сетевых сообщений на основе открытых сокетов.
<p align="center"><img src="images/vpo_arch.jpg" width="600"/></p>
<p align="center"><em>Рисунок 3</em></p>


<a name="Tablo"/><h5>Табло</h5>
Табло - группа из 8 индикационных устройств, являющихся внешними абонентами по отношению к остальной части СМУ. Каждое Табло находится в одном из двух состояний: ОТКЛ (отключено) или ВКЛ (включено), а также в одном или комбинации из следующих режимов: RED, YELLOW, GREEN. Табло находится в состоянии включено, если оно перед этим получило команду на включение от ВПО. Табло находится в определенном режиме(-ах), если при получении от ВПО указан соответствующий режим(-ы).


<a name="VPI"/><h5>ВПИ</h5>
ВПИ (внешний пользовательский интерфейс) - программный компонент ЭВМ, предназначенный для отправки команд пользователя в ВПО и визуального отображения работы СМУ на экране. Взаимодействие с ВПО осуществляется на основе пересылки сетевых [сообщений](#Exchange) при помощи открытых сокетов. При помощи ВПИ возможно осуществление следующих функций:
1. Изменение состояния работы Устройства (только в режиме ВУ).
2. Получение состояния и режимов работы Устройства.
3. Изменение состояния и режимов работы Табло (только в режиме ВУ).
4. Получение состояния и режимов работы Табло.
5. Получение информации о состоянии работы компонентов программного обеспечения СМУ.


<a name="Panel"/><h5>Панель</h5>
Передняя Панель - часть Устройства, представляющая возможность непосредственного управления СМУ. Взаимодействие с ВПО осуществляется на основе открытия именованных каналов и записи команд/приема [сообщений](#Exchange) в них. При помощи ВПИ возможно осуществление следующих функций:
1. Изменение состояния работы Устройства (только в режиме ЛУ).
2. Изменение режима работы Устройства.
3. Получение состояния и режимов работы Устройства.
4. Изменение состояния и режимов работы Табло (только в режиме ЛУ).

[comment]: <> (возможно тут стоит добавить функцию: 5. Получение состояния и режимов работы Табло.)


<a name="Exchange"/><h4>Информационный обмен</h4>
Информационный обмен между всеми компонентами СМУ происходит при помощи пакетов, имеющих следующий формат:
|  №  | Название поля  | Размер (бит) | Назначение поля                                    |
| :-: | :------------: | :----------: | :--------------------------------------------------|
|  1  | Стартовые биты | 2            | 0b10                                               |
|  2  | Заголовок      | 4            | Число (id), определяющее тип сообщения (см. далее) |
|  3  | Данные         | 8*N          | Смысловая часть сообщения                          |
|  4  | Стоповые биты  | 2            | 0b01                                               |

Стоить отметить, что поле данных должно быть выравнено на целое число байт (N), таким образом пакет будет так же кратен целому числу байт и иметь размер N+1 байт = 8*(N+1) бит.

В составе каждого сообщения имеется ЭКТ (Элементарный Квант Табло), имеющий следующую структуру:
| № бита | Значение| Назначение        |
| :----: | :-----: | :---------------- |
| 0      |    0    | Табло ОТКЛ        |
| 0      |    1    | Табло ВКЛ         |
| 1      |    0    | Режим RED ОТКЛ    |
| 1      |    1    | Режим RED ВКЛ     |
| 2      |    0    | Режим YELLOW ОТКЛ |
| 2      |    1    | Режим YELLOW ВКЛ  |
| 3      |    0    | Режим GREEN ОТКЛ  |
| 3      |    1    | Режим GREEN ВКЛ   |

Примечание: в случае, если подается ЭКТ с командой отключить Табло и включить какой-либо из режимов, команда на включение режимов игнорируется.


<a name="VPI-VPO"/><h4>ВПИ-ВПО</h4>
В разработке


<a name="VPO-VPI"/><h4>ВПО-ВПИ</h4>
В разработке


<a name="VPO-Tablo"/><h4>ВПО-Табло</h4>
Данное сообщение предназначено для пересылки из ВПО в нужное Табло (устанавливается адрес от 0 до 7) команды по установлению/снятию состояния или режимов. В случае необходимости послать несколько сообщений на разные Табло, формируется последовательность сообщений, отправляемых получателю с интервалом в 1 мс (при этом бит 0 принимает значение "1" для всех сообщений последовательности, кроме последнего).

Заголовок = 0b0100  
Размер N = 1 байт  
Структура (здесь и далее "X" - произвольное значение):
| № бита | Значение| Назначение                          |
| :----: | :-----: | :---------------------------------- |
| 0      | 0       | Сообщения "ВПО-Табло" не ожидаются  |
| 0      | 1       | Ожидаются еще сообщения "ВПО-Табло" |
| 1-3    | XXX     | Адрес (порядковый номер) Табло      |
| 4-7    | XXXX    | ЭКТ                                 |


<a name="Tablo-VPO"/><h4>Табло-ВПО</h4>
Данное сообщение предназначено для пересылки из Табло в ВПО информации о состояниях и режимах всех Табло.

Заголовок = 0b1000  
Размер N = 4 байта
Структура:
| № бита | Значение| Назначение |
| :----: | :-----: | :--------- |
| 0-3    | XXXX    | ЭКТ        |
| 4-7    | XXXX    | ЭКТ        |
| 8-11   | XXXX    | ЭКТ        |
| 12-15  | XXXX    | ЭКТ        |
| 16-19  | XXXX    | ЭКТ        |
| 20-23  | XXXX    | ЭКТ        |
| 24-27  | XXXX    | ЭКТ        |
| 28-31  | XXXX    | ЭКТ        |

<a name="VPO-Panel"/><h4>ВПО-Панель</h4>
В разработке


<a name="Panel-VPO"/><h4>Панель-ВПО</h4>
В разработке


<a name="Requirements"/><h3>Требования</h3>
В разработке


<a name="Building"/><h3>Скачивание и сборка</h3>
В разработке


<a name="Usage"/><h3>Использование</h3>
В разработке


***
[comment]: <> (<p align="center"><a href="https://github.com/VasiliyMatlab"><img src="https://github.com/VasiliyMatlab.png" width="100" alt="VasiliyMatlab"/></a></p>)
<p align="center"><a href="https://github.com/VasiliyMatlab">VasiliyMatlab</a></p>
