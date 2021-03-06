# Yii2-Simple-Filter
Yii2 Simple Filter Module

Yii2 Simple Filter Module v0.9.1dev
#####Не совместимо с более ранними версиями, т.к. изменились названия методов, а также способ установки параметров! 

Видео: http://www.youtube.com/watch?v=Vah2j5WzXIs

####Принцип работы:
Модуль делает 2 вещи: 
- рисует в View файле список с 'чекбоксами', которыми можно задавать параметры фильтра;
- на основе выбранных параметров фильтра выводит данные указанной модели в указанный дополнительный Ajax View, в основном View контроллера. Следует знать, что этим модулем в Ajax View данные только доставляются, без их визуального отображения. Отобразить данные пользователю необходимо с помощью таблицы или виджета GridView.



####Установка:

Видео установки и настройки на чистое Yii2 Base приложение: http://www.youtube.com/watch?v=Wnn_xVcTun0

Установите модуль через [composer](http://getcomposer.org/download/)

Выполните в терминале следующую команду:
```
$ php composer.phar require --prefer-dist sanex/yii2-simple-filter "dev-master"
```
или добавьте
```
"sanex/yii2-simple-filter": "dev-master"
```
в секцию `require` файла `composer.json`.

Далее, добавьте `'enablePrettyUrl' => true` в настройках `urlManager` в конфигурационном файле Yii2.
######Модуль не работает с параметром `'enablePrettyUrl' => false`!


####Как использовать?

####Контроллер
В контроллере, к которому пренадлежит основной View, в котором мы хотим вывести чекбоксы и подключить Ajax View, необходимо создать объект с инстансом модуля и задать ему параметры:
```
use sanex\simplefilter\SimpleFilter;

...

$model = new Catalog;

$ajaxViewFile = '@sanex/catalog/views/catalog/catalog-ajax';

$filter = SimpleFilter::getInstance();
$filter->setParams([
    'model' => $model,
    'query' => $query,
    'useAjax' => true,
    'useCache' => true,
    'useDataProvider' => true,
]);
```

`$ajaxViewFile` - алиас (или путь) Ajax View файла, куда необходимо передавать результаты фильрации. Этот файл с Ajax видом необходимо создать, перед использованием фильтра.

Параметры метода `setParams()`:

`model` - модель, данные которой неотходимо отфильтровать;

`query` - (опционально) - объект \yii\db\ActiveQuery, см. ниже;

`useAjax` - (опционально, если не задано - `true`) можно выбирать между Ajax или не-Ajax фильтрацией, путем задания значения этого параметра в (bool) `true` или `false`;

`useCache` - (опционально, если не задано - `false`) если (bool) true - возвращает кешированные данные, если (int) число - длительность кеширования в секундах;

`useDataProvider` - (опционально, если не задано - `false`) если (bool) true - возвращает результат фильтрации в Ajax View в виде dataProvider, если (bool) false или не задано - возвращает выборку в виде модели;

По умолчанию, запрос, сформированный фильтром выглядит вот так: `SELECT COUNT(*) FROM 'catalog' WHERE 'color' IN ('Green', 'Red')`

Чаще всего, возникает необходимость расширить запрос дополнительными условиями. Для этого необходимо установить параметр `query` метода `setParams` объекта Фильтр. В качестве значения этого параметра необходимо задать объект `\yii\db\ActiveQuery`, который будет сожержать изначальные условия для выборки, поверх которых будет производиться фильтрация.

```
$query = new \yii\db\ActiveQuery($model);
$query->select(['id', 'name', 'size', 'price', 'country'])->where(['country' => 'Canada'])->orderBy(['price' => SORT_ASC]); 
```

Методом `limit()` объекта \yii\db\ActiveQuery можно задать параметр `'pagination' => ['pageSize' => $this->limit]`, (кол-во записей на странице) у объекта ActiveDatapProvider.

```
$query = new \yii\db\ActiveQuery($model);
$query->limit(25); 
```
Если метод `limit()` не указан, то `limit` для всех запросов устанавливается в стандартное значение - 50 записей на странице.

Если с данным фильтром возникла необходимость в создании кастомного виджета постраничной навигации `pagination`, то необходимый для работы виджета сдвиг `offset` можно получить из GET-параметра `page`. Следует учесть, что для значений `page` в `0` и `1`, значение `offset` будет равняться `0`, для остальных значений - по формуле `(page - 1) * limit`.

####Основной вид
В основном View необходимо вызвать метод Фильтра `setFilter()`, в котором задать параметры фильтрации.
######Значения `property` должны быть такими же, как имена столбцов таблице БД, а `values`, совпадать с содержимым этих столбцов!

```
$filter->setFilter([
    [
        'property' => 'color',
        'caption' => 'Color',
        'values' => [
            'Red',
            'Green',
            'Blue',
            'Black'
        ],
        'class' => 'horizontal'
    ],
    [
        'property' => 'size',
        'caption' => 'Size',
        'values' => [
            '45x45',
            '50x50',
            '60x60'
        ]
    ]
]);
```

Вы можете задать дополнительные классы для checkbox'ов, путем задания свойства `class`. Данный фильтр содержит 2 стиля для checkbox'ов фильтра: класс `horizontal` и класс `vertical` для вертикального расположения. Если свойство `class` не установлено, то будет использоваться стиль `horizontal` по умолчанию.
Значение свойства `class` может быть строкой:
`'class' => 'horizontal additional class'` 
или массивом: 
`'class' => ['vertical', 'additionalClass']`


В том месте, где необходимо вывести Ajax View с отфильтрованными данными, необходимо вызвать метод Фильтра renderDataView(), с заданными параметрами:
```
$filter->renderDataView($ajaxViewFile, ['testParam' => $testParam]);
```

`$ajaxViewFile` - Ajax view файл, заданный в контроллере;

`(array)$ajaxViewParams` - параметры, которые будут переданы в AjaxView файл. Принцип работы полностью аналогичен 2-му параметру метода render($view, $params = []).


####Ajax View
В Ajax View, отфильтрованные данные передаются в переменной `$simpleFilterData`.
Далее они могут быть помещены в GridView или на их основе, через цикл, может быть построена таблица.