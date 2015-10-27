# Контентные блоки

Контентные блоки используются для избежания дублирования повторяющихся блоков или элементов в контенте или представлениях сайта.

> Аналогичный функционал в MODX называется чанками(chunks), а в WordPress - шорткодами(shortcodes).

## Определение блока

Для определения контентного блока - создайте соответствующую запись в разделе "Контентные блоки" панели администрирования:

- **Название** - используется для визуальной идентификации блока
- **Ключ** - тот ключ, по которому будет вызываться этот блок внутри контента
- **Значение** - собственно то, на что будет заменяться оператор вызова контентного блока непосредственно в контенте
- **Предварительная загрузка** - этот блок контента будет загружаться при каждом запросе к сайту. Включите эту опцию для наиболее часто используемых блоков.

Контентный блок может принимать дополнительные параметры, например:

```
Это простой блок, который сейчас выведет отформатированный параметр
[[+paramName:format,format_attribute1,'format_attribute_n']].
А здесь мы просто выведем значение параметра как есть [[+anotherParam]].
```

Здесь:

- **paramName** - название параметра(именно это название нужно будет передавать вместе соператором вызова блока). Плюс перед названием параметра обязателен.
- **format** - функция форматирования(опционально)
- **format_attribute1** и **format_attribute_n** - произвольное количество значений параметров, которые будут передаваться функции форматирования. Разделитель - запятая. Если значение в ковычках - оно трактуется как строка(string), без ковычек - как число с плавающей точкой(float).
- **[[+anotherParam]]** - название параметра. В данном примере он выводится "как есть".

## Использование контентного блока

В контенте вставляете оператор вызова блока, который имеет следующий формат:

```
[[$contentBlockKey paramName='paramValue' anotherParam=3.14]]
```
Здесь:
- **contentBlockKey** - замените на ключ вашего контентного блока. **Важно:** знак доллара перед ключем обязателен.
- **paramName** - название параметра
- **paramValue** - значение параметра. Если оно в одинарных ковычках - трактуется как строка(string). Иначе - как число с плавающуй точкой(float).

## Рекурсивные контентные блоки
Допускается в контентном блоке вызывать другой контентный блок. Его вызов должен осуществляться точно таким же образом, как и вызов в контенте страницы. При этом будут обработаны все переданные параметры и другие контентные блоки, если они содержатся во вложенных контентных блоках.

**Недопустимо вызывать контентный блок внутри себя, а так же вызывать родительский контентный блок внутри вызываемого дочернего!**
## Использование контентного блока в шаблоне
Хорошим тоном считается вынесение всей текстовой информации, включая статичные блоки верстки, из шаблонов в контентные блоки. Это позволит контент-менеджерам сайта гораздо проще и нагляднее редактировать эту информацию.
Для вывода контентных блоков в шаблонах необходимо вызвать специальный хелпер-метод:
```php
<?= \app\modules\core\helpers\ContentBlockHelper::getChunk($key, $params = [], yii\base\Model $model = null) ?>
```
Он принимает в качестве параметров 3 значения:
- ключ чанка;
- массив параметров;
- модель, в чьем view-файле мы вызываем данный метод.

Обязательным является только первый параметр - ключ чанка. В этом случае, минимальный вызов метода будет выглядеть так:
```php
<?= \app\modules\core\helpers\ContentBlockHelper::getChunk('chunkKey') ?>
```
Если в вызываемом контентном блоке имеются плейсхолдеры, которые необходимо заменить реальными значениями, то необходимо вторым параметром передать ассоциативный массив вида:
```php
[
    'paramName' => 'string param value',
    'param2Name' => 3.14,
]
```
В качетсве третьего необязательного параметра в метод передается модель (Product, Category, Page), в чьем view файле вызывается хелпер-метод.   Она будет использована для кеширования результатов рендера чанка. Модель имеет смысл передавать только если обрабатывается чанк с параметрами, причем значения параметров зависят от конкретной страницы, в противном случае этот параметр можно опустить.
Полный вызов метода выглядит следующим образом:
```php
<?= \app\modules\core\helpers\ContentBlockHelper::getChunk(
    'chunkKey',
    ['paramName' => 'string param value','param2Name' => 3.14,],
    $model
) ?>
```
Если в вызываемом чанке вызываются другие чанки, то они также будут отрендерены.
## Вызов формы в контенте
По аналогии с контентными блоками в контенте можно вызывать созданные в административной части сайта формы.
Минимальный вызов выгладит так:
```php
[[%3]]
```
где:
- % - обязательный префикс;
- 3 - id созданной в админке формы.
Вызов вида:
```php
[[%3#form-id]]
```
Вызовет форму, установив в качестве id - form-id
Вызов вида:
```php
[[%3#form-id;isModal]]
```
Отрендерит модальную форму с идентификатором form-id