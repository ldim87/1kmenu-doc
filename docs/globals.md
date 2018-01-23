# Глобальные переменные

## $GLOBALS['arr_url_go']

Этот массив содержит УРЛ путь, разбитый знаком "/"

> Напр. при запросе **//1000.menu/test/my/request**  
> массив выглядит так: **Array ('test', 'my', 'request')**

## $GLOBALS['now_time']

Unixtime сервера

## $GLOBALS['now_time_tz_offset']

Unixtime с учетем временного пояса пользователя

## $GLOBALS['session']

Данные сессии пользователя

## $GLOBALS['user']

Данные пользователя актуальной сессии

## $GLOBALS['param_general']

Массив с глобальными настройками. Загружается из файла **/sys/vars.php**

## $GLOBALS['param']

Массив с параметрами сайта. Собирается из нескольких частей. Имеющиеся переменные переписываются новыми в следующем порядке:

1. [Массив **$GLOBALS['param_general']**](/globals?id=globals39param_general39)
1. [Файл настроек **/sys/param.ini**](/folder_struct?id=%d0%a4%d0%b0%d0%b9%d0%bb%d1%8b)
1. [Настройки из базы данных](http://vgit1.aynos.cz/phpdoc/1kmenu/index.html#method_get_param_from_db)

## $GLOBALS['body']

Массив с настройками шаблона страницы

### $GLOBALS['body']['tema']

Название шаблона страницы. Сценарий шаблона выполняется через файл:
> **/sys/tema/*{название_шаблона}*.html**

### $GLOBALS['body']['title']

Тайтл страницы

### $GLOBALS['body']['header']

Заголовок страницы

### $GLOBALS['body']['description']

Описание страницы

### $GLOBALS['body']['arr_history_link']

Массив с хлебными крошками. Добавление ссылки в хлебные крошки:

```php
<?
$GLOBALS['body']['arr_history_link'][] = '<a href="/test">Test</a>';
?>
```

## $GLOBALS['head']['value']

Добавление любого контента в &lt;head&gt; страницы, напр. дополнительных стилей или скриптов:

```php
<?
$GLOBALS['head']['value'] .= '<link rel="stylesheet" type="text/css" href="//path/to.css">';
?>
```

## $GLOBALS['menu_link']['this']

Массив с ссылками в администраторском меню. Если нужно добавить ссылку для администратора, касающуюся данной страницы, то это можно сделать следующим образом:

```php
<?
$GLOBALS['menu_link']['this'][] = '<a href="/test/admin">Админка чего-либо</a>';
?>
```

## $GLOBALS['xrlocale']

Объект для загрузки мультиязычных сообщений. Более подробно и мультизячности читай [здесь](/globals/xrlang).

## $GLOBALS['sql_debug']

Отладка SQL запросов. Включается с помощью УРЛ параметра **sql_stats_debug=1**

> Напр.: //1000.menu/test?**sql_stats_debug=1**

Конструктор "**/index.php**" отображает для администраторов с минимальным [уровнем **7**](http://vgit1.aynos.cz/phpdoc/1kmenu/index.html#method_user_admin) все запускаемые SQL запросы после отрисовки шаблона страницы (в самом низу). Для скриптов, запускаемых через "**/script.php**", необходимо вручную выписывать переменную **$GLOBALS['sql_debug_arr']**, в которую собираются все запускаемые SQL запросы.

## $GLOBALS['hide_stats']

В случае TRUE, запрещает вывод блока со статистикой выполнения сценария конструктором "**/index.php**" в самом низу страницы включая список выполняемых SQL запросов с помощью [$GLOBALS['sql_debug']](/globals?id=globals39sql_debug39)