SmartDorm API Документация
1️⃣ Введение
Базовый URL: https://api.smartdorm.ru/v1

Ссылка на коллекцию postman: https://miles-294568.postman.co/workspace/miles's-Workspace~fcad82cc-0972-4227-8734-b4a4a9e034e8/collection/44856962-d76f0bcd-e3a3-455c-9e88-138a33518819?action=share&creator=44856962

Описание: API платформы SmartDorm предоставляет программный доступ к функциям умного общежития: управление доступом, учет ресурсов, бронирование зон, коммуникация и финансовые операции.

Формат данных: JSON для всех запросов и ответов.

Версия API: v1 (текущая)

Пример базового запроса:

```bash
curl -X GET "https://api.smartdorm.ru/v1/health" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```
2️⃣ Аутентификация
SmartDorm API использует OAuth 2.0 Bearer токены для аутентификации.

Получение токена:
```http
POST /auth/token
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=resident@dorm.ru&password=your_password&client_id=web_app
```
Пример ответа:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600,
  "refresh_token": "def50200e9b2a...",
  "scope": "resident"
}
```
Использование токена:
Добавьте заголовок в каждый запрос:

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
```
Роли и scope:

resident — жилец (доступ к своим данным)

admin — администратор (полный доступ)

service — сервисный работник (ограниченный доступ)

3️⃣ Коды ответов и ошибки
Успешные ответы:
200 OK — запрос выполнен успешно

201 Created — ресурс создан

204 No Content — успешно, но тело ответа отсутствует

Ошибки клиента:
400 Bad Request — неверный формат запроса

401 Unauthorized — требуется аутентификация

403 Forbidden — недостаточно прав

404 Not Found — ресурс не найден

409 Conflict — конфликт (например, двойное бронирование)

422 Unprocessable Entity — ошибка валидации данных

Ошибки сервера:
500 Internal Server Error — внутренняя ошибка сервера

503 Service Unavailable — сервис временно недоступен

Формат ошибки:
```json
{
  "error": {
    "code": "DOOR_LOCK_ERROR",
    "message": "Не удалось открыть дверь. Проверьте подключение замка.",
    "details": {
      "door_id": "room-205",
      "timestamp": "2024-03-15T14:30:00Z"
    }
  }
}
```
4️⃣ Описание Endpoints
📍 Управление доступом (Doors)
GET /doors/{door_id}/status
Получить текущий статус двери.

```URL: GET https://api.smartdorm.ru/v1/doors/{door_id}/status```

Path Parameters:

Параметр	Тип	Описание
door_id	string	Уникальный идентификатор двери (например: "room-205", "gym-1")
Headers:

```text
Authorization: Bearer YOUR_ACCESS_TOKEN
```
Пример запроса:

```bash
curl -X GET "https://api.smartdorm.ru/v1/doors/room-205/status" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```
Пример ответа: 200 OK

```json
{
  "door_id": "room-205",
  "status": "locked",
  "battery_level": 85,
  "last_opened": "2024-03-15T08:30:00Z",
  "last_opened_by": "Иван Петров",
  "online": true,
  "permissions": {
    "can_open": true,
    "can_manage": false,
    "temporary_keys_allowed": true
  }
}
```
Возможные ошибки:

401 — Требуется аутентификация

403 — Нет доступа к этой двери

404 — Дверь не найдена

503 — Сервис замков недоступен

```POST /doors/{door_id}/open```
Открыть дверь удаленно.

```URL: POST https://api.smartdorm.ru/v1/doors/{door_id}/open```

Path Parameters:

Параметр	Тип	Описание
door_id	string	Уникальный идентификатор двери
Request Body:

```json
{
  "method": "remote", // "remote", "nfc", "bluetooth"
  "reason": "entering_room",
  "force": false // Принудительное открытие (только для администраторов)
}
```
Пример запроса:

```bash
curl -X POST "https://api.smartdorm.ru/v1/doors/room-205/open" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "method": "remote",
    "reason": "entering_room"
  }'
```
Пример ответа: 200 OK

