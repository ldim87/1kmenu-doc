# Работа с базой данных

## mysql_do

Ф-ция служит для выполнения INSERT / UPDATE. Более подробно читай в [документации](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_mysql_do).

Как пользоваться:

```php
<?
// If second argument of mysql_do is array, it is used as values for prepared statement

$values = array('foo', 'bar');
$result = mysql_do('INSERT INTO some_table (column) VALUES ('.arr_in($values).')', $values);

// And $result is array
if($result['status']){
	otvet_ok('Action done. New row ID is: '.$result['insert_id']);
} else {
	otvet_err('Action failed! ' . (user_admin() ? $result['message'] : ''));
}

// If second argument is FALSE (default),
// $return contains status (on UPDATE) or new ID (on INSERT)

$result = mysql_do('INSERT INTO some_table (column) VALUES (\'foo\', \'bar\')');
// $result is integer
if($result){
	otvet_ok("New row ID is {$result}");
}

$result = mysql_do('UPDATE some_table SET column = \'foo\' WHERE id = 2');
// $result is boolean
if($result){
	otvet_ok("Successfully updated");
}
?>
```

## mysql_get_arr

Ф-ция служит для загрузки строк из БД. Более подробно читай в [документации](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_mysql_get_arr).

Как пользоваться:

```php
<?
// Select something by these IDs
$selected_ids = array(1, 2, 3);

// Query settings
$query_settings = array(
	// Debug mode
	'debug' => !empty($_GET['debug']) && user_admin(),
	// Cache mode
	'cache' => true,
	// Cache key 
	// It can be manual string or auto by current URL path:
	// if script URL is "/script/do/some/stuff"
	// key would be - "do_some_stuff" (see $GLOBALS['arr_url_go'] for more info)
	'cache_key' => key_by_url(),
	// Cache expiration in seconds
	'cache_time' => 3600
	// Cache flush
	'renew_cache' => !empty($_GET['renew_cache']) && user_admin()
);

// Get items
$products = mysql_get_arr(
	// Query
	'SELECT id, name, some_column FROM some_table WHERE id IN ('.arr_in($selected_ids).')',
	// Values for query
	$selected_ids,
	// Settings
	$query_settings
);

// Now you can iterate through the items array
if($products){
	foreach($products as $key => $product){
		echo "Product ID: {$product['id']} \n";
		echo "Product name: {$product['name']} \n";
		// ..
	}
} else {
	echo "Products are not found";
}
?>
```

## mysql_get_list

Ф-ция служит для загрузки одной строки из БД. Более подробно читай в [документации](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_mysql_get_list).

Как пользоваться:

```php
<?
// Query settings are the same as in example above
$query_settings = array(
	// Debug mode
	'debug' => !empty($_GET['debug']) && user_admin(),
	// Cache mode
	'cache' => true,
	// Cache key 
	// It can be manual string or auto by current URL path:
	// if script URL is "/script/do/some/stuff"
	// key would be - "do_some_stuff" (see $GLOBALS['arr_url_go'] for more info)
	'cache_key' => key_by_url(),
	// Cache expiration in seconds
	'cache_time' => 3600
	// Cache flush
	'renew_cache' => !empty($_GET['renew_cache']) && user_admin()
);

// Get item
$product = mysql_get_list(
	// Query
	'SELECT id, name, some_column FROM some_table WHERE foo = ?',
	// Values for query
	array('bar'),
	// Settings
	$query_settings
);

// Now you can access data
if($product){
	echo "Product ID: {$product['id']} \n";
	echo "Product name: {$product['name']} \n";
	// ..
} else {
	echo "Product is not found";
}
?>
```

## mysql_get_val

Ф-ция служит для загрузки одной ячейки из БД. Более подробно читай в [документации](http://vgit1.aynos.cz/phpdoc/1kmenu/#method_mysql_get_val).

Как пользоваться:

```php
<?
// Query settings are the same as in example above
$query_settings = array(
	// Debug mode
	'debug' => !empty($_GET['debug']) && user_admin(),
	// Cache mode
	'cache' => true,
	// Cache key 
	// It can be manual string or auto by current URL path:
	// if script URL is "/script/do/some/stuff"
	// key would be - "do_some_stuff" (see $GLOBALS['arr_url_go'] for more info)
	'cache_key' => key_by_url(),
	// Cache expiration in seconds
	'cache_time' => 3600
	// Cache flush
	'renew_cache' => !empty($_GET['renew_cache']) && user_admin()
);

// Get total items number
$products_count = mysql_get_val(
	// Query
	'SELECT COUNT(*) FROM some_table WHERE foo = ?',
	// Values for query
	array('bar'),
	// Settings
	$query_settings
);

// Now you can access data
echo "Total products count: {$products_count}";
?>
```