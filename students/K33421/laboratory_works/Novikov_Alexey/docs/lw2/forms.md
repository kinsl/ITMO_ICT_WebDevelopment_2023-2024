# Формы

Перед тем как писать представления, нам нужно создать некоторые формы. Для этого создадим файл `forms.py` в нашем приложении.

=== "Регистрация"

    ```Python
    from django import forms
    from django.contrib.auth.forms import UserCreationForm
    from django.contrib.auth.models import User


    --8<-- "laboratory_work_2/flights/forms.py:8:24"
    ```

    В Django уже существует встроенная форма для регистрации новых пользователей - `UserCreationForm`. Но она состоит 
    только из 3 полей: никнейм, пароль и подтверждение пароля.  
    Создадим новый класс, наследуя эту форму.

    В мета-классе напишем, что форма будет основана на модели `auth.models.User` и перечислим необходимые нам поля: 
    фамилия, имя, никнейм, почта, пароль и подтверждение пароля.

    Напишем, какие виджеты хотим использовать для наших новый полей и пометим их обязательными.

    Переопределим метод `save`. В начале вызовем метод `save` из наследуемого класса, но пометим его как `commit=False`, 
    чтобы не отправлять запрос в БД. Получим из словаря `cleaned_data` (данные, прошедшие валидации) значения наших новых полей 
    и присвоим их атрибутам нашего нового объекта. Сохраним объект и вернём его.

=== "Отзыв"

    ```Python
    from django import forms
    
    from .models import Feedback


    --8<-- "laboratory_work_2/flights/forms.py:27:30"
    ```

    Самая базовая форма, основанная на модели. Нам требуются только 2 поля: текст отзыва и рейтинг.