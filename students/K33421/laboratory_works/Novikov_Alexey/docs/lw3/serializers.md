# Сериализаторы

Перед тем как писать представления, нам нужно создать для них сериализаторы. 
Для этого создадим файл `serializers.py` в наших приложениях.

=== "adapter"

    ```Python title="Адаптеры"
    from rest_framework import serializers
    
    from apps.adapter.models import Adapter


    --8<-- "laboratory_work_3/src/apps/adapter/serializers.py:6:14"
    ```

    Один из самых базовых сериализаторов, основанных на модели. 

    Создадим атрибут `full_name` со значением `serializers.SerializerMethodField()`, и затем создадим метод 
    с тем же названием и приставкой `get`. В этом методе будем возвращать фамилию и имя. 

    Сделано это для того, чтобы выводить их в одном поле в итоговом JSON. Также будем возвращать id адаптера.

=== "oidc"

    ```Python title="JWT токены"
    from rest_framework import serializers


    --8<-- "laboratory_work_3/src/apps/oidc/serializers.py:4:8"
    ```

    В этом сериализаторе определим четыре поля: два текстовых и два с датой и временем. Этот сериализатор нужен для возврата 
    информации о JWT токенах после авторизации, и его нужно указать в настройках, пункт об этом есть [здесь](settings.md).

=== "survey"

    === "Вопросы"

        ```Python
        from rest_framework import serializers
        
        from apps.survey.models import SurveyQuestion
        from apps.adapter.models import Adapter
        from apps.adapter.serializers import AdapterSerializer
        from apps.oidc.models import ItmoIdProfile

    
        --8<-- "laboratory_work_3/src/apps/survey/serializers.py:10:27"
        ```

        В этом сериализаторе необходимо возвращать всю информацию, необходимую для генерации "формы" опроса: 
        все нужные компоненты, id вопросов, их названия, а также список привязанных к группе адаптеров при необходимости.

        Для этого создадим два новых поля с помощью методов: в первом будем просто возвращать человекочитаемое название 
        компонента (которое соответствует названию файла с компонентом, чтобы на фронте было проще), а во втором из контекста 
        (который в дальнейшем при использовании сериализатора "прокинем" из представления) получим запрос пользователя, чтобы 
        идентифицировать его. 

        Если тип используемого для этого вопроса компонента является AdapterSlider, то мы должны по учебной группе 
        пользователя получить список привязанных к ней адаптеров (используя ранее созданный сериализатор для адаптеров), 
        в ином случае будем просто возвращать пустой список.

    === "Ответы"

        ```Python
        from rest_framework import serializers
        from rest_framework.exceptions import ValidationError
        
        from apps.survey.models import SurveyQuestion

    
        --8<-- "laboratory_work_3/src/apps/survey/serializers.py:30:63"
        ```

        Особенностью сериализатора `SurveyAnswerSerializer` является то, что он будет использоваться не для выходных 
        данных, а для входных.

        Использовать будем всего 2 поля: `id` (id вопроса) и `value` (полученный ответ для этого вопроса). Первое поле будет 
        численным, а второе - JSON, так как JSON позволяет принимать практически любые встроенные типы данных, что нам и нужно.

        Для каждого из этих полей напишем валидатор. В случае `id` будем проверять существование вопроса с таким id. 

        В случае `value` будем проверять его тип и в зависимости от этого варьировать действия. Если получена строка или число, 
        вернём, как есть. Если получен список, то проверим, что каждый элемент этого списка является словарём и дополнительно 
        провалидируем этот список с помощью ещё одного сериализатора, специально написанного для AdapterSlider компонента, 
        возвращающего данные по-особенному. Если валидация успешна, то вернём данные, в любом ином случае (в том числе при получении 
        других типов данных) вернём ошибку валидации.