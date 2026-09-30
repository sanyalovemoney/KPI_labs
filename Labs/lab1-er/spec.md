# Spec: Завдання 1 — ER-модель онлайн-бібліотеки з підпискою

## Намір

Описати дані онлайн-бібліотеки з підпискою як ER-модель: сутності, атрибути,
зв'язки й кардинальності. Модель стане основою для наступних завдань —
вимог (Lab2), взаємодії (Lab3), процесу (Lab4) і архітектури (Lab5).

## Сутності та атрибути

Ідентифікатори всіх сутностей — `string` (uuid). Це фіксується тут, у spec,
а не лише в DEFENSE.

**User** — користувач сервісу.
`user_id` (PK), `email` (унікальний), `display_name`, `password_hash`,
`locale`, `registered_at`, `status`.

**Plan** — тариф підписки.
`plan_id` (PK), `code` (унікальний), `title`, `price_cents`, `currency`,
`duration_days`, `max_concurrent_loans`.

**Subscription** — оформлена підписка користувача.
`subscription_id` (PK), `user_id` (FK), `plan_id` (FK), `started_at`,
`expires_at`, `status`, `auto_renew`.

**Payment** — платіж за підписку.
`payment_id` (PK), `subscription_id` (FK), `amount_cents`, `currency`,
`paid_at`, `method`, `status`, `external_txn_id` (унікальний).

**Book** — книга каталогу.
`book_id` (PK), `isbn` (унікальний), `title`, `publisher_id` (FK),
`published_year`, `language`, `page_count`, `description`.

**Publisher** — видавництво.
`publisher_id` (PK), `name`, `country`.

**Author** — автор.
`author_id` (PK), `full_name`, `country`, `bio`.

**BookAuthor** — асоціативна сутність «книга — автор».
`book_id` (FK), `author_id` (FK), `role`, `ordinal`.
Складений PK: (`book_id`, `author_id`).

**Genre** — жанр, ієрархічний.
`genre_id` (PK), `code` (унікальний), `title`, `parent_genre_id` (FK на Genre).

**Loan** — видача книги користувачу.
`loan_id` (PK), `user_id` (FK), `book_id` (FK), `borrowed_at`, `due_at`,
`returned_at`. Стан позики (активна / повернена / прострочена) — похідний,
обчислюється з `returned_at` і `due_at`, не зберігається.

**Review** — відгук користувача про книгу.
`review_id` (PK), `user_id` (FK), `book_id` (FK), `rating`, `body`, `created_at`.
Пара (`user_id`, `book_id`) унікальна — один відгук на книгу від користувача.

**ReadingProgress** — прогрес читання.
`progress_id` (PK), `user_id` (FK), `book_id` (FK), `last_page`,
`updated_at`. Пара (`user_id`, `book_id`) унікальна. Відсоток прочитаного —
похідний від `last_page` і `Book.page_count`, не зберігається.

## Зв'язки та кардинальності

| Зв'язок | Кардинальність | Коментар |
|---------|----------------|----------|
| User — Subscription | 1 : N | у користувача багато підписок за історію |
| Plan — Subscription | 1 : N | тариф має багато підписок |
| Subscription — Payment | 1 : N | за строк підписки може бути кілька платежів |
| Publisher — Book | 1 : N | книга має одне видавництво |
| Book — BookAuthor | 1 : N | асоціативна сутність |
| Author — BookAuthor | 1 : N | асоціативна сутність |
| Book — Genre | M : N | чистий зв'язок, без власних атрибутів |
| Genre — Genre | 0..1 : N | ієрархія жанрів, self-reference; кореневі жанри не мають батька |
| User — Loan | 1 : N | |
| Book — Loan | 1 : N | |
| User — Review | 1 : N | |
| Book — Review | 1 : N | |
| User — ReadingProgress | 1 : N | |
| Book — ReadingProgress | 1 : N | |

## Критерії прийняття

- [ ] Кожна сутність має явно вказаний первинний ключ
- [ ] Складені обмеження унікальності (пара `user_id`,`book_id` у Review і
      ReadingProgress) виражені в моделі позначкою `UK`, а не лише текстом
- [ ] Усі ідентифікатори мають тип `string` (uuid) — без змішування `string` і `number`
- [ ] Модель у 3НФ: немає часткових залежностей від складеного ключа і немає транзитивних залежностей
- [ ] Похідні значення (`Loan.status`, `ReadingProgress.percent`) у моделі не
      зберігаються — обчислюються при читанні
- [ ] Зв'язок Book — Genre подано як чистий M:N, без сполучної сутності
- [ ] Зв'язок Book — Author подано як асоціативну сутність `BookAuthor`, бо зв'язок несе власні атрибути `role` і `ordinal`
- [ ] Назви атрибутів у spec і в `model/er.mmd` збігаються символ у символ
- [ ] Кардинальності на діаграмі відповідають таблиці вище
- [ ] Кардинальність самозв'язку `Genre` допускає жанр без батька (кореневі
      жанри) — `|o--o{`, а не `||--o{`
- [ ] Немає сутностей, не з'єднаних із рештою моделі
- [ ] Рендер у `renders/` збігається з `model/er.mmd`
- [ ] Немає фізичної схеми БД (SQL DDL / ORM) — лише концептуальна модель

## Поза межами

- Фізична схема БД, індекси, типи СУБД — предмет курсу баз даних.
- Ролі та права доступу (RBAC) — не модельовано, у домені їх немає.
- Аудіокниги, офлайн-читання — не входять у поточну версію домену.