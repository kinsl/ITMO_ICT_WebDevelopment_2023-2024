# Задание №4

???+ question "Задание"

    Реализовать двухпользовательский или многопользовательский чат. Реализация
    многопользовательского чата позволяет получить максимальное количество
    баллов.

    Реализовать с помощью протокола TCP – 100% баллов, с помощью UDP – 80%.  
    Обязательно использовать библиотеку `threading`.

    Для реализации с помощью UDP, `threading` использовать для получения
    сообщений у клиента.  
    Для применения с TCP необходимо запускать клиентские подключения И прием
    и отправку сообщений всем юзерам на сервере в потоках.  
    Не забудьте сохранять юзеров, чтобы потом отправлять им сообщения.

=== "Сервер"

    ```Python title="server.py"
    --8<-- "laboratory_work_1/task4/server.py"
    ```
    
    Создадим `#!py3 class Client`, в атрибутах которого будут храниться клиентский сокет, клиентский адрес, 
    булевое значение `is_closed` (`True`, если сокет закрыт. Необходимо для понимания других потоков, 
    что с этим сокетом больше работать не нужно). Также поставим таймаут для сокета, чтобы была возможность выйти 
    с помощью ++ctrl+c++.

    Создадим `#!py class ClientThread`, наследуемый от `threading.Thread`, к его атрибутам добавим объект 
    класса `#!py3 Client`, с которым этот поток работает, событие `threading.Event` - `should_stop`, которое будем проверять 
    для завершения потока, и `threading.Lock` для того, чтобы в нужный момент блокировать поток при доступе к общим изменяемым данным.

    В этом классе переопределим метод `#!py3 def run`, который запускается при использовании метода `.start` у потока.

    Первым делом этот метод запускает метод `broadcast`, отвечающий за отправку сообщений всем пользователям, 
    о его работе поговорим чуть позже. 
    Далее метод запускает цикл `#!py3 while`, который работает до тех пор, пока `Event` не скажет о том, что потоку следует 
    остановиться.

    Дополнительно проверяем статус клиента `is_closed`, на случай, если сокет клиента уже закрылся другим потоком.

    Получаем сообщение клиента. Если оно пустое, значит, клиент отключился, и мы должны удалить его, вызвав метод `remove_client`, 
    о нём тоже позже.

    Если сообщение есть - вызываем метод `broadcast` с текстом сообщения.

    Дополнительно проверяем ошибку `OSError` на случай, если поток всё-таки решится обратиться к уже закрытому сокету, 
    и ошибку `ConnectionResetError` на случай, если мы будет ожидать сообщение от клиента в момент, когда он отключается.

    Теперь рассмотрим метод `remove_client`. В этом методе меняем атрибут `is_closed` на True, 
    чтобы другой поток больше не использовал этот сокет. Также отправляем сообщение всем пользователям о том, что этот клиент 
    покинул чат. 
    С помощью `Lock` удаляем клиента из списка клиентов. Закрываем сокет и включаем событие `should_stop` для текущего потока.

    В методе `broadcast` с помощью `Lock` делаем копию списка клиентов. Пробегаемся в цикле `#!py3 for` по всем клиентам 
    и отправляем всем клиентам, не являющимся тем клиентом, вызвал этот метод, сообщение с необходимым текстом.

    В основной части программы создаём сокет сервера, так же выставляем для него таймаут. Создаём пустой список клиентов 
    и создаём объект `Lock`. 
    В цикле `#!py3 while` принимаем соединение клиента, на основе полученных данных создаём объекта `Client`. 
    С помощью `Lock` добавляем этого клиента в список. 
    На основе объектов `Client` и `Lock` создаём наш кастомный поток и запускаем его.

    Если программа завершается нажатием на ++ctrl+c++, закрываем все клиентские сокеты, также изменяя их атрибут `is_closed`, 
    и затем закрываем сокет сервера.
    

=== "Клиент"

    ```Python title="client.py"
    --8<-- "laboratory_work_1/task4/client.py"
    ```

    Создадим `#!py class ReceiveThread`, наследуемый от `threading.Thread`, к его атрибутам добавим объект 
    сокета и булевое значение `server_closed`, которое становится True, если сервер закрывает соединение.

    В этом классе переопределим метод `#!py3 def run`, который запускается при использовании метода `.start` у потока.

    В этом методе запускаем цикл `#!py3 while`, пока не поступило команды завершить потоки (глобальное булевое `stop_threads`, 
    становится True, когда клиент завершает работу нажатием ++ctrl+c++) и пока ещё не известно, что сервер отключился.

    В цикле получаем сообщение от сервера. Если оно пустое, значит сервер закрыл соеднение. Выводим об этом сообщение клиенту 
    и просим завершить сессию. Также меняем значение атрибута `server_closed` на True.

    В основной части программы инициализируем сокет, подключаем к сокету сервера, ставим ему таймаут. 
    С помощью сокета создаём объект нашего кастомного потока и запускаем его.
    Запускаем цикл `#!py3 while`, работающий до тех пор, пока сервер не закроет соединение или клиент на завершит программу 
    нажатием на ++ctrl+c++. В самом цикле получаем ввод пользователя через `#!py3 input()` и отправляем полученное сообщение на сервер.

    При завершении программы меняем булевое `stop_threads` на True и дожидаемся завершения потока "приёмника" с помощью join. 
    Затем закрываем сокет.

    === "Клиент 1"

        <figure markdown>
          ![Клиент 1](https://kinsl.github.io/ITMO_ICT_WebDevelopment/img/lw1/task4/client1_console.png)
        </figure>

    === "Клиент 2"

        <figure markdown>
          ![Клиент 2](https://kinsl.github.io/ITMO_ICT_WebDevelopment/img/lw1/task4/client2_console.png)
        </figure>

    === "Клиент 3"

        <figure markdown>
          ![Клиент 3](https://kinsl.github.io/ITMO_ICT_WebDevelopment/img/lw1/task4/client3_console.png)
        </figure>