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

	function __construct($opt = array()){
		// загрузка мультиязычных сообщений
		$this->init_locale(
			!empty($opt['locale_lang']) ? $opt['locale_lang'] : '',
			!empty($opt['locale_settings']) ? $opt['locale_settings'] : array()
		);
	}

	// инициализация сообщений
	function init_locale($lang = '', $sys = array()){
		// сохраняем в локальную переменную
		$this->XrLang = new XrLang();
		
		// настройка языка (опционально, по умолчанию и так используется $GLOBALS['user']['lang'])
		// $this->XrLang->set_lang($lang ? $lang : $GLOBALS['user']['lang']);

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

// возможные настройки для админа
$debug = !empty($_GET['debug']) && user_admin();
$cache = empty($_GET['nocache']) || !user_admin();

// инициализация
$some_class = new SomeClass();

echo $some_class->say_hello(); // Привет всем!
echo $some_class->imes('bye'); // Пока!

?>
```