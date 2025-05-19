### NOVATALKS SDK API DOCUMENTATION 

---

## 1. Postman Collection

Посилання на **Postman** колекцію для роботи:

[SDK Collection](sdk-openapi.json)

---

## 2. Documentation

- [SDK Authorization](#6-sdk-authorization)
- [SDK API](#7-sdk-api)
- [SDK WebSocket](#8-sdk-websocket)
- [SDK Interactive Messages](#9-sdk-interactive-messages)

---

## 3. Host

### Stage Environment:

#### UI:

- [https://devlight.cloud.novatalks.com.ua](https://devlight.cloud.novatalks.com.ua/)

#### WebSocket:

- wss://devlight.cloud.novatalks.com.ua/ws

---

## 4. Credentials

Були раніше передані при проведені тренінгу з продукту. При необхідності можемо скинути пароль чи нагадати логін.

---

## 6. SDK Authorization

Авторизація **websocket** клієнта і всіх запитів до **API** йде з використання **pubsubToken** (header access_token)

### Опис алгоритму авторизації:

1. Зробити **POST** запит на **/api/v1/sdk/conversations** для створення чату
2. У відповіді сервісу буде проперті **contactInbox** в якому буде **pubsubToken** токен
3. Передавати токен потрібно в **access_token** header при запиті на API

### Приклад запиту на отримання токену:

```http
POST /api/v1/sdk/conversations HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json

{
  "name": "sdk-contact-494",
}
```

### Приклад відповіді:

```json
{
    "payload": {
        "contactInbox": {
            "pubsubToken": "kJGO53QxaEuhHaOU3h5Dc5"
        },
         "conversation": {
          "id": 1,
          "status": "open"
        },
        ...(other data)
    }
}
```

---

## 7. SDK API

### Create new conversation for contact

Для створення чату потрібно передати обовзʼякове поле **name** в тілі запиту. Всі інші поля - опціональні.

**Запит:**

```http
POST /api/v1/sdk/conversations HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json

{
  "name": "sdk-contact-494",
}
```

✅ **Успішна відповідь (200 OK)**

```json
{
  "payload": {
    "contact": {
      "id": 36,
      "name": "sdk-contact-494",
      "email": null,
      "phone_number": null,
      "custom_attributes": {}
    },
    "contactInbox": {
      "pubsubToken": "kJGO53QxaEuhHaOU3h5Dc5"
    },
    "account_id": 1,
    "conversation": {
      "id": 27,
      "status": "resolved"
    }
  }
}
```

### Get all messages for conversation(could be used for polling)

Для отримання повідомлень по чату потрібно виконати запит
**Запит:**

```http
GET /api/v1/sdk/conversations HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
access_token: kJGO53QxaEuhHaOU3h5Dc5
```

✅ **Успішна відповідь(немає повідомлень) (200 OK)**

```json
[]
```

### Get selected messages for conversation

Для отримання повідомлень по чату та **before** чи **after** query параметри з **id** повідомлення в чаті. Сервіс поверне 20 повідомлень, які підходять під критерії пошуку. Також є можливісь передати query параметр **updated_at** з timestamp в мілісекудах для пошуку повідомлень які більше за це значеня.
**Запит:**

```http
GET /api/v1/sdk/conversations/messages?before=99 HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
access_token: kJGO53QxaEuhHaOU3h5Dc5
```

✅ **Успішна відповідь(немає повідомлень) (200 OK)**

```json
[]
```

### Create new message for contact

Для створення повідомлення потрібно передати повідомлення в тілі запиту.

**Запит:**

```http
POST /api/v1/sdk/conversations/messages HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
access_token: kJGO53QxaEuhHaOU3h5Dc5

{
   "message":{
      "content":"hello",
      "echo_id":"724"
   }
}
```

✅ **Успішна відповідь(тіла відповіді немає) (200 OK)**

**Запит з файлом та текстом:**

```http
POST /api/v1/sdk/conversations/messages HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: multipart/form-data
access_token: kJGO53QxaEuhHaOU3h5Dc5

attachments[]: (binary)
message[content]:"hello",
message[echo_id]: 209
```

✅ **Успішна відповідь(тіла відповіді немає) (200 OK)**

---

**Запит з файлом:**

```http
POST /api/v1/sdk/conversations/messages HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: multipart/form-data
access_token: kJGO53QxaEuhHaOU3h5Dc5

attachments[]: (binary)
message[echo_id]: 209
```

✅ **Успішна відповідь(тіла відповіді немає) (200 OK)**

**Запит з реплай повідомленням:**

```http
POST /api/v1/sdk/conversations/messages HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
access_token: kJGO53QxaEuhHaOU3h5Dc5

{
   "message":{
      "content":"[RepliedMessageDialogId:]76af69fb-2366-40ed-b335-454d4725bc20[RepliedMessageId:]20163[RepliedContent:]Доброго дня[Content:]Вітаю",
      "echo_id":"724"
   }
}
```

✅ **Успішна відповідь(тіла відповіді немає) (200 OK)**

---

### Update message status

```http
PATCH /api/v1/sdk/conversations/messages/status HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
access_token: kJGO53QxaEuhHaOU3h5Dc5

{
  "status":"seen", // sent | delivered | seen | failed
  "messageIds": [1,2]
}
```

### Update interactive message attributes

```http
PATCH /api/v1/sdk/conversations/messages/477/interactive-attribute HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
access_token: kJGO53QxaEuhHaOU3h5Dc5

{
   "selected_content_attribute_value": "selected value or feedback input"
}

```

✅ **Успішна відповідь(тіла відповіді немає) (200 OK)**

## 8. SDK WebSocket

Після отримання **pubsubToken** токен ви можете встановити **websocket** зʼєднання для отримання ріалтайм данних:

```http
GET /ws HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Сonnection: Upgrade
Upgrade: websocket
```

Сервер відправляє такі івенти для **sdk** клієнта:
| Івент | Опис |
|------------|--------------------------------|
| `subscribe` | Підписка клієнта |
| `presence.update` | keepalive повідомлення |
| `conversation.status_changed` | Івент про зміну статусу розмови чату |
| `message.created` | Івент про створення повідомлення |
| `message.updated` | Івент про оновлення повідомлення |
| `typing` | Івент про те, щоб користувач бачив, що агент набирає повідомлення |

### WebSocket Subscribe

Після підключення треба обязково зробити підписку на отримання івентів для **pubsubToken** токену:

```json
{
  "event": "subscribe",
  "data": {
    "pubsub_token": "kJGO53QxaEuhHaOU3h5Dc5",
    "account_id": 1
  }
}
```

✅ **Успішна відповідь від сервісу на підписку**

```json
{
  "event": "confirm_subscription",
  "data": "confirmed"
}
```

### WebSocket Keepalive

Сервіс буде відпрявляти івенти для перевірки статусу клієнта:

```json
{
  "event": "presence.update",
  "data": {}
}
```

Очікуєма відповідь від клієнта на запит сервісу:

```json
{
  "event": "update_presence"
}
```

### WebSocket conversation.status_changed event

Нашим віджетом ми цей івент ігноруємо, але може бути корисний для вас:

```json
{
  "event": "conversation.status_changed",
  "data": {
    "account_id": 1,
    "id": 28,
    "inbox_id": 5,
    "inbox": {
      "id": 5,
      "name": "Demo",
      "additional_settings": {
        "autoanswer": true,
        "utilization": 20,
        "wrapup_code": false,
        "wrapup_timeout": 30,
        "alerting_timeout": 30,
        "conversation_type": "chat",
        "auto_assignment_limit": 5
      }
    },
    "status": "open",
    "substatus": "oninbox",
    "priority": 5,
    "subpriority": 0,
    "assignee_id": null,
    "team_id": null,
    "channel": "Sdk",
    "additional_attributes": {
      "chatId": null,
      "contactSource": "chat"
    },
    "custom_attributes": {},
    "dialog_id": "48f195c1-b9ff-4ee5-85ec-9031aed264ef",
    "messages": [],
    "meta": {
      "sender": {
        "additional_attributes": {},
        "custom_attributes": {},
        "email": null,
        "id": 37,
        "identifier": "1ReoP6Le5GFuP5xKDOSwHD",
        "name": "sdk-contact-67",
        "phone_number": null,
        "thumbnail": "",
        "type": "contact",
        "labels": [],
        "avatar_url": null
      },
      "assignee": null,
      "team": null,
      "labels": [],
      "mentions": []
    },
    "labels": [],
    "can_reply": true,
    "snoozed_until": null,
    "agent_last_seen_at": 0,
    "contact_last_seen_at": 0,
    "timestamp": 1745404967805,
    "created_at": 1745404967805,
    "session": null,
    "source_id": null
  }
}
```

### WebSocket message.created event

Сервіс відпрявляє івент при створення нового повідомлення в рамках чату.

Івент про створення нового повідомлення агентом:

```json
{
  "event": "message.created",
  "data": {
    "account_id": 1,
    "id": 8326,
    "content": "hello",
    "message_type": "outgoing",
    "content_type": "text",
    "private": false,
    "content_attributes": {},
    "source_id": null,
    "external_source_ids": {},
    "inbox_id": 5,
    "sender_id": 1,
    "status": "sent",
    "pinned": false,
    "conversation_id": 27,
    "echo_id": "fad29d60472f",
    "dialog_id": "3181f2ce-2f84-438f-a04b-b97e794ba7e7",
    "attachments": [],
    "sender": {
      "id": 1,
      "name": "Support",
      "display_name": "",
      "avatar_url": null
    },
    "created_at": 1745404967805
  }
}
```

Івент про створення нового повідомлення клієнтом (зі сторони sdk api):

```json
{
  "event": "message.created",
  "data": {
    "account_id": 1,
    "id": 8330,
    "content": "hello",
    "message_type": "incoming",
    "content_type": "text",
    "private": false,
    "content_attributes": {},
    "source_id": null,
    "external_source_ids": {},
    "inbox_id": 5,
    "sender_id": 36,
    "status": "sent",
    "pinned": false,
    "conversation_id": 27,
    "echo_id": "589",
    "dialog_id": "cfda15a5-1d7c-46cf-b0b9-c3f0a5c5afaa",
    "attachments": [],
    "sender": {
      "id": 36,
      "name": "sdk-contact-494",
      "avatar_url": null
    },
    "created_at": 1745404967805
  }
}
```

Івент про створення нового повідомлення з вкладенням:

```json
{
  "event": "message.created",
  "data": {
    "account_id": 1,
    "id": 8333,
    "content": null,
    "message_type": "outgoing",
    "content_type": "text",
    "private": false,
    "content_attributes": {},
    "source_id": null,
    "external_source_ids": {},
    "inbox_id": 5,
    "sender_id": 1,
    "status": "sent",
    "pinned": false,
    "conversation_id": 27,
    "echo_id": "34e8c7b6476c",
    "dialog_id": "cfda15a5-1d7c-46cf-b0b9-c3f0a5c5afaa",
    "attachments": [
      {
        "id": 41,
        "message_id": 8333,
        "account_id": 1,
        "file_type": "image",
        "fallback_title": null,
        "extension": null,
        "data_url": "https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
        "thumb_url": "https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
        "file_size": 9473
      }
    ],
    "sender": {
      "id": 1,
      "name": "Support",
      "display_name": "",
      "avatar_url": null
    },
    "created_at": 1745404967805,
    "reply": {
      "id": 3726,
      "sender": {
        "id": 244,
        "name": "sdk-1",
        "display_name": null,
        "avatar_url": null
      },
      "message_type": "incoming",
      "content_type": "text",
      "attachments": [
        {
          "id": 41,
          "message_id": 8333,
          "account_id": 1,
          "file_type": "image",
          "fallback_title": null,
          "extension": null,
          "data_url": "https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
          "thumb_url": "https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
          "file_size": 9473
        }
      ],
      "created_at": 1747090960816
    } // or null
  }
}
```

### WebSocket message.updated event

Сервіс відпрявляє івент при зміну аттрибутів повідомлення в рамках чату. На данний момент, івент потрібен тільки для інформування про зміну статусу повідомлення(поле **status**):

```json
{
  "event": "message.updated",
  "data": {
    "account_id": 1,
    "id": 8326,
    "content": "hello",
    "message_type": "outgoing",
    "content_type": "text",
    "private": false,
    "content_attributes": {},
    "source_id": null,
    "external_source_ids": {},
    "inbox_id": 5,
    "sender_id": 1,
    "status": "delivered", // sent | delivered | seen | failed
    "pinned": false,
    "conversation_id": 27,
    "echo_id": "fad29d60472f",
    "dialog_id": "3181f2ce-2f84-438f-a04b-b97e794ba7e7",
    "attachments": [],
    "sender": {
      "id": 1,
      "name": "Support",
      "display_name": "",
      "avatar_url": null
    },
    "created_at": 1745404967805,
    "reply": {
      "id": 3726,
      "sender": {
        "id": 244,
        "name": "sdk-1",
        "display_name": null,
        "avatar_url": null
      },
      "message_type": "incoming",
      "content_type": "text",
      "attachments": [
        {
          "id": 41,
          "message_id": 8333,
          "account_id": 1,
          "file_type": "image",
          "fallback_title": null,
          "extension": null,
          "data_url": "https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
          "thumb_url": "https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
          "file_size": 9473
        }
      ],
      "created_at": 1747090960816
    } // or null
  }
}
```

### WebSocket typing event

Сервіс відпрявляє івент агент(User) набирає повідомлення:

```json
{
  "event": "typing",
  "data": {
    "conversation_id": 1,
    "sender": {
      "id": 1,
      "name": "Support"
    },
    "sender_type": "User" // Contact || User
  }
}
```

Сервіс відпрявляє івент клієнт(Contact) набирає повідомлення:

```json
{
  "event": "typing",
  "data": {
    "conversation_id": 1,
    "sender": {
      "id": 1,
      "name": "ContactName"
    },
    "sender_type": "Contact" // Contact || User
  }
}
```

### Send WebSocket typing event from Contact:

```json
{
  "event": "typing",
  "data": {
    "conversation_id": 1,
    "account_id": 1
  }
}
```

---

## 9. SDK Interactive Messages

### Card

```json
{
  "content_type": "card",
  "content": "Explore the Mountains",
  "content_attributes": {
    "image_url": "https://example.com/images/alps.jpg", // optional
    "buttons": [
      {
        "type": "link",
        "title": "View Details",
        "url": "https://example.com/trips/alps"
      },
      {
        "type": "postback", // click on postback button - create message
        "title": "Book Now",
        "value": "BOOK_ALPS_TRIP"
      }
    ]
  }
}
```

### Link

```json
{
  "content_type": "card",
  "content": "Explore the Mountains",
  "content_attributes": {
    "buttons": [
      {
        "type": "link",
        "title": "View Details",
        "url": "https://example.com/trips/alps"
      }
    ]
  }
}
```

### Feedback

```json
{
  "content_type": "feedback",
  "content": "How would you rate your experience?",
  "content_attributes": {
    "selected_value": null // or feedback input value
  }
}
```

### QuickReply

```json
{
  "content_type": "quick_reply",
  "content": "How would you rate your experience?",
  "content_attributes": {
    "image_url": "https://example.com/images/alps.jpg", // optional
    "buttons": [
      { "title": "Standard (3–5 days)", "value": "SHIP_STANDARD" },
      { "title": "Express (1–2 days)", "value": "SHIP_EXPRESS" },
      { "title": "Overnight", "value": "SHIP_OVERNIGHT" }
    ],
    "selected_value": null // or value(SHIP_OVERNIGHT)
  }
}
```
