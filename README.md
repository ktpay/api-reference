# API Reference

## Ktwidget
##### Что необходимо для выполнения запроса:
- Идентификатор вашего приложения в системе ktpay
- Ключ от идентификатора вашего приложения в системе ktpay
- Публичную часть `sodium` ключа которую вы сгенерировали* (не потеряйте приватную часть ключа), нужно скинуть нам.

\* Для генерации ключа сначала сгенерируйте [пару ключей](https://libsodium.gitbook.io/doc/public-key_cryptography/authenticated_encryption#key-pair-generation),
 потом возмите [публичную часть]() из него.

##### Платеж осуществляется в несколько этапов:
- Вы создаете [json](https://www.json.org/json-ru.html) строку, шифруете еe (описание подписи будет ниже)
 с помощью ранее выданного вам ключа от идентификатора вашего приложения в системе ktpay.
    - Получившуюся зашифрованную строку вы отправляеете в теле запроса.
    - В заголовках запроса вы отправляете `Auth-Identifier` со значением идентификатора вашего приложения, полученого ранее.
- В ответ от сервера вы получаете `url` страницы на которую нужно редиректнуть пользователя.
- Пользователь вводит карточные данные и проходит 3DSecure, если в этом есть необходимость.
- Редиректим пользователя на сайт клиента на страницу успеха или отклонения платежа*
- В "бэкграунде" сообщаем серверу клиента о результате транзакции**.

\* `url` для возвращения на сайт клиента может быть указан двумя способами:
- Указать при создании клиента
- Передать в параметрах запроса 
    - callback_back_url - Общий `url` для возвращения на сайт клиента 
    - callback_success_url - `url` для успешных платежей
    - callback_failed_url - `url` для НЕ успешных платежей

\*\* `callback_post_url` - `url` на который будет отправлятся POST запрос с результатом платежа.
Передается с параметерами в запросе на создания платежа.

##### Шифрование запроса
Шифрование запроса делается с помошью библиотеки `libsodium`

[Sodium](https://libsodium.gitbook.io/doc) - это современая, легкая в использование проргаммная библиотека для шифрования, дишифрования, подписи, хэшированию паролей и другова.

Пример на js:
Сам запрос в формате json:
```json
{
  "order": {
    "amount": 1000,
    "order_id": "Order1",
    "description": "Тестовая оплата за Order1",
    "callback_post_url": "https://site.com/callback_post_url",
    "callback_back_url": "https://site.com/callback_back_url",
    "callback_success_url": "https://site.com/callback_success_url",
    "callback_failed_url": "https://site.com/callback_failed_url",
    "type": "ecom",
    "service": {
        "service_id": "1",
        "contract_value": "17-25022"
    }
  },
  "user": {
    "merchant_user_id": "user1",
    "phone": "+77778889900",
    "email": "test@example.com"
  }
}
```

```js
let recipientPk = sodium.from_base64("h09acJ8QSLPw40XZxdNfLbq4hXzWInC5zXL319RdXUI=");
let cipher = sodium.crypto_box_seal(json_string, recipientPk);
let encrypted = sodium.to_base64(cipher);
```

`encrypted` - строка котороя передется в теле запроса

### Создание платежа
### Проверка статуса платежа
### Отмена платежа
### Возврат платежа

