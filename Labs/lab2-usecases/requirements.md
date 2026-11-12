# Вимоги: онлайн-бібліотека з підпискою

Формат EARS. ID: REQ-NN, нумерація наскрізна, REQ-01—REQ-12; форму позначаю
біля id кожної вимоги — подійна WHEN, станова WHILE…WHEN; IF…THEN описує
небажану поведінку (REQ-05, REQ-12).
Оперую термінами з ER-моделі Lab1: `User`, `Plan`, `Subscription`, `Payment`,
`Book`, `Loan`, `Review`, `ReadingProgress`. Нових сутностей і полів не
вигадую.

Похідні поля не згадую: прострочення описую тільки через `Loan.due_at` і
`Loan.returned_at`, а те, скільки книги прочитано, виводиться з
`ReadingProgress.last_page` та `Book.page_count` і у вимогах не фігурує.
Окремої позначки «приховано» в `Review` немає, тому модерація відгуку — це
видалення запису; пояснюю це в REQ-11.

## EARS-вимоги

**REQ-01 (WHEN).** WHEN читач надсилає форму реєстрації з `email`, якого ще
немає в `User`, система SHALL створити `User` зі `status = pending` і
`registered_at` = поточний момент та ініціювати надсилання листа підтвердження
на `User.email` через Поштовий сервіс.

**REQ-02 (WHEN).** WHEN читач підтверджує email переходом за посиланням з
листа, система SHALL встановити `User.status = active`.

**REQ-03 (WHEN).** WHEN читач із `User.status = active` обирає `Plan` і
підтверджує оформлення підписки, система SHALL створити `Subscription` зі
`status = pending` для обраного `Plan`.

**REQ-04 (WHEN).** WHEN Платіжний шлюз повертає успішну авторизацію списання
для `Subscription`, система SHALL створити `Payment` з
`amount_cents = Plan.price_cents`, `currency = Plan.currency`,
`status = success`, `external_txn_id` з відповіді шлюзу та `paid_at` = поточний
момент, а також перевести `Subscription` у `status = active` зі
`started_at = Payment.paid_at` і `expires_at` = `started_at` +
`Plan.duration_days` днів.

**REQ-05 (IF…THEN).** IF Платіжний шлюз повернув відмову авторизації списання
для `Subscription`, THEN система SHALL встановити
`Subscription.status = canceled`; `Payment` зі `status = success` для такої
`Subscription` не створюється.

**REQ-06 (WHILE…WHEN).** WHILE `Subscription` читача має `status = active` І
кількість його `Loan` із порожнім `returned_at` менша за
`Plan.max_concurrent_loans`, WHEN читач запитує позичити `Book`, система SHALL
створити `Loan` з `borrowed_at` = поточний момент і `due_at` = `borrowed_at` +
30 днів.

**REQ-07 (WHEN).** WHEN читач зберігає сторінку під час читання `Book`,
система SHALL записати `ReadingProgress` для пари (`user_id`, `book_id`):
`last_page` = збережена сторінка, `updated_at` = поточний момент; якщо запису
для цієї пари ще немає, система створює його.

**REQ-08 (WHEN).** WHEN для `Loan` із порожнім `returned_at` настає момент за
2 дні до `due_at`, система SHALL ініціювати надсилання нагадування про
повернення на `User.email` через Поштовий сервіс.

**REQ-09 (WHEN).** WHEN читач повідомляє про повернення `Book`, система SHALL
встановити `Loan.returned_at` = поточний момент.

**REQ-10 (WHEN).** WHEN читач надсилає відгук на `Book` із `rating` від 1 до 5
і `body`, система SHALL створити `Review` для пари (`user_id`, `book_id`)
з `created_at` = поточний момент. Повторний відгук на та саму книгу відкидає:
IF `Review` для цієї пари вже є, THEN новий відгук не створюється, а наявний
запис лишається без змін. Унікальність пари узгоджена з Lab1 (UK на парі
в `Review`).

**REQ-11 (WHEN).** WHEN Бібліотекар позначає `Review` як недоречний, система
SHALL видалити цей запис `Review` з каталогу. Примітка: окремої позначки
«приховано» модель Lab1 для `Review` не має, тому видалення запису — єдиний
модельно-легальний спосіб модерації.

**REQ-12 (IF…THEN).** IF лист підтвердження не підтверджено протягом 24 годин
після створення `User`, THEN система SHALL надіслати його повторно на
`User.email` рівно один раз.

Примітка: варіанта використання в REQ-12 немає — це фоновий таймерний процес,
який жоден актор не ініціює. Пояснюю це в `traceability.md`; носій вимоги
з'явиться в архітектурі Lab5.

## User stories

- **US-01.** Як Читач, я хочу зареєструватися і підтвердити email, щоб
  отримати доступ до сервісу. *(REQ-01, REQ-02)*
- **US-02.** Як Читач, я хочу обрати тариф і отримати підписку одразу після
  успішної оплати, щоб читати без очікування. *(REQ-03, REQ-04, REQ-05)*
- **US-03.** Як Читач, я хочу тримати кілька книг одночасно в межах тарифу і
  вчасно повертати їх, щоб не втрачати доступ до читання. *(REQ-06, REQ-09)*
- **US-04.** Як Читач, я хочу, щоб сервіс пам'ятав мою останню сторінку,
  нагадував про строк повернення і давав оцінити прочитане один раз на книгу,
  щоб не тримати все це в голові. *(REQ-07, REQ-08, REQ-10)*
- **US-05.** Як Бібліотекар, я хочу прибрати з каталогу недоречні відгуки, щоб
  читачі бачили коректні оцінки. *(REQ-11)*

## Gherkin

Три сценарії: один успішний (REQ-04) і два альтернативних (REQ-05, REQ-06).
У кожному Background достатньо стану, щоб Then не брав цифри з повітря.

```gherkin
Feature: Оформити підписку

  Background:
    Given у каталозі є Plan з code = "standard": price_cents = 49900, currency = "UAH", duration_days = 30, max_concurrent_loans = 5
    And читач "olya@example.com" має User зі status = "active"
    And для читача вже створено Subscription зі status = "pending" для обраного Plan (REQ-03 виконався)

  Scenario: успішна авторизація активує підписку (REQ-04)
    When Платіжний шлюз повертає успішну авторизацію списання 49900 UAH з id транзакції "gw-8842"
    Then Payment створено: amount_cents = 49900, currency = "UAH", status = "success", external_txn_id = "gw-8842", paid_at = поточний момент
    And Subscription.status = "active", started_at = Payment.paid_at, expires_at = started_at + 30 днів

  Scenario: відмова авторизації скасовує підписку (REQ-05)
    When Платіжний шлюз повертає відмову авторизації списання для Subscription
    Then Subscription.status = "canceled"
    And кількість Payment зі status = "success" для цієї Subscription = 0
```

```gherkin
Feature: Позичити книгу

  Background:
    Given читач "petro@example.com" має Subscription зі status = "active" (Plan з code = "standard", max_concurrent_loans = 5)
    And у читача вже 5 Loan з порожнім returned_at

  Scenario: шоста книга перевищує ліміт тарифу (REQ-06, заперечення передумови)
    When читач запитує позичити шосту Book
    Then новий Loan не створено
    And кількість Loan з порожнім returned_at лишається 5
```
