Расширённая ER-схема и домены для "Системы управления театром"

Цель файла: подробное описание сущностей, их атрибутов и доменов (множеств допустимых значений) в формате, совместимом с методичкой "трехэтапное проектирование базы данных".

1. Список сущностей и краткие описания

- shows (Спектакли): основная информация о спектакле
- actors (Актёры): данные актёров
- actor_show (Роль актёра в спектакле): pivot-таблица с ролью
- venues (Залы/Площадки): места, где проходят представления
- seat_sections (Секции зала): секции (балкон, партер и т.д.)
- seats (Места): конкретные места в секции
- performances (Даты показов / Экземпляры спектакля): конкретные показы (дата/время) привязанные к залу
- price_tiers (Тарифы/цены): цены по секциям/премиум/скидки
- tickets (Билеты): билеты на конкретное представление и место
- orders (Заказы): покупательские заказы, содержащие билеты
- payments (Платежи): информация о платёжных транзакциях
- users (Пользователи): зарегистрированные пользователи системы
- genres (Жанры): жанры спектаклей и pivot show_genre
- reviews (Отзывы): отзывы пользователей о спектаклях
- performance_stats (Статистика посещаемости): агрегированная статистика по показам


2. Схема отношений (основные связи)

- shows (1) -- (M) performances
- venues (1) -- (M) performances
- venues (1) -- (M) seat_sections
- seat_sections (1) -- (M) seats
- performances (1) -- (M) tickets
- tickets (M) -- (1) seats (опционально, если свободные места)
- users (1) -- (M) orders
- orders (1) -- (M) tickets (через order_items или связь через tickets.order_id)
- orders (1) -- (M) payments
- shows (M) -- (M) actors (через actor_show pivot с полем role)
- shows (M) -- (M) genres (через show_genre)
- performances (1) -- (1) performance_stats


3. Таблицы с атрибутами и доменами (формат: Атрибут — Домeн)

-- Таблица: users
id — Целое положительное число (bigint), primary key
name — Строка, varchar(255), не пустая
email — Строка, varchar(255), валидный email, уникальная
password — Строка, varchar(255), хеш пароля
is_admin — Булево (tinyint(1)), {0,1}, по умолчанию 0
created_at — Timestamp
updated_at — Timestamp

-- Таблица: venues
id — bigint
name — varchar(255)
address — varchar(500) (строка до 500 символов)
capacity — integer, >=0
description — text, nullable
created_at — timestamp
updated_at — timestamp

-- Таблица: seat_sections
id — bigint
venue_id — bigint, foreign key -> venues.id
name — varchar(100) (например: Партер, Балкон 1)
rows — smallint, количество рядов (>=0)
cols — smallint, приблизительное количество мест в секции (>=0)
description — varchar(255), nullable

-- Таблица: seats
id — bigint
section_id — bigint, foreign key -> seat_sections.id
row — smallint, >=1
number — smallint, >=1
seat_type — enum('standard','premium','vip','disabled'), default 'standard'
is_active — boolean, true если место используется

-- Таблица: shows
id — bigint
title — varchar(255) NOT NULL
director — varchar(150) nullable
duration_minutes — integer, >0
description — text nullable
poster_path — varchar(255) nullable
status — enum('draft','running','archived') default 'draft'
created_at, updated_at

-- Таблица: actor_show (pivot)
id — bigint
actor_id — bigint -> actors.id
show_id — bigint -> shows.id
character_name — varchar(150) nullable (роль)
billing_order — smallint nullable (порядок в афише)

-- Таблица: actors
id — bigint
full_name — varchar(150) NOT NULL
bio — text nullable
birth_date — date nullable
photo_path — varchar(255) nullable

-- Таблица: genres
id — bigint
name — varchar(80) NOT NULL, уникальное

-- Таблица: show_genre (pivot)
show_id — bigint -> shows.id
genre_id — bigint -> genres.id

-- Таблица: performances
id — bigint
show_id — bigint -> shows.id
venue_id — bigint -> venues.id
performance_date — date NOT NULL
performance_time — time NOT NULL
duration_minutes — integer nullable (если отличается от show.duration)
status — enum('scheduled','cancelled','postponed','completed') default 'scheduled'
seats_map_version — varchar(50) nullable (версия/снимок посадочных мест)

