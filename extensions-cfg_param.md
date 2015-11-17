# Настройка переменных конфигурации

Механизм Cotonti спроектирован таким образом, что позволяет при создании Расширения задать набор переменных, которые будут доступны для настройки через панель администрирования. Они отображаются на странице (Управление сайтом → расширения → *'имя расширения'* → Конфигурация). Таким образом разработчик может создавать более гибкие расширения.

# Формат задания переменных

Настройки переменных конфигурации загружаются из файла настроек Расширения (extension_name.setup.php) во время установки Расширения в панели администрирования. 

*Поэтому, если вы разрабатываете свое Расширение и изменили или добавили переменные в файл установки (setup файл), то вам необходимо произвести процедуру переустановки Расширения в Админ панели (Управление сайтом → Расширения → 'ваше расширение', кнопка `Обновить`).*

Для описания переменных конфигурации в установочном файле Расширения (extension_name.setup.php) должен быть размещен текстовый блок, выделенный следующими маркерами:

```
[BEGIN_COT_EXT_CONFIG]
...
[END_COT_EXT_CONFIG]
```
Внутри этого блока должно находится описание переменных настройки. Описание каждой переменной находится на отдельной строке и задается отдельными «полями» в следующем формате:

```
Variable=[Order]:[Type]:[Values]:[Default]:[Description]
```

Квадратные скобки указаны как знак того, что некоторые поля могут быть опущены при задании переменной. Знаки `:` и `=` обязательны.

Рассмотрим подробнее составляющие формата: 

* `Variable`: Имя переменной для доступа из програмного кода, без знака $.
* `Order`: Порядковый номер. Служит для контроля порядка отображения (сортировки) полей ввода на странице Конфигурации.
* `Type`: Тип переменной конфигурации. Подробнее о типах смотри следующий раздел «***Используемые типы переменных***».
* `Values`: Допустимые значения. Для переменных типа **`select`** and **`radio`** задается список значений разделенных запятыми. Для типа **`range`** в этом поле задаются минимальное и максимальное значение для задания диапазона возможных вариантов выбора. Для типа **`callback`** или **`custom`** здесь указывается имя функции обработчика (см. описание в следующем разделе). В остальных случаях поле остается пустым.
* `Default`: Значение по умолчанию. Для типов `select` и `radio` в качестве значения по умолчанию может быть только одно из тех, что указано в поле `Values`. Для остальных указывается значение (строковое), которое будет использовано как начальное, или восстановлено при нажатии кнопки `сброс` в панели Конфигурации. Значение по умолчанию может быть не задано (пустым).
* `Description`: Описание. Текст который будет отображен в панели Конфигурации для описания назначения данной настройки. [При использовании мультиязычных сайтов описания переменных могут быть локализованы (переведены) посредством языковых файлов. Подробнее об этом смотри в статье «***Локализация расширений***».]

Используемые типы переменных
============================


`string`
--------

Строка данных, максимальной длинной 255 символов.
отображается на странице настроек как обычное однострочное поле ввода.
Значение поля `Values` для этого типа не используется.

`select`
--------

Меню выбора. Отображается как стандартный раскрывающийся список. Элементы списка задаются в поле `Values` через запятую. Позволяет выбрать только один вариант из списка. 

`radio`
-------

Меню выбора в виде переключателя (или как их еще называют радио-кнопок) «да|нет». Используется для переменных, которые отображают параметр, с двумя возможными состояниями «вкл|выкл». Поле `Values` тут не используется. Значение по умолчанию может быть 0 или 1 (для состояний «Нет» и «Да» соответственно).

**Обновлено:** с версии `Сиена 0.9.19` поддерживается расширенный вариант использования, когда можно задать более двух пунктов с произвольными значениями. В этом случае в поле `Values` задаются значения списка (в точности аналогично типу `Select`).

`text`
------

Тип `text` используется для ввода текстовых данных без ограничения на длину строк (актуальный максимальный размер данных определяется настройками серверного ПО, например смотри параметр `post_max_size` из `php.ini` файла). 
Значение поля `Values` для этого типа не используется.
Надо также отметить, что тип `text` используется как тип по умолчанию в том случае, если актуальный тип не указан в описании переменной (или указан с ошибкой).

