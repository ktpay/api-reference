# API Reference

## Ktwidget

#### Оглавление
- [Общее описание](#Общее-описание)
- [Что необходимо для выполнения запроса](#Что-необходимо-для-выполнения-запроса)
- [Генерация ключа](#Генерация-ключа)
- [Описание запросов к API](#Описание-запросов-к-API)
- [Описание ответа от API](#Описание-ответа-от-API)
- Методы
    - [Создание платежа](#Создание-платежа)
    - [Проверка статуса платежа](#Проверка-статуса-платежа)
    - [Отмена платежа](#Отмена-платежа)
    - [Возврат платежа](#Возврат-платежа)

##### Общее описание:
`Merchant` - это клиент использующий API сервиса. В текущий момент, личный кабинет `merchant'a` в разработке,
поэтому создание и изменения настроек, осуществляется через вашего менеджера.

При создании `merchant'a` обязательным является, лишь `название`, `домены` и `публичная часть ключа`
полученная на этапе ["Генерация ключа"](#Генерация-ключа).
###### Необязательные параметры:
- callback_back_url - Общий `url` для возвращения на сайт `merchant'a`
- callback_success_url - `url` для успешных платежей
- callback_failed_url - `url` для НЕуспешных платежей
- callback_post_url - `url` на который будет отправляться POST запрос с результатом платежа.

После создания менеджером `merchant'a` вы получите `Идентификатор приложения` (далее `app_id`) и ключ (далее `app_key`)

URL к тестовой среде: [https://card.test.nplus.tech/api/v1/merchant-widget](https://card.test.nplus.tech/api/v1/merchant-widget)

##### Что необходимо для выполнения запроса:
- Идентификатор вашего приложения (`app_id`) в системе ktpay.
- Ключ от идентификатора вашего приложения (`app_key`) в системе ktpay.
- Публичную часть `sodium ключа` которую вы [сгенерировали](#Генерация-ключа), нужно обернуть функцией `base64_encode`
и отправить на почту [widget@ktpay.kz](mailto:widget@ktpay.kz?subject=Публичная%20часть%20ключа%20от%20компании%20xxx)
с пометкой "Публичная часть ключа от компании xxx"

##### Генерация ключа:
Шифрование запроса делается с помощью библиотеки `libsodium`

[Sodium](https://libsodium.gitbook.io/doc) - это современая, легкая в использование проргаммная библиотека для шифрования,
дшифрования, подписи, хэшированию паролей и другова. На вашем [ЯП](https://libsodium.gitbook.io/doc/bindings_for_other_languages)

Для генерации `sodium ключа` сначала сгенерируйте [пару ключей](https://libsodium.gitbook.io/doc/public-key_cryptography/authenticated_encryption#key-pair-generation),
затем его публичную часть оберните функцией хеширования base64encode вашего ЯП, для отправки его менеджеру.
Сохраните приватную часть `sodium ключа`.

Пример на Cи: 
```clang
unsigned char publickey[crypto_box_PUBLICKEYBYTES];
unsigned char secretkey[crypto_box_SECRETKEYBYTES];
crypto_box_keypair(publickey, secretkey);
```
***Внимание!!! Если в вашей `sodium` библиотеке или ее обертке нет функции
с хешированием в base64. Оберните `cipher` функцией base64 самого ЯП.***

Примеры:
[php](https://www.php.net/manual/ru/function.base64-encode.php) |
[rust](https://docs.rs/base64/0.3.1/base64/fn.encode.html) |
[java](https://docs.oracle.com/javase/8/docs/api/java/util/Base64.Encoder.html) |
[go](https://golang.org/pkg/encoding/base64) |
[еще пара](https://www.yeahhub.com/encode-base64-popular-programming-languages/)

##### Платеж осуществляется в несколько этапов:
1. Вы создаете [json](https://www.json.org/json-ru.html) строку, [шифруете](#Описание-запросов-к-API) еe с помощью ранее выданного вам `ключа` в системе ktpay.
    - Получившуюся зашифрованную строку вы отправляете в теле запроса.
    - В заголовках запроса вы отправляете `Auth-Identifier` со значением `app_id`, полученного ранее.
2. В [ответ от сервера](#Описание-ответа-от-API) вы получаете [json](https://www.json.org/json-ru.html) строку.
Там будет `url` страницы на которую нужно редиректить пользователя.
3. Пользователь вводит карточные данные и проходит 3DSecure, если в этом есть необходимость.
4. Редирект пользователя на сайт клиента на страницу успеха или отклонения платежа*
5. В "бэкграунде" сообщаем серверу клиента о результате транзакции**.

\* `url` для возвращения на сайт `merchant'a`. Передается с параметрами в запросе на создания платежа или 
в [настройках](#Необязательные-параметры) `merchant'a`.

\*\* `callback_post_url` - Передается с параметрами в запросе на создания платежа или
в [настройках](#Необязательные-параметры) `merchant'a`.

##### Описание запросов к API:
Пример на js:
```js
const app_key = "h09acJ8QSLPw40XZxdNfLbq4hXzWInC5zXL319RdXUI=";
const order = {
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
};
const recipientPk = sodium.from_base64(app_key);
const cipher = sodium.crypto_box_seal(JSON.stringify(order), recipientPk);
const encrypted = sodium.to_base64(cipher);
```
В заголовках запроса вы отправляете `Auth-Identifier` со значением `app_id`.
`encrypted` - строка которая передается в теле запроса.

##### Описание ответа от API:
Успешный ответ:
```json
{
  "success": true,
  "payload": {
    "key": "value",
    "key2": "value2"
  }
}
```
Неуспешный ответ:
```json
{
  "success": false,
  "error": {
    "code": 4220103,
    "message": "Ошибка валидации",
    "validation": {
      "app_id": [
        "Неверный/несуществующий идентификатор приложения мерчанта."
      ]
    },
    "hint": "Для решения проблемы обратитесь в тех. поддержку с указанием кода ошибки."
  }
}
```
В `success` параметре значение результата запроса, если это `true` значит нужно смотреть параметр `payload` иначе
смотрим `error`.

В `payload` в зависимости от метода будут параметры результата запроса в виде ключ/значение.

В `error` собраны данные об ошибке:
 - `code` код ошибки
 - `message` сообщение об ошибке
 - `hint` более детальное пояснение
 
Если первые три цифры кода ошибки `422` это значит, что не прошла валидация и есть параметр `validation`
с массивом ошибок по полям, которые не прошли валидацию.

### Методы
##### Создание платежа
##### Проверка статуса платежа
##### Отмена платежа
##### Возврат платежа