-- Таблица: price_tiers
id — bigint
show_id — bigint nullable (если цена специфична для шоу)
section_id — bigint nullable (если цена привязана к секции)
name — varchar(100) nullable (например: "Премиум", "Детский")
price — decimal(10,2) NOT NULL
currency — varchar(3) default 'RUB'
valid_from — date nullable
valid_to — date nullable

-- Таблица: tickets
id — bigint
performance_id — bigint -> performances.id
seat_id — bigint nullable -> seats.id (если свободная нумерация)
order_id — bigint nullable -> orders.id
purchaser_id — bigint nullable -> users.id
price — decimal(10,2)
status — enum('reserved','sold','checked_in','refunded','cancelled') default 'reserved'
qr_code — varchar(255) nullable
issued_at — timestamp nullable
checked_in_at — timestamp nullable

-- Таблица: orders
id — bigint
user_id — bigint nullable -> users.id (или nullable для гостя)
total_amount — decimal(12,2)
status — enum('pending','paid','cancelled','refunded') default 'pending'
created_at, updated_at

-- Таблица: payments
id — bigint
order_id — bigint -> orders.id
provider — varchar(50) (e.g., 'stripe','yoomoney')
amount — decimal(12,2)
currency — varchar(3) default 'RUB'
status — enum('pending','paid','failed','refunded')
transaction_id — varchar(255) nullable
paid_at — timestamp nullable

-- Таблица: performance_stats
id — bigint
performance_id — bigint -> performances.id
date_calculated — date
tickets_sold — integer
revenue — decimal(12,2)
checked_in_count — integer

-- Таблица: reviews
id — bigint
user_id — bigint -> users.id
show_id — bigint -> shows.id
rating — tinyint (1..5)
comment — text nullable
created_at, updated_at


4. Примеры доменов в формате методички (примеры)

Ниже — образцовая таблица доменов для некоторых сущностей (полная таблица аналогичная для всех атрибутов выше может быть сгенерирована по запросу).

Сущность | Атрибут | Домен
---------|---------|------
Shows | title | Строка, varchar(255), не пустая
Shows | director | Строка, varchar(150), может быть NULL
Shows | duration_minutes | Целое число > 0
Shows | description | Текст, произвольной длины, NULL

Users | name | Строка, varchar(255), не пустая
Users | email | Строка, varchar(255), валидный email, уникальная
Users | is_admin | Булево: {0,1}

Venues | name | Строка, varchar(255)
Venues | address | Строка, varchar(500)
Venues | capacity | Целое число >= 0

SeatSections | rows | Целое число >= 0
SeatSections | cols | Целое число >= 0

Seats | row | Целое >=1
Seats | number | Целое >=1
Seats | seat_type | Перечислимый: 'standard','premium','vip','disabled'

Performances | performance_date | Дата (YYYY-MM-DD)
Performances | performance_time | Время (HH:MM:SS)
Performances | status | enum('scheduled','cancelled','postponed','completed')

Tickets | status | enum('reserved','sold','checked_in','refunded','cancelled')
Tickets | price | Дробное число с 2 знаками после запятой, >=0

Orders | status | enum('pending','paid','cancelled','refunded')
Payments | provider | Строка, varchar(50)
Payments | amount | Decimal(12,2)

Reviews | rating | Целое 1..5


5. Ограничения и индексы (рекомендации)

- Уникальный индекс на users.email
- Внешние ключи с ON DELETE CASCADE для зависимых сущностей (например, если performance удалён — удалить билеты), но продумать логику (возможно soft-delete)
- Индексы на performances.performance_date, tickets.status, orders.status для быстрого поиска
- Индекс составной: (performance_id, seat_id) в tickets для быстрой проверки занятости места


6. ER-диаграмма (текстовое описание)

Главные узлы: shows, performances, venues, seat_sections, seats, tickets, orders, users.
- shows связаны с performances (1:N)
- performances связаны с venues (N:1)
- venues -> seat_sections -> seats
- tickets привязаны к performance и к конкретному seat
- orders агрегируют несколько билетов, привязаны к пользователю


7. Что дальше

- Если видите структуру и домены ок — я сгенерирую SQL CREATE TABLE либо Laravel migrations для выбранной СУБД (MySQL/SQLite/PostgreSQL).
- Также могу экспортировать ER-схему в `.drawio` (xml) файл, если хотите визуальный файл.

