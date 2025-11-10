# План тестирования API 

### Цель тестирования:

#### Проверить корректность работы REST API метода

```commandline
GET /products/{productId}/status?authToken=&recalculate=&owner=&region=
```

Цель тестирования — убедиться, что API корректно:
- обрабатывает входные параметры (`recalculate`, `owner`, `region`);
- проверяет и валидирует токен авторизации (`authToken`);
- возвращает корректные коды ответов (`200`, `401`, `404`, `500`);
- формирует ответы в правильном формате JSON;
- устойчиво работает при некорректных данных и граничных условиях.


## Объект тестирования

**Метод:** `GET /products/{productId}/status`

**Описание параметров:**

| Параметр | Обязательный | Тип | Формат / Допустимые значения |
|-----------|-----|------|-------------------------------|
| `productId` | Да | int/string | Существующий или несуществующий ID продукта |
| `authToken` | Да | string (16 символов) | Только латинские буквы и цифры (без спецсимволов) |
| `recalculate` | Нет | boolean | `true`, `false`, `null` |
| `owner` | Нет | string | `Создатель`, `Пользователь`, `null` |
| `region` | Нет | string | `Северо-Запад`, `Сибирь`, `Поволжье` |


### 1. Функциональное тестирование

* Проверка валидных комбинаций параметров

* Проверка граничных значений

* Проверка обработки ошибок

### 2. Нефункциональное тестирование

* Валидация входных параметров

* Обработка исключительных ситуаций

* Проверка форматов данных

## Тест-кейсы

### Группа 1: Позитивные тесты (HTTP 200)

#### Тест кейс 1.1: Успешный запрос со всеми параметрами

```commandline
URL: GET /products/123/status?authToken=Abc123def456gh78&recalculate=true&owner=Создатель&region=Северо-Запад
Ожидаемый результат: HTTP 200, {"productStatus": 1} или {"productStatus": 0}
```

#### Тест кейс 1.2: Запрос только с обязательными параметрами

```commandline
URL: GET /products/123/status?authToken=Abc123def456gh78
Ожидаемый результат: HTTP 200, корректный JSON с productStatus
```

### Тест кейс 1.3: Попарное тестирование (recalculate + owner, recalculate + region, owner + region)
| ID   | recalculate | owner         | region         | Ожидаемый результат |
|------|------------|---------------|----------------|-------------------|
| TC1  | true       | Создатель     | Северо-Запад   | HTTP 200, {"productStatus": 0 или 1} |
| TC2  | true       | Пользователь  | Сибирь         | HTTP 200, {"productStatus": 0 или 1} |
| TC3  | true       | null          | Поволжье       | HTTP 200, {"productStatus": 0 или 1} |
| TC4  | false      | Создатель     | Сибирь         | HTTP 200, {"productStatus": 0 или 1} |
| TC5  | false      | Пользователь  | Поволжье       | HTTP 200, {"productStatus": 0 или 1} |
| TC6  | false      | null          | Северо-Запад   | HTTP 200, {"productStatus": 0 или 1} |
| TC7  | null       | Создатель     | Поволжье       | HTTP 200, {"productStatus": 0 или 1} |
| TC8  | null       | Пользователь  | Северо-Запад   | HTTP 200, {"productStatus": 0 или 1} |
| TC9  | null       | null          | Сибирь         | HTTP 200, {"productStatus": 0 или 1} |


### Группа 2: Ошибки аутентификации (HTTP 401)

#### Тест кейс 2.1: Отсутствует authToken

```commandline
URL: GET /products/123/status
Ожидаемый результат: HTTP 401, доступ запрещен
```

#### Тест кейс 2.2: Неверный формат authToken (менее 16 символов)

```commandline
URL: GET /products/123/status?authToken=short
Ожидаемый результат: HTTP 401
```

#### Тест кейс 2.3: Неверный формат authToken (более 16 символов)

