# Query REST

Формат query параметров REST API запросов для эффективной разработки фронта и бэка.
Решает проблему согласованности и выборки одним запросом связанных данных, сохраняя простоту REST.

Спецификация определяет несколько query параметров url запроса и не ограничивает применение 
дополнительных. Параметры опциональные. Именование допустимо изменять. 

```
GET /objects?fields=name,author(age)&search[author.age]=18;30&sort=-name&limi=10&skip=0&lang=ru
```

Формат имеет минимальную![v1](v1.png) версию и расширенные ![v2](v2.png) ![v3](v3.png).
При частичной реализации опций второй версии, указывается первая версия с перечислением номеров
опций, например `QueryREST v1.fields-5.7.search-3.4.5`. Если реализуются не все опции первой 
версии, то указывается нулевая версия формата.

__[`fields`](fields.md)__ - что выбрать.
  1. [Свойства по умолчанию](fields.md)![v1](v1.png)
  2. [Все свойства](fields.md)![v1](v1.png) - `*`
  3. [Выборочные свойства](fields.md)![v1](v1.png)   - `prop1, prop2(prop3)`
  4. [Свойства списка](fields.md)![v2](v2.png) - `items(prop1), count`
  5. [Исключение свойства](fields.md)![v2](v2.png) - `!prop1` 
  6. [Зависимость от типа объекта](fields.md)![v3](v3.png) - `id, author(name, manager:rating)`
  7. [Рекурсивные шаблоны](fields.md)![v3](v3.png)  - `comments(text, children(^))`
  