```json
{
  "success": true,
  "door_id": "room-205",
  "action": "open",
  "timestamp": "2024-03-15T14:35:22Z",
  "request_id": "req_7y2b8c9d0e1f2",
  "message": "Дверь успешно открыта"
}
```
Пример ошибки: 409 Conflict

```json
{
  "error": {
    "code": "DOOR_BUSY",
    "message": "Дверь уже открыта или в процессе открытия",
    "details": {
      "door_id": "room-205",
      "current_state": "unlocking",
      "retry_after": 5
    }
  }
}
```
📍 Бронирование помещений (Bookings)
GET /bookings/available-slots
Получить доступные слоты для бронирования.

```URL: GET https://api.smartdorm.ru/v1/bookings/available-slots```

Query Parameters:

Параметр	Тип	Обязательный	Описание
zone_type	string	Да	Тип зоны: laundry, kitchen, gym, study_room
date	string	Да	Дата в формате YYYY-MM-DD
duration	integer	Нет	Длительность в минутах (30, 60, 90, 120)
Пример запроса:

```bash
curl -X GET "https://api.smartdorm.ru/v1/bookings/available-slots?zone_type=laundry&date=2024-03-20&duration=90" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```
Пример ответа: 200 OK

```json
{
  "zone_type": "laundry",
  "date": "2024-03-20",
  "available_slots": [
    {
      "start_time": "2024-03-20T08:00:00Z",
      "end_time": "2024-03-20T09:30:00Z",
      "zone_id": "laundry-1",
      "max_capacity": 2,
      "current_bookings": 0,
      "price": 0
    },
    {
      "start_time": "2024-03-20T10:00:00Z",
      "end_time": "2024-03-20T11:30:00Z",
      "zone_id": "laundry-1",
      "max_capacity": 2,
      "current_bookings": 1,
      "price": 0
    }
  ],
  "pricing_info": {
    "base_price": 0,
    "currency": "RUB",
    "overtime_rate": 50
  }
}
```
POST /bookings
Создать новое бронирование.

```URL: POST https://api.smartdorm.ru/v1/bookings```

Request Body:

Поле	Тип	Обязательный	Описание
zone_id	string	Да	ID зоны для бронирования
start_time	string	Да	Время начала (ISO 8601)
end_time	string	Да	Время окончания (ISO 8601)
participants	array	Нет	Список ID участников
purpose	string	Нет	Цель использования
Пример запроса:

```bash
curl -X POST "https://api.smartdorm.ru/v1/bookings" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "zone_id": "gym-2",
    "start_time": "2024-03-20T18:00:00Z",
    "end_time": "2024-03-20T19:30:00Z",
    "participants": ["user_123", "user_456"],
    "purpose": "Тренировка"
  }'
```
Пример ответа: 201 Created

```json
{
  "booking_id": "book_abc123def456",
  "zone_id": "gym-2",
  "zone_name": "Спортзал (2 этаж)",
  "booked_by": "user_123",
  "start_time": "2024-03-20T18:00:00Z",
  "end_time": "2024-03-20T19:30:00Z",
  "status": "confirmed",
  "qr_code_url": "https://api.smartdorm.ru/v1/bookings/book_abc123def456/qr",
  "access_code": "4287",
  "created_at": "2024-03-15T15:20:00Z",
  "total_price": 0,
  "participants": [
    {
      "user_id": "user_123",
      "name": "Иван Петров",
      "status": "confirmed"
    },
    {
      "user_id": "user_456",
      "name": "Мария Сидорова",
      "status": "pending"
    }
  ]
}
```
Ошибки валидации (422 Unprocessable Entity):

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Ошибки валидации запроса",
    "details": {
      "start_time": ["Время начала должно быть в будущем"],
      "zone_id": ["Эта зона уже забронирована на указанное время"]
    }
  }
}
```
📍 Учет ресурсов (Meters)
GET /meters/consumption
Получить данные о потреблении ресурсов.

```URL: GET https://api.smartdorm.ru/v1/meters/consumption```

Query Parameters:

