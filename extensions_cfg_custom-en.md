> *Note*: **This is working draft. Translation had not been complete.**
> 
# 'custom' type in config variables

If you are not familiar with configuration variables for Extensions you can read some on [Configuration values](docs/ext/extensions/configvalues) page.
Here we going deeper to 'custom' type of it.

Till Siena `0.9.19` we can use only these types of config variables:
`string` (string input), `text` (text field), `radio` (radio switch — yes/no), `select` (dropdown select), `callback` (custom select list), `range` (integer range).
And more 2 of special type — `hidden` (hidden value) and `separator` (visual delimiter of variable groups).

The bottle neck for developers these times was limited number of predefined types and lack of ability to filter user input values before saving in database.

As a long wait extension we introduce a `custom` type variables since `Siena 0.9.19`. This type intended to extend flexibility of development and customizing of data types as developer needs, allowing custom data filtering.

The new `custom` type allow to:

1. Filtering data with Cotonti internal filters;
2. Filtering data with custom function;
3. Create custom fields for configuration page.

Common format of `custom` type is identical to `callback` type definition:
```
var_name=01:custom:custom_function_name(param1,…):Default value:Description
```

Let's looking closer...

## Filtering data with built-in filters

If you are not novice to Cotonti development you must know `cot_import` function, allowing filter incoming user data with defined type (more info you can find [here](docs/devel/validation_messages)). 
Some of predefined filter types can be used to filer configuration data on Extension setting page.
All you need for that is to define (in extension setup file — ext_name.setup.php) `custom` type and specify filter type after it:
```
alpha_only=05:custom:function_name_ALP()::Alphanumeric input
```
where `ALP` is a predefined filter type used to filter data. 