Скажите: желаемая СУБД для миграций? Хотите, чтобы я сразу сгенерировал все Laravel migration-файлы? Или предпочитаете сначала просмотреть и откорректировать домены и сущности?

8. Спланируйте структуру приложения (итоговая схема)

Ниже — сжатая и понятная схема компонентов приложения и их ответственности. Я подготовил соответствующий файл диаграммы `theater_app_structure.drawio` в репозитории (простая схема с основными блоками).

- Модели: Users, Shows, Performances, Venues, SeatSections, Seats, PriceTiers, Tickets, Orders, Payments, Actors, Genres, Actor_Show, Show_Genre, Reviews, Performance_Stats
- Миграции: по одной миграции на каждую основную таблицу, индексами и FK согласно ER-схеме
- Контроллеры: Resource-контроллеры для основных сущностей (ShowController, PerformanceController, TicketController, OrderController, UserController, VenueController)
- Views (Blade): layout, index/show/create/edit для основных ресурсов, компоненты форм, ошибки валидации
- Auth: регистрация/вход (Laravel Breeze/Jetstream можно подключить), middleware для ролей (admin)
- Службы (Services): опционально — PaymentService, TicketAllocationService, QRCodeService
- Очереди/Jobs: TicketIssuanceJob, SendPaymentReceiptJob (если используются асинхронные операции)

9. Оформление таблиц (образец)

Формат для каждой таблицы — заголовок с именем таблицы, затем список столбцов в виде: `имя: тип — описание`.

Пример (Users):

-- Таблица: users
id: bigint PRIMARY KEY — уникальный идентификатор
name: varchar(255) NOT NULL — полное имя
email: varchar(255) NOT NULL UNIQUE — email пользователя
password: varchar(255) NOT NULL — хеш пароля
is_admin: boolean DEFAULT false — флаг администратора
created_at: timestamp
updated_at: timestamp

10. Окончательная таблица со всеми полями (переработанная, готовая к миграциям)

-- users
id: bigint PRIMARY KEY
name: varchar(255) NOT NULL
email: varchar(255) NOT NULL UNIQUE
password: varchar(255) NOT NULL
is_admin: boolean NOT NULL DEFAULT false
created_at: timestamp
updated_at: timestamp

-- venues
id: bigint PRIMARY KEY
name: varchar(255) NOT NULL
address: varchar(500)
capacity: integer NOT NULL DEFAULT 0
description: text NULL
created_at: timestamp
updated_at: timestamp

-- seat_sections
id: bigint PRIMARY KEY
venue_id: bigint NOT NULL REFERENCES venues(id) ON DELETE CASCADE
name: varchar(100) NOT NULL
rows: smallint NOT NULL DEFAULT 0
cols: smallint NOT NULL DEFAULT 0
description: varchar(255) NULL

-- seats
id: bigint PRIMARY KEY
section_id: bigint NOT NULL REFERENCES seat_sections(id) ON DELETE CASCADE
row: smallint NOT NULL
number: smallint NOT NULL
seat_type: varchar(20) NOT NULL CHECK (seat_type IN ('standard','premium','vip','disabled'))
is_active: boolean NOT NULL DEFAULT true

-- shows
id: bigint PRIMARY KEY
title: varchar(255) NOT NULL
director: varchar(150) NULL
duration_minutes: integer NOT NULL
description: text NULL
poster_path: varchar(255) NULL
status: varchar(20) NOT NULL DEFAULT 'draft' -- enum('draft','running','archived')
created_at: timestamp
updated_at: timestamp

-- actors
id: bigint PRIMARY KEY
full_name: varchar(150) NOT NULL
bio: text NULL
birth_date: date NULL
photo_path: varchar(255) NULL

-- actor_show
id: bigint PRIMARY KEY
actor_id: bigint NOT NULL REFERENCES actors(id) ON DELETE CASCADE
show_id: bigint NOT NULL REFERENCES shows(id) ON DELETE CASCADE
character_name: varchar(150) NULL
billing_order: smallint NULL

-- genres
id: bigint PRIMARY KEY
name: varchar(80) NOT NULL UNIQUE

-- show_genre
show_id: bigint NOT NULL REFERENCES shows(id) ON DELETE CASCADE
genre_id: bigint NOT NULL REFERENCES genres(id) ON DELETE CASCADE

