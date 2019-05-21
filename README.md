# API Reference

## Ktwidget

#### Оглавление
- [Общее описание](#Общее-описание)
- [Что необходимо для выполнения запроса](#Что-необходимо-для-выполнения-запросa)
- [Генерация ключа](#Генерация-ключа)
- [Описание запросов к API](#Описание-запросов-к-API)
- [Описание ответа от API](#Описание-ответа-от-API)
- Методы
    - [Создание платежа](#Создание-платежа)
    - [Проверка статуса платежа](#Проверка-статуса-платежа)
    - [Отмена платежа](#Отмена-платежа)
    - [Возврат платежа](#Возврат-платежа)

##### Общее описание:
URL к тестовой среде: [https://card.test.nplus.tech/api/v1/merchant-widget](https://card.test.nplus.tech/api/v1/merchant-widget)

##### Что необходимо для выполнения запроса:
- Идентификатор вашего приложения в системе ktpay. Далее `id`
- Ключ от идентификатора вашего приложения в системе ktpay. Далее `ключ`
- Публичную часть `sodium` ключа которую вы [сгенерировали](#Генерация-ключа), нужно обернуть функцией `base64_encode`
и отправить на почту [widget@ktpay.kz](mailto:widget@ktpay.kz?subject=Публичная%20часть%20ключа%20от%20компании%20xxx)
с пометкой "Публичная часть ключа от компании xxx"

##### Генерация ключа:
Для генерации ключа сначала сгенерируйте [пару ключей](https://libsodium.gitbook.io/doc/public-key_cryptography/authenticated_encryption#key-pair-generation),
 потом возмите [публичную часть]() из него.

##### Платеж осуществляется в несколько этапов:
1. Вы создаете [json](https://www.json.org/json-ru.html) строку, шифруете еe (описание подписи будет ниже)
 с помощью ранее выданного вам `ключа` в системе ktpay.
    - Получившуюся зашифрованную строку вы отправляеете в теле запроса.
    - В заголовках запроса вы отправляете `Auth-Identifier` со значением `id`, полученого ранее.
2. В ответ от сервера вы получаете `url` страницы на которую нужно редиректнуть пользователя.
3. Пользователь вводит карточные данные и проходит 3DSecure, если в этом есть необходимость.
4. Редиректим пользователя на сайт клиента на страницу успеха или отклонения платежа*
5. В "бэкграунде" сообщаем серверу клиента о результате транзакции**.

\* `url` для возвращения на сайт клиента может быть указан двумя способами:
- Указать при создании клиента
- Передать в параметрах запроса 
    - callback_back_url - Общий `url` для возвращения на сайт клиента 
    - callback_success_url - `url` для успешных платежей
    - callback_failed_url - `url` для НЕ успешных платежей

\*\* `callback_post_url` - `url` на который будет отправлятся POST запрос с результатом платежа.
Передается с параметерами в запросе на создания платежа.

##### Описание запросов к API:
Шифрование запроса делается с помошью библиотеки `libsodium`

[Sodium](https://libsodium.gitbook.io/doc) - это современая, легкая в использование проргаммная библиотека для шифрования,
дшифрования, подписи, хэшированию паролей и другова. На вашем [ЯП](https://libsodium.gitbook.io/doc/bindings_for_other_languages)

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
`encrypted` - строка котороя передется в теле запроса.

***Внимание!!! Если в вашей `sodium` библиотеке или ее обертке нет функции
с хешированием в base64. Оберните `cipher` функцией base64 самого ЯП.***

Примеры:
[php](https://www.php.net/manual/ru/function.base64-encode.php) |
[rust](https://docs.rs/base64/0.3.1/base64/fn.encode.html) |
[java](https://docs.oracle.com/javase/8/docs/api/java/util/Base64.Encoder.html) |
[go](https://golang.org/pkg/encoding/base64) |
[еще пара](https://www.yeahhub.com/encode-base64-popular-programming-languages/)

##### Описание ответа от API:

### Методы
##### Создание платежа
##### Проверка статуса платежа
##### Отмена платежа
##### Возврат платежа