Параметр	Тип	Обязательный	Описание
meter_type	string	Нет	Тип счетчика: electricity, water, heating
period	string	Да	Период: day, week, month, year
date_from	string	Нет	Начальная дата (YYYY-MM-DD)
date_to	string	Нет	Конечная дата (YYYY-MM-DD)
granularity	string	Нет	Детализация: hour, day, month
Пример запроса:

```bash
curl -X GET "https://api.smartdorm.ru/v1/meters/consumption?meter_type=electricity&period=month&granularity=day" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```
Пример ответа: 200 OK

```json
{
  "room_id": "room-205",
  "meter_type": "electricity",
  "period": "2024-03",
  "total_consumption": 145.2,
  "unit": "kWh",
  "total_cost": 870.12,
  "currency": "RUB",
  "daily_data": [
    {
      "date": "2024-03-01",
      "consumption": 4.8,
      "cost": 28.8,
      "peak_hour": "19:00"
    },
    {
      "date": "2024-03-02",
      "consumption": 5.2,
      "cost": 31.2,
      "peak_hour": "21:00"
    }
  ],
  "comparison": {
    "previous_period": 138.5,
    "change_percent": 4.8,
    "building_average": 152.7,
    "your_position": "below_average"
  },
  "tips": [
    "Вы потребляете на 5% меньше среднего по общежитию",
    "Пиковое потребление в 21:00 - попробуйте перенести стирку на утро"
  ]
}
```
📍 Финансы и сплит-оплаты (Finance)
POST /finance/split-bills
Создать счет для раздельной оплаты.

```URL: POST https://api.smartdorm.ru/v1/finance/split-bills```

Request Body:

```json
{
  "title": "Пицца и напитки",
  "description": "Заказ пиццы на вечер кино",
  "total_amount": 2500,
  "currency": "RUB",
  "payer_id": "user_123",
  "split_type": "equal", // equal, percentage, custom
  "participants": [
    {
      "user_id": "user_123",
      "share": 1
    },
    {
      "user_id": "user_456",
      "share": 1
    },
    {
      "user_id": "user_789",
      "share": 1
    }
  ],
  "category": "food",
  "receipt_photo_url": "https://storage.smartdorm.ru/receipts/photo123.jpg"
}
```
Пример ответа: 201 Created

```json
{
  "bill_id": "bill_xyz789abc123",
  "title": "Пицца и напитки",
  "status": "pending",
  "total_amount": 2500,
  "currency": "RUB",
  "amount_per_person": 833.33,
  "payer": {
    "user_id": "user_123",
    "name": "Иван Петров",
    "amount_paid": 2500
  },
  "participants": [
    {
      "user_id": "user_123",
      "name": "Иван Петров",
      "amount_owed": 0,
      "status": "paid"
    },
    {
      "user_id": "user_456",
      "name": "Мария Сидорова",
      "amount_owed": 833.33,
      "status": "pending",
      "payment_link": "https://pay.smartdorm.ru/bill_xyz789abc123/user_456"
    },
    {
      "user_id": "user_789",
      "name": "Алексей Иванов",
      "amount_owed": 833.33,
      "status": "pending",
      "payment_link": "https://pay.smartdorm.ru/bill_xyz789abc123/user_789"
    }
  ],
  "created_at": "2024-03-15T20:15:00Z",
  "due_date": "2024-03-22T20:15:00Z",
  "qr_code": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUg..."
}
```
📍 Чат и коммуникация (Chat)
GET /chat/rooms/{room_id}/messages
Получить сообщения из чата комнаты.

```URL: GET https://api.smartdorm.ru/v1/chat/rooms/{room_id}/messages```

Query Parameters:

Параметр	Тип	Обязательный	Описание
limit	integer	Нет	Количество сообщений (default: 50)
before	string	Нет	ID сообщения, после которого загружать
after	string	Нет	ID сообщения, до которого загружать
Пример запроса:

```bash
curl -X GET "https://api.smartdorm.ru/v1/chat/rooms/floor-2-east/messages?limit=20" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```
Пример ответа: 200 OK