`callback`
----------

Тип `callback` отображается аналогично типу `select`, с той лишь разницей, что список элементов для выбора не указан жестко при описании, а генерируется заданной функцией. Имя такой функции указывается в поле `Values`. Для примера, если поле `Values` содержит запись `cot_get_parsers()`, то для формирования списка будет использован массив возвращаемый этой функцией. Подробнее об этом типе смотри в разделе «***Расширенные типы переменных***».

`hidden`
--------

Переменная аналогичная типу `String`, но скрытая от взгляда пользователя (не выводится на странице Конфигурации) и используемая программно из кода Расширения.

`separator`
-----------

Тип разделитель. По сути это не переменная, а описание визуального элемента в интерфейсе панели администрирования, который служит для отделения одной группы параметров от другой. Чтобы разделитель оказался в нужном месте списка настроек ему, как и остальным переменным надо правильно задать порядковый номер (в поле `Order`). Текст указанный в поле `Description` будет использован как заголовок при отображении разделителя.

`range`
-------

Создает меню выбора (в виде раскрывающегося списка) целых значений из диапазона заданного в настройках переменной. Границы диапазона (минимум и максимум) указываются в поле `Values` через запятую.

`custom`
--------

Тип `custom` добавлен в Cotonti в версии 0.9.18 (и дополнен в версии 0.9.19) для расширения ограниченного набора стандартных типов. Тип `custom` позволяет разработчику написать самостоятельный обработчик поля ввода данных, т.е. определить:
 * как оно будет выглядеть;
 * как будет обрабатывать введенные значения. 
 
Имя функции обработчика, как и в случае с типом `callback` задается в поле `Values`. Подробнее об этом типе смотри в разделе «***Расширенные типы переменных***».


### Примеры использования: ###

```
[BEGIN_COT_EXT_CONFIG]
myvar1=01:string::50:This is my first variable
myvar2=02:select:1,2,3,4,5,6,7,8,9,10:7:This is my second variable
myvar3=03:radio::1:This is my third variable
myvar4=04:text::Default text goes here:This is my fourth variable
myvar5=05:callback:cot_get_parsers():none:This is one of the available parsers
myvar6=06:hidden::123foo:This option is not visible for admins
myvar7=07:separator:::Separator is displayed here
myvar8=08:range:1,100:50:A selection from 100 numbers
myvar9=09:custom:custom_input_func(a,b,c):50:Custom function field 
[END_COT_EXT_CONFIG]
```

Теперь пользователь сможет изменять настройки вашего Расширения через панель администрирования. Доступ к указанным значениям в программном коде осуществляется через обращение к системному массиву `$cfg`.
Для плагина значения переменных доступны таким образом:

```php
$cfg['plugin']['имя_плагина']['myvar1']
$cfg['plugin']['имя_плагина']['myvar2']
...
```
где `имя_плагина` надо заменить на название вашего плагина.
Для модулей доступ к переменным несколько отличается:

```php
$cfg['имя_модуля']['myvar1']
```
где `имя_модуля` это название модуля.

Подробнее о некоторых типах
===========================

## Тип 'Select' ##
В описании переменной типа `select` в поле `Values` в качестве элементов списка могут быть указаны только простые строки. Эти строки будут использованы и как значения и как описание элемента списка при формировании кода. Т.е. строка описания `var=01:select:v1,v2,v3:v1:Description` будет преобразована примерно в следующий html код для использования на странице Конфигурации:
```html
<select name="var">
  <option value="v1" selected>v1</option>
  <option value="v2">v2</option>
  <option value="v3">v3</option>
</select>
```
В тех случаях, когда надо задать описания пунктов списка отличные от реальных значений (`v1`,`v2`,`v3`) можно воспользоваться типом `callback` (см. ниже) или  механизмом локализации переменных (об этом подробнее смотри статье [«Локализация Расширений»](docs/ext/lang/ext-l10n)).

## Тип 'Callback' ##

