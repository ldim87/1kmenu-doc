# Пример работы с формой

На данном примере рассматривается создание интефейса управления любого объекта (рецепта, продукта итд). В данном случае мы создаем страницы добавления продукта, редактирования продукта и сам скрипт обработки данных продукта (сохранение и удаление данных). Для работы с формой используется класс [FormManager](http://vgit1.aynos.cz/phpdoc/1kmenu/classes/FormManager.html). Смотри [демо страницы](https://1000.menu/test/product/add).

## Структура создаваемых файлов

С начала ппределяемся с УРЛ создаваемых страниц:

> /product/{ID продукта} - карточка продукта

> /product/edit/{ID продукта} - редактирование продукта

> /product/add - добавление продукта

> /script/user/product_info - адрес скрипта обработки данных

Структура файлов будет следующая

> /sys/html/product.html - Обработка маршрутов УРЛ /product/...

> sys/html/product/info.html - Шаблон карточки продукта

> /sys/html/product/add.html - Обработка маршрута /product/add

> /sys/html/product/edit.html - Обработка маршрута /product/edit/...

> /sys/html/product/form.php - Шаблон формы добавления / редактирования продукта

> /sys/include/locale/form_product_info/i18n/ru.ini - Текстовые сообщения формы на русском

> /sys/include/locale/product_info/i18n/ru.ini - Текстовые сообщения страниц на русском

> /sys/script/user/product_info.php - Скрипт обработки данных формы

## Описание файлов 

Файл, который отвечает за загрузку страниц продукта:

> /sys/html/**product**.html

```php
<?
// Загружаем часть УРЛ (/product/{$path}/)
$path = get_url_part(1); // можно так: $GLOBALS['arr_url_go'][1] ?? false;

// Загружаем мультиязычные сообщения самих страниц
// Хоть и название схоже с названием формы, ini файл будет 
// отдельный, так как к форме еще добавляется префикс "form_"
$locale_messages = new XrLang(array('type'=>'product_info'));

// Если это ID - карточка продукта
// т.е. если это цифра > 0 (см.док. is_num())
if(is_num($path, true)){
	// Загружаем файл "sys/html/product/info.html" (в данном примере не описывается)
	include_or_err(__DIR__.'/'.get_url_part(0).'/info.html', 404, $locale_messages);
}
// Если это не ID, пытаемся загрузить файл в папке "sys/html/product/"
else {
	// Загружаем файл "/sys/html/product/$path.html"
	// Если не найден, то загрузится страница 404
	// Также передаем переменную с текстовыми сообщениями, которая
	// во вложенный файл передается как переменная "$arr_lenta"
	include_or_err(__DIR__.'/'.get_url_part(0).'/'.$path.'.html', 404, $locale_messages);
}
?>
```

Текстовые сообщения страниц /product/... добавляем в файл:

> /sys/include/locale/product_info/i18n/ru.ini

```ini
add_new_product = "Добавление нового продукта"
edit_product = "Редактирование продукта"
```

Создаем файл для добавления нового продукта. Здесь все просто, так как не нужно загружать данные:

> sys/html/product/add.html

```php
<?
// Настраиваем заголовок страницы
// Переменная $locale_messages передалась как $arr_lenta (см. док. include_or_err())
$GLOBALS['body']['title'] = $arr_lenta->get('add_new_product');

// Загружаем форму
require __DIR__.'/form.php';
?>
```

Создаем файл для редактирования продукта:

> sys/html/product/edit.html

```php
<?
// Берем ID продукта из УРЛ
$id = get_url_part(2);

// Если формат ID неверный, возвращаем 404 страницу
if(!is_num($id, true)){
	include 'sys/html/404.html';
	return;
}

// Загружаем данные продукта
$data = mysql_get_list('SELECT id, name, info, work FROM products WHERE id=?', array($id));

// Настраиваем заголовок страницы
// Переменная $locale_messages передалась как $arr_lenta (см. док. include_or_err())
// Ф-ция ival экранирует все специальные знаки (см. док. ival())
$GLOBALS['body']['title'] = $arr_lenta->get('edit_product').' "'.ival($data['name']).'"';

// Загружаем форму
require __DIR__.'/form.php';
?>
```

## Шаблон формы

Если мы хотим использовать локализацию формы, то нам необходимо создать файл для мультиязычных сообщений отдельно для формы.

> /sys/include/locale/form_{Название локализации}/i18n/{Язык}.ini

Название локализации равно названию формы, если не задано иначе. В данном случае мы назовем нашу форму "product_info". 

Создаем файл для русских сообщений:

> /sys/include/locale/**form_product_info**/i18n/**ru**.ini

```ini
product_name = "Название продукта"
product_info = "Описание продукта"
status = "Состояние"
show_product = "Показывать этот продукт"
# Сообщения обработки
product_not_found = "Продукт не найден"
name_is_empty = "Вы не заполнили название продукта"
info_is_empty = "Вы не заполнили описание продукта"
product_successfully_added = "Продукт был успешно добавлен"
product_successfully_updated = "Продукт был успешно отредактирован"
product_successfully_deleted = "Продукт был успешно удален"
```

Помимо этого нужно иметь в виду, что в ходе действий загружаются еще стандартные сообщения, которыми можно пользоваться и которые не желательно дублировать в новых ini файлах

> /sys/include/locale/**form**/i18n/ru.ini - загружается классом FormManager вне зависимости от указанного названия формы. Все сообщения доступны по вызову метода $form->imes($message_id).

> /sys/include/locale/**general**/i18n/ru.ini - загружается всегда. Все сообщения доступны по вызову глобальной ф-ции imes($message_id).

Теперь все готово для создания формы. Так как форма используется в 2 местах ("/add" и "/edit"), то мы вынесли ее в отдельный файл:

> /sys/html/product/form.php

```php
<?
// Подключаем класс форм
require 'sys/include/FormManager.php';

// Инициализация формы "product_info" вместе с данными формы
$form = new FormManager(
	'product_info', 	// название формы
	$data ?? array(),	// данные формы (в edit.html наполняется данными)
	array(				// настройки:
		// режим отладки (выкл. по умолчанию, админ может включать get параметром debug=1)
		'debug' => !empty($_GET['debug']) && user_admin(),
		// кэш языковых сообщений (вкл. по умолчанию, админ может выключать get параметром nocache=1)
		'form_locale_cache' => empty($_GET['nocache']) || !user_admin(),
		// если необходимо загрузить языковые сообщения из другого файла
		// напр. /sys/include/locale/form_products/i18n/ru.ini, то используем следующую настройку:
		// 'locale_name' => 'products'
	)
);
?>
<form method="POST" action="/script/user/product_info">
	<table class="admin_table_form"><tbody>
		<tr>
			<th><?= $form->imes('product_name') ?>:</th>
			<td><input type="text" name="name" size="30" value="<?= $form->get('name') ?>"></td>
		</tr>
		<tr>
			<th><?= $form->imes('product_info') ?>:</th>
			<td class="fb-pd">
				<textarea name="info" cols="60"><?= $form->get('info') ?></textarea>
			</td>
		</tr>
		<tr>
			<th><?= $form->imes('status') ?>:</th>
			<td class="fb-pd">
				<label>
					<input type="checkbox" name="work"<?= $form->get('work') ? ' checked' : '' ?>> 
					<?= $form->imes('show_product') ?>
				</label>
			</td>
		</tr>
	</tbody></table>
	<div class="fb">
		<input type="hidden" name="id" value="<?= $form->get('id') ?>">
		<input type="submit" value="<?= $form->imes('save') ?>">
		<?
		// Добавляем кнопку "Удалить", если это редактирование продукта
		if(!$form->is_new){
			echo '<input type="submit" name="delete_product" value="'.$form->imes('delete').'" onclick="return confirm(\''.$form->imes('confirm_deletion').'\')">';
		}
		?>
	</div>
</form>
```

## Обработка формы

Так как форма ссылается на "/script/user/product_info", то обрабатывающий файл будет находиться здесь:

> /sys/script/user/product_info.php

```php
<?
// Подключаем класс форм
require 'sys/include/FormManager.php';

// Инициализация формы "product_info" вместе с данными формы
$form = new FormManager(
	'product_info', 	// название формы должно быть схоже с названием в шаблоне формы
	$_POST,				// данные формы (отсылаются вместе с формой)
	array(				// настройки:
		// режим отладки (выкл. по умолчанию, админ может включать get параметром debug=1)
		'debug' => !empty($_GET['debug']) && user_admin(),
		// кэш языковых сообщений (вкл. по умолчанию, админ может выключать get параметром nocache=1)
		'form_locale_cache' => empty($_GET['nocache']) || !user_admin(),
		// запоминаем данные в сессии, чтобы человек не заполнял форму дважды в случае ошибки
		// если валидация формы проводится с помощью javascript на странице формы, можно это не ипользовать
		'session_store' => true
	)
);

// Если это удаление продукта
if($form->getval('delete_product')){
	// Проверка ID
	if($form->is_new){
		// Выдача ошибки
		otvet_err($form->imes('product_not_found'));

		// Прекращение сценария
		return;
	}

	// удаляем продукт из базы
	$result = mysql_do(
		'DELETE FROM products WHERE id=?',
		array($form->getval('id'))
	);

	// Если произошла ошибка
	if(!$result['status']){
		// Выписываем ошибку
		otvet_err($form->imes('error_saving_data_try_later'));
	}
	// Если удаление прошло успешно
	else {
		// Выписываем сообщение и очищаем данные формы
		$form->done('product_successfully_deleted');

		// Перенаправляем браузер на какую-нибудь другую страницу,
		// иначе будет осуществлен переход на страницу редактирования
		// уже не существующего продукта /product/edit/{id}
		$_REQUEST['referer'] = '/products';
	}

	// Прекращение сценария
	return;
}

// Проводим обработку и проверку данных для сохранения продукта

// Обрабатываем поле name - обрезаем края и переводим первую букву в верхний реестр
$form->set('name', str_ucfirst(trim($form->getval('name'))));

// Проверяем, есть ли название после применения фильтров
if(!$form->getval('name')){
	// записываем ошибку
	$form->error('name_is_empty');
}

// Обрабатываем поле info
$form->set('info', trim($form->getval('info')));

// Проверяем, заполнено ли описание
if(!$form->getval('info')){
	// записываем ошибку
	$form->error('info_is_empty');
}

// Настраиваем статус 1 или 0
$form->set('work', $form->getval('work') ? 1 : 0);

// Если форма заполнена неверно (есть ошибки, заданные через $form->error())
if(!$form->is_valid()){
	// Возващаем пользователю ошибки
	otvet_err($form->get_errors());

	// Прекращаем сценарий
	return;
}

// Записываем данные в базу
$result = mysql_insert_update(
	// данные для записи
	$form->getvals(
		array('name', 'info', 'work')
	),
	// название таблицы
	'products',
	// ID (если есть) - от этого зависит, будет ли проводится UPDATE или INSERT
	$form->getval('id')
);

// Если запись в базу не удалась
if(!$result){
	// Выписываем ошибку
	otvet_err($form->imes('error_saving_data_try_later'));

	// Прекращаем сценарий
	return;
}

// Если это было добавление нового продукта
if($form->is_new){
	// Сообщаем об успешном действии и очищаем данные формы
	$form->done('product_successfully_added');

	// Делаем редирект на страницу редактирования нового продукта
	// (в $result при INSERT сохраняется ID нового элемента)
	$_REQUEST['referer'] = '/product/edit/' . $result;
}
// Если это было редактирование продукта
else {
	// Сообщаем об успешном действии и очищаем данные формы
	$form->done('product_successfully_updated');
}
?>
```