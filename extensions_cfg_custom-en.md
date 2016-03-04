# 'custom' type in config variables

If you are not familiar with configuration variables for Extensions you can read some on [Configuration values](docs/ext/extensions/configvalues) page.
Here we going deeper to 'custom' type of it.

Prior to Siena `0.9.19` we can use only these types of config variables:
`string` (string input), `text` (text field), `radio` (radio switch — yes/no), `select` (drop-down select), `callback` (custom select list), `range` (integer range).
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
  
  Example: inputing integer in defined range of values, when user set value beyond permitted, function can alter input to set value as corresponding range edge.
  
  Example for emitting warning message:
  
  ```php
  cot_message('msg', 'warning', $var_name);
  ```

  * If user input not permitted and can not be corrected, function should emmit `error` type message:

  ```php
  cot_error('msg', $var_name);
  ```

In both cases the last argument to message function is *variable name*. It's required for `Display messages separately for each source` mode (see `Administration panel` → `Configuration` → `Themes` settings). 


## Creating custom input fields

With `custom` type variables it's possible to alter default input fields styling and behavior. In fact it's possible use custom HTML code for input field or even use several input fields for one config variable.
There are special function to be called while system is building configuration page. The name of function must match with name set in variable definition (with type `custom`):
```
mobile_num=01:custom:mobtel_input('+7')::Contact phone
```
so function can be define like this:
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

As a first argument in function we get array of variable definition data  (see description above). The last parameters will be that defined in «setup» file. 


## Example of use ##

### Password input fields ###

Let's try to do some custom fields relies on info from this page.

Assume we need to store password for third party service inside our extension, and we want to get standard password input field in configuration page (by «standard» means fields with masked symbols and second input to verify password type is correct).

First — define our setup file variable, that should store our password:
```
cloud_psw=01:custom:cfg_password(5)::Cloud server access password
```
Type of variable is set to `custom`. `cfg_password()` — is a function for generating input fields. As an argument we pass number of minimal allowed length of password (5 symbols).

To input we needs 2 input fields with type `password`. 
Input data will be get by array with keys `0` and `1`. 

```php
function cfg_password($cfg_var, $minlength = 4){
	if (!$minlength) $minlength = 4;
	$value = $cfg_var['config_value'];
	$var_name = $cfg_var['config_name'];
	$type = 'password'; // тип полей ввода
	// append length check as HTML5 do
	$attr = array('pattern' => '[^\s]{'.$minlength.',}');
	// generating HTML code to display two input fields
	$input_code = 
		cot_inputbox($type, $var_name.'[0]', $value, $attr) . // first field
		' <br>To change password — type it once more:<br>' .
		cot_inputbox($type, $var_name.'[1]', '', $attr); // second field
	return $input_code;
}
```
By using custom function for field generation we allow also to set any attributes for input field. 

As next step we need to set filtration function to process inputed data. 

```php
function cfg_password_filter(&$input_value, $cfg_var, $minlength = 4){
	if (!is_array($input_value)) return NULL;

	if ($input_value[0] == $input_value[1]) { 
		// if both inputed password matched
		// checking for minimal required length
		if ($input_value[0] && mb_strlen($input_value[1]) < $minlength) 
		{
			// if it too short — emit error
			cot_error('minimal length required: '.$minlength, $cfg_var['config_name']);
		}
		else
		{
			// in case of good password return it for further saving in DB
			return $input_value[1];
		}
	} else {
		// if password not matched emit corresponding error
		if ($input_value[0]) cot_error('Entered passwords not matched', $cfg_var['config_name']);
	}
	// in case of error we must set current password for input field
	$input_value = $cfg_var['config_value'];
	return NULL;
}
```
By `&$input_value` reference we get array with two elements that contains user entered passwords. We use reference here to allow change inputed data for default one (current password) in case of errors.

So, by defining two simple functions we get new custom type for configuration page. 
Making this functions more universal you can use it across different of your Extensions without any addition corrections.

### Default values ###

As you could been noted in «Filtering data with custom function» section, we not required to always set filtration and field generation functions. We can use built-in filters, and a default field for input (in this case it would be common text field to string input — `<input type="text">`).

This way we can use one of predefined filters but meka a custom field for input:
```
ui_color=01:custom:color_input_alp():Black:Select color, type name or HEX code
```
In this sample case we get `color_input_alp()` function call for field generation, but filtration will be done by system defined `ALP` filter (means `aplhanumeric` — filters all except alpha, digits, hyphen and underscore symbols).