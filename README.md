# Payment REST API

## Скачивание и настройка проекта
```bash
$ git clone https://github.com/Arcady1/go-rest-api.git
$ cd go-rest-api/
$ mv .env.dev .env
```

## Docker

Запуск: 

```bash
$ docker-compose up --build
```

Остановка:
```bash
$ docker-compose down
```

## Настройка базы данных

Для создания необходимых таблиц, нужно запустить SQL скрипт `/sql/init.sql`.

## API

Базовый URL: 
```c
http://<HOST>:<PORT>
```

Значения `HOST` и `PORT` устанавливаются в `.env` файле.

Заголовки в каждом запросе:
```c
'Content-Type: application/json'
```

### Метод начисления средств на баланс

Запрос:

```HTTP PUT /api/v1/user/refill```

Пример запроса:

```
HTTP PUT /api/v1/user/refill
Body {
    "userId": "<ID пользователя>",
    "amount": <Сумма зачисления>
}
```

Параметры запроса:

| Параметр | Описание                                         | Тип    | Формат                                  | Обязательный |
| -------- | ------------------------------------------------ | ------ | --------------------------------------- | ------------ |
| userId   | ID пользователя, на чей счёт зачисляются деньги. | String | Любая непустая строка.                  | Да           |
| amount   | Сумма зачисления.                                | Float  | Число типа Float. Мин: 0.01, шаг: 0.01. | Да           |

Пример ответа:

```json
{
    "status": 201,
    "message": "success",
    "data": null
}
```

Параметры ответа:

| Свойство | Описание          | Тип    |
| -------- | ----------------- | ------ |
| status   | Статус ответа.    | Int    |
| message  | Сообщение ответа. | String |
| data     | Данные ответа.    | None   |

#### Возможные ошибки

Пример ответа с ошибкой:

```json
{
    "status": 400,
    "message": "Wrong data",
    "data": null
}
```

| Код | Описание                    |
| --- | --------------------------- |
| 500 | Ошибка на стороне сервера.  |
| 400 | Ошибка в переданных данных. |

#### Описание метода

* Метод позволяет начислить деньги на счёт пользователя.
* Если указанный пользователь не существует, то создаются новый пользователь и счёт. Деньги зачисляются на счёт сразу же.
* Если пользователь существует, то указанная сумма зачисляется ему на счёт, и баланс увеличивается.

---

### Метод получения баланса пользователя

Запрос:

```HTTP POST /api/v1/user/balance```

Пример запроса:

```
HTTP POST /api/v1/user/balance
Body {
    "userId": "<ID пользователя>",
}
```

Параметры запроса:

| Параметр | Описание                                            | Тип    | Формат                 | Обязательный |
| -------- | --------------------------------------------------- | ------ | ---------------------- | ------------ |
| userId   | ID пользователя, баланс счёта которого надо узнать. | String | Любая непустая строка. | Да           |

Пример ответа:

```json
{
    "status": 200,
    "message": "success",
    "data": {
        "balance": 10.55
    }
}
```

Параметры ответа:

| Свойство        | Описание              | Тип    |
| --------------- | --------------------- | ------ |
| status          | Статус ответа.        | Int    |
| message         | Сообщение ответа.     | String |
| data            | Информация о балансе. | Object |
| data -> balance | Баланс пользователя.  | Float  |

#### Возможные ошибки

Пример ответа с ошибкой:

```json
{
    "status": 404,
    "message": "The user with ID '1' is not found",
    "data": null
}
```

| Код | Описание                    |
| --- | --------------------------- |
| 500 | Ошибка на стороне сервера.  |
| 400 | Ошибка в переданных данных. |
| 404 | Пользователь не найден.     |

#### Описание метода

* Метод позволяет узнать баланс пользователя.
* Если указанный пользователь существует, вернётся информация о его балансе, иначе - ошибка.

---

### Метод резервирования средств с основного баланса на отдельном счете

Запрос:

```HTTP POST /api/v1/payments/reserve```

Пример запроса:

```
HTTP POST /api/v1/payments/reserve
Body {
    "userId": "<ID пользователя>",
    "serviceId": "<ID услуги>",
    "orderId": "<ID заказа>",
    "cost": <Стоимость>
}
```

Параметры запроса:

| Параметр  | Описание                                             | Тип    | Формат                                  | Обязательный |
| --------- | ---------------------------------------------------- | ------ | --------------------------------------- | ------------ |
| userId    | ID пользователя, с чьего счёта резервируются деньги. | String | Любая непустая строка.                  | Да           |
| serviceId | ID предоставляемой услуги.                           | String | Любая непустая строка.                  | Да           |
| orderId   | ID заказа.                                           | String | Любая непустая строка.                  | Да           |
| cost      | Стоимость услуги.                                    | Float  | Число типа Float. Мин: 0.01, шаг: 0.01. | Да           |

