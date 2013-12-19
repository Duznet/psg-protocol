# PSG-protocol (Platform shooter game protocol)

Протокол обмена сообщениями между клиентом и сервером

Сообщения передаются в формате JSON методом POST

## Содержание

* [PSG-protocol (Platform shooter game protocol)](#psg-protocol-platform-shooter-game-protocol)
    * [Содержание](#Содержание)
    * [Формат сообщений](#Формат-сообщений)
        * [Запрос (Request) ](#Запрос-request)
            * [Пример](#Пример)
        * [Ответ (Response) ](#Ответ-response)
            * [Базовые коды ошибок, расположенные по убыванию приоритета](#Базовые-коды-ошибок-расположенные-по-убыванию-приоритета)
                * [Примечание](#Примечание)
            * [Пример ответа в случае успеха](#Пример-ответа-в-случае-успеха)
            * [Пример ответа в случае ошибки](#Пример-ответа-в-случае-ошибки)
    * [Действия](#Действия)
        * [startTesting](#starttesting)
        * [signup](#signup)
        * [signin](#signin)
        * [signout](#signout)
        * [sendMessage](#sendmessage)
        * [getMessages](#getmessages)
            * [Пример массива messages](#Пример-массива-messages)
        * [uploadMap](#uploadmap)
            * [Примечание](#Примечание-1)
            * [Спецификация карты](#Спецификация-карты)
        * [getMaps](#getmaps)
            * [Пример массива maps](#Пример-массива-maps)
        * [createGame](#creategame)
            * [Примечание](#Примечание-2)
        * [getGames](#getgames)
            * [Пример массива games](#Пример-массива-games)
        * [joinGame](#joingame)
        * [leaveGame](#leavegame)
            * [Примечание](#Примечание-3)
        * [getGameConsts](#getgameconsts)
        * [getStats](#getstats)
    * [Websocket](#websocket)
        * [Взаимодействие сервера и клиентов](#Взаимодействие-сервера-и-клиентов)
        * [Игровые клетки](#Игровые-клетки)
        * [Система координат](#Система-координат)
        * [Игровые параметры](#Игровые-параметры)
        * [Взаимодействие игроков с картой](#Взаимодействие-игроков-с-картой)
        * [Физика](#Физика)
        * [Использование респаунов и телепортов](#Использование-респаунов-и-телепортов)
            * [Пример использования респаунов](#Пример-использования-респаунов)
        * [Синхронный режим](#Синхронный-режим)
        * [Запросы, требующие websocket](#Запросы-требующие-websocket)
            * [Сообщение, которое сервер рассылает по всем соединениям websocket каждый тик](#Сообщение-которое-сервер-рассылает-по-всем-соединениям-websocket-каждый-тик)
                * [Пример сообщения от сервера](#Пример-сообщения-от-сервера)
            * [Общие параметры сообщений, которые отправляет клиент](#Общие-параметры-сообщений-которые-отправляет-клиент)
            * [empty](#empty)
            * [move](#move)
            * [fire](#fire)
            * [Пример сообщения клиента](#Пример-сообщения-клиента)

## Формат сообщений

### Запрос (Request)

* action => строка, название действия
* params => объект, описывающий параметры, специфичные для этого действия. *Необязательный в том случае, когда действие не подразумевает ни одного параметра (Например, для действия startTesting)*

#### Пример

    {
        "action": "signup",
        "params": {
            "login": "admin",
            "password": "banana"
        }
    }


### Ответ (Response)

* result => строка, результат выполнения действия
    - В случае успешного выполнения действия значение поля result должно быть равно "ok"
    - В случае ошибки передается ее код. Если ошибок несколько, то передается наиболее приоритетная из них. Ошибки с одинаковым приоритетом выбираются произвольно

* message => *строка, необязательный текстовый комментарий. Может быть произвольным*

#### Базовые коды ошибок, расположенные по убыванию приоритета

* badJSON (Запрос не соответствует [стандарту JSON](http://www.json.org/json-ru.html))
* badRequest (Какое-либо обязательное поле не было обнаружено, т.е. нарушен формат сообщений)
* badAction (Команда не найдена либо поле action некорректно)

##### Примечание

* Ошибки типа bad&lt;Property&gt;, т.е. описывающие несоответствие параметров ограничениям, имеют больший приоритет среди специфичных для запросов. *Например, в действии signin более приоритетной будет ошибка "badLogin" над "incorrect"*
* В общем случае, приоритеты ошибок расставлены следующим образом (т.е. ошибки, относящиеся к одному пункту из следующего списка. имеют одинаковый приоритет), по убыванию:
    - badJSON
    - badRequest
    - badAction
    - bad&lt;Property&gt;
    - все остальные
* Если в запросе пришло больше параметров, чем требуется, лишние параметры следует игнорировать

#### Пример ответа в случае успеха

    {
        "result": "ok"
    }

#### Пример ответа в случае ошибки

    {
        "result": "userExists",
        "message": "Такой пользователь уже существует"
    }


## Действия

### startTesting

Параметры:

* websocketMode => строковое значение, описывает механизм взаимодействия сервера и клиентов по websocket
    + Допустимые значения: "sync", "async"

Коды ошибок:

- notInTestMode (Сервер не в тестирующем режиме)

### signup

Параметры:

* login => строковое значение.
    - минимальная длина: 4 символа
    - макисмальная длина: 40 символов
* password => строковое значение.
    - минимальная длина: 4 символа

Коды ошибок:

* userExists (Такой пользователь уже существует)
* badLogin (Некорректный логин)
* badPassword (Некорректный пароль)

### signin

Параметры:

* login => строковое значение
* password => строковое значение

Ответ:

* sid => латинские буквы и цифры, не короче 1 символа

Коды ошибок:

* incorrect (Не найден пользователь с таким логином и паролем)
* badLogin (Логин не является строкой)
* badPassword (Пароль не является строкой)

### signout

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)

---

### sendMessage

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* game => число, идентификатор игры. Пустая строка -- для общего чата
* text => текст сообщения, строковое значение

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badGame (Некорректный идентификатор игры либо пользователь не может отправлять сообщение в эту игру)
* badText (Некорректный текст сообщения)

### getMessages

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* game => число, идентификатор игры. Пустая строка -- для общего чата
* since => Unix timestamp UTC

Ответ:

* messages => массив объектов, описывающих сообщения, отсортированный *по возрастанию* времени получения сообщения на сервере
    - time => время получения сообщения на сервере в формате Unix timestamp UTC. Должно быть не меньше параметра since
    - text => строка, текст сообщения
    - login => строка, логин отправителя сообщения

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badGame (Некорректный идентификатор игры либо пользователь не может получать сообщения из этой игры)
* badSince (Некорректное время)

#### Пример массива messages

    [
        {
            "time": 1382454174,
            "text": "Hello there",
            "login": "OldGuy1915"
        },
        {
            "time": 1382713375,
            "text": "Are you still here?",
            "login": "Newbie"
        }
    ]

---

### uploadMap

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* name => название игры, строковое значение
* map: ["<row1>", "<row2>", ...] => массив строк, описывающих карту. Длины строк должны быть равны
* maxPlayers => максимальное число игроков для этой карты, число

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badName (Некорректное название карты)
* badMap (Некорректные данные карты)
* badMaxPlayers (Некорректное максимальное число игроков)
* mapExists (Карта с таким названием уже существует)

#### Примечание

* Допускается создание карт, использующих разные названия, но одинаковые массивы строк map

#### Спецификация карты

* Элементы массива, описывающего карту, должны быть одинаковой длины (Карта прямоугольная)
* Может быть только два телепорта, помеченных одной цифрой. Они считаются соответственно входом и выходом.
* Одной клетке соответствует 1 символ

Допустимые символы:

+ ```.``` - пустое место
+ ```$``` - место респауна
+ ```#``` - стенка (считается, что стенки непробиваемые)
+ [a-z] - предметы. *Пока про предметы не договорились*
+ [A-Z] - оружиe:
    - ```K``` - knife
    - ```P``` - pistol
    - ```M``` - machine gun
    - ```R``` - rocket launcher
    - ```A``` - railgun
+ [0-9] - телепорты (Одинаковые цифры обозначают входы и выходы одного телепорта)

Пример:

    .2.....1....2.
    .#R..$.1.M..#.
    .############.
    ..............

Минимальный размер карты - 1 x 1 клетка

### getMaps

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа

Ответ:

* maps => массив объектов, описывающих карты. Порядок произвольный
    - id => число, идентификатор карты
    - name => строка, название карты
    - maxPlayers => число, максимально допустимое количество игроков для этой карты
    - map => данные карты в виде массива строк

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)

#### Пример массива maps

    [
        {
            "id": 1,
            "name": "Small map",
            "maxPlayers": 2,
            "map": [
                ".$",
                "##"
            ]
        },
        {
            "id": 500,
            "name": "Another map",
            "maxPlayers": 3,
            "map": [
                "#$.......$#",
                "###########"
            ]
        }
    ]

### createGame

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* name => название игры, строковое значение, не короче одного символа
* map => число, идентификатор карты
* maxPlayers => максимально допустимое число игроков
* consts => *объект, описывающий игровые константы, необязательный*
    * accel => вещественное число, константа ускорения
        * Больше 0
        * Не больше 0.1
    * maxVelocity => вещественное число, максимальная скорость
        * Больше 0
        * Меньше 1
    * friction => вещественное число, константа трения
        * Больше 0
        * Не больше 0.1
    * gravity => вещественное число, константа гравитации
        * Больше 0
        * Не больше 0.1

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badName (Некорректное название игры)
* badMap (Некорректный идентификатор карты либо карта с таким идентификатором не найдена)
* badMaxPlayers (Некорректное максимальное число игроков либо оно меньше допустимого картой)
* gameExists (Игра с таким названием уже существует)
* alreadyInGame (Пользователь, находящийся в какой-либо игре, не может создавать игры)

#### Примечание

* Пользователь, создавший игру, автоматически в нее заходит (т.е. не требуется отправка дополнительного запроса на joinGame)
* При попытке два раза подряд отправить запрос createGame с одними и теми же аргументами, возникает две ошибки (alreadyInGame и gameExists). Соответственно, корректным будет считаться ответ, в котором содержится любая из них
* Допускается создание игр, использующих одну и ту же карту
* Максимальное количество игроков, допустимое для игры, не должно превышать максимальное количество игроков, предусмотренных картой
* Параметр consts является необязательным, при его отсутствии либо при отправке некорректных данных все константы игры устанавливаются [по умолчанию](#Игровые-параметры)

### getGames

Параметры:

* sid => латинские буквы и цифры, не короче одного символа
* *status => строковое значение. Необязательный параметр. При указании статуса возвращаются игры только с этим статусом. По умолчанию возвращаются все*
    + Допустимые значения описаны в описании ответа к этому запросу

Ответ:

* games => массив объектов, описывающих игры. Порядок произвольный
    - id => число, идентификатор игры
    - name => строка, название игры
    - map => число, идентификатор карты
    - maxPlayers => число, максимальное количество игроков для этой игры
    - players => массив логинов пользователей, участвующих в этой игре, отсортированный в порядке их подключения к игре
    - status => строковое значение, состояние игры
        + Допустимые значения: "running", "finished"

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)

#### Пример массива games

    [
        {
            "id": 0,
            "name": "Super cool game",
            "map": 1,
            "maxPlayers": 12,
            "players": [
                "Creator", "User1"
            ],
            "status": "running"
        },
        {
            "id": 2,
            "name": "Platformer cup 2013",
            "map": 12,
            "maxPlayers": 4,
            "players": [
                "android lover", "**apple-fan-boy**", "windows_user313"
            ],
            "status": "finished"
        },
        {
            id: 123,
            "name": "Dm",
            "map": 1,
            "maxPlayers": 5,
            "players": [
                "Lonely guy"
            ],
            "status": "running"
        }
    ]

### joinGame

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* game => число, идентификатор игры

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* gameFull (Игра заполнена)
* badGame (Некорректный идентификатор игры)
* alreadyInGame (Пользователь, находящийся в какой-либо игре, не может подключаться к играм (в том числе к своей))

### leaveGame

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа

Коды ошибок:

* notInGame (Игрок не участвует ни в одной игре)

#### Примечание

После выхода последнего игрока, игра должна быть переведена в статус finished, после чего в ней доступен только просмотр статистики

### getGameConsts

Запрос на получение параметров игры, в которой находится пользователь

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа

Ответ:

* tickSize => целое число, размер игрового тика
* accuracy => вещественное число, допустимая точность при сравнении координат
* accel => вещественное число, ускорение при движении
* maxVelocity => вещественное число, максимальная скорость
* gravity => вещественное число, гравитация
* friction => вещественное число, трение

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* notInGame (Пользователь с таким sid не находится ни в одной игре)

### getStats

Параметры:

* sid => латинские буквы и цифры, не короче 1 символа
* game => число, идентификатор игры

Ответ:

* players => массив объектов, описывающих статистику каждого игрока
    - login => строка, логин игрока
    - kills => целое число, количество убийств, совершенных этим игроком
    - deaths => целое число, количество смертей этого игрока

Коды ошибок:

* badSid (Некорректный sid либо пользователь с таким sid не найден)
* badGame (Некорректный идентификатор игры)
* gameRunning (Игра еще не закончена)

## Websocket

### Взаимодействие сервера и клиентов

* Игровое время измеряется в тиках. Тики нумеруются на сервере
* Текущим называется тик:
    - для сервера: последний тик, состояние игры на момент которого было отправлено
    - для клиента: последний тик, состояние игры на момент которого было получено
* Следующим называется тик:
    - для сервера: тик, состояние игры на момент которого рассчитывается
    - для клиента: тик, состояние игры на момент которого ожидается
* Временной промежуток между текущим и следующим игровым тиком определяется размером тика
* Сервер принимает сообщения от клиентов с номерами тиков равными текущему, либо на 1 меньше. Остальные игнорируются
* Клиент принимает сообщения от сервера с номером тика больше либо равным текущему. Остальные игнорируются. При получении состояния игры, клиент обновляет игровое время (приравнивает номер тика к полученному от сервера)
* Первый запрос от клиента после установки websocket-соединения должен содержать параметр sid. Все остальные параметры первого запроса игнорируются. Остальные запросы от клиента не обязаны содержать sid
* При расчете координат и скорости игрока на следующий тик сервер:
    1. Анализирует сообщения от клиента согласно [физическим требованиям](#Физика)
    2. Суммирует векторы {dx, dy}, полученные из всех сообщений move за тик от соответствующего клиента, и нормирует этот вектор, получая направление движения игрока
    3. Проводит корректировку скорости и координат игрока согласно [физическим требованиям](#Физика)
    4. Нормализует скорость игрока согласно [игровым ограничениям](#Игровые-параметры)

### Игровые клетки

* Клетка - квадрат со стороной 1
* Игрок - квадрат размером с клетку
* Координаты игрока - координаты центра квадрата игрока

### Система координат

* координаты 0, 0 соответствуют левой верхней вершине левой верхней клетки
* x направлен вправо
* y направлен вниз

### Игровые параметры

* Размер тика (tickSize): 30 мс
* Максимальная скорость передвижения по модулю по каждой компоненте (maxVelocity): 0.2
* Ускорение при движении (accel): 0.02
* Трение (friction): 0.02
* Гравитация (gravity): 0.02
* Допустимая точность при сравнении координат (accuracy): 0.000001

### Взаимодействие игроков с картой

* Область за границей карты считается стенкой
* Считается, что игрок столкнулся со стенкой, если игрок пересекается с квадратом стенки
* Считается, что игрок вошел в телепорт (портал), если центр телепорта находится внутри квадрата игрока
* Считается, что игрок находится на опоре (стенке), если нижняя сторона игрока пересекается с горизонтальной границей стенки

### Физика

* Объекты и стенки не уничтожаются при попадании в них снарядов
* Игроки могут проходить сквозь друг друга
* Игроки не могут проходить сквозь стенки
* Прыжком считается:
    - со стороны клиента: отправка действия move с параметром dy < 0
    - со стороны сервера: ситуация при расчете координат и скоростей игрока на следующий тик, когда вектор скорости игрока {dx,dy} имеет dy < 0
* Падением считается состояние, когда игрок на текущий тик находится в воздухе (не на опоре)
* Прыжок можно осуществлять только, когда игрок находится на опоре. В противном случае считается, что dy в сообщении равен 0
* При отправке запроса move с dy > 0 параметр dy в сообщении приравнивается к 0
* При падении к вертикальной составляющей скорости за каждый тик должна прибавляться постоянная гравитации
* Столкновение со стенкой является абсолютно неупругим, т.е. при столкновении с вертикальной границей стенки обнуляется скорость по x, при столкновении с горизонтальной - по y
* При столкновении с вертикальной/горизонтальной стенкой координаты игрока должны измениться так, чтобы расстояние
 (по х/y соответственно) от центра квадрата игрока до центра стенки было равно единице
* Столкновение с углом обрабатывается, как столкновение с двумя стенками, кроме случая, когда на текущем тике игрок касается этого угла одной точкой (т.е. расстояние от игрока до центра клетки со стенкой по обеим осям равно единице). В этом случае столкновение с углом следует обрабатывать, как столкновение с горизонтальной стенкой
* При прыжке скорость по y следует делать максимально допустимой по модулю
* Если за тик не приходит ни одного запроса move, считается, что {dx,dy} = {0,0}
* Если игрок находится на опоре и при расчете скоростей на сервере итоговое dx получается равной нулю, следует начать торможение, то есть уменьшить скорость по х по модулю на постоянную трения, либо приравнять к нулю, если скорость по x по модулю меньше постоянной трения
* При расчете скорости по x, если итоговая компонента dx не равна нулю, следует увеличить скорость по x на величину ускорения при движении, если dx > 0, либо уменьшить на ту же величину, если dx < 0


### Использование респаунов и телепортов

* Порядок обхода респаунов -- слева направа, сверху вниз по кругу
* Если персонаж появился из какого-то респауна, считается, что этот респаун - последний использованный. Каждое перерождение происходит в следующем за последним использованным респауном
    - Из респауна игрок появляется без движения
* При телепортации игрока его скорость остается неизменной, а координаты изменяются на координаты центра выхода из телепорта
* Телепортация через портал &lt;n&gt; происходит, только если игрок на текущем тике находился вне портала &lt;n&gt;

#### Пример использования респаунов

Рассмотрим карту:

    ..$......$.
    .###...###.
    ...$.......
    .######....

Порядок появлений из респаунов будет следующий:

1. Третья клетка первой строки
2. Предпоследняя клетка первой строки
3. Четвертая клетка третьей строки
4. Третья клетка первой строки
5. И так далее

### Синхронный режим

* После получения запроса "startTesting" с параметром "websocketMode" равным "sync" включается синхронный режим работы websocket сервера.
* После перехода в данный режим останавливается отправка состояния игры клиентам по таймеру.
* Отправка состояния игры осуществляется после получения какого-либо запроса от всех игроков подключенных к данной игре.
* Каждый раз после отправки состояния игры считается, что ни один игрок не отправлял никаких запросов.
* Для получения нового состояния игры и симуляции "бездействия" клиенту нужно отправить "empty" запрос.

---

### Запросы, требующие websocket

Сообщения, требующие websocket, предлагается отправлять на /websocket (Например, http://localhost/websocket, если запросы без websocket отправлялись на localhost)

При установлении websocket-соединения, первым запросом требуется отправить move с нулевыми параметрами dx, dy

#### Сообщение, которое сервер рассылает по всем соединениям websocket каждый тик

* tick => число, номер игрового тика
* players => массив **массивов**, описывающих каждого игрока. В порядке подключения к игре. Игрок описывается в следующем порядке:
    - x => вещественное число, координата x игрока
    - y => вещественное число, координата y игрока
    - vx => вещественное число, скорость по оси x игрока
    - vy => вещественное число, скорость по оси y игрока
    - weapon => символ, обозначающий текущее оружие игрока. Значение соответствует символу из [спецификации карты](#Спецификация-карты)
    - weaponAngle => вещественное число, угол наклона оружия (в градусах) к оси x. Отсчитывается по часовой стрелке. 0 - направление горизонтально вправо
        + Значения: 0 <= weaponAngle < 360. -1 -- по умолчанию (направление оружия не известно)
    - login => строковое значение, логин игрока
    - health => целое число, процент здоровья игрока
    - respawn => целое число, время в тиках до респауна игрока или 0, если игрок жив
    - kills => целое число, количество убийств, совершенных этим игроком с начала игры
    - deaths => целое число, количество смертей этого игрока с начала игры
* projectiles => массив **массивов**, описывающих снаряды. Один снаряд описывается в следующем порядке:
    - x => вещественное число, x-компонента координат снаряда
    - y => вещественное число, y-компонента координат снаряда
    - vx => вещественное число, скорость по оси x снаряда
    - vy => вещественное число, скорость по оси y снаряда
    - weapon => символ, обозначающий оружие, с помощью которого был сделан выстрел. Значение соответствует символу из [спецификации карты](#Спецификация-карты)
* items => массив, каждый элемент - время до восстановления i-го предмета на карте. Предметы нумеруются относительно расположения на карте слева направо сверху вниз
    - Значения: 0 - для предметов, которые доступны игрокам, больше 0 - для предметов, которые еще восстанавливаются

##### Пример сообщения от сервера

    {
        "tick": 20,
        "players": [
            [12.12, 10.6, 0.06, 0, "K", 10.07, "qwerty", 100, 0, 1, 0]
        ],
        "projectiles": [
            [12.14, 10.8, -0.6, 0.1, "M"],
            [10.14, 11.22, -0.6, 0.1, "M"]
        ]
    }

#### Общие параметры сообщений, которые отправляет клиент

* *sid => латинские буквы и цифры, не менее 1 символа. Передача параметра sid является обязательной только при первом сообщении после открытия websocket-соединения*

#### empty

Предназначен для симуляции бездействия клиента

#### move

Параметры:

* tick => число, номер игрового тика
* dx => число, смещение игрока по x
* dy => число, смещение игрока по y

#### fire

Параметры:

* tick => число, номер игрового тика
* dx => число, смещение координат выстрела по x
* dy => число, смещение координат выстрела по y

#### Пример сообщения клиента

    {
        "params": {
            "sid": "usersid",
            "tick": 10,
            "dx": 12.123,
            "dy": 123.123
        },

        "action": "move"
    }