-- performances
id: bigint PRIMARY KEY
show_id: bigint NOT NULL REFERENCES shows(id) ON DELETE CASCADE
venue_id: bigint NOT NULL REFERENCES venues(id) ON DELETE CASCADE
performance_date: date NOT NULL
performance_time: time NOT NULL
duration_minutes: integer NULL
status: varchar(20) NOT NULL DEFAULT 'scheduled' -- enum('scheduled','cancelled','postponed','completed')
seats_map_version: varchar(50) NULL

-- price_tiers
id: bigint PRIMARY KEY
show_id: bigint NULL REFERENCES shows(id) ON DELETE SET NULL
section_id: bigint NULL REFERENCES seat_sections(id) ON DELETE SET NULL
name: varchar(100) NULL
price: decimal(10,2) NOT NULL
currency: varchar(3) NOT NULL DEFAULT 'RUB'
valid_from: date NULL
valid_to: date NULL

-- tickets
id: bigint PRIMARY KEY
performance_id: bigint NOT NULL REFERENCES performances(id) ON DELETE CASCADE
seat_id: bigint NULL REFERENCES seats(id) ON DELETE SET NULL
order_id: bigint NULL REFERENCES orders(id) ON DELETE SET NULL
purchaser_id: bigint NULL REFERENCES users(id) ON DELETE SET NULL
price: decimal(10,2) NOT NULL
status: varchar(20) NOT NULL DEFAULT 'reserved' -- enum('reserved','sold','checked_in','refunded','cancelled')
qr_code: varchar(255) NULL
issued_at: timestamp NULL
checked_in_at: timestamp NULL

-- orders
id: bigint PRIMARY KEY
user_id: bigint NULL REFERENCES users(id) ON DELETE SET NULL
total_amount: decimal(12,2) NOT NULL
status: varchar(20) NOT NULL DEFAULT 'pending' -- enum('pending','paid','cancelled','refunded')
created_at: timestamp
updated_at: timestamp

-- payments
id: bigint PRIMARY KEY
order_id: bigint NOT NULL REFERENCES orders(id) ON DELETE CASCADE
provider: varchar(50) NOT NULL
amount: decimal(12,2) NOT NULL
currency: varchar(3) NOT NULL DEFAULT 'RUB'
status: varchar(20) NOT NULL -- enum('pending','paid','failed','refunded')
transaction_id: varchar(255) NULL
paid_at: timestamp NULL

-- performance_stats
id: bigint PRIMARY KEY
performance_id: bigint NOT NULL REFERENCES performances(id) ON DELETE CASCADE
date_calculated: date NOT NULL
tickets_sold: integer NOT NULL
revenue: decimal(12,2) NOT NULL
checked_in_count: integer NOT NULL

-- reviews
id: bigint PRIMARY KEY
user_id: bigint NOT NULL REFERENCES users(id) ON DELETE CASCADE
show_id: bigint NOT NULL REFERENCES shows(id) ON DELETE CASCADE
rating: smallint NOT NULL CHECK (rating >=1 AND rating <=5)
comment: text NULL
created_at: timestamp
updated_at: timestamp

11. Обязательные компоненты приложения (минимум для РГР)

- Модели и миграции: не менее 3 связанных моделей (например, Shows, Performances, Tickets) + остальные модели и миграции по схеме
- Отношения: hasMany/belongsTo/belongsToMany в моделях
- Контроллеры: Resource controllers для основных сущностей (Shows, Performances, Tickets, Orders, Users)
- Представления: Blade layout + CRUD-шаблоны для ключевых сущностей
- Формы и валидация: серверная валидация в контроллерах/FormRequest, отображение ошибок во view
- Аутентификация: регистрация/вход, middleware для админ-панели
- Дополнительно: поиск, пагинация, загрузка постеров (show.poster_path), генерация QR для билетов

12. Дальше — миграции

Если подтверждаете эту структуру, я сгенерирую Laravel migration-файлы для PostgreSQL (в `database/migrations/`) и добавлю пример базового сидера и инструкции по запуску миграций.

Файл диаграммы структуры приложения: `theater_app_structure.drawio` (создан рядом с README).

---
Обновил файл согласно вашему формату; сообщите, если нужно изменить имена полей/доменов перед генерацией миграций.
