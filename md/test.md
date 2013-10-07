# Юнит-тесты

- [Введение](#introduction)
- [Определение тестов для выполнения](#defining-and-running-tests)
- [Тестовое окружение](#test-environment)
- [Обращение к маршрутам](#calling-routes-from-tests)
- [Тестирование фасадов](#mocking-facades)
- [Проверки (assertions)](#framework-assertions)
- [Вспомогательные методы](#helper-methods)

<a name="introduction"></a>
## Введение

Laravel построен с учётом тестирования. Фактически, поддержка PHPUnit доступна по умолчанию, а файл `phpunit.xml` уже настроен для вашего приложения. В дополнение к PHPUnit Laravel также использует компоненты Symfony HttpKernel, DomCrawler и BrowserKit для проверки и манипулирования шаблонами для симуляции работы браузера.

Папка `app/tests` уже содержит файл теста для примера. После установки нового приложения Laravel просто выполните команду `phpunit` для запуска процесса тестирования.

<a name="defining-and-running-tests"></a>
## Определение тестов для выполнения

Для создания теста просто создайте новый файл в папке `app/tests`. Класс теста должен наследовать класс `TestCase`. Вы можете объявлять методы тестов как вы обычно объявляете их для PHPUnit.

**Пример тестового класса:**

	class FooTest extends TestCase {

		public function testSomethingIsTrue()
		{
			$this->assertTrue(true);
		}

	}

Вы можете запустить все тесты в вашем приложении через командную строку командой `phpunit`.

> **Внимание:** если вы определили собственный метод `setUp`, не забудьте вызвать `parent::setUp`.

<a name="test-environment"></a>
## Тестовое окружение

Во время выполнения тестов Laravel автоматически установит текущую среду в `testing`. Кроме этого Laravel подключит настройки тестовой среды для сессии (`session`) и кэширования (`cache`). Оба эти драйвера устанавливаются в `array`, что позволяет данным существовать в памяти, пока работают тесты. Вы можете свободно создать любое другое тестовое окружение по необходимости.

<a name="calling-routes-from-tests"></a>
## Обращение к маршрутам

You may easily call one of your routes for a test using the `call` method:

**Вызов routing маршрута из теста:**

	$response = $this->call('GET', 'user/profile');

	$response = $this->call($method, $uri, $parameters, $files, $server, $content);

После этого вы можете обращаться к свойствам объекта `Illuminate\Http\Response`:

	$this->assertEquals('Hello World', $response->getContent());

You may also call a controller from a test:

**Вызов контроллера из теста:**

	$response = $this->action('GET', 'HomeController@index');

	$response = $this->action('GET', 'UserController@profile', array('user' => 1));

Метод `getContent` вернёт содержимое-строку ответа маршрута или контроллера. Если был возвращён `View` вы можете получить его через свойство `original`:

	$view = $response->original;

	$this->assertEquals('John', $view['name']);

Для вызова HTTPS-маршрута можно использовать метод `callSecure`:

	$response = $this->callSecure('GET', 'foo/bar');

> **Внимание:** фильтры маршрутов отключены в тестовой среде. Для их включения добавьте в тест вызов `Route::enableFilters()`.

### DOM Crawler

Вы можете вызвать ваш маршрут и получить объект `DomCrawler`, который может использоваться для проверки содержимого ответа:

	$crawler = $this->client->request('GET', '/');

	$this->assertTrue($this->client->getResponse()->isOk());

	$this->assertCount(1, $crawler->filter('h1:contains("Hello World!")'));

Для более подробной информации о его использовании обратитесь к [официальной документации](http://symfony.com/doc/master/components/dom_crawler.html).

<a name="mocking-facades"></a>
## Тестирование фасадов

При тестировании вам может потребоваться отловить вызов к одному из статических классов-фасадов Laravel. К примеру, у вас есть такой контроллер:

	public function getIndex()
	{
		Event::fire('foo', array('name' => 'Дейл'));

		return 'All done!';
	}

Вы можете отловить обращение к `Event` с помощью метода `shouldReceive` этого фасада, который вернёт объект [Mockery](https://github.com/padraic/mockery).

**Тестирование фасада `Event`:**

	public function testGetIndex()
	{
		Event::shouldReceive('fire')->once()->with(array('name' => 'Дейл'));

		$this->call('GET', '/');
	}

> **Внимание:** не делайте этого для объекта `Request. Вместо этого, передайте желаемый ввод методу `call` во время выполнения вашего теста.

<a name="framework-assertions"></a>
## Проверки (assertions)

Laravel предоставляет несколько `assert`-методов, чтобы сделать ваши тесты немного проще.

**Проверка на успешный запрос:**

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertResponseOk();
	}

**Проверка статуса ответа:**

	$this->assertResponseStatus(403);

**Проверка переадресации в ответе:**

	$this->assertRedirectedTo('foo');

	$this->assertRedirectedToRoute('route.name');

	$this->assertRedirectedToAction('Controller@method');

**Проверка наличия данных в шаблоне:**

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertViewHas('name');
		$this->assertViewHas('age', $value);
	}

**Проверка наличия данных в сессии:**

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertSessionHas('name');
		$this->assertSessionHas('age', $value);
	}

<a name="helper-methods"></a>
## Вспомогательные методы

Класс `TestCase` содержит несколько вспомогательных методов для упрощения тестирования вашего приложения.

Вы можете установить текущего авторизованного пользователя с помощью метода `be`:

**Установка текущего авторизованного пользователя:**

	$user = new User(array('name' => 'John'));

	$this->be($user);

Вы можете заполнить вашу БД начальными данными изнутри теста методом `seed`.

**Заполнение БД тестовыми данными:**

	$this->seed();

	$this->seed($connection);

Больше информации на тему начальных данных доступно в разделе [Миграции и начальные данные](/docs/migrations#database-seeding).
