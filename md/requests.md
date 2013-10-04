# Запросы и ввод

- [Базовый ввод](#basic-input)
- [Cookies](#cookies)
- [Старый ввод](#old-input)
- [Файлы](#files)
- [Информация о запросе](#request-information)

<a name="basic-input"></a>
## Базовый ввод

Вы можете получить доступ ко всем данным, переданным приложению, используя всего несколько простых методов. Вам не нужно думать о том, какой тип HTTP-запроса был использован (GET, POST и т.д.) - методы работают одинаково для любого из них.

**Получение переменной:**

	$name = Input::get('name');

**Получение переменной или значения по умолчанию, если переменная не была передана:**

	$name = Input::get('name', 'Sally');

**Была ли передана переменная?**

	if (Input::has('name'))
	{
		//
	}

**Получение всех переменных запроса:**

	$input = Input::all();

**Получение некоторых переменных:**
	// Получить только перечисленные:
	$input = Input::only('username', 'password');

	// Получить все, кроме перечисленных:
	$input = Input::except('credit_card');

Некоторые JavaScript-библиотеки, такие как Backbone, могут передавать переменные в виде JSON. Вне зависимости от этого `Input::get` будет работать одинаково.

<a name="cookies"></a>
## Cookies

Все cookie, создаваемые Laravel, шифруются и подписываются специальным кодом - таким образом, если клиент изменит их значение, то они станут неверными.


**Чтение cookie:**

	$value = Cookie::get('name');

**Добавление cookie к ответу:**

	$response = Response::make('Hello World');

	$response->withCookie(Cookie::make('name', 'value', $minutes));

**Создание cookie, которая хранится вечно:**

	$cookie = Cookie::forever('name', 'value');

<a name="old-input"></a>
## Старый ввод

Вам может пригодиться сохранение пользовательского ввода между двумя запросами. Например, после проверки формы на корректность вы можете заполнить её старыми значениями в случае ошибки.

**Сохранение всего ввода для следующего запроса:**

	Input::flash();

**Сохранение некоторых переменных для следующего запроса:**

	// Сохранить только перечисленные:
	Input::flashOnly('username', 'email');

	// Сохранить все, кроме перечисленных:
	Input::flashExcept('password');

Обычно требуется сохранить ввод при переадресации на другую страницу - это делается легко:

	return Redirect::to('form')->withInput();

	return Redirect::to('form')->withInput(Input::except('password'));

> **Примечание:** Вы можете сохранять и другие данные внутри сессии, используя класс [Session](/docs/session).

**Получение старого ввода:**

	Input::old('username');

<a name="files"></a>
## Файлы

**Получение объекта загруженного файла:**

	$file = Input::file('photo');

**Определение успешной загрузки:**

	if (Input::hasFile('photo'))
	{
		//
	}

Метод `file` возвращает объект класса `Symfony\Component\HttpFoundation\File\UploadedFile`, который в свою очередь расширяет стандартный класс `SplFileInfo`, который предоставляет множество методов для работы с файлами.

**Перемещение загруженного файла:**

	Input::file('photo')->move($destinationPath);

	Input::file('photo')->move($destinationPath, $fileName);

**Получение пути к загруженному файлу:**

	$path = Input::file('photo')->getRealPath();

**Получение имени файла на клиентской системе (до загрузки):**

	$name = Input::file('photo')->getClientOriginalName();

**Получение расширения загруженного файла:**

	$extension = Input::file('photo')->getClientOriginalExtension();

**Получение размера загруженного файла:**

	$size = Input::file('photo')->getSize();

**Определение MIME -типа загруженного файла:**

	$mime = Input::file('photo')->getMimeType();

<a name="request-information"></a>
## Информация о запросе

Класс `Request` содержит множество методов для изучения входящего запроса в вашем приложении. Он расширяет класс `Symfony\Component\HttpFoundation\Request`. Ниже - несколько полезных примеров.

**Получение URI (пути) запроса:**

	$uri = Request::path();

**Соответствует ли запрос маске пути?**

	if (Request::is('admin/*'))
	{
		//
	}

**Получение URL запроса:**

	$url = Request::url();

**Извлечение сегмента URI (пути):**

	$segment = Request::segment(1);

**Чтение заголовка запроса:**

	$value = Request::header('Content-Type');

**Чтение значения из $_SERVER**

	$value = Request::server('PATH_INFO');

**Сделан ли запрос через AJAX:**

	if (Request::ajax())
	{
		//
	}

**Использует ли запрос HTTPS:**

	if (Request::secure())
	{
		//
	}
