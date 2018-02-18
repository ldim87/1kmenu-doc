# Загрузка мультиязычных сообщений

Для загрузки стандартных мультиязычных сообщений используется ф-ция **imes** (см. [документацию](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_imes)).

> языковые сообщения находятся в папке **/sys/include/locale/*{type}*/i18n/*{lang}*.ini**

## Создание дополнительных сообщений

Если же необходимо создать дополнительные сообщения в отдельном классе или для специального интерфейса, то используется следующая схема.

```php
<?
class SomeClass {

	// подкласс для мультиязычных сообщений
	private $XrLang;

	// готовность мультиязычности (для ленивой загрузки класса)
	private $locale_initiated;

	// ключ мультиязычных сообщений
	private $locale_key = 'my_type';

	// инициализация сообщений
	function init_locale($sys = array()){
		// сохраняем в локальную переменную
		$this->XrLang = new XrLang();
		
		// настройка языка (опционально, по умолчанию и так используется $GLOBALS['user']['lang'])
		if(!empty($sys['lang'])){
			$this->XrLang->set_lang($sys['lang']);
		}
		
		// загрузка сообщений
		$this->locale_initiated = $this->XrLang->load( $this->locale_key );

		// возвращаем статус
		return $this->locale_initiated;
	}

	// загрузка сообщений
	function imes($message_id){
		// ленивая загрузка мультиязычности
		if(is_null($this->locale_initiated)){
			$this->init_locale();
		}

		// если не удалось подгрузить класс мультиязычности
		if($this->locale_initiated == false){
			return $message_id;
		}

		// если все ок, возвращаем мультиязычное сообщение
		return call_user_func_array(
			array($this->XrLang, 'get'), 
			func_get_args()
		);
	}

	// теперь можно вызывать любые сообщения
	function say_hello(){
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

// инициализация
$some_class = new SomeClass();

// выдача мультиязычных сообщений
echo $some_class->say_hello(); // Привет всем!
echo $some_class->imes('bye'); // Пока!

?>
```