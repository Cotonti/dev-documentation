Использование Composer
======================

![php-composer](https://github.com/Cotonti/dev-documentation/assets/1021886/d32e7170-0270-44ac-bf12-668a52222942)

В версии Cotonti Siena 0.9.23 добавлена поддержка [Composer](https://getcomposer.org/) - менеджера пакетов для PHP.

## Что такое Composer

**Composer – это пакетный менеджер зависимостей**, предназначенный для упрощения загрузки и установки сторонних php библиотек в проект. 
Например, с его помощью очень легко добавить в разрабатываемое расширение или проект необходимые библиотеки.

**composer.json** - это текстовый файл, в котором в формате JSON описаны все сторонние библиотеки (пакеты) от которых зависит данный
проект. Кроме того он может содержать информацию о проекте, минимальную версию PHP, необходимую для работы проекта, необходимые 
расширения PHP.

**composer.lock** - содержит текущий список всех установленных зависимостей и их версии. Основное назначение этого файла
заключается в полном сохранении среды, в которой ведется разработка и тестирование проекта.  
Например если вы ведёте коллективную разработку, то ваш коллега скачав (pull) проект с гит-репозитория должен получить 
тоже окружение и версии всех пакетов, что и у вас.  
При деплое проекта в продакшен нужно на сервере получить те же самые версии пакетов, что и в dev среде. 
Это позволяет убедиться в том, что в любом месте, где разворачивается получает ваш проект, пакетное окружение будет 
идентично тому, которое вы использовали при разработке, и помогает избежать ошибок, которые могли бы возникнуть из-за
обновления версий.

## Использование Composer при разработке расширений.

Прежде всего у Вас должен быть [установлен сам Composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos).

Здесь мы добавим в создаваемое расширение Cotonti HTTP клиент [Guzzle](https://docs.guzzlephp.org), который позволяет отправлять как синхронные,
так и асинхронные запросы к удаленном серверам, используя один и тот же интерфейс. Отличный инструмент для интеграции с различными API.

А также мы будем использовать [Flysystem](https://flysystem.thephpleague.com) для загрузки файлов в удаленное хранилище.

Эти фичи будут использоваться только для примера и никак между собой не связаны.

Итак приступим.

Следуя инструкциям [по установке Guzzle](https://docs.guzzlephp.org/en/stable/overview.html#installation) добавим в файл **composer.json**
который находится в корне проекта в секцию `require`

```json
"guzzlehttp/guzzle": "^7.8"
```

Получится что то вроде этого:

```json
"require": {
    "php": ">=5.6",
    "ext-gd": "*",
    "ext-mbstring": "*",
    "ext-json": "*",
    "ext-hash": "*",
    "ext-pdo": "*",
    "guzzlehttp/guzzle": "^7.8"
},
```

Далее в командной строке выполнить:
```shell
> composer update
```
Эта команда обновит все зависимости установленные в проекте до последних версий (в соответствии с **composer.json**), 
установит новые зависимости, которые появились в файле **composer.json** и удалит те, которых в этом файле больше нет.
После этого Composer обновит файл **composer.lock**.

Теперь наш HTTP клиент готов к использованию.

В PHP-файл нашего расширения добавим

```php
<?php

use GuzzleHttp\Client;

// Создаем экземпляр клиента с базовым URI
$client = new Client(['base_uri' => 'https://foo.com/api/']);

// Отправим POST-запрос application/x-www-form-urlencoded на адрес https://foo.com/api/test
$response = $client->request('POST', '/test', [
  'form_params' => [
      'field_name' => 'abc',
      'other_field' => '123',
      'nested_field' => [
          'nested' => 'hello',
      ],
  ],
]);

$body = $response->getBody();
echo $body;
```

Просто, не так ли?


Теперь займемся загрузкой файлов на удаленный сервер:

Как и указано в [документации](https://flysystem.thephpleague.com/docs/getting-started/)  добавим в файл **composer.json**
в секцию `require`

```json
"league/flysystem": "^3.0",
"league/flysystem-aws-s3-v3": "^3.0"
```

Снова:
```shell
> composer update
```

Все готово. Можем использовать библиотеку в своем расширении.

```php
<?php

use Aws\S3\S3Client;
use League\Flysystem\AwsS3V3\AwsS3V3Adapter;
use League\Flysystem\Filesystem;

// For some reason AWS adapter works this way only
putenv('AWS_ACCESS_KEY_ID=my-access-key');
putenv('AWS_SECRET_ACCESS_KEY=my-secret-key');

/** @var S3ClientInterface $client */
$client = new S3Client([
    'version' => 'latest',
    'endpoint' => 'https://storage.yandexcloud.net',
    'region' => 'ru-central1',
]);

// The internal adapter
$adapter = new AwsS3V3Adapter(
    $client, // S3Client
    'test-new-bucket', // Bucket name
    'path/to/upload/dir' // path prefix
);


// FilesystemOperator готов и сконфигурирован. Можно использовать.
$fileSystem = new Filesystem($adapter);

// Запись в файл
$filesystem->write($path, $contents);
```

Вот и все. Не забудем добавить в инструкцию по установке нашего расширения необходимость добавить в **composer.json** 
строки:

```json
"guzzlehttp/guzzle": "^7.8",
"league/flysystem": "^3.0",
"league/flysystem-aws-s3-v3": "^3.0"
```
И выполнить **composer update**, чтобы пользователи знали как правильно установить наше расширение.


## Деплой проекта на продакшен сервер

Наш проект готов. Он находится под контролем версий Git. Пора перенести его на продакшен сервер.

У нас в папке **lib/vendor** появились новые вложенные папки с зависимостями, которые добавил туда Composer.

Добавим их в `.gitignore`. Нет необходимости добавлять их в репозиторий нашего проекта и увеличивать его
размер.

Коммитим изменения и отправляем их в репозиторий.

Заходим на сервер по SSH и переходим в папку с проектом.

Выполняем
```shell
> git pull
```

Потом 
```shell
> composer install
```
В отличие от `composer update` эта команда установит все зависимости, перечисленные в **composer.lock**
и именно те версии зависимостей, которые в нем указаны. Если этого файла нет в корне проекта, то эта
команда ведет себя аналогично `composer update` и использует **composer.json** для установки зависимостей.

Ну вот и все. Заходим через браузер на наш сервер и радуемся тому что получилось.
