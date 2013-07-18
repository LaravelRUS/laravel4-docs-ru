# Installation

- [Установка Composer](#install-composer)
- [Установка Laravel](#install-laravel)
- [Загрузка архива](#server-requirements)
- [Настройка](#configuration)
- [Красивые URL](#pretty-urls)

<a name="install-composer"></a>
## Установка Composer

Laravel использует [Composer](http://getcomposer.org) для управления зависимостями. Для начала скачайте файл `composer.phar`. Дальше вы можете либо оставить этот Phar-архив в своей локальной папке с проектом, либо переместить его в `usr/local/bin`, чтобы использовать его в рамках всей системы. Для Windows вы можете использовать [официальный установщик](https://getcomposer.org/Composer-Setup.exe).

<a name="install-laravel"></a>
## Установка Laravel

### Создание проекта Composer

Вы можете установить Laravel с помощью команды `create-project`:

	composer create-project laravel/laravel

### Загрузка архива

Как только Composer установлен скачайте [последнюю версию фреймворка](https://github.com/laravel/laravel/archive/master.zip) и извлеките архив в папку на вашем сервере. Дальше, в корне вашего приложения на Laravel выполните `php composer.phar install` (или `composer install`) для установки всех зависимостей библиотеки. Этот процесс требует, чтобы на сервере был установлен Git.

Если вы хотите обновить Laravel выполните команду `php composer.phar update`.

<a name="server-requirements"></a>
## Требования к серверу

У Laravel всего несколько требований к вашему серверу:

- PHP >= 5.3.7
- MCrypt PHP Extension

<a name="configuration"></a>
## Настройка

Laravel практически не требует начальной настройки - вы можете сразу начинать разработку. Однако вам может пригодиться файл `app/config/app.php` и его документация - он содержит несколько настроек вроде `timezone` и `locale`, которые вам может потребоваться изменить в соответствии с нуждами вашего приложения.

> **Примечание:** Единственная настройка, которую вам нужно изменить `key` в файле `app/config/app.php`. Это значение должно быть случайной строкой длиной 32 символа. Этот ключ используется при шифровании и зашифрованные строки не будут безопасными, пока вы не измените эту настройку. Это можно быстро сделать с помощью следующей команды: `php artisan key:generate`.

<a name="permissions"></a>
### Права доступа
Laravel требует, чтобы у сервера были права на запись в папку `app/storage`.

<a name="paths"></a>
### Пути

Некоторые системные пути Laravel - настраиваемые; для этого обратитесь к файлу `bootstrap/paths.php`.

> **Примечание:** Laravel спроектирован так, чтобы защитить код вашего приложения и локальное хранилище - для этого общедоступные файлы помещаются в папку `public`. Рекомендуется, чтобы вы установили эту папку корневой папкой вашего сайта (DocumentRoot в Apache) или переместили её содержимое в корневую папку сайта, а все другие файлы фреймворка - за её пределы.

<a name="pretty-urls"></a>
## Красивые URL

Laravel поставляется вместе с файлом `public/.htaccess`, который настроен для обработки URL без указания `index.php`. Если вы используете Apache в качестве вёб-сервера обязательно включите модуль `mod_rewrite`.

Если стандартный `.htaccess` не работает для вашего Apache, попробуйте следующий:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]
