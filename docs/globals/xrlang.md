# Загрузка мультиязычных сообщений

Для загрузки стандартных мультиязычных сообщений используется ф-ция **imes** (см. [документацию](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_imes)).

> языковые сообщения находятся в папке **/sys/include/locale/*{type}*/i18n/*{lang}*.ini**

## Создание дополнительных сообщений

Если же необходимо создать дополнительные сообщения в отдельном классе или для специального интерфейса, то используется следующая схема.

```php
<?
class SomeClass {

	// подкласс для мультиязычных сообщений
	protected $XrLang;

	// инициализации сообщений
	function init_locale(XrLang $lang_class, $sys = array()){
		// сохраняем в локальную переменную
		$this->XrLang = $lang_class;
		
		// настройка языка
		$this->XrLang->set_lang($GLOBALS['user']['lang']);

		// загрузка сообщений
		$this->XrLang->load(
			'my_type',	// ключ для мультиязычных сообщений
			array(
				// debug - default off
				'debug' => !empty($sys['debug']),
				// cache - default on
				'cache' => !isset($sys['cache']) || !empty($sys['cache'])
			)
		);
	}

	// загрузка сообщений
	function imes($message_id){
		// Если подкласс не был настроен, возвращаем код сообщения
		if(is_null($this->XrLang)){
			return $message_id;
		}
		
		// Загрузка сообщений
		return call_user_func_array(
			array($this->XrLang, 'get'), 
			func_get_args()
		);
	}

	// теперь можно вызывать любые сообщения
	function say_hello(){
		// напр.
		return $this->imes('say_hello');
	}
}

?>
```

Далее создаем файл с языковыми сообщениями на русском языке. Так как в примере выше ключ **my_type**, создаем папку с таким же названием.

> /sys/include/locale/**my_type**/i18n/**ru.ini**:
>
> say_hello = "Привет всем!"  
> bye = "Пока!"

Использование самого класса происходит следующим образом:

```php
<?

// загрузка класса
require '/path/to/the/some_class.php';

// возможные настройки
$debug = !empty($_GET['debug']) && user_admin();
$cache = true;

// инициализация
$some_class = new SomeClass(
	new XrLang(),
	array(
		'debug' => $debug,
		'cache' => $cache
	)
);

echo $some_class->say_hello(); // Привет всем!
echo $some_class->imes('bye'); // Пока!

?>
```