```json
{
  "room_id": "floor-2-east",
  "room_name": "2 этаж, восточное крыло",
  "messages": [
    {
      "id": "msg_001",
      "user_id": "user_123",
      "user_name": "Иван Петров",
      "user_avatar": "https://storage.smartdorm.ru/avatars/user_123.jpg",
      "content": "Кто сегодня хочет пиццу? Заказываю через приложение",
      "timestamp": "2024-03-15T19:30:00Z",
      "type": "text",
      "attachments": [],
      "reactions": {
        "🍕": ["user_456", "user_789"],
        "👍": ["user_101"]
      }
    },
    {
      "id": "msg_002",
      "user_id": "system",
      "user_name": "Система",
      "content": "Напоминание: завтра с 10:00 до 14:00 плановое отключение горячей воды",
      "timestamp": "2024-03-15T18:00:00Z",
      "type": "announcement",
      "priority": "high"
    }
  ],
  "has_more": true,
  "last_message_id": "msg_002",
  "unread_count": 3
}
```
POST /chat/messages
Отправить сообщение в чат.

```URL: POST https://api.smartdorm.ru/v1/chat/messages```

Request Body:

```json
{
  "room_id": "floor-2-east",
  "content": "Я оставил зарядку в спортзале. Кто видел?",
  "type": "text",
  "reply_to": "msg_001",
  "attachments": [
    {
      "type": "image",
      "url": "https://storage.smartdorm.ru/uploads/charger.jpg",
      "caption": "Вот такая зарядка"
    }
  ]
}
```
Пример ответа: 201 Created

```json
{
  "message_id": "msg_003",
  "room_id": "floor-2-east",
  "user_id": "user_456",
  "timestamp": "2024-03-15T20:45:00Z",
  "content": "Я оставил зарядку в спортзале. Кто видел?",
  "status": "sent",
  "attachments": [
    {
      "id": "att_001",
      "type": "image",
      "url": "https://storage.smartdorm.ru/uploads/charger.jpg",
      "thumbnail_url": "https://storage.smartdorm.ru/uploads/charger_thumb.jpg",
      "caption": "Вот такая зарядка",
      "size": 2048576
    }
  ]
}
```
📍 Пользователи и профили (Users)
PUT /users/me
Обновить данные текущего пользователя.

```URL: PUT https://api.smartdorm.ru/v1/users/me```

Request Body:

```json
{
  "display_name": "Иван П.",
  "phone_number": "+79991234567",
  "notifications": {
    "door_opened": true,
    "new_bill": true,
    "chat_message": true,
    "booking_reminder": true,
    "resource_alert": false
  },
  "privacy_settings": {
    "show_consumption_public": false,
    "show_in_roommate_search": true,
    "allow_contact_by_phone": false
  }
}
```
Пример ответа: 200 OK


```json
{
  "user_id": "user_123",
  "email": "ivan@dorm.ru",
  "display_name": "Иван П.",
  "room_id": "room-205",
  "phone_number": "+79991234567",
  "avatar_url": "https://storage.smartdorm.ru/avatars/user_123.jpg",
  "status": "active",
  "role": "resident",
  "notifications": {
    "door_opened": true,
    "new_bill": true,
    "chat_message": true,
    "booking_reminder": true,
    "resource_alert": false
  },
  "privacy_settings": {
    "show_consumption_public": false,
    "show_in_roommate_search": true,
    "allow_contact_by_phone": false
  },
  "updated_at": "2024-03-15T21:00:00Z"
}
```

DELETE /devices/{device_id}
Отвязать устройство от аккаунта.

```URL: DELETE https://api.smartdorm.ru/v1/devices/{device_id}```

Path Parameters:

Параметр	Тип	Описание
device_id	string	ID устройства (смартфон, планшет и т.д.)
Пример запроса:

```bash
curl -X DELETE "https://api.smartdorm.ru/v1/devices/phone_android_123abc" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```
Пример ответа: 204 No Content

```text
(тело ответа отсутствует)
```
Возможные ошибки:

404 — Устройство не найдено

403 — Нельзя удалить последнее активное устройство
