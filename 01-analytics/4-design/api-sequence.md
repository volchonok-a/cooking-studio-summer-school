# API Sequence Diagrams

## Описание
Документ описывает последовательности вызовов API для ключевых сценариев клиентского приложения в формате PlantUML. Каждая диаграмма показывает взаимодействие между Клиентом, Приложением, API и внешними сервисами.

**Обозначения:**
- `Клиент` — пользователь (актор)
- `Приложение` — клиентское веб-приложение (Frontend)
- `API` — серверная часть (Backend, black-box источник истины)
- `OAuth` — внешний OAuth-провайдер (Google/Яндекс)
- `Push` — сервис push-уведомлений (Web Push API)

---

## Сценарий 1: Авторизация через Email + пароль (UC-1, FR-01)

```plantuml
@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "API (auth)" as API

Client -> App: Ввод Email + пароль
App -> API: POST /auth/login {email, password}
API -> API: Проверка учётных данных
API -> App: 200 OK {accessToken, refreshToken, client}
App -> App: Сохранение токенов в httpOnly-куки (NFR-14)
App -> Client: Редирект на /slots (SCR-002)
@enduml

@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "OAuth Provider" as OAuth
participant "API (auth)" as API

Client -> App: Клик «Войти через Google/Яндекс»
App -> OAuth: GET /authorize (redirect)
OAuth -> Client: Страница авторизации
Client -> OAuth: Ввод учётных данных
OAuth -> App: Callback с code
App -> API: POST /auth/oauth {code, provider}
API -> API: Обмен кода на токены
API -> App: 200 OK {accessToken, refreshToken, client}
App -> App: Сохранение токенов в httpOnly-куки
App -> Client: Редирект на /slots
@enduml

@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "API (slots)" as API

Client -> App: Открытие главного экрана (SCR-002)
App -> API: GET /slots?date_from=now&date_to=now+7days
API -> API: Выборка слотов за 7 дней
API -> App: 200 OK [SlotResponse]
App -> App: Группировка по дате, сортировка по времени
App -> Client: Отображение списка слотов

alt Нет слотов
  App -> Client: Empty state «Пока нет доступных классов» (FR-08)
end

opt Офлайн-режим (NFR-9)
  App -> App: Чтение из localStorage/IndexedDB
  App -> Client: Отображение с плашкой «Данные могли устареть»
end
@enduml

@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "API (bookings)" as API
participant "Push Service" as Push

Client -> App: Заполнение формы (SCR-004)\nместа, экипировка, аллергии
App -> App: Валидация (аллергии обязательны, FR-12)
App -> App: Генерация Idempotency-Key (UUID, NFR-9)

Client -> App: Клик «Подтвердить бронь»
App -> API: POST /bookings {\n  slot_id,\n  guest_count,\n  equipment: "own"|"rental",\n  allergies,\n  idempotency_key\n}

API -> API: Атомарная проверка мест (NFR-7)\nФиксация цены, создание брони

alt Успех (201 Created)
  API -> App: 201 Created {BookingResponse}
  App -> App: Сохранение брони локально
  App -> Client: Экран подтверждения (SCR-005)\nСтатус «Ожидает оплаты» (FR-17)
  API -> Push: Push «Бронь подтверждена» (FR-26)
  Push -> Client: Push-уведомление
  
else Нет мест (409 Conflict)
  API -> App: 409 Conflict {error: "slot_full"}
  App -> Client: Сообщение «Места закончились» (BS-003)
  App -> Client: Кнопка «Вернуться к списку»
  
else Клиент заблокирован (403 Forbidden)
  API -> App: 403 Forbidden {error: "client_blocked"}
  App -> Client: Баннер «Записи недоступны до <дата>» (FR-29)
end
@enduml

@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "API (bookings)" as API

Client -> App: Клик «Отменить» (SCR-006)
App -> API: DELETE /bookings/{booking_id}
API -> API: Проверка времени (≥12 часов, FR-19)
API -> API: Освобождение места
API -> API: Статус → "cancelled_client"
API -> App: 200 OK {status: "cancelled"}
App -> App: Обновление списка бронирований
App -> Client: Бронь в статусе «Отменено клиентом»
@enduml

@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "API (bookings)" as API
participant "Push Service" as Push

Client -> App: Клик «Отменить» (SCR-006)
App -> App: Модалка подтверждения (SCR-007)
App -> Client: ️ «Отмена <12 часов → блокировка 7 дней» (FR-20)

Client -> App: Подтверждение (явное согласие)
App -> API: DELETE /bookings/{booking_id}

API -> API: Проверка времени (<12 часов)
API -> API: Освобождение места
API -> API: Статус → "cancelled_client"
API -> API: Блокировка клиента на 7 дней (FR-29)

API -> App: 200 OK {\n  status: "cancelled",\n  blocked_until: "2026-07-14"\n}

App -> App: Обновление списка
App -> Client: Бронь отменена

API -> Push: Push «Аккаунт заблокирован» (FR-26)
Push -> Client: Push-уведомление о блокировке
@enduml

@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "API (ratings)" as API

Client -> App: Клик «Оценить» (SCR-008)\nдля брони со статусом «Состоялся»
App -> App: Форма оценки (1-5 звёзд, FR-23)

Client -> App: Выбор оценки + клик «Отправить»
App -> API: POST /ratings {\n  booking_id,\n  chef_id,\n  stars: 1-5\n}

API -> API: Проверка статуса брони ("completed")
API -> API: Проверка: оценка ещё не ставилась
API -> API: Сохранение оценки
API -> API: Пересчёт рейтинга шефа (FR-24)

API -> App: 201 Created {RatingResponse}
App -> Client: «Спасибо за оценку!» (SCR-008)
App -> App: Обновление рейтинга в карточках слотов
@enduml

@startuml
actor "Клиент" as Client
participant "Приложение" as App
participant "Browser Push API" as PushAPI
participant "API (client)" as API

App -> Client: Запрос разрешения на push\n(при первом входе/брони, FR-27)
Client -> PushAPI: Разрешение дано/отклонено

alt Разрешение дано
  PushAPI -> PushAPI: Подписка на push
  PushAPI -> App: Push subscription {endpoint, keys}
  App -> API: POST /client/push-token {endpoint, keys}
  API -> API: Сохранение токена
  API -> App: 200 OK
  
else Разрешение отклонено
  App -> App: Push-уведомления отключены
  App -> App: Работает только in-app (колокольчик, SCR-010)
end
@enduml