```commandline
URL: GET /products/123/status?authToken=thisistoolongtoken123
Ожидаемый результат: HTTP 401
```

#### Тест кейс 2.4: Недопустимые символы в authToken

```commandline
URL: GET /products/123/status?authToken=invalid!@#chars
Ожидаемый результат: HTTP 401
```

#### Тест кейс 2.5: Пустой authToken

```commandline
URL: GET /products/123/status?authToken=
Ожидаемый результат: HTTP 401
```

#### Тест кейс 2.6: Валидный токен, но без прав доступа

```commandline
URL: GET /products/123/status?authToken=ValidTokenWithoutAccess
Ожидаемый результат: HTTP 401 или HTTP 403, доступ запрещен
```

### Группа 3: Ошибки "Продукт не найден" (HTTP 404)

#### Тест кейс 3.1: Несуществующий productId

```commandline
URL: GET /products/999999/status?authToken=Abc123def456gh78
Ожидаемый результат: HTTP 404, продукт не найден
```

#### Тест кейс 3.2: productId = 0

```commandline
URL: GET /products/0/status?authToken=Abc123def456gh78
Ожидаемый результат: HTTP 404
```

#### Тест кейс 3.3: Отрицательный productId

```commandline
URL: GET /products/-1/status?authToken=Abc123def456gh78
Ожидаемый результат: HTTP 404
```

### Группа 4: Валидация параметров

#### Тест кейс 4.1: Некорректные значения recalculate

```commandline
URL: GET /products/123/status?authToken=...&recalculate=invalid
Ожидаемый результат: HTTP 400 или корректая обработка
```

#### Тест кейс 4.2: Некорректные значения owner

```commandline
URL: GET /products/123/status?authToken=...&owner=НекорректныйВладелец
Ожидаемый результат: HTTP 400, {"errorMessage": "Invalid parameter owner"}
```

#### Тест кейс 4.3: Некорректные значения region

```commandline
URL: GET /products/123/status?authToken=...&region=Марс
Ожидаемый результат: HTTP 400, {"errorMessage": "Invalid parameter region"}
```

### Группа 5: Граничные случаи и нагрузка

#### Тест кейс 5.1: Длинный productId

```commandline
URL: GET /products/999999999999999/status?authToken=Abc123def456gh78
Ожидаемый результат: HTTP 200 или 404
```

#### Тест кейс 5.2: Проверка времени отклика 

```commandline
URL: Любой валидный запрос
Ожидаемый результат: Время ответа ≤ 2000 мс
```

#### Тест кейс 5.3: Повторяющиеся параметры

```commandline
URL: GET /products/123/status?authToken=...&recalculate=true&recalculate=false
Ожидаемый результат: Определить поведение (первое/последнее значение)
```

### Группа 6: Формат ответа и структура ответа

#### Тест кейс 6.1: Проверка формата JSON ответа

```commandline
URL: Валидный запрос
Ожидаемый результат: 
- Content-Type: application/json
- Валидный JSON
- Соответствие схеме {"productStatus": number}
```

#### Тест кейс 6.2: Проверка допустимых значений productStatus

```commandline
URL: Множество валидных запросов
Ожидаемый результат: productStatus всегда 0 или 1
```

### Группа 7: Ошибки сервера (HTTP 500)

#### Тест кейс 7.1: Проверка обработки внутренних ошибок

```commandline
URL: Валидный запрос, вызывающий ошибку сервера
Ожидаемый результат: HTTP 500, {"errorMessage": "описание ошибки"}
```

### Чек-лист валидации:

* Все комбинации параметров работают корректно
* Формат authToken строго проверяется
* Коды ответов соответствуют спецификации
* JSON ответы валидны и соответствуют схеме
* Обработка отсутствующих параметров
* Обработка некорректных значений
* Логирование ошибок
* Время ответа в допустимых пределах
* API стабильно при множественных параллельных запросах









































