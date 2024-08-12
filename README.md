# TerminalAtService API

API `TerminalAtService` обрабатывает операции по переводу через терминал, такие как проверка и осуществление платежа.

## Конечная точка

**URL**: `/payment_app.at`  
**Метод**: `GET`

## Параметры запроса

| Параметр       | Тип    | Обязательный | Описание                          | Возможные значения               |
|----------------|--------|--------------|-----------------------------------|----------------------------------|
| `ACTION`       | string | Да           | Действие, которое нужно выполнить | `check`, `payment`               |
| `AMOUNT`       | number | Да           | Сумма перевода в TJS              | Любое положительное число        |
| `OPER_ID`      | string | Да           | Идентификатор операции            | UUID                             |
| PAY_DATE       | string | Да           | Дата и время платежа              | Дата и время в формате YYYY-MM-DD HH:MM:SS |
| `SERVICE_NAME` | string | Да           | Название услуги                   | Любое допустимое название услуги |
| `PHONE_NUMBER` | string | Да           | Номер телефона                    | Валидный номер телефона          |
| `TERMINAL_ID`  | string | Да           | идентификатор терминала           | UUID                             |

## Примеры запросов

**Примеры запросов:**

**Запрос на проверку (check)** `GET`

```HTTP
 /payment_app.at?action=check&oper_id=123e4567-e89b-12d3-a456-426614174000&pay_date=2024-07-25&service_name=example_service&phone_number=9988776655
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
    "phone_number": "998877665",
    "fio": "Иван И.",
    "oper_id": "123e4567-e89b-12d3-a456-426614174000",
    "transaction_date": "2024-07-25"
  }
}
```

Запрос на оплату (payment) `GET`

```HTTP
 /payment_app.at?action=payment&amount=100.50&oper_id=123e4567-e89b-12d3-a456-426614174000&pay_date=2024-07-25&service_name=example_service&phone_number=9988776655
```

Ответ на запрос на оплату (payment)

```json
{
  "error_code": {
    "code": 202,
    "message": "Перевод зачислен"
  },
  "payload": {
    "service_name": "example_service",
    "action": "payment",
    "terminal_id": "12345",
    "phone_number": "998877665",
    "fio": "Иван И.",
    "oper_id": "123e4567-e89b-12d3-a456-426614174000",
    "transaction_date": "2024-07-25",
    "amount": 100.50
  }
}
```
