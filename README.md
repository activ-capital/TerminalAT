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
http://10.64.20.103:8080/payment_app.at?ACTION=check&OPER_ID=123e4567-e89b-12d3-a456-426614174000&PAY_DATE=2024-07-25 14:30:00&SERVICE_NAME=example_service&PHONE_NUMBER=examplenumber&TERMINAL_ID=terminal123
```

Ответ на запрос на проверку (check)

```json
{
    "code": 302,
    "message": "Получатель найден"
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
/payment_app.at?ACTION=payment&AMOUNT=100.50&OPER_ID=123e4567-e89b-12d3-a456-426614174000&PAY_DATE=2024-07-25 14:30:00&SERVICE_NAME=example_service&PHONE_NUMBER=examplenumber&TERMINAL_ID=terminal123
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


## Авторизация
### Для обеспечения безопасности запросы к API должны быть подписаны с использованием асимметричной криптографии (публичного и приватного ключа)

### Механизм авторизации:
	1. Подготовка строки запроса: Клиент должен собрать строку из обязательных параметров запроса в следующем порядке: OPER_ID, ACTION, SERVICE_NAME, PHONE_NUMBER.
 Пример строки:
 
 ```plaintext
OPER_ID=123e4567-e89b-12d3-a456-426614174000&ACTION=check&SERVICE_NAME=example_name&PHONE_NUMBER=99255000000
```
#### Важно: В строку для подписи включаются только поля: OPER_ID, ACTION, SERVICE_NAME, PHONE_NUMBER. Остальные параметры запроса не включаются в подпись.

      2. Генерация подписи: Используйте приватный ключ для подписания строки запроса. Подпись создаётся с использованием алгоритма HMAC-SHA256.
 Заголовки запроса:

	•	X-Public-Key: публичный ключ клиента (используется сервером для проверки подписи).
	•	X-Signature: подпись, сгенерированная с помощью приватного ключа клиента.
### Пример запроса:
```http
GET /payment_app.at?ACTION=check&OPER_ID=123e4567-e89b-12d3-a456-426614174000&PAY_DATE=2024-07-25%2014:30:00&SERVICE_NAME=example_service&PHONE_NUMBER=992937870880 HTTP/1.1
Host: example.com
X-Public-Key: -----BEGIN PUBLIC KEY-----
<ваш публичный ключ>
-----END PUBLIC KEY-----
X-Signature: <сгенерированная подпись>
```

### Пример кода для генерации подписи (Go):
```GO
func GenerateHMACSignature(message, secret string) string {
    // Создаем HMAC-SHA256
    h := hmac.New(sha256.New, []byte(secret))
    h.Write([]byte(message))

    // Возвращаем подпись в формате Base64
    return base64.StdEncoding.EncodeToString(h.Sum(nil))
}
```
## Пример строки для подписи:
```plaintext
OPER_ID=123e4567-e89b-12d3-a456-426614174000&ACTION=check&SERVICE_NAME=example_service&PHONE_NUMBER=992550000000
```
## Пример сгенерированной подписи:
```plaintext
Подпись (Base64): mFz5T2yA1y3YhjOY+pLlfgfMT9k2Oy3wJ/nYRw9y1uo=
```
### Процесс генерации подписи
# HMAC-SHA256: Подпись генерируется с использованием этого алгоритма и вашего секретного ключа.
# Base64: Результат кодируется в формате Base64 и передаётся в заголовке X-Signature.

### Ошибки авторизации:
	•	401: Unauthorized – Неверная подпись или отсутствует публичный ключ.
	•	400: Bad Request – Отсутствуют обязательные параметры запроса.
	•	500: Internal Server Error – Ошибка загрузки ключей на сервере.
 ### Примечания
 	•	Убедитесь, что используете один и тот же приватный ключ для генерации подписи и соответствующий публичный ключ для проверки подписи на сервере.
	•	Строка для подписи должна быть сформирована строго в указанном порядке параметров: OPER_ID, ACTION, SERVICE_NAME, PHONE_NUMBER.
	•	Любые дополнительные параметры запроса не включаются в строку для подписи.

