# Контроллеры

- [Простейшие контроллеры](#basic-controllers)
- [Фильтры для контроллеров](#controller-filters)
- [RESTful-контроллеры](#restful-controllers)
- [Контроллеры ресурсов](#resource-controllers)
- [Обработка неопределённых методов](#handling-missing-methods)

<a name="basic-controllers"></a>
## Простейшие контроллеры

Вместо того, чтобы определять всю маршрутизацию (routing) вашего проекта в файле `routes.php` вы можете организовать её, используя класс Controller. Контроллеры могут группировать связанную логику в отдельные классы, а кроме того использовать дополнительные возможности Laravel, такие как автоматическое [внедрение зависимостей](/docs/ioc).

Контроллеры обычно хранятся в папке `app/controllers`, а этот путь по умолчанию зарегистрирован в настройке `classmap` вашего файла `composer.json`.

Вот пример простейшего класса контроллера:

	class UserController extends BaseController {

		/**
		 * Отобразить профиль соответствующего пользователя.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

Все контроллеры должны наследовать класс `BaseController`. Этот класс также может хранится в папке `app/controllers` и в него можно поместить общую логику для других контроллеров. The `BaseController` расширяет стандартный класс Laravel, `Controller` class. Теперь, определив контроллер, мы можем зарегистрировать маршрут для его действия (action):

	Route::get('user/{id}', 'UserController@showProfile');

Если вы решили организовать ваши контроллеры в пространства имён, просто используйте полное имя класса при определении маршрута:

	Route::get('foo', 'Namespace\FooController@method');

> **Примечание:** Помните, что имена класса в строках следуют обычным правилам PHP и если ваш класс начинается с `n`, `t` и других букв в нижнем регистре, то обратный слэш перед ними нужно экранировать - иначе они преобразуются в разрыв строки, табуляцию или иной спецсимвол: `namespace\new_controller` - прим. пер.


Вы также можете присвоить имя этому маршруту:

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

Вы можете получить URL к действию методом `URL::action` method:

	$url = URL::action('FooController@method');

Получить имя действия, которое выполняется в данном запросе, можно методом  `currentRouteAction`:

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## Фильтры для контроллеров

[Фильтры](/docs/routing#route-filters) могут указываться для контроллеров аналогично "обычным" маршрутам:

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

Однако вы можете указывать их и изнутри самого контроллера:

	class UserController extends BaseController {

		/**
		 * Создать экземпляр класса UserController.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth');

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

Можно устанавливать фильтры в виде функции-замыкания:

	class UserController extends BaseController {

		/**
		 * Создать экземпляр класса UserController.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

<a name="restful-controllers"></a>
## RESTful-контроллеры

Laravel позволяет вам легко создавать единый маршрут для обработки всех действий контроллера используя простую схему именования REST. Для начала зарегистрируйте маршрут методом `Route::controller`:

**Регистрация RESTful-контроллера:**

	Route::controller('users', 'UserController');

Метод `controller` принимает два аргумента. Первый - корневой URI (путь), который обрабатывает данный контроллер, а второй - имя класса самого контроллера.После регистрации просто добавьте методы в этот класс с префиксом в виде типа HTTP-запроса (HTTP verb), который они обрабатывают.

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

Методы `index` обрабатывают корневой URI контроллера - в нашем случае это `users`.

Если имя действия вашего контроллера состоит из нескольких слов вы можете обратиться к нему по URI, используя синтаксис с дефисами (-). Например, следующее действие в нашем классе `UserController` будет доступно по адресу `users/admin-profile`:

	public function getAdminProfile() {}

<a name="resource-controllers"></a>
## Контроллеры ресурсов

Они упрощают построение RESTful-контроллеров, работающих с ресурсами. Например, вы можете создать контроллер, обрабатывающий фотографии, хранимые вашим приложением. Вы можете быстро создать такой контроллер с помощью команды `controller:make` интерфейса (Artisan) и метода `Route::resource`.

Для создания контроллера выполните следующую консольную команду:

	php artisan controller:make PhotoController

Теперь мы можем зарегистрировать его как контроллер ресурса:

	Route::resource('photo', 'PhotoController');

Этот единственный вызов создаёт множество маршрутов для обработки различный RESTful-действий на ресурсе photo. Сам сгенерированный контроллер уже имеет методы-заглушки для каждого из этих маршрутов с комментариями, которые напоминают вам о том, какие типы запросов они обрабатывают.

**Запросы, обрабатываемые контроллером ресурсов:**

Тип       | Путь                  | Действие     | Имя маршрута
----------|-----------------------|--------------|---------------------
GET       | /resource             | index        | resource.index
GET       | /resource/create      | create       | resource.create
POST      | /resource             | store        | resource.store
GET       | /resource/{id}        | show         | resource.show
GET       | /resource/{id}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{id}        | update       | resource.update
DELETE    | /resource/{id}        | destroy      | resource.destroy

Иногда вам может быть нужно обрабатывать только часть всех возможных действий:

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

Вы можете указать этот набор и при регистрации маршрута:

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));
					
	// либо:

  	Route::resource('photo', 'PhotoController',
                  array('except' => array('create', 'store', 'update', 'delete')));

<a name="handling-missing-methods"></a>
## Обработка неопределённых методов

Можно определить "catch-all" метод, который будет вызываться для обработки запроса, когда в контроллере нет соответствующего метода. Он должен называться `missingMethod` и принимать массив параметров запроса в виде единственного своего аргумента.

**Defining A Catch-All Method**

	public function missingMethod($parameters)
	{
		//
	}
