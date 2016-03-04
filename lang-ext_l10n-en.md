Extensions localization
=======================

Common rules
============

Localization files (or 'Lang files' for short )for Extensions located in Extension folder within 'lang' subfolder, so the root path to language file should be: 
* for Plugins: `plugins/plug_name/lang/plugin_name.*.lang.php`
* for Modules: `modules/module_name/lang/module_name.*.lang.php`
… where `*` sign should be replaced with 2-character language code.

Common rules for lang files are described in [this article](docs/ext/lang/localization).

Main language strings
=====================

Main language strings are these strings that you want to use in your Extension for page localization. 
The system use unified registry for lang strings and it's important to pay attention for defining string names. In other case you can accidentally rewrite some system strings. So there is recommended rule make lang string names with a prefix by name of your Extension. E.g. for 'comments' plugin this could be 'com_':

```php
$L['com_closed'] = 'Adding comments has been disabled for this item';
```
Actually it's a common PHP Array with key-string pairs. But you already know it, right?

Plural forms strings
====================

You can be familiar with `cot_declension` that simplifies string generation that contains some numerals for proper write plural forms of word.
As example: "1 page" / "2 pages".
One of the function parameter using string with plural forms of word or lang string reference.
Direct form of plural form definition:
```php
$text = 'You wrote '.cot_declension(12, 'page, pages');
// $text variable assigned with string "You wrote 12 pages"
```
or you can user Lang file definition:
```php
$Ls['pages'] = 'page, pages'; // in common case this defines in language file
$text = 'You wrote '.cot_declension(12, 'pages');
// $text variable assigned with string "You wrote 12 pages"
```
As you can notice we used `$Ls` variable, but not `$L` — it's a way to avoid mess in main Array and to avoid rewriting some system strings.

