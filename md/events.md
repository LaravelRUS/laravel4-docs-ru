# События

- [Простейшее использование](#basic-usage)
- [Обработчики по шаблону](#wildcard-listeners)
- [Классы-обработчики](#using-classes-as-listeners)
- [Queued Events](#queued-events)
- [Event Subscribers](#event-subscribers)

<a name="basic-usage"></a>
## Простейшее использование

Класс `Event`содержит простую реализацию концепции "Наблюдатель", что позволяет вам подписываться на уведомления о событиях в вашем приложении.

**Подписка на событие:**

	Event::listen('user.login', function($user)
	{
		$user->last_login = new DateTime;

		$user->save();
	});

**Возбуждение события:**

	$event = Event::fire('user.login', array($user));

При подписывании на событие вы можете указать приоритет. Обработчики с более высоким приоритетом будут вызваны перед теми, чей приоритет ниже, а обработчики с одинаковым приоритетом будут вызываться в порядке их регистрации.

**Подписка на событие с приоритетом:**

	Event::listen('user.login', 'LoginHandler', 10);

	Event::listen('user.login', 'OtherHandler', 5);

Иногда вам может быть нужно пропустить вызовы других обработчиков события. Вы можете сделать это, вернув значение `false`:

**Прерывание обработки события:**

	Event::listen('user.login', function($event)
	{
		// Обработка события...

		return false;
	});

<a name="wildcard-listeners"></a>
## Обработчики по шаблону

При регистрации обработчика вы можете использовать звёздочки (*) для привязки его ко всем подходящим событиям.

**Registering Wildcard Event Listeners**

	Event::listen('foo.*', function($param, $event)
	{
		// Обработка событий...
	});

Этот обработчик будет вызываться при любом событии, начинающемся `foo.`. Обратите внимание, что в качестве первого аргумента ему передаётся полное имя обрабатываемого события.

<a name="using-classes-as-listeners"></a>
## Классы-обработчики

В некоторых случаях вы можете захотеть обрабатывать события внутри класса, а не функции-замыкания. Классы-обработчики получаются из [Laravel IoC container](/docs/ioc), поэтому вы можете использовать все его возможности по автоматическому внедрению зависимостей.

**Регистрация обработчика в классе:**

	Event::listen('user.login', 'LoginHandler');

По умолчанию будет вызываться метод `handle` класса `LoginHandler`.

**Создание обработчика внутри класса:**

	class LoginHandler {

		public function handle($data)
		{
			//
		}

	}

Если вы не хотите использовать метод `handle`, вы можете указать другое имя при регистрации.

**Регистрация обработчика в другом методе:**

	Event::listen('user.login', 'LoginHandler@onLogin');

<a name="queued-events"></a>
## Запланированные события

С помощью методов `queue` и `flush` вы можете запланировать события к возникновению, но не возбуждать их сразу.

**Регистрация цепочки событий:**

	Event::queue('foo', array($user));

**Регистрация "пускателя":**

	Event::flusher('foo', function($user)
	{
		//
	});

Наконец, вы можете "запустить" все запланированные события методом `flush`:

	Event::flush('foo');

<a name="event-subscribers"></a>
## Классы-подписчики

Классы-подписчики содержат обработчики множества событий. Подписчики должны содержать метод `subscribe`, которому будет передан экземпляр обработчика событий для регистрации.

**Определение класса-подписчика:**

	class UserEventHandler {

		/**
		 * Обработка событий входа пользователя в систему.
		 */
		public function onUserLogin($event)
		{
			//
		}

		/**
		 * Обработка событий выхода из системы.
		 */
		public function onUserLogout($event)
		{
			//
		}

		/**
		 * Регистрация всех обработчиков данного подписчика.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen('user.login', 'UserEventHandler@onUserLogin');

			$events->listen('user.logout', 'UserEventHandler@onUserLogout');
		}

	}

Как только класс-подписчик определён, он может быть зарегистрирован внутри `Event`.

**Регистрация класса-подписчика:**

	$subscriber = new UserEventHandler;

	Event::subscribe($subscriber);
