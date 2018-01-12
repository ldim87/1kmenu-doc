# Глобальные JS ф-ции и методы

JS файлы с яваскриптами находятся в папке "**/style/js/**".

## /style/js/general.js

Файл с глобальными ф-циями и объектами, принципы которых описаны ниже.

### load_img

*string* - Изображение gif для уведомления о загрузке чего-либо.

### load_img_big

*string* - Большое изображение gif для уведомления о загрузке чего-либо.

### imes

Объект для мультиязычных сообщений. Методы:

* **get** (message_id *string*, prefix *string*) - Загрузка сообщений с указанным **message_id**. Чтобы использовать мультиязычные сообщения в разных объектах, можно использовать **prefix** (по умолчанию используется префикс "**globals**", который служит исключительно для файла general.js). Префикс является атрибутом объекта **xr_lang_mes**, который предназначен для хранения текстовых сообщений. Пример использования:

```javascript
// init localized messages (can be in separate file)
xr_lang_mes.test_prefix = {
	alert: "This is localized alert message",
	warning: "This is localized warning message",
	test: "This is some random test message"
};

// use in object
var some_object = {
	// shortcut to imes function with prefix "test_prefix"
	imes: function (message_id) {
		return imes.get(message_id, "test_prefix");
	},
	// use it in any method
	test_method: function(){
		alert(this.imes("alert"));
		alert(this.imes("warning"));
		alert(this.imes("test"));
	}
};
```

* **add** (prefix *string*, messages *object*) - Добавление в объект дополнительных сообщений. Пример:

```javascript
// xr_lang_mes.test_prefix must be already set!

// Add more messages
imes.add("test_prefix", {
	new_message: "Some new message text",
	next_new_message: "Some next new message text"
});

// use it the same way as in previous example
var some_object = {
	// init local method with prefix "test_prefix"
	imes: function(message_id){
		return imes.get(message_id, "test_prefix");
	},
	// use in methods
	test_method: function(){
		// call existing messages
		alert(this.imes("test"));

		// call new messages
		alert(this.imes("new_message"));
		alert(this.imes("next_new_message"));
	}
};
```

### delayed_fc

*function* (**callback** *function*, **ms** *integer*) - Отложенный вызов ф-ции. Удобно для отложенных событий **onkeyup**, **onchange**, или отложенных показов/скрытий блоков итп.

> Счетчик времени **глобальный**! Это означает, что вторичный вызов ф-ции всегда скидывает счетчик предыдущего вызова!

Пример:

```javascript
// This delayed function is ignored!
delayed_fc (
	function(){
		alert("This message will NOT be alerted");
	},
	1000 // time in milliseconds
);

// Delayed calls CAN NOT be stacked. This will override the first alert!
delayed_fc (
	function(){
		alert("This message will be alerted!");
	},
	2000 // time in milliseconds
);
```

### preloadimages

*function* (**arr** *array*) - Запуск ф-ции после загрузки всех картинок в массиве. Возвращает объект с методом **done()**, в котором описывается колбэк.

Пример:

```javascript
// Preload images and alert message when done
preloadimages([
	'//path/to/first.image.jpg',
	'//path/to/second.image.jpg',
	'//path/to/third.image.jpg',
]).done(function(){
	alert("Images were loaded!");
});
```

### mes.fire

*function* (**message_text** *string*, **type** *string|null*, **options** *object|null*) - Открытие popup сообщения.

Пример:

```javascript
// you can pass HTML in message popup
mes.fire("This is <strong>POPUP</strong> message");

// message can have an error format
mes.fire("This is error message", "err");

// message HTML can be picked via JQuery selector
mes.fire("#block_id", "jqsel");

// popup window class can be set via options
mes.fire("Some message", null, {pclass: "custom-class"});
```

### iface

Основной объект с методами управления интерфейсом сайта. Самые используемые методы:

#### iface.get_console 

function(**selector** *string*) - Добавляет **&lt;div class=&quot;console&quot;&gt;&lt;/div&gt;** после выбранного **selector** и возвращает этот блок как JQuery объект. Пример:

```html
<div id="some_box"></div>
<script>
	// init console box
	var console_box = iface.get_console("#some_box");

	// append some messages
	console_box.append("Some text");
	console_box.append("Some more text");
</script>
```

#### iface.formdata 

function(**form_name** *string*, **required_data** *object*) - Служит для валидации формы на стороне клиента перед ее отправкой. Пример:

```html
<form method="POST" name="test_form">
	Your name:
	<input type="text" name="form[name]">
	Your age:
	<input type="text" name="form[age]">
	<input type="submit">
</form>
<script>
	$(document["test_form"]).submit( function(){
		 // init checking
		var formdata = new iface.formdata (
			// pass form name
			$(this).attr("name"),
			// set required data and messages if values are empty
			{
				"form[name]" : "Please, enter your name!"
				"form[age]" : "Please, enter your age!"
			}
		);

		return formdata.check();
	});
</script>
```

