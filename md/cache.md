# Кэш

- [Настройка](#configuration)
- [Использование кэша](#cache-usage)
- [Увеличение и уменьшение значений](#increments-and-decrements)
- [Группы элементов](#cache-sections)
- [Кэширование в базе данных](#database-cache)

<a name="configuration"></a>
## Настройка

Laravel предоставляет унифицированное API для различных систем кэширования. Настройки кэша содержатся в файле `app/config/cache.php`.Здесь вы можете указать драйвер, используемый по умолчанию внутри вашего приложения. Laravel изначально поддерживает многие популярные системы, такие как [Memcached](http://memcached.org) и [Redis](http://redis.io).

Этот файл также содержит множество других настроек, которые в нём же документированы, поэтому обязательно ознакомьтесь с ними. По умолчанию, Laravel настроен для использования драйвера `file, который хранит упакованные объекты кэша в файловой системе.  Для больших приложений рекомендуется использование систем кэширования в памяти - таких как Memcached или APC.

<a name="cache-usage"></a>
## Использование кэша

**Запись нового элемента в кэш:**

	Cache::put('key', 'value', $minutes);

**Запись элемента, если он не существует:**

	Cache::add('key', 'value', $minutes);

**Проверка существования элемента в кэше:**

	if (Cache::has('key'))
	{
		//
	}

**Чтение элемента из кэша:**

	$value = Cache::get('key');

**Чтение элемента или значения по умолчанию:**

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

**Запись элемента на постоянное хранение:**

	Cache::forever('key', 'value');

Иногда вам может быть нужно получить элемент из кэша или сохранить его там, если он не существует. Вы можете сделать это методом `Cache::remember`:

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

Вы также можете совместить `remember` и `forever`:

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

Обратите внимание, что все кэшируемые данные упаковываются (сериализуются), поэтому вы можете хранить любые типы.

**Удаление элемента из кэша:**

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## Увеличение и уменьшение значений

Все драйверы, кроме `file` и `database` , поддерживают операции `инкремента` и `декремента`.

**Увеличение числового значения:**

	Cache::increment('key');

	Cache::increment('key', $amount);

**Уменьшение числового значения:**

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-sections"></a>
## Группы элементов

> **Внимание:** группы не поддерживаются драйверами `file` и `database`.

Вы можете объединять элементы кэша в группы (секции), а затем сбрасывать всю группу целиком. Для доступа к группе используйте метод `section`.

**Обращение к группе элементов кэша:**

	Cache::section('people')->put('John', $john);

	Cache::section('people')->put('Anne', $anne);

Вы можете использовать обычные операции над элементами группы, такие как чтение, запись, `increment` и `decrement`.

**Чтение элемента группы:**

	$anne = Cache::section('people')->get('Anne');

Вы также можете удалить всю группу:

	Cache::section('people')->flush();

<a name="database-cache"></a>
## Кэширование в базе данных

Перед использовании драйвера `database` вам нужно создать таблицу для хранения элементов кэша. Ниже приведён пример её структуры `Schema`:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