Пример ответа:

```json
{
    "status": 200,
    "message": "success",
    "data": null
}
```

Параметры ответа:

| Свойство | Описание          | Тип    |
| -------- | ----------------- | ------ |
| status   | Статус ответа.    | Int    |
| message  | Сообщение ответа. | String |
| data     | Данные ответа.    | None   |


#### Возможные ошибки

Пример ответа с ошибкой:

```json
{
    "status": 400,
    "message": "The order already exists",
    "data": null
}
```

| Код | Описание                                              |
| --- | ----------------------------------------------------- |
| 500 | Ошибка на стороне сервера.                            |
| 400 | Ошибка в переданных данных.                           |
| 404 | Пользователь не найден.                               |
| 406 | Стоимость услуги превышает баланс счёта пользователя. |

#### Описание метода

* Метод позволяет зарезервировать деньги пользователя за заказ на отдельном счёте.
* У одного пользователя может быть неограниченное количество заказов.
* В одном заказе (`orderId`) услуги (`serviceId`) повторяться не могут.
* Чтобы повторно заказать услугу, необходимо создать заказ с уникальным `orderId`.
* Если у пользователя на счёте недостаточно средств, то резерв средств произвести невозможно - вернётся ошибка. 
* В случае успешного резервирования денег, статус заказка меняется на `reserved`.

---

### Метод признания выручки

Запрос:

```HTTP PUT /api/v1/payments/accept```

Пример запроса:

```
HTTP PUT /api/v1/payments/accept
Body {
    "userId": "<ID пользователя>",
    "serviceId": "<ID услуги>",
    "orderId": "<ID заказа>",
    "amount": <Сумма списания>
}
```

Параметры запроса:

| Параметр  | Описание                                             | Тип    | Формат                                  | Обязательный |
| --------- | ---------------------------------------------------- | ------ | --------------------------------------- | ------------ |
| userId    | ID пользователя, с чьего счёта резервируются деньги. | String | Любая непустая строка.                  | Да           |
| serviceId | ID предоставляемой услуги.                           | String | Любая непустая строка.                  | Да           |
| orderId   | ID заказа.                                           | String | Любая непустая строка.                  | Да           |
| amount    | Сумма списания.                                      | Float  | Число типа Float. Мин: 0.01, шаг: 0.01. | Да           |

Пример ответа:

```json
{
    "status": 201,
    "message": "success",
    "data": null
}
```

Параметры ответа:

| Свойство | Описание          | Тип    |
| -------- | ----------------- | ------ |
| status   | Статус ответа.    | Int    |
| message  | Сообщение ответа. | String |
| data     | Данные ответа.    | None   |


#### Возможные ошибки

Пример ответа с ошибкой:

```json
{
    "status": 406,
    "message": "The order is not reserved. The order status is 'succeed'",
    "data": null
}
```

| Код | Описание                          |
| --- | --------------------------------- |
| 500 | Ошибка на стороне сервера.        |
| 400 | Ошибка в переданных данных.       |
| 404 | Пользователь или заказ не найден. |
| 406 | Статус заказа не 'reserved'.      |

#### Описание метода

* Метод позволяет признать зарезервированную сумму.
* Если указанная сумма больше, чем зарезервированная, то происходит разрезервирование денег:
  * Статус заказа меняется на `canceled`.
  * Сумма заказа возвращается на счёт пользователя.
* Если указанная сумма меньше или равна зарезервированной, то происходит списание денег:
  * Статус заказа меняется на `succeed`.
  * Стоимость заказа становится равной переданной сумме.
  * Разница между исходной стоимостью заказа и переданной суммой возвращается на счёт пользователя.

## Схема базы данных Postgres

#### users

| Поле       | Тип     | Свойства             |
| ---------- | ------- | -------------------- |
| user_id    | VARCHAR | PRIMARY KEY NOT NULL |
| account_id | VARCHAR | NOT NULL             |

#### accounts

| Поле       | Тип     | Свойства             |
| ---------- | ------- | -------------------- |
| account_id | VARCHAR | PRIMARY KEY NOT NULL |
| balance    | FLOAT   | NOT NULL             |

#### orders

| Поле       | Тип     | Свойства             |
| ---------- | ------- | -------------------- |
| id         | VARCHAR | PRIMARY KEY NOT NULL |
| order_id   | VARCHAR | NOT NULL             |
| account_id | VARCHAR | NOT NULL             |
| service_id | VARCHAR | NOT NULL             |
| cost       | FLOAT   | NOT NULL             |
| status     | VARCHAR | NOT NULL             |

## License
MIT