#### iface.load_html

function(**html** *string*, **callback** *function*) - Вызов колбэка после загрузки заданного HTML. Пример:

```javascript
// Some HTML with images
var html = "<div> Some images: "
		+ "<img src='/path/to/image'>"
		+ "<img src='/path/to/another_image'>"
	+ "</div>";

// call function after HTML is loaded with images
iface.load_html (html, function(){
	alert("HTML is loaded!");
	document.write(html);
});
```

#### iface.interval

Служит для запуска действий в интервалах только при открытой вкладке браузера. Является заменой ф-ции window.setInterval, которая запускает колбэк независимо от того, открыто ли актуальное окно. Пример:

```javascript
// call action every second
var timer = iface.interval.set( 1000, function(){ 
	console.log( "Action executed!" );
}); 

// you can unset the interval like this
iface.interval.unset( timer );
```

### delete_from_array

function(**array** *array*, **needle** *string|number|array*) - Удаление элемента из массива.

Пример:

```javascript
var some_array = [1,2,3,4,5];

delete_from_array (some_array, 3);
// result: [1,2,4,5]

delete_from_array (some_array, [1,5]);
// result: [2,4]
```

### nl2br

function(**str** *string*, **is_xhtml** *boolean*) - Перевод новых строк на &lt;br&gt; (или на &lt;br /&gt; в случе **is_xhtml = true**)

### switchCheck

function(**selector** *string|object*, **obj** *string|object*) - Настройка статусов чекбоксов. 

Пример:

```html
Checkboxes:
<input type="checkbox" class="selected_items"> First
<input type="checkbox" class="selected_items"> Second
<br>
<a href="#" onclick="return switchCheck('.selected_items')">Click to check / uncheck all</a>
<br>
<input type="checkbox" onchange="switchCheck('.selected_items', this)"> Or via this checkbox
```

### scrollto_byid

function(**object_id** *string*, **duration** *integer|boolean*, **callback_function** *function*) - Прокрутка страницы к блоку с заданным ID. 
Пример:

```javascript
// scroll to div#test_id
scrollto_byid("test_id");

// scroll slower (500 is default duration)
scrollto_byid("test_id", 1000);

// scroll with callback
scrollto_byid(
	"test_id",
	true, // use default value
	function(){
		alert("Scrolled successfully!");
	}
);
```

### createCookie

function(**name** *string*, **value** *string*, **days** *integer*, **date** *string*) - Запись куки.

Пример:

```javascript
// create cookie for 5 days
createCookie("test", "some value", 5);

// create cookie with precise expiration date (UTC date)
createCookie("test", "some value", 0, new Date("2018-02-01 13:30:00").toUTCString());
```


### readCookie

function(**name** *string*) - Чтение куки.

Пример:

```javascript
var cookie_value = readCookie("cookie_name");
```

### readCookie

function(**name** *string*) - Чтение куки.

Пример:

```javascript
var cookie_value = readCookie("cookie_name");
```

### eraseCookie

function(**name** *string*) - Удаление куки.

Пример:

```javascript
eraseCookie("cookie_name");
```

### load_next_page_html

function(**backend_url** *string*, **next_page_class** *string*, **custom_selector** *string|object*, **callback** *function*) - Аяксовая постраничность. 
Пример:

```html
<div class="items">
	...
</div>
<div class="next_page">
	<button type="button" onclick="load_next_page_html('/path/to/next_page', 'next_page')">Load more...</button>
</div>
```

Если классы на странице повторяются, можно использовать this в качестве **custom_selector**. 
Пример:

```html
<div class="items">
	<div class="sub-items">
		<button type="button" onclick="load_next_page_html('/path/to/next_page', 'sub-items', this)">Load more...</button>
	</div>
</div>
<div class="items">
	<div class="sub-items">
		<button type="button" onclick="load_next_page_html('/path/to/next_page', 'sub-items', this)">Load more...</button>
	</div>
</div>
```

Также можно запустить колбэк после загрузки данных. 
Пример:

```html
<div class="items">
	<div class="sub-items">
		<button type="button" class="load_more">Load more...</button>
	</div>
</div>
<div class="items">
	<div class="sub-items">
		<button type="button" class="load_more">Load more...</button>
	</div>
</div>

<script>
	// bind an event
	$("body").on("click", "button.load_more", function(){
		load_next_page_html(
			'/url/path/to/next/page', // next page html url
			'sub-items' // next page block class
			this, // use closest sub-items to this object
			function(data){
				// data are already converted to $(data)
				// so you can work with it via JQuery
				data.find(".whatever").val("whatever");
			}
		);
	});
</script>
```