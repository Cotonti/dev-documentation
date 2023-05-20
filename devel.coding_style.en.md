Cotonti CMF Code Style
======================

The following code style is used for Cotonti CMF core and official extensions development. 
If you want to pull-request code into the core, consider using it. We aren't forcing you 
to use this code style for your application based on Cotonti. 
Feel free to choose what suits you better.

## 1. General rules

This common rules consider all source code files in the project (HTML, PHP, CSS, JS, etc.) if other is not defined.

- Code files MUST use only UTF-8 encoding without BOM.
- A new line SHOULD be LF.
- Indentation SHOULD be 4 spaces for indenting, not tabs. _(todo discussing...)_
- No extra spaces at the end of lines (set up your text editor, so that it removes extra spaces when saving).

There is not a hard limit on line length. Each line of text in your code should be at most 120 characters long.  
Put the line break wherever it makes the most aesthetic sense, not necessarily the breaking
point closest to 120 characters.

## 2. PHP guidelines

### 2.1 A brief view

…in addition to general rules

PHP code should follow the [PSR-12](https://www.php-fig.org/psr/psr-12/), and the
[PSR-1](https://www.php-fig.org/psr/psr-1/) coding standards and the [PSR-4](https://www.php-fig.org/psr/psr-4/)
autoloading standard.

If you are using PhpStorm, it can help you to maintain the required code style. You can turn it on in
`Settings` → `Editor` → `Code Style` → `PHP` → `Set from` → `PSR12`.

- PHP code MUST use either `<?php` or `<?=` opening tags and MUST NOT use the other tag variations such as `<?`.
- In case file contains PHP only it should not have trailing `?>`.
- Use one space around all operators, except for increment/decrement, e.g. `$a + $b`, `$string = $foo . 'bar'`, `$i++` 
not `$a+$b`.
- Use one space after comma `$one, $two`, but not `$one,$two`.
- Use strict equality `===` (inequality `!==` ).
- One-line comments should be started with `//` and not `#`.
- Avoid using global variables. Don't use undefined variables.

### 2.2 Documentation
- Refer to [PHPDoc](https://phpdoc.org/) for documentation syntax.
- Code without documentation is not allowed.
- All class files must contain a "file-level" docblock at the top of each file and a "class-level" docblock immediately
above each class.

#### 2.2.1 File Header
All PHP files should contain some documentation info formatted as [PHPDoc](https://phpdoc.org/).

It is very good when the header describes the variables that are used in this file, but are defined somewhere else.

Standard header for new files (this template of the header must be included at the start of all Cotonti files):
```php
/**
 * @package {PACKAGENAME}
 * @version {VERSION}
 * @copyright (c) 2008-2023 Cotonti Team
 * @license BSD License
 * 
 * @var array<string, mixed> $user
 * @var XTemplate $t
 */
````
`@version` is optional. For example, if you are developing some extension, you do not need to define `@version` tag as
version info already set in extension setup file and may confuse other developers.

#### 2.2.2 Classes
Do not forget to comment the class. Classes need a separate @package definition, it is the same as the header package
name.

```php
/**
 * Cotonti MySQL Database adapter class.
 * A compact extension to standard PHP PDO class with slight Cotonti-specific needs,
 * handy functions and query builder.
 * 
 * @package cotonti
 * @see http://www.php.net/manual/en/class.pdo.php
 *
 * @property-read int $affectedRows Number of rows affected by the most recent query
 * @property-read int $count Total query count
 * @property-read int $timeCount Total query execution time
 */
class MySQL extends Adapter
```

#### 2.2.3 Methods and functions 
Do not forget to comment the functions. Each function should have at least a comment of what this function does. Also, 
it is recommended to document the parameters too. Use `@param`, `@return`, `@throws` in that order.
```php
/**
 * Returns total number of records contained in a table
 * @param string $tableName Table name
 * @return int
 * @throws Exception if the table name is exists
 */
public function countRows($tableName)
```

### 2.3 Class, Function and Variable naming
All names must be descriptive, but concise. Seeing the name of a class or variable, it should be clear what they are
needed for. For example: `$CurrentUser`, `getUserData($id)`.

**Class names** MUST be declared in `PascalCase`. For example: `Controller`, `Model`.

**Constants** MUST be declared in all upper case with underscore separators. For example: `STATUS_PUBLISHED`.

**Directory/namespace names** in lower case

We DO NOT USE underscores to denote class private properties and methods.

#### 2.3.1 Variable naming
Variable names MUST be in **camelCase**. Each word, except the first one, must begin with a capital letter. 
For example: `$currentUser` is correct, but `$current_user` and `$currentuser` are not

Situations when one-character variable names are allowed: language strings: `$L`, string resources: `$R`, and when the
variable is an index for some looping construct.

**Loop Indices:** \
In this case, the index of the outer loop should always be `$i`. If there's a loop inside that loop, its index should 
be `$j`, followed by $k, and so on. If the loop is being indexed by some already-existing variable with a meaningful
name, this guideline does not apply, example:
```php
for ($i = 0; $i < $outerSize; $i++) {
   for ($j = 0; $j < $innerSize; $j++) {
      foo($i, $j);
   }
}
```

#### 2.3.2 Functions and methods names
Functions and methods names MUST be in **camelCase**. The exception is Cotonti core functions. Good function names are
`printLoginStatus()`, `getUserData()`, etc.

In Cotonti we are using standard `cot_` prefix for any function to avoid naming conflicts. It is standard for the
core API to be reusable: `cot_mailPrepareAddress($address)`

But you are not forced for `cot_` prefix in your plugins, unless those functions are going to
be reused as the core API. You can use your own prefix within your extension API.

**Function Arguments** are subject to the same guidelines as variable names.

#### 2.3.3 Summary
The basic philosophy here is to not hurt code clarity for the sake of laziness. This has to be balanced by a little bit
of common sense, though; `printLoginStatusForAGivenUser()` goes too far, for example - that function would be
better named `printUserLoginStatus()`, or just `printLoginStatus()`.

### 2.4 Types
All PHP types and values should be used lowercase. That includes `true`, `false`, `null` and `array`.

We use operators for type casting, not functions. And only their short forms. Put one space after the operator.
```php
$value1 = (int) $argument1;     // right
$value2 = (bool) $argument2;    // right

$value3 = (integer) $argument3;  // wrong
$value4 = invtal($argument2, 8); // only if it is really necessary
```

Changing type of an existing variable is considered as a bad practice. Try not to write such code unless it is really
necessary.
```php
public function save(Transaction $transaction, $argument2 = 100)
{
    $transaction = new Connection; // bad
    $argument2 = 200; // good
}
```

### 2.5 Strings

If string doesn't contain variables or single quotes, use single quotes.
```php
$str = 'Like this.';
```
If string contains single quotes you can use double quotes to avoid extra escaping.

#### Variable substitution
```php
$str1 = "Hello $username!";
$str2 = "Hello {$username}!";
```
The following is not permitted:
```php
$str3 = "Hello ${username}!";
```

#### Concatenation
Add spaces around dot when concatenating strings:
```php
$name = 'Cotonti' . ' CMF';
```

When string is long format is the following:
```php
$sql = 'SELECT * '
. 'FROM post '
. 'WHERE id = 121 ';
```

### 2.6 Arrays
For arrays we're using short array syntax.

```php
$arr = [3, 14, 15, 'Cotonti', 'CMF'];
```

If there are too many elements for a single line, each element is located on a separate line. After the last element we
put a comma.
```php
$mail = [
    'to'  => 'user@domain.com',
    'from' => ['admin@site.com', 'SiteTitle'],
    'cc' => [['user2@example.com', 'User2'], 'user3@example.com', 'User4 <user4@example.com>'],
];
```

### 2.7 Control Structures
- Control statement condition MUST have single space before and after parenthesis.
- Operators inside of parenthesis SHOULD be separated by one space.
- Opening brace MUST be on the same line.
- The closing brace MUST be on the next line after the body.
- Always use braces for single line statements.

#### if, elseif, else

```php
if ($expr1) {
    // if body
} elseif ($expr2) {
    // elseif body
} else {
    // else body;
}

// The following is NOT allowed:
if (!$model && null === $event) throw new Exception('test');
```
The keyword `elseif` SHOULD be used instead of `else if` so that all control keywords look like single words.

Expressions in parentheses MAY be split across multiple lines, where each subsequent line is indented at least once. 
When doing so, the first condition MUST be on the next line. The closing parenthesis and opening brace MUST be placed 
together on their own line with one space between them. Boolean operators between conditions MUST always be at the 
beginning of the line.
```php
if (
    $expr1
    && $expr2
) {
    // if body
}
```

Prefer avoiding `else` after `return` where it makes sense:
```php
$result = $this->getResult();
if (empty($result)) {
    return false;
} else {
    // process result
}
```
is better as:
```php
$result = $this->getResult();
if (empty($result)) {
    return false;
}

// process result
```

#### switch, case
A `switch` structure looks like the following. Note the placement of parentheses, spaces, and braces.
```php
switch ($expr) {
    case 0:
        echo 'First case, with a break';
        break;
    case 1:
        echo 'Second case, which falls through';
        // no break
    case 2:
    case 3:
    case 4:
        echo 'Third case, return instead of break';
        return;
    default:
        echo 'Default case';
        break;
}
```

#### Ternary operator
Ternary operators should only be used to do very simple things. Preferably, they will only be used to do assignments,
and not for function calls or anything complex at all. They can be harmful to readability if used incorrectly, 
so don't fall in love with saving typing by using them, examples:
```php
// Bad place to use them
($i < $size && $j > $size) ? do_stuff($foo) : do_stuff($bar);

// OK place to use them
$min = ($i < $j) ? $i : $j;
```

### 2.8 Classes

The term "class" refers to all classes, interfaces, and traits.

- The opening brace for the class MUST go on its own line; the closing brace for the class MUST go on the next line
after the body.
- Classes SHOULD be named using `PascalCase`.
- Every class MUST have a documentation block that conforms to the [PHPDoc](https://phpdoc.org/)
- There should be only one class in a single PHP file.
- All classes should be namespaced.
- Visibility MUST be declared on all properties and methods.
- The `extends` and `implements` keywords MUST be declared on the same line as the class name. Lists of implements and,
in the case of interfaces, extends MAY be split across multiple lines, where each subsequent line is indented once.
When doing so, the first item in the list MUST be on the next line, and there MUST be only one interface per line.

#### Properties
Public and protected variables SHOULD be declared at the top of the class before any method declarations. 
Private variables should also be declared at the top of the class but may be added right before the methods that are
dealing with them in cases where they are only related to a small subset of the class methods.

```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements \ArrayAccess, \Countable
{
    public $publicProp1;
    public $publicProp2;

    protected $protectedProp;

    private $_privateProp;


    public function someMethod()
    {
        // ...
    }
}
```

### 2.9 Methods and Functions
Opening brace of a function SHOULD be on the line after the function declaration, and the closing brace MUST go on the 
next line following the body.

In the argument list there MUST be one space after each comma.

```php
function foo(int $arg1, &$arg2, $arg3 = [])
{
    // function body
}
```

Argument lists MAY be split across multiple lines, where each subsequent line is indented once. When doing so, the 
first item in the list MUST be on the next line, and there MUST be only one argument per line.

```php
public function aVeryLongMethodName(
        ClassTypeHint $arg1,
        &$arg2,
        array $arg3 = []
    ) {
        // method body
    }
```

When making a method or function call, there MUST NOT be a space between the method or function name and the opening 
parenthesis, there MUST NOT be a space after the opening parenthesis, and there MUST NOT be a space before the closing 
parenthesis. In the argument list, there MUST NOT be a space before each comma, and there MUST be one space after each
comma.

## 3. SQL guidelines

SQL Statements are often unreadable without some formatting, since they tend to be big at times.
Though the formatting of sql statements adds a lot to the readability of code. SQL statements should 
be formatted in the following way, basically writing keywords:
```php
$sql = 'SELECT p.*, u.* ' 
    . 'FROM ' . Cot::$db->pages . ' AS p ' .
    . 'LEFT JOIN ' . Cot::$db->pages . ' AS u ON u.user_id = p.page_ownerid '
    . "WHERE p.page_ownerid = 1 AND p.page_cat = 'news' " 
    . 'LIMIT 10';
```

Use parameters to safely substitute user-supplied data into the query (even if you are sure the variable cannot contain
single quotes - never trust your input):
```php
$result = Cot::$db->query(
    'SELECT * FROM ' . Cot::$db->forum_topics . ' WHERE ft_cat = :cat  AND ft_id = :topicId',
    ['cat' => $section, 'topicId' => $param]
);
```

#### Avoid DB specific SQL
- The `not equals` operator, as defined by the [SQL-92 standard](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)
, is "<>". `!=` is also supported by most databases, but it's better to use `<>`.
- `INSERT ... ON DUPLICATE KEY UPDATE` - is a MySQL specific extension to SQL.
- Backticks `` ` `` for table and column names are MySQL specific. Better use
`CotDB::quoteTableName()` and `CotDB::quoteColumnName()` or abbreviations of these methods: `CotDB::quoteT()`,
`CotDB::quoteC()`

## 4. Javascript guidelines
- Use [JSDoc](https://jsdoc.app/) for classes, member variables, and methods
- The documenting, commenting and naming rules are applicable from point **2. PHP**
- Don't use [jQuery](https://jquery.com /) where you can do without it.
- Use `let` rather than `var` to declare variables and `const` to declare constants.
- Constants whose values are always hard-coded throughout the program are named in uppercase with
an underscore as a word separator. For example: `const COLOR_ORANGE = "#ffa500"`.
Most variables are constants in a different sense: they don't change value after assignment. But with different
function launches their value may vary. Such variables should use `const` and **camelCase** in the name.
- Semicolons are always placed at the end of the line.
- Always use strict equality `===` (inequality `!==`).

### 4.2 Strings
- use single quotes: `let single = 'single-quoted';`.
- If string contains single quotes you can use double quotes to avoid extra escaping.
- For expressions substitute use backticks: ``alert(`1 + 2 = ${sum(1, 2)}.`); // 1 + 2 = 3.``.

## 5. Helpful links and tools
- [PHPStorm Quality Tools](https://www.jetbrains.com/help/phpstorm/php-quality-tools.html)
- [Changelog files rules](https://keepachangelog.com) — useful how-to maintain CHANGELOG files
