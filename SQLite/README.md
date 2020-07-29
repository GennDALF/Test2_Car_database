## Справочник автомобилей: SQLite реализация

### Введение
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;В файл ***initial.py*** вынесен весь код, отвечающий за работу с базой данных. В том числе, DDL скрипты для создания таблиц и триггеров, а также шаблоны запросов.<br/>Тестовый файл базы данных ***database.sqlite*** уже создан и заполнен несколькими записями.<br/><br/>  

### Схема
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;База данных состоит из двух таблиц **cars** и **history**, со связью один-ко-многим между ними. Таблица **history** хранит все изменения, происходящие с каждой записью в таблице **cars** – заполняется и очищается автоматически с помощью триггеров.<br/><br/>

### Описание методов API
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Для добавления одного или нескольких автомобилей возможно передать JSON строку. Альтернативой является ручное именование передаваемых спецификаций автомобиля или, что то же самое, передача словаря – в таком случае за одно обращение можно добавить только одну запись. В качестве фильтров функции `get_cars()` возможно использовать те же поля, что и для создания записи – параметры *`**filters`* и *`**car_specs`* проверяются на соответствие одному и тому же словарю (см. ниже).

| Метод | Параметры | Возвращаемый результат |
| :--- | :---: | :---: |
| `get_cars()` <br/> Вывод списка автомобилей | *`**filters`*<br/>Словарь для задания фильтров \*<br/>*`_and=True`*<br/>В значении *`True`* – логическое И для фильтров<br/>В значении *`False`* – логическое ИЛИ для фильтров | list <br/> |
| `add_car()` <br/> Добавление автомобиля(-ей) | *`entry="[]"`*<br/>Строка с данными в формате JSON<br/>*`**car_specs`*<br/>Словарь для задания спецификаций автомобиля \* | str <br/> |
| `del_car()` <br/> Удаление автомобиля | *`*plate_numbers`*<br/>Множество строк с номерами автомобилей для удаления | str |
| `get_stats()` <br/> Статистика по базе данных |   –   | JSON str \*\* |

\* поля для создания записи об автомобиле: помеченные `'r'` обязательны
```
{
  "plate": 'r',
  "brand": 'r',
  "model": 'r',
  "year": 'r',
  "color": 'r',
  "VIN": '',
  "power": '',
  "body": ''
}
```
\*\* структура JSON вывода функции `get_stats()`:
```
[
  {
    "total_entries": <int>,   # общее количество всех автомобилей в базе
    "first_entry": <str>,     # дата и время добавления первой записи
    "last_update": <str>,     # дата и время добавления последней записи
    "total_queries": <int>,   # количество всех совершённых обращений
    "last_query": <str>       # дата и время последнего обращения
  }
]
```
Все значения времени и даты выводятся в следующем формате: "HH:MM DD.mm.YYYY", например "15:40 03.05.2020"
<br/><br/>

### Примеры
```
>>> car1 = {
...     'plate': "l924jf",
...     'brand': "GAZ",
...     'model': "2410",
...     'year': "1987",
...     'color': "white",
...     'power': "70",
... }
>>> add_car(**car1)
Successful

>>> car2 = '[{"plate": "e456az", "brand": "Lada", "model": "Kalina", "year": "2013", ' \
...        '"color": "grey", "VIN": "ASDFG8YQ34WIR3","body": "sedan"}]'
>>> add_car(entry=car1)
Successful

>>> get_cars(color="white", brand="Lada")
[{'plate': 'v483qw', 'brand': 'Lada', 'model': 'Kalina', 'year': '2013', 'color': 'white', 'VIN': 'ASDFG8YQ34WIR3S', 'body': 'sedan'}, 
 {'plate': 'g631qe', 'brand': 'Lada', 'model': 'Kalina', 'year': '2014', 'color': 'white', 'VIN': 'ADFG89QWERUIJK2', 'body': 'sedan'}]

>>> get_cars(color="white", brand="Lada", _and=False))
[{'plate': 'l924jf', 'brand': 'GAZ', 'model': '2410', 'year': '1987', 'color': 'white', 'power': '70'}, 
 {'plate': 'v483qw', 'brand': 'Lada', 'model': 'Kalina', 'year': '2013', 'color': 'white', 'VIN': 'ASDFG8YQ34WIR3S', 'body': 'sedan'}, 
 {'plate': 'g631qe', 'brand': 'Lada', 'model': 'Kalina', 'year': '2014', 'color': 'white', 'VIN': 'ADFG89QWERUIJK2', 'body': 'sedan'}, 
 {'plate': 'e456az', 'brand': 'Lada', 'model': 'Kalina', 'year': '2013', 'color': 'grey', 'VIN': 'ASDFG8YQ34WIR3S', 'body': 'sedan'}]

>>> del_car("l924jf", "a222aa", "l098we")
Warning: ['a222aa', 'l098we'] plate number(-s) have not been found
```