[There are more addition parameters for `cot_declension` function. Check the API for this [API](https://www.cotonti.com/reference/api%20-%20functions/package-functions.html#cot_declension%28%29)]


System data localization
========================

System string called that way as used by Cotonti system itself on some system pages (in Administration panel). Here it is:

```php
$L['info_name'] = ''; // Name of Extension
$L['info_desc'] = ''; // Extension description 
$L['info_notes'] = ''; // Addition notes
```
This variables can be used for both Extension types — Plugins and Modules. Addition notes (`info_notes`) can be viewed on Extension administration page (Administration panel → Extensions → *'selected extension'*).

One more variable that is used only in some Plugins:
```php
$L[$plugname . '_title']
// where plugname should be replaced for actual Plugin name
```
This string will be used only for `standalone` part of Plugin and only for system defined (default) plugin template.

If your Extension needs some administration within own page (called by `tools` hook), you can consider these two variables: `$adminhelp` and `$adminsubtitle`. First one is defined help message shown in bottom block of Extension administration page (Administration panel → Extensions → *'selected extension'* → Administration). The second one had used for HTML page title (e.g.: "My Personal File Space - Administration" ).
These variables are not defined by default and you need to initialize it yourself:
```php
$adminhelp = $L['userimages_help'];
$adminsubtitle = $L['userimages_title'];
```

Configuration variables localization
====================================

More over you can use Language file to localize Extension configuration variables used on config page (Administration panel → Extensions → *'selected extension'* → Configuration).

To get variable localized you must know it's name defined in `[BEGIN_COT_EXT_CONFIG]` block in the pluginname.setup.php (more details about configuration variable definition see «***Setting up Configuration variables***»).

Lets see config var definition:
```php
sort=01:select:ID,Title,Date:ID:Default sorting column for search results
```
…and it's localization:
```php
$L['cfg_sort'] = 'Default sorting column for tag search results';
$L['cfg_sort_params'] = 'Title: Title, Date: Date';
$L['cfg_sort_hint'] = 'by default used ascending sorting';
```

Title and description for Config setting can be defined for any config variable. List string localization is actual only for config var of type `select`.
Config localization strings should be placed in `$L` array with names as follows: 
```php
$L['cfg_'.$varname] // Description
$L['cfg_'.$varname.'_params'] // Values list localization 
$L['cfg_'.$varname.'_hint']  // hint for input field
```
… where `$varname` replaces by actual Variable name.

Localizing extra fields description
===================================

As a developer/administrator of your site you can add «Extra fields» to some data tables (Admin panel → Other → Extra fields). While adding Extra field configuration we can add default description. It will be accessed via `{XXX_EXTRAFIELDNAME_TITLE}` tag, where `XXX` — name of table for extra field, and `EXTRAFIELDNAME` — name of extra field itself. 
Inspite of Cotonti do not provides standard way to localize extra field description, in many cases it should use this variable for it: 
```php
$L['page_{extrafieldname}_title'] = 'Extra field description';
```
where `{extrafieldname}` — it's a name of an extra field.



Arrays / list localization
==========================

In some Lang files (mainly in old Extension) you can find Arrays used for List localization or for plural form list:
```php
$L['cfg_array1_params'] = array('Foo', 'Bar');
$L['cfg_array2_params'] = array('foo' => 'Foo', 'bar' => 'Bar');
$Ls['Guests'] = array('guest', 'guests');

```
**It's a outdated format** and **not recommended for use**.
Now we actively used "Transifex", a collaborative translation service ([more info on that](en/docs/ext/lang/transifex)), so **only string type can be used** in language strings. 
Let's see how we can convert it to actual form:
```php
$L['cfg_array1_params'] = 'Foo, Bar';
$L['cfg_array2_params'] = 'foo:Foo, bar:Bar';
$Ls['Guests'] = 'guest, guests');
```
Thus any Lists becomes a strings with comma separated values and Arrays becomes comma separated string but with 'Key:Value' pairs delimited with `:` sign.

Example language file
=====================

Here you can look at simplified version of Language file for `comments` plugin. Note the single blocks separated with comments for differen types of Lang strings:

```php
<?php
/**
 * English Language File for Comments Plugin
 *
 * @package Comments
 * @copyright (c) Cotonti Team
 * @license https://github.com/Cotonti/Cotonti/blob/master/License.txt
 */

defined('COT_CODE') or die('Wrong URL.');

/**
 * Plugin Config
 */

$L['cfg_order'] = 'Sorting order';
$L['cfg_order_hint'] = 'Chronological or most recent first';
$L['cfg_order_params'] = 'Chronological,Recent';

$L['info_desc'] = 'Comments system for Cotonti with API and integration with pages, lists, polls, RSS and other extensions';

/**
 * Plugin Body
 */

$L['comments_comment'] = 'Comment';
$L['comments_comments'] = 'Comments';

/**
 * cot_declension arrays
 */

$Ls['Comments'] = "comments,comment";

/**
 * Comedit
 */

$L['plu_title'] = 'Comment Editing';
```


Using localization data
=======================


Using in Resource strings
-------------------------

Resource string it's some kind of micro-template string where some special markers should by replaced with actual data.

Some Resource string definition (common used in ext_name.resources.php):
```php
$R['comments_code_pages_info'] = $L['Total'].': {$totalitems}, '.$L['comm_on_page'].': {$onpage}';
```
Using of Resource string:
```php
// `cot_rc` function returns string with actual array data
$stat_data = array(
	'totalitems'=> $totalitems, 
	'onpage' => $page_title
);
$stat_msg = cot_rc('comments_code_pages_info', $stat_data);
```

Using in program code
---------------------

As mentioned above you can use data from `$L`, `$Ls` (or other) arrays itself:
```php
$email_title = $L['plu_comlive'];
cot_error($L['com_commenttooshort'], 'comtext');
```
Same way we can assign Lang string to template tag:
```php
$t->assign(array(
		'COMMENTS_POSTER_TITLE' => $L['Poster']
));
```
Besides that, Language data can be used indirect way with some functions:
```php
// `cot_rc` funciton used data from $L['com_msg']
$message = cot_rc('com_msg', array('number'=> $com_count));

// `cot_declension` function used data from $Ls['Comments']
$text = $L['com_posted_comments'].' '.cot_declension($com_count, 'Comments');
```


Using in templates
------------------

In templates it can be accessed as [common PHP variables](docs/ext/themes/cotemplate_statements):
```
{PHP.L.lang_string_id}
``` 
where `lang_string_id` is a Key in Language array `$L`.

You can use proper plural forms right from your template:
```
<!-- BEGIN: SOME_BLOCK -->
You wrote {PHP.num_of_comments|cot_declension($this, 'Comments')}
<!-- END: SOME_BLOCK -->
```

This example assumes that `$num_of_comments` represents a number of user comments, and plural forms is defined as follows:
```php
$Ls['Comments'] = "комментарий,комментария,комментариев";
```

In this case we get «You wrote 5 comments» (for case `$num_of_comments == 5`).









