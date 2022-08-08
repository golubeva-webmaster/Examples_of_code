upload.vue - компонент загрузки изображений или файлов, в зависомости от того, что передано в качестве входного параметра
Загрузка изображений вызывается так:
```html
              <files-upload
                :card-title="Загрузка изображений">
                :input-label="Изображения">
                :card-icon="`mdi-camera-plus-outline`"
                :files="item.images"
                :type="`image`"
              />
              <files-upload
                :card-title="Загрузка файлов
                :input-label="Файлы
                :card-icon="`mdi-file-document-plus-outline`"
                :files="item.files"
                :type="`file`"
              />
```
  item.images ={path: string, name: string}[]
  
  upload_drop_down.vue - тот же самый компонет, но с возможностью загрузки файлов при помощи DropDown
