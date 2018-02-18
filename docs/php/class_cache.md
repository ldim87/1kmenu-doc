#  Кэширование запросов в классах

За кэширование данных отвечают 2 основных ф-ции:

- [mc_get](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_mc_get) - загрузка данных из кэша
- [mc_set](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_mc_set) - запись данных в кэш
- [mc_del](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_mc_del) - удаление данных в кэше

Следующий пример описывает, как можно использовать кэширование в классе. В данном случае класс управления "Продуктом" использует кэширование благодаря этим основным элементам:

> private $cache_keys = array(**ключи**) - список используемых ключей кэша

> function get_cache_key($keyname) - возвращает ключ по его названию (индексу в $this->cache_keys)

> function get_cache_keys() - возвращает все ключи кэша, т.е. целый $this->cache_keys

На примере можно показать, как использовать кэш в многостраничных списках, или как инвалидировать кэш при обновлении данных.

```php
<?
// Псевдо-класс управления "Продуктом"
class SomeProduct {

	// ID продукта
	private $product_id;

	// ключи кэша (ко всем по умолчанию добавляется ID продукта)
	private $cache_keys = array(
		// инфо продукта
		'info' => 'product_info_',
		// похожие продукты (с постраничностью)
		'similars' => 'product_similars_',
		// общее кол-во похожих продуктов
		'similars_count' => 'product_similars_count_',
		// версия списка похожих продуктов (т.к. это многостраничный список)
		'similars_list_version' => 'product_similars_list_ver_'
	);

	// конструктор
	function __construct($product_id){
		// ID продукта
		$this->product_id = (int) $product_id;

		// настройка ключей
		foreach ($this->cache_keys as $k => $keyname) {
			$this->cache_keys[$k] = $keyname . $this->product_id;
		}
	}

	// возвращает ключ кэша
	function get_cache_key($keyname){
		return $this->cache_keys[$keyname] ?? false;
	}

	// возвращает все ключи кэша
	function get_cache_keys(){
		return $this->cache_keys;
	}
	
	//
	// Теперь можно использовать кэширование
	//

	// загрузка продукта
	function get_product_info($opt = array()){
		// настройки запроса
		$debug = !empty($opt['debug']);	// default off
		$cache = !isset($opt['cache']) || $opt['cache']; // default on
		
		// берем ключ
		$cache_key = $this->get_cache_key('info');

		// возвращаем данные с учетем кэша
		return mysql_get_list(
			'SELECT id, name, description FROM products WHERE id=?',
			array($this->product_id),
			array(
				'debug' => $debug,
				'cache' => $cache,
				'cache_key' => $cache_key,
				'cache_time' => 3600
			)
		);
	}

	// загрузка похожих продуктов с постраничностью
	function get_product_similars($page = 1, $opt = array()){
		// настройки запроса
		$debug = !empty($opt['debug']);	// default off
		$cache = !isset($opt['cache']) || $opt['cache']; // default on
		
		// кол-во результатов на странице
		$on_page = 50;

		// время кэширования
		$cache_time = 3600;

		// 
		// ключи кэша 
		// 
		
		// ключ одной страницы списка
		$cache_key = $this->get_cache_key('similars') . '_' . $page;

		// ключ версии списка (для правильной работы многостраничных списков)
		$cache_key_version = $this->get_cache_key('similars_list_version');

		//
		// возвращаем данные с учетем кэша
		// 
		
		return mysql_get_arr(
			'SELECT id, name, description FROM products_similars WHERE similar_id=? LIMIT ' . (($page-1)*$on_page) . ', ' . $on_page,
			array($this->product_id),
			array(
				'debug' => $debug,
				'cache' => $cache,
				'cache_key' => $cache_key,
				'cache_version_key' => $cache_key_version,
				'cache_time' => $cache_time				
			)
		);
	}

	// загрузка кол-ва похожих продуктов
	function get_product_similars_count($opt = array()){
		// настройки запроса
		$debug = !empty($opt['debug']);	// default off
		$cache = !isset($opt['cache']) || $opt['cache']; // default on

		// время кэширования
		$cache_time = 3600;
		
		// ключ кол-ва всех похожих продуктов
		$cache_key = $this->get_cache_key('similars_count');

		//
		// возвращаем данные с учетем кэша
		// 
		
		return mysql_get_val(
			'SELECT COUNT(*) FROM products_similars WHERE similar_id=?',
			array($this->product_id),
			array(
				'debug' => $debug,
				'cache' => $cache,
				'cache_key' => $cache_key,
				'cache_time' => $cache_time
			)
		);
	}

	//
	// Скидывать кэш можно следующим образом
	//

	// напр. при обновлении данных продукта
	function update_product_info(array $update_data, $opt = array()){
		// настройки запроса
		$debug = !empty($opt['debug']);	// default off

		// обновляем продукт в БД
		$result = mysql_set_data($update_data, 'products', $this->product_id, array('debug' => $debug));

		// если обновление прошло успешно
		if($result['status']){
			// очищаем кэш данных продукта
			mc_del($this->get_cache_key('info'));
		}

		// возвращаем результат
		return $result;
	}

	// очистка кэша списка похожих продуктов (все страницы + кол-во найденных результатов)
	function reload_similars_list(){
		// все ключи
		$cache_keys = $this->get_cache_keys();
		
		// выбираем ключи списка
		$flush_keys = array(
			$cache_keys['similars_count'],			// скинет счетчик кол-ва результатов
			$cache_keys['similars_list_version']	// скинет версию списка (заставит обновить кэш на каждой странице списка)
		);

		// чистим выбранные ключи
		return mc_del($flush_keys);
	}
}
?>
```
