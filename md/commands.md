# Команды Artisan

- [Введение](#introduction)
- [Создание команды](#building-a-command)
- [Регистрация команд](#registering-commands)
- [Вызов других команд](#calling-other-commands)

<a name="introduction"></a>
## Введение

В дополнение к стандартным командам Artisan вы можете также добавлять свои собственные команды для работы с приложением. Вы можете поместить их в папку `app/commands`, либо в любое другое место, в котором их смжет найти автозагрузчик в соответствии с вашим файлов `composer.json`.

<a name="building-a-command"></a>
## Создание команды

### Создание класса

Для создания новой команды можно использовать команду `command:make`, которая создаст заглушку, с которой вы можете начать работать.

**Генерация нового класса команды:**

	php artisan command:make FooCommand

По умолчанию сгенерированные команды помещаются в папку `app/commands`, однако вы можете указать произвольное расположение или пространство имён:

	php artisan command:make FooCommand --path=app/classes --namespace=Classes

### Написание команды

Когда вы сгенерировали класс команды, вам нужно заполнить его свойства `name` и `description` , которые используются при отображении вашей команды в списке команд `list`.

Метод `fire` будет вызван при вызове вашей команды. В него вы можете поместить любую нужную логику.

### Параметры и ключи

Методы `getArguments` и `getOptions` служат для определения параметров и ключей, которые ваша команда принимает на вход. Оба эти метода могут возвращать массив команд, которые описываются как массив ключей.

При определении `arguments` массив имеет такую форму:

	array($name, $mode, $description, $defaultValue)

Параметр `mode` может быть объектом `InputArgument::REQUIRED` или `InputArgument::OPTIONAL`.

При определении `options`, массив выглядит так:

	array($name, $shortcut, $mode, $description, $defaultValue)

`mode` для ключей может быть любым из этих объектов: `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`.

Режим `VALUE_IS_ARRAY` указывает, что ключ может быть передан несколько раз:

	php artisan foo --option=bar --option=baz

Режим `VALUE_NONE`  указывает, что ключ является простым переключателем:

	php artisan foo --option

### Чтение ввода

Во время выполнения команды вам, конечно, потребуется доступ к параметрам и ключам, которые были переданы ей на вход. Для этого вы можете использовать методы  `argument` и `option`.

**Чтение параметра команды:**

	$value = $this->argument('name');

**Чтение всех параметров:**

	$arguments = $this->argument();

**Чтение ключа команды:**

	$value = $this->option('name');

**Чтение всех ключей:**

	$options = $this->option();

### Вывод

Для вывода сообщений вы можете использовать методы `info`, `comment`, `question` и `error`. Каждый из них будет использовать подходящие цвета ANSI при отображении текста.

**Вывод сообщения в консоль:**

	$this->info('Показать это на экране');

**Вывод сообщения об ошибке:**

	$this->error('Что-то пошло не так!');

### Запрос ввода

Вы можете также использовать методы `ask` и `confirm` для получение ввода от пользователя.

**Запрос ввода от пользователя:**

	$name = $this->ask('Как вас зовут?');

**Запрос скрытого ввода:**

	$password = $this->secret('Какой пароль?');

**Запрос подтверждения:**

	if ($this->confirm('Вы хотите продолжить (да/нет)?'))
	{
		//
	}

Вы можете также передать значение по умолчанию в метод `confirm`, которое должно быть либо `true`, либо `false`:

	$this->confirm($question, true);

<a name="registering-commands"></a>
## Регистрация команд

Как только ваша команда написана вам нужно зарегистрировать её в Artisan, чтобы она стала доступна для использования. Это обычно делается в файле `app/start/artisan.php`. Внутри него вы можете использовать метод `Artisan::add` для регистрации команд.

**Регистрация команды Artisan:**

	Artisan::add(new CustomCommand);

Если ваша команда зарегистрирована внутри [контейнера IoC](/docs/ioc), то вы можете использовать метод `Artisan::resolve`, чтобы сделать её доступной для Artisan.

**Регистрация команды, находящейся в IoC:**

	Artisan::resolve('binding.name');

<a name="calling-other-commands"></a>
## Вызов других команд

Иногда вам потребуется вызвать другую команду изнутри вашей. Вы можете сделать это методом `call`.

**Calling Another Command**

	$this->call('command.name', array('argument' => 'foo', '--option' => 'bar'));