__[`search`](#search)__ - условия выборки.
  1. [Равенство значению](search.md)![v1](v1.png) - `search[prop1]=value`
  2. [Вхождение в строку](search.md)![v2](v2.png) - `search[prop1]=*value`
  3. [Вхождение с сначала строки](search.md)![v2](v2.png) - `search[prop1]=^value`
  4. [Полнотекстовый поиск](search.md)![v2](v2.png) - `search[prop1]=~value`  
  5. [Неравенство значению](search.md)![v2](v2.png) - `search[prop1]=!value` 
  6. [Равенство значению со спец. символами](search.md)![v2](v2.png) - `search[prop1]="value-with!~^*<>;|`
  7. [Больше, меньше значения](search.md)![v2](v2.png) - `search[prop1]=>value`, `search[prop1]=<value`
  8. [Больше или равно, меньше или равно](search.md)![v2](v2.png) - `search[prop1]=>>value`, `search[prop1]=<<value`
  9. [Диапазон значений](search.md)![v2](v2.png) - `search[prop1]=min;max`
  10. [Интервал значений](search.md)![v2](v2.png) - `search[prop1]=min~max`
  11. [Отсутствие свойства или значения](search.md)![v2](v2.png) `search[prop1]=null`
  12. [Выполнение любого условия](search.md)![v2](v2.png) - `search[prop1]=exp1|exp2`
  13. [Выполнение всех условий](search.md)![v2](v2.png) - `search[prop1]=exp1&exp2`
  14. [Множество любых условий]()![v2](v2.png) - `search[prop1]=value1|min2;max2|min3~max3|!value4|*value5|^value6|null`   
  15. [Условие по вложенному свойству](search.md)![v1](v1.png) - `search[prop1.prop2]=value`
  16. [Условие по свойству в неопредленной вложенности]()![v3](v3.png) - `search[prop1..parent.title]=value`
  17. [Условие вывода свойства prop1](search.md)![v3](v3.png) - `search.prop1[prop2]=value`
  
__[`sort`](#sort)__ - сортировка.
  1. [По убванию]()![v1](v1.png) - `sort=-name`
  2. [По нескольким свойствам]()![v1](v1.png) - `sort=-name,date`
  3. [По вложенным свойствам]()![v1](v1.png) - `sort=contacts.address.street`
  4. [Сортировка в множественном свойстве]()![v3](v3.png) - `sort.contacts=address.street`
  
__[`limit`](#limit-skip)__ - ограничение количества.
  1. [По умолчанию]()![v1](v1.png)
  2. [Не больше указанного]()![v1](v1.png) - `limit=10`
  3. [В множественом свойстве]()![v3](v3.png) - `limit.comments=10`

__[`skip`](#limit-skip)__ - с какой позиции ограниченное количество.
  1. [С указанной]()![v1](v1.png) - `skip=10`
  2. [В множественом свойстве]()![v3](v3.png) - `skip.comments=10`
  
__[`depth`](#depth)__ - ограничение вложенности.
  1. [По умолчанию]()![v3](v3.png)
  2. [Для свойства]()![v3](v3.png) - `depth.parent=10`

__[`lang`](#lang-unit-version)__ - язык при мультиязычности.
  1. [Для всех свойств]()![v2](v2.png) `lang=en`
  2. [Все варинты языков]()![v2](v2.png) - `lang=*`
  3. [Несколько языков]()![v3](v2.png) - `lang=ru,en`
  4. [Для одного свойства]()![v3](v3.png) - `lang.title=en`

[Сложные запросы]()

## fields

Выбираемые данные. 
Названия свойств объектов и их отношений, которые нужно получить одним запросом.

> Альтернативные названия: fields, props, select

В классическом REST API структура ответа жестко определяется сервером. Клиент получает лишние 
данные, либо сталкивается с нехваткой нужных. Например, связанные объекты нужно запрашивать 
отдельными запросами, имея их идентфикатор. Обработка параметра `fields` решает эту проблему. 
Одним запросом можно выбрать все необходимые связанные данные и не запрашивать лишние.

_Выборка пользователей с указанием нужных свойств_
```
GET /users?fields=id,email,avatar(url,extension)
```
```json
{
  "id":1,
  "email":"some@mail.com",
  "avatar":{
    "url": "/uploads/1928-212/5c2f3ed1fee590496c63759f.png",
    "extension": "png"
  }
}
```

## search

Условие выборки.

> Альтернативные названия: search, search, where

Параметром условия выборки указываются значения полей и способы сравнения значения со свойствами
объекта, напрмиер "больше или равно", "вхождение в диапазон", "перечисление значений" и другие. 
Допустимо указывать виртуальные поля, по которым формируются сложные условия на стороне бэкенда. 
Допустимые поля и условия для них определяет бэкенд. Назанчение параметра - передать бэку 
информацию, из которой будет формироваться полноценное условие поиска в базе данных.

Параметром можно указать несколько условий на разные поля. Поэтому применяется синтаксис квадратных
скобок. Формат не определет как логически услвоия будут объединться - И, ИЛИ, НЕ. Это зависит от
реализации сервера. Обычно улсовия объединяются через И. Но на усмотрнения бэкенда выборочные 
условия объдинять в соответсвии со спецификой запроса.

Условие равенства `search[prop1]=value` сервер может реализовывать по любой логике, не обязательно
строгим равенством00. Потомучто параметр search не стремиться определить все возможные условия, в ином 
случаи он станет сложным для использования и интерпретации бэком. 

> @todo: Отдельным параметром передавать логику объединения условий. Например 
```logic=price && (title || age)```

Условие может быть как на выбираемое множество объектов, так и на конкретное свойство объекта 
(обычно тоже множественное). Например, у товара множество точек продаж, выбирая сам товар, можем 
ограничить список точек продаж по определенному условию.

_Выборка товара с условием на него_
```
search[price]=100;200  // Цена в диапазоне от 100 до 200 включительно
search.stores[title]=*Magnit  // Магазины отфильтровать по названию - содержат Magnit
```

Через точку указывается, к какому свойству применить условие фильтра. Будет отфильтровано
само свойство у выбранных объектов. Если свойство не указано, то фильтр применяется к множеству 
запрашиваемых объектов. В квадратных скобках указывается свойство (или виртуальное поле), по 
которому будет сформировно условие фильтра.

## sort

Сортировка списков.

> Альтернативные названия: order

Параметром сортировки перечисляются поля с указанием направления сортировки.
Сортировать можно выбираемые объекты и их множественные свойства. Для сортировки свойств в названи
параметра sort через точку указывается свойство, которое сортировать. Свойства, по которым 
сортировать в строковом значении параметра.

_Сортировка товара и его свойства магазинов_
```
sort=-price,date // Сортировка по убыванию цены и возаратсанию даты
sort.stores=title // Список магазинов отсортировать по названию магазинов
```

## limit, skip

Ограничения списков

> Альтернативные названия: count, offset

Лимит и сдвиг ограничивают выборку объектов. Можно ограничить выборку множественных свойств объекта.

_Ограничение выбираемых товаров и их свойства магазинов_
```
limit=100
limit.stores=10

skip=10
skip.stores=0
```

## depth

Ограничение вложенности (глубины) при выборке иерерхических структур или рекурсивных связей.
Обычно указывается для свойств объекта.

_Выбор дерева страниц 2 уровня и комменответов на 4 уровня вложенности_
```
GET /pages?depth.chilren=2&depth.comments=4
```

Отсутствие параметра `depth` может трактоваться как выборка плоских данных, а при наличии depth c 
любым значением структуировать их в дерево, в качестве древовидных отношений бэкенд применятся
установки по умолчанию, например свойство children.

## lang, unit, version

Параметр языка для случаев с мультиязычными свойствами, чтобы вернуть объекты со свойствами
в нужном языке. Можно указать разные языки для конкретных свойств объекта. 

_Язык для товаров и названия магазинов_
```
lang=en // язык для всех свойств
lang.stores.title=ru  // язык для заголовка магазинов
```

Если указать all, то сервер должен вернуть объекты на всех языках и свойства на всех языках. В 
том случаи структура свойства зависит от модели. Предполагается, что образуется вложенность

```json
{
   "title": {
     "ru": "Заголвоок",
     "en": "Title",
     "it": "Testata"
   }
}
```

Параметр языка предагается как пример, по его подобию можно указывать валюту `unit` для цен, версию
изменений `ver` и другие.

## {custom}

Кастомные параметры предлагается реализовывать по образу и подобию предложенных форматов. 
Применять точки в названии для определений условий на соответсвующие свойства объектов.
Формат у значения параметра может быть любым, так как зависим от назначения, учитывать толкьо 
экранирование спец. символов URL.