You can use these predefined filters: `INT`, `BOL`, `PSW`, `ALP`, `TXT`, `NUM`.
If you confused with these types see link above. Developer can check filter internals [within `cot_import` function](https://github.com/Cotonti/Cotonti/blob/master/system/functions.php#L332).

If we had not need to use custom data field (see description below), so we can omit `function_name`:
```
alpha_only=05:custom:_ALP()::Alphanumeric input
```
**Note:** underscore sign and parentheses is required. Filter name is case insensitive. 

## Filtering data with custom function

If we had lack of predefined types of filters we can define own one. We can do it two ways:

### Defining User filter for `cot_import` function ###

Creating of user filters is described in [Section 5 of «Validation and Messages»](docs/devel/validation_messages#ch5) article. Format of use these kind of filter is same as described above: 
```
some_var=01:custom:_MYFILTER()::User data
```
where `MYFILTER` is a user defind filter name, registered in filer list:
```php
$cot_import_filters['MYFILTER'][] = 'myfilter_function';
```
Here `myfilter_function` name is a function name used to filter data.

The only limitation to use user defined filters is filter name (`MYFILTER`), that should not consist of underscore sign.
 
### User custom filtration function ###

Unlike user filters this functions is called directly (not with `cot_import`), that allow use addition parameters for function. 

To be used in filtration callback, the name of function should be defined with some rules — match variable name with `_filter` suffix. 
As example for mobile phone input we can define variable these way:
```
mobile_num=01:custom:mobtel_input('+7',10)::Contact phone
```
In this case `mobtel_input_filter()` function would be called (if defined) for filtration data.

Filtration function should return filtered data of NULL otherwise (in case filtration can not be done).

#### Callback parameters for filtration function ####

Common format of filter function looks like:
```php
function myCustomCfgVar_filter($input_value, $cfg_var, ...)
```
As a first argument `$input_value` we get user input value (the data should be filtered).
As a second argument function gets array of variable data `$cfg_var` (as used in `cot_config_add` function. [See function](https://github.com/Cotonti/Cotonti/blob/master/system/configuration.php#L108-L196)). Example of config variable data:
```php
array(
  'config_owner'    => 'plug',
  'config_cat'      => 'plug_name',
  'config_subcat'   => null,
  'config_order'    => '02',
  'config_name'     => 'mobile_num',
  'config_type'     => 8, // COT_CONFIG_TYPE_CUSTOM
  'config_value'    => '9119999999',
  'config_default'  => '',
  'config_variants' => 'mobtel_input()',
  'config_text'     => 'Contact phone',
  'config_donor'    => null
)
```

Next we get arguments defined in config variable — `'+7',10`. So filtration function definition can looks like this (for mobile number example): 
```php
function mobtel_input_filter($phone, $cfg_var, $prefix='', $length=null)
```
and rest of function like this:
```php
function mobtel_input_filter(&$phone, $cfg_var, $prefix='', $length=null) {
  $var_name = $cfg_var['config_name'];
  $filtered = preg_replace("/[^\d]/", '', $input_value);
  if ($length && strlen($filtered) != $length) 
  {
    $filtered = null;
    $phone = '';
  }
  return $filtered;  
``` 

By default, incorrect inputed data not saving in database, but left in field input for correction. In other case you can use call by reference and filtered value can be altered inside filtration function. 

#### Notification display ####

There should be user feed back to provide information about adoption of inputed value.
It can be done using standard notification functions `cot_error()` and  `cot_message()` inside filtration function.
But some rules should be complied: 
  * if user inputed value are changes by filter function warning  (тип `warning`).
  
  Пример: ввод целого числа в указанном диапазоне, когда пользователь ввел число выходящее за границы диапазона, функция может скорректировать ввод пользователя установив значение равное соответствующей границе диапазона.
  
  Пример вывода предупреждающего сообщения:
  
  ```php
  cot_message('msg', 'warning', $var_name);
  ```

  * если пользователь ввел недопустимое значение, которое не может быть скорректировано или отфильтровано, необходимо вывести сообщение с ошибкой (тип `error`):

  ```php
  cot_error('msg', $var_name);
  ```

В обоих случаях последним параметром в функцию вывода сообщения передается имя переменной. Это необходимо для отображения ошибок рядом с конкретным полем ввода (при включенной в настройках системы опции `Показывать сообщения отдельно для каждого источника`).


## Создание пользовательских полей ввода

С помощью типа `custom` так же можно контролировать внешний вид и функционал полей ввода. По сути разработчик имеет возможность полностью переопределить HTML код поля ввода и даже создать для одного параметра несколько отдельных полей ввода.
За формирование кода элемента отвечает отдельная функция, вызываемая при создании страницы настроек. Имя этой функции должно соответствовать имени указанном в определении `custom` переменной:
```
mobile_num=01:custom:mobtel_input('+7',11)::Contact phone
```
тогда функция может выглядеть так:
```php
function mobtel_input($cfg_var, $prefix=''){
	$name = $cfg_var['config_name'];
	$value = $cfg_var['config_value'];
	if ($value)
	{
		$mobile_code = substr($value, 0, 3);
		$mobile_num =  substr($value, 3, 7);
		$formatted = $prefix . ' (' .$mobile_code. ') ' . $mobile_num;
	}
	return cot_inputbox('text', $name, $formatted, array('placeholder' => $prefix.' (000) 0000000' ));
}
```

Первым параметром в функцию передается массив данных самой переменной конфигурации (описание см. выше). Следом будут переданы все параметры указанные в описании переменной в «setup» файле Расширения. 


## Пример использования ##

### Поле ввода пароля ###

Попробуем применить полученные в предыдущих разделах знания. 

Допустим для нашего Расширения нам надо хранить пароль от стороннего сервиса, и мы хотим реализовать «стандартное» поле для ввода пароля (с сокрытием ввода и дублирующим полем для подтверждения правильности ввода).

Первое — определим нашу переменную файла конфигурации, в которой будет храниться значение пароля (например для доступа к стороннему облачному сервису):
```
cloud_psw=01:custom:cfg_password(5)::Test password input
```
Тип переменной, естественно, `custom`. `cfg_password()` — функция для вывода поля ввода. В качестве переменной этой функции будем передавать минимально допустимую длину пароля (5 символов).

Для формирования элемента ввода пароля нам потребуется 2 поля типа `password`. 
Данные из полей будем получать в виде массива с ключами 0 и 1. 

```php
function cfg_password($cfg_var, $minlength = 4){
	if (!$minlength) $minlength = 4;
	$value = $cfg_var['config_value'];
	$var_name = $cfg_var['config_name'];
	$type = 'password'; // тип полей ввода
	// дополнительно установим проверку длины средствами HTML5
	$attr = array('pattern' => '[^\s]{'.$minlength.',}');
	// создаем HTML код для вывода 2-х полей
	$input_code = 
		cot_inputbox($type, $var_name.'[0]', $value, $attr) . // первое поле
		' <br>Для смены пароля введите его повторно:<br>' .
		cot_inputbox($type, $var_name.'[1]', '', $attr); // второе поле
	return $input_code;
}
```
Написание своей функции вывода поля ввода, в том числе, позволяет нам гибко настраивать любые атрибуты. 

Далее приступим к функции фильтрации введенного пароля (т.е. проверки на соответствие заданным условиям). 

```php
function cfg_password_filter(&$input_value, $cfg_var, $minlength = 4){
	if (!is_array($input_value)) return NULL;

	if ($input_value[0] == $input_value[1]) { 
		// если пароли в обоих полях совпали
		// проверяем соответствие минимальной длине
			if ($input_value[0] && mb_strlen($input_value[1]) < $minlength) {
			// если длина не удовлетворяет — выводим ошибку стандартными средствами
			cot_error('Минимальная длина: '.$minlength, $cfg_var['config_name']);
		}
		else
		{
			// в случае успеха возвращаем значения пароля 
			// для последующего сохранения в БД
			return $input_value[1];
		}
	} else {
		// если пользователь ошибся при вводе выводим сообщение
		if ($input_value[0]) cot_error('Введенные пароли не совпадают', $cfg_var['config_name']);
	}
	// в случае ошибки мы должны вывести в поле текущий пароль
	$input_value = $cfg_var['config_value'];
	return NULL;
}
```
Через ссылку на `&$input_value` мы получаем массив с двумя элементами, в котором содержится данные введенные пользователем в соответствующие поля. Передача параметра по ссылке здесь необходима, чтобы в случае некорректного ввода переписать входящие данне значением текущего пароля, который будет отражен в поле после перегрузки страницы и вывода ошибки.

Вот так, написав 2 небольшие функции, мы получили совершенно новый пользовательский тип для переменной страницы настроек. 
Сделав данные функции максимально универсальными вы сможете применять их в различных своих Расширениях без каких-либо изменений.

### Данные по умолчанию ###

Как вы могли заметить из раздела «Фильтрация данных через встроенные фильтры», при использовании типа `custom` не обязательно всегда писать свой фильтр-обработчик и функцию вывода поля. Для фильтрации может быть использован встроенный фильтр, а для вывода поля по умолчанию (если пользовательская функция не определена) будет использовано обычное «строковое» поле ввода (`<input type="text">`).

Таким образом, мы можем, например, использовать встроенный фильтр, но создать для него собственное поле ввода:
```
myvar=01:custom:color_input_alp():Black:Select color, type name or HEX code
```
В этом случае, для создания поля ввода будет вызвана функция `color_input_alp()`б а обработка введенного значения будет произведена «встроенным» фильтром `ALP` (`aplhanumeric` — удаляет все символы, кроме символов латинского алфавита, цифр, дефиса и символа подчеркивания).