Визуально переменная типа `callback` на странице настроек отображается аналогично типу `select`, как выпадающий список, но перечень пунктов этого списка задается не в описании переменной, а формируется из массива данных, возвращаемого указанной функцией. Имя такой функции задается в описании (см. выше «*Формат задания переменных*»). Функция должна возвращать массив.

Для версий `Сиена 0.9.18 и ниже` допустим только «простой» нумерованный формат массива:
```php
array(
  'value1',
  'value2',
  'value3',
);
```
значения массива (value1, value2, value3) будут использованы и как значения пунктов списка и как их названия (см. пример HTML кода выше в описании типа 'Select').

Начиная с версии `Сиена 0.9.19` допустимо использовать ассоциативный массив
ключи которого будут использованы как значения элементов списка, а строки массива, как описание (названия) элементов списка. 
Т.е. массив
```php
array(
  'key1' => 'title1',
  'key2' => 'title2',
  'key3' => 'title3',
);
```
будет преобразован в следующий список:
```html
  <option value="key1">title1</option>
  <option value="key2">title2</option>
  <option value="key3">title3</option>
```
Не зависимо от версии может быть использован механизм локализации переменных настройки (см. ссылку на статью выше). Локализованные описания элементов списка имеют приоритет над заданными массивом.

Тип `callback` может потребоваться в случаях, когда элементы списка зависят от дополнительных условий (например настроек другого Расширения).
При описании переменной, функция может быть задана с параметрами. Эти параметры будут переданы ей при вызове.
```
myvar5=05:callback:cot_get_parsers(html):none:This is one of the available parsers
```

## Тип 'Custom' ##
Тип `custom` призван исправить проблему ограниченного числа стандартных типов т.к. позволяет написать код обработки и контролировать отображение поля ввода и фильтрацию введенных пользователем данных. Это почти не ограниченно расширяет возможности разработчика по взаимодействию с пользователем через панель администрирования.

Механизм работы переменных данного типа будет изложен в отдельной статье на примере конкретных задач («Тип 'custom' в переменных конфигурации»).

 **Внимание**: *Данный тип будет работать только в Cotonti начиная с версии `Сиена 0.9.19`. Если вы разрабатываете Расширение для более старых версий имейте это в виду.*


Локализация переменных 
======================

Подробно процесс локализации переменных конфигурации изложен на странице [«Локализация Расширений»](docs/ext/lang/ext-l10n). Здесь лишь отметим, что описания переменных и отображаемые в списках значения могут быть переведены на необходимые языки и быть отражены в зависимости от выбранного на сайте языка.

Переменные конфигурации для дерева категорий
============================================

Некоторые Расширения, а точнее *Модули* могут использовать такой, встроенный в Cotonti, механизм, как «структура (дерево) категорий». Как пример модули 'page' и 'forums' имеют категории для страниц и разделов форума. По умолчанию каждая, создаваемая в дереве категория, имеет предустановленный набор параметров (заголовок, описание и т.п.). Однако этот набор может быть программно расширен, путем добавления дополнительных переменных конфигурации, аналогично тому, как мы это можем делать для добавления параметров настройки Расширений. Точно так же описания этих переменных должно находится в установочном файле модуля (`module_name.setup.php`) в специальном блоке `COT_EXT_CONFIG_STRUCTURE`. Вот пример подобного блока из модуля 'page':

```
[BEGIN_COT_EXT_CONFIG_STRUCTURE]
order=01:callback:cot_page_config_order():title:
way=02:select:asc,desc:asc:
maxrowsperpage=03:string::30:
truncatetext=04:string::0:
allowemptytext=05:radio::0:
keywords=06:string:::
[END_COT_EXT_CONFIG_STRUCTURE]
```
Как можно увидеть формат описания переменных в точности повторяет таковой описанный для Расширений.

Настройки по умолчанию для вновь создаваемых категорий можно изменить на странице общих настроек Модуля (Управление сайтом → Расширения → *Имя модуля* → Конфигурация). Конечные настройки для каждой из категорий могут быть настроены следующим образом: Управление сайтом → Расширения → *Имя модуля* → кнопка `Структура`, далее напротив нужной категории кнопка `Конфиг`.