### NOVATALKS WEBWIDGET API DOCUMENTATION

---

## 1. Postman Collection
Посилання на **Postman** колекцію для роботи:

[Widget Collection](widget-openapi.json)

---

## 2. Documentation
- [Widget Demo](#5-widget)
- [Widget Authorization](#6-widget-authorization)
- [Widget API](#7-widget-api)
- [Widget WebSocket](#8-widget-websocket)

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

## 5. Widget
Код для запуску віджету для якого було розроблено **widget api**:

```json
 (function (d) {
      if (!window.hasOwnProperty('novaTalks')) {
      window.novaTalks = {
        websiteToken: "ClCy4HYUKqkf7LABDB1foL",
        baseUrl: "https://devlight.cloud.novatalks.com.ua",
        isWidgetDynamic: true,
        widgetAssetsURL: "https://storage.novatalks.ai/static/widget/v2", 
        
      };
      try {
        const htmlHead = d.getElementsByTagName("head")[0];
        const htmlBody = d.body;
        const ntkWidgetBaseURL = "https://storage.novatalks.ai/static/widget/v2";
        const ntkWidgetJS = d.createElement("script");
        ntkWidgetJS.setAttribute("type", "text/javascript");
        ntkWidgetJS.setAttribute("async", false);
        ntkWidgetJS.setAttribute("src", ntkWidgetBaseURL.concat("/ntk-widget-sdk.js")); 
        htmlBody.appendChild(ntkWidgetJS);
      } catch (e) {
			  console.log('Exception: ', e);
		  }
    }
     })(document);
```

Запускати можно прямо на сторінці з UI. Можете його використувувати для розуміння роботи інтеграції.

---

## 6. Widget Authorization
Авторизація **websocket** клієнта і всіх запитів до **API** йде з використання **pubsubToken**

### Опис алгоритму авторизації:
1. Зробити **POST** запит на **/widget/conversations** для створення чату
2. У відповіді сервісу буде проперті **contactInbox** в якому буде **pubsubToken** токен
3. Передавати токен потрібно в **Auth-Widget** header при запиті на API

### Приклад запиту на отримання токену:
```http
POST /widget/conversations HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json

{
  "name": "widget-contact-494",
}
```

### Приклад відповіді:
```json
{
    "payload": {
        "contactInbox": {
            "pubsubToken": "kJGO53QxaEuhHaOU3h5Dc5"
        },
        ...(other data)
    }
}
```

---

## 7. Widget API
### Create new conversation for contact
Для створення чату потрібно передати **website_token** токен та обовзʼякове поле **name** в тілі запиту. Всі інші поля - опціональні.

**Запит:**
```http
POST /widget/conversations?website_token=Y5fUwtJiV1Tr6z7GQfKknW HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json

{
  "name": "widget-contact-494",
}
```

✅ **Успішна відповідь (200 OK)**
```json
{
    "payload": {
        "contact": {
            "id": 36,
            "name": "widget-contact-494",
            "email": null,
            "phone_number": null,
            "custom_attributes": {}
        },
        "contactInbox": {
            "pubsubToken": "kJGO53QxaEuhHaOU3h5Dc5"
        },
        "account_id": 1,
        "conversation": {
            "id": 27
        }
    }
}
```

### Get all messages for conversation(could be used for polling)
Для отримання повідомлень по чату потрібно передати **id** чату та **website_token** токен.
**Запит:**
```http
GET widget/conversations/27/messages?website_token=Y5fUwtJiV1Tr6z7GQfKknW HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
Auth-Widget: kJGO53QxaEuhHaOU3h5Dc5
```

✅ **Успішна відповідь(немає повідомлень) (200 OK)**
```json
[]
```

### Get selected messages for conversation
Для отримання повідомлень по чату потрібно передати **id** чату та **before** чи **after** query параметри з **id** повідомлення в чаті. Сервіс поверне 20 повідомлень, які підходять під критерії пошуку.
**Запит:**
```http
GET widget/conversations/27/messages?before=99 HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
Auth-Widget: kJGO53QxaEuhHaOU3h5Dc5
```

✅ **Успішна відповідь(немає повідомлень) (200 OK)**
```json
[]
```

### Create new message for contact
Для створення чату потрібно передати **id** чату, **website_token** токен та саме повідомлення в тілі запиту.

**Запит:**
```http
POST /widget/conversations/30/messages?website_token=Y5fUwtJiV1Tr6z7GQfKknW HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: application/json
Auth-Widget: kJGO53QxaEuhHaOU3h5Dc5

{
   "message":{
      "content":"hello",
      "echo_id":"724"
   },
   "conversationId":30,
   "accountId":1,
   "contactId":39
}
```

✅ **Успішна відповідь(тіла відповіді немає) (200 OK)**

**Запит з файлом:**
```http
POST /widget/conversations/30/messages?website_token=Y5fUwtJiV1Tr6z7GQfKknW HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Content-Type: multipart/form-data
Auth-Widget: kJGO53QxaEuhHaOU3h5Dc5

attachments[]: (binary)
message[echo_id]: 209
conversationId: 30
contactId: 39
accountId: 1
```

✅ **Успішна відповідь(тіла відповіді немає) (200 OK)**

---

## 8. Widget WebSocket
Після отримання **pubsubToken** токен ви можете встановити **websocket** зʼєднання для отримання ріалтайм данних:
```http
GET /ws HTTP/1.1
Host: devlight.cloud.novatalks.com.ua
Сonnection: Upgrade
Upgrade: websocket
```

Сервер відправляє такі івенти для **wіdget** клієнта:
| Івент | Опис |
|------------|--------------------------------|
| `subscribe` |  Підписка клієнта |
| `presence.update` | keepalive повідомлення |
| `conversation.created` | Івент про створення чату |
| `message.created` | Івент про створення повідомлення |
| `message.updated` | Івент про оновлення повідомлення |

### WebSocket Subscribe
Після підключення треба обязково зробити підписку на отримання івентів для **pubsubToken** токену:
```json
{
   "event":"subscribe",
   "data":{
      "pubsub_token":"kJGO53QxaEuhHaOU3h5Dc5",
      "account_id":1
   }
}	
```

✅ **Успішна відповідь від сервісу на підписку**
```json
{
   "event":"confirm_subscription",
   "data":"confirmed"
}
```

### WebSocket Keepalive
Сервіс буде відпрявляти івенти для перевірки статусу клієнта:
```json
{
   "event":"presence.update",
   "data":{
      
   }
}		
```

Очікуєма відповідь від клієнта на запит сервісу:
```json
{   
   "event":"update_presence"
}
```

### WebSocket conversation.created event
Нашим віджетом ми цей івент ігноруємо, але може бути корисний для вас:
```json
{
   "event":"conversation.created",
   "data":{
      "account_id":1,
      "id":28,
      "inbox_id":5,
      "inbox":{
         "id":5,
         "name":"Demo",
         "additional_settings":{
            "autoanswer":true,
            "utilization":20,
            "wrapup_code":false,
            "wrapup_timeout":30,
            "alerting_timeout":30,
            "conversation_type":"chat",
            "auto_assignment_limit":5
         }
      },
      "status":"open",
      "substatus":"oninbox",
      "priority":5,
      "subpriority":0,
      "assignee_id":null,
      "team_id":null,
      "channel":"WebWidget",
      "additional_attributes":{
         "chatId":null,
         "contactSource":"chat"
      },
      "custom_attributes":{
         
      },
      "dialog_id":"48f195c1-b9ff-4ee5-85ec-9031aed264ef",
      "messages":[
         
      ],
      "meta":{
         "sender":{
            "additional_attributes":{
               
            },
            "custom_attributes":{
               
            },
            "email":null,
            "id":37,
            "identifier":"1ReoP6Le5GFuP5xKDOSwHD",
            "name":"widget-contact-67",
            "phone_number":null,
            "thumbnail":"",
            "type":"contact",
            "labels":[
               
            ]
         },
         "assignee":null,
         "team":null,
         "labels":[
            
         ],
         "mentions":[
            
         ]
      },
      "labels":[
         
      ],
      "can_reply":true,
      "snoozed_until":null,
      "agent_last_seen_at":0,
      "contact_last_seen_at":0,
      "timestamp":1742150872,
      "created_at":1742150872,
      "session":null,
      "source_id":null
   }
}
```

### WebSocket message.created event
Сервіс відпрявляє івент при створення нового повідомлення в рамках чату.

Івент про створення нового повідомлення агентом:
```json
{
   "event":"message.created",
   "data":{
      "account_id":1,
      "id":8326,
      "content":"hello",
      "message_type":"outgoing",
      "content_type":"text",
      "private":false,
      "content_attributes":{
         
      },
      "source_id":null,
      "external_source_ids":{
         
      },
      "inbox_id":5,
      "sender_id":1,
      "status":"sent",
      "pinned":false,
      "conversation_id":27,
      "echo_id":"fad29d60472f",
      "dialog_id":"3181f2ce-2f84-438f-a04b-b97e794ba7e7",
      "attachments":[
         
      ],
      "sender":{
         "id":1,
         "name":"Support",
         "displayName":""
      },
      "created_at":1742149000
   }
}	
```
Івент про створення нового повідомлення клієнтом (зі сторони widget api):
```json
{
   "event":"message.created",
   "data":{
      "account_id":1,
      "id":8330,
      "content":"hello",
      "message_type":"incoming",
      "content_type":"text",
      "private":false,
      "content_attributes":{
         
      },
      "source_id":null,
      "external_source_ids":{
         
      },
      "inbox_id":5,
      "sender_id":36,
      "status":"sent",
      "pinned":false,
      "conversation_id":27,
      "echo_id":"589",
      "dialog_id":"cfda15a5-1d7c-46cf-b0b9-c3f0a5c5afaa",
      "attachments":[
         
      ],
      "sender":{
         "id":36,
         "name":"widget-contact-494"
      },
      "created_at":1742149759
   }
}
```

Івент про створення нового повідомлення з вкладенням:
```json
{
   "event":"message.created",
   "data":{
      "account_id":1,
      "id":8333,
      "content":null,
      "message_type":"outgoing",
      "content_type":"text",
      "private":false,
      "content_attributes":{
         
      },
      "source_id":null,
      "external_source_ids":{
         
      },
      "inbox_id":5,
      "sender_id":1,
      "status":"sent",
      "pinned":false,
      "conversation_id":27,
      "echo_id":"34e8c7b6476c",
      "dialog_id":"cfda15a5-1d7c-46cf-b0b9-c3f0a5c5afaa",
      "attachments":[
         {
            "id":41,
            "message_id":8333,
            "account_id":1,
            "file_type":"image",
            "fallback_title":null,
            "extension":null,
            "data_url":"https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
            "thumb_url":"https://store-url/store/30cbe6ea7db5752885bc5/telegram.png",
            "file_size":9473
         }
      ],
      "sender":{
         "id":1,
         "name":"Support",
         "displayName":""
      },
      "created_at":1742149989
   }
}
```

### WebSocket message.updated event
Сервіс відпрявляє івент при зміну аттрибутів повідомлення в рамках чату. На данний момент, івент потрібен тільки для інформування про зміну статусу повідомлення(поле **status**):
```json
{
   "event":"message.updated",
   "data":{
      "account_id":1,
      "id":8326,
      "content":"hello",
      "message_type":"outgoing",
      "content_type":"text",
      "private":false,
      "content_attributes":{
         
      },
      "source_id":null,
      "external_source_ids":{
         
      },
      "inbox_id":5,
      "sender_id":1,
      "status":"delivered",
      "pinned":false,
      "conversation_id":27,
      "echo_id":"fad29d60472f",
      "dialog_id":"3181f2ce-2f84-438f-a04b-b97e794ba7e7",
      "attachments":[
         
      ],
      "sender":{
         "id":1,
         "name":"Support",
         "displayName":""
      },
      "created_at":1742149000
   }
}	
```

---

