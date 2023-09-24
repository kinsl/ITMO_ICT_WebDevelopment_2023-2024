# Маршрутизация

В директории нашего проекта перейдём в файл `urls.py` и зарегистрируем маршруты для админ-панели и для нашего приложения:

```Python title="laboratory_work_2/urls.py"
--8<-- "laboratory_work_2/laboratory_work_2/urls.py:17:24"
```


Теперь в директории нашего приложения создадим файл `urls.py` и пропишем маршруты для каждого представления:

```Python title="flights/urls.py"
--8<-- "laboratory_work_2/flights/urls.py:17:31"
```

Для каждого маршрута пропишем значение аргумента `name`, чтобы можно было на них ссылаться.

Также для некоторых маршрутов мы использовали конвертеров типов. Например, `#!py3 <str:tab>` передаст в наше представление аргумент 
`tab` типа `#!py3 str`, а `#!py3 <int:flight_id>` передаст аргумент `flight_id` типа `#!py3 int`.