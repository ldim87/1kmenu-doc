# script.php
Главный обработчик форм и аяксовых запросов.

## Обработка форм
Пример обработки формы:

```html
<form method="POST" action="/script/free/test">
	<input type="text" name="my_text">
	<input type="submit">
</form>
```

Атрибут "**action**" может содержать и полный неукороченный адрес:
```html
<form method="POST" action="/script.php?go=free/test">
```

Путь обрабатывающего скрипта в обоих случаях следующий:
> **/sys/script/free/test.php**

```php
<?
otvet_ok("Your text: ".filter_input(INPUT_POST, 'my_text', FILTER_SANITIZE_SPECIAL_CHARS));
?>
```


## Обработка аяксовых запросов

Пример обработки аяксового запроса
```html
<button type="button" onclick="$('#result_box').load('/ajax/free/test_ajax')">
	Load some HTML via AJAX
</button>
<div id="result_box"></div>
```

Список возможных адресов запроса, которые обрабатываются одним и тем же скриптом:
* /ajax/free/test_ajax
* /ajax.php?go=free/test_ajax
* /script/free/test_ajax?ajax=1 (или referer_not=1)
* /script.php?go=free/test_ajax&ajax=1 (или referer_not=1)

Путь обрабатывающего скрипта во всех вышеперечисленных случаях:
> **/sys/script/free/test_ajax.php**

```php
<?
echo "<span>Some HTML</span>";
echo "<br><span>Some more HTML</span>";
?>
```

## ajax.php / script.php

Запросы "**/ajax/...**" (или "**ajax.php?go=...**")  являются ссылкой на файл /script.php с дополнительным УРЛ параметром "**ajax=1**" и обрабатываются одним и тем же файлом! Т.е. **физически файла ajax.php не существует**!  
  
Разница в вызове заключается лишь в том, что запросы "**/script/...**" (или "**script.php?go=...**") после исполнения сценария по умолчанию перенаправляют браузер обратно на страницу, с которой был произведен запрос (т.е. на **$_SERVER['HTTP_REFERER']**). Управлять переадресацией можно следующим образом:

### Параметр referer

Параметр "**referer**" можно указать прямо в ссылке или в атрибуте формы "**action**":

```html
<form method="POST" action="/script.php?go=free/test&referer=/test">
```

или в коде PHP:

```php
<?
$_REQUEST['referer'] = "/test";
?>
```

Тогда после выполнения скрипта будет произведена переадресация браузера на страницу "**/test**"

> Параметры "**ajax=1**" и "**referer_not=1**" приоритетнее и отключают переадресацию даже если указан параметр "**referer=...**", который просто в данном случае игнорируется!