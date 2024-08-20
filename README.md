# TerminalAtService API

API `TerminalAtService` обрабатывает операции по переводу через терминал, такие как проверка и осуществление платежа.

## Конечная точка

**URL**: `/payment_app.at`  
**Метод**: `GET`

## Параметры запроса

| Параметр       | Тип    | Обязательный | Описание                          | Возможные значения               |
|----------------|--------|--------------|-----------------------------------|----------------------------------|
| `ACTION`       | string | Да           | Действие, которое нужно выполнить | Возможные значения: `check` для проверки данных или `payment` для выполнения платежа.|
| `AMOUNT`       | number | Да           | Сумма перевода в TJS              | Любое положительное число        |
| `OPER_ID`      | string | Да           | Идентификатор операции            | Должен быть строковым значением, представляющим идентификатор операции.                             |
| `PAY_DATE`       | string | Да           | Дата и время платежа              | Дата и время в формате YYYY-MM-DD HH:MM:SS |
| `SERVICE_NAME` | string | Да           | Название провайдера или банка отправителя  | Любое допустимое название|
| `PHONE_NUMBER` | string | Да           | Номер телефона                    | Должен быть числовым значением длиной ровно 12 символов без пробелов, тире или других специальных символов.|
| `TERMINAL_ID`  | string | Да           | идентификатор терминала           | Должен быть строковым значением, представляющим идентификатор терминала.                             |

## Примеры запросов

**Примеры запросов:**

**Запрос на проверку (check)** `GET`

```HTTP
http://10.64.20.103:8080/payment_app.at?ACTION=check&OPER_ID=123e4567-e89b-12d3-a456-426614174000&PAY_DATE=2024-07-25 14:30:00&SERVICE_NAME=example_service&PHONE_NUMBER=examplenumber
```

Ответ на запрос на проверку (check)

```json
{
  "error_code": {
    "code": 302,
    "message": "Получатель найден"
  },
  "payload": {
    "service_name": "example_service",
    "action": "check",
    "terminal_id": "12345",
    "phone_number": "992937870880",
    "fio": "Иван И.",
    "oper_id": "123e4567-e89b-12d3-a456-426614174000",
    "transaction_date": "2024-07-25 14:30:00"
  }
}
```

Запрос на оплату (payment) `GET`

```HTTP
/payment_app.at?ACTION=payment&AMOUNT=100.50&OPER_ID=123e4567-e89b-12d3-a456-426614174000&PAY_DATE=2024-07-25 14:30:00&SERVICE_NAME=example_service&PHONE_NUMBER=examplenumber
```

Ответ на запрос на оплату (payment)

```json
{
    "code": 202,
    "message": "Перевод зачислен"
  "payload": {
    "service_name": "example_service",
    "action": "payment",
    "terminal_id": "12345",
    "phone_number": "992937870880",
    "fio": "Иван И.",
    "oper_id": "123e4567-e89b-12d3-a456-426614174000",
    "transaction_date": "2024-07-25 14:30:00"
    "amount": 100.50
  }
}
```

### Успешные статус-коды (Success Status Codes)

| Статус-код | Сообщение                 | Описание                                                   |
|------------|---------------------------|------------------------------------------------------------|
| `202`      | `Перевод зачислен`        | Платеж успешно выполнен, сумма зачислена на счет получателя. |
| `302`      | `Получатель найден`       | Информация о получателе найдена и возвращена.              |

### Ошибочные статус-коды (Fail Status Codes)

| Статус-код | Сообщение                                | Описание                                                                                                 |
|------------|------------------------------------------|----------------------------------------------------------------------------------------------------------|
| `102`      | `Получатель не найден`                   | Запрос не может быть обработан, так как указанный получатель не найден.|
| `201`      | `Не найден результат проверки`           | Запрос на завершение не может быть обработан, так как результат предварительной проверки не найден.|
| `400`      | `Неверный запрос`                        | Один или несколько параметров запроса отсутствуют или неверны.  |
| `401`      | `Неавторизован`                          | Доступ запрещен, необходимо предоставить корректные учетные данные. |
| `107`      | `Дубликат запроса оплаты`                | Запрос на оплату не может быть обработан, так как нарушена уникальность `OPER_ID`.|
| `500`      | `Внутренняя ошибка сервера`              | Произошла ошибка на сервере при обработке запроса.              |
| `503`      | `Сервис недоступен`                      | Сервис временно недоступен, попробуйте позже.                   |
