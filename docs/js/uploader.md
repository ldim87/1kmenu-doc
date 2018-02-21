# Загрузчик файлов

Для загрузки файлов используем ф-цию интерфейса [iface.file_uploader](http://vgit1.aynos.cz/jsdoc/1kmenu/iface.file_uploader.html)

В примере ниже указано, как можно сделать загрузку изображений. По умолчанию разрешены только файлы jpg, jpeg и png.
Единственный обязательный блок, необходимый для загрузчика в данном примере: 

```html
<div id="file_uploader_product_images"></div>
```

так как в конструкторе мы указале первым параметром:

> "product_images"

Остальные блоки мы используем в колбэках. Весь пример:

```html
<div id='uploaded_images' class="ib"></div>
<div id='upload_message' class="hidden bi ib">
	Потащите за фото, чтобы поменять их местами...
</div>
<div id="file_uploader_product_images"></div>

<script>
	// загружаем модуль загрузчика
	iface.js_lib.load('file_uploader').done(function(){
		// инициализируем загрузчик
		var uploader_images = new iface.file_uploader(
			// постфикс для блока загрузки
			// (заставляем выводить процесс загрузки в блоке "file_uploader_product_images")
			'product_images', 
			// настройки загрузчика
			{
				// после успешной загрузки каждого файла выполняется следующий callback
				file_process: function(uploaded){
					// можно посмотреть в консоли инфо загружееного файла
					// console.log('Uploaded file info: ', uploaded);

					// отключаем кэш для загруженных изображений
					var img_url = '/tmp/'+uploaded['target_name']+'?t='+Date.now();
					
					// оболочка для загруженных файлов (напр. для настроек CSS)
					// сразу же туда помещаем анимацию загрузки
					var wrapper = $('<span class="uploaded_image">').html(load_img);

					// помещаем оболочку в наш контейнер на странице
					var container = $("#uploaded_images").append(wrapper);

					// теперь ждем, когда вернется загруженная картинка
					$('<img src="'+img_url+'">').on('load', function(){
						// убираем иконку загрузки из оболочки и кладем туда все необходимые элементы 
						wrapper.html(
							// путь к загруженному файлу
							'<input type="hidden" name="files[]" value="' + uploaded['target_name'] + '">' +
							// ID загруженного файла
							'<input type="hidden" name="file_ids[]" value="' + uploaded['id'] + '">' +
							// название файла (как назывался на стороне клиента)
							'<input type="hidden" name="file_names[]" value="'+ uploaded['name'] +'">' + 
							// размер файла (возможна проверка на стороне клиента, но ей не стоит слишком доверять - может обманывать)
							'<input type="hidden" name="file_sizes[]" value="'+ uploaded['size'] +'">' + 
							// само изображение само собой
							'<img src="' + img_url + '">'	
						);
					});
				},
				// мы также можем выполнить какое-то действие после загрузки всех выбранных файлов
				on_uploaded: function(){
					// напр. мы можем сделать изображения сортируемыми
					$("#uploaded_images").sortable();
					$("#upload_message").removeClass('hidden');
				}
			}
		);
	});
</script>

<style>
	#uploaded_images .uploaded_image {
	    display: inline-block;
	    margin: 0 5px;
	}
	#uploaded_images .uploaded_image img {
	    display: block;
	    max-height: 200px;
		max-width: 300px;
	}
</style>
```