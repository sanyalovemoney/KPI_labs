# Вимоги: онлайн-бібліотека з підпискою (цикл v2)

Цикл v2 деталізує один UC — «Оформити підписку». EARS і Gherkin нижче
описують саме його; решта UC лишається на діаграмі й у матриці без
переписування. Причина в spec: для Lab3 потрібен потік із зовнішнім актором
і перевірюваними альтернативами, а не дванадцять однорядкових вимог по всіх
UC.

Формат EARS. ID: REQ-01—REQ-05, нумерація наскрізна в межах циклу. Форму
позначаю біля id кожної вимоги: подійна WHEN, умовна IF…THEN. Оперую
термінами з ER-моделі Lab1: `User`, `Plan`, `Subscription`, `Payment`.
Нових сутностей і полів не вигадую.

Похідні поля не згадую: прострочення описую тільки через `Loan.due_at` і
`Loan.returned_at`, а те, скільки книги прочитано, виводиться з
`ReadingProgress.last_page` та `Book.page_count`.

## EARS-вимоги

**REQ-01 (WHEN).** WHEN читач із `User.status = "active"` обирає `Plan` і
підтверджує оформлення підписки, система SHALL створити `Subscription` зі
`status = "pending"` для обраного `Plan`.

**REQ-02 (WHEN).** WHEN Платіжний шлюз повертає успішну авторизацію списання
з `external_txn_id`, якого ще немає серед `Payment`, система SHALL створити
`Payment` з `amount_cents = Plan.price_cents`, `currency = Plan.currency`,
`status = "success"`, цим `external_txn_id` і `paid_at` = поточний момент,
а також перевести `Subscription` у `status = "active"` зі
`started_at = Payment.paid_at` і `expires_at` = `started_at` +
`Plan.duration_days` днів.

**REQ-03 (IF…THEN).** IF Платіжний шлюз повернув відмову авторизації
списання для `Subscription`, THEN система SHALL встановити
`Subscription.status = "canceled"`; `Payment` зі `status = "success"` для
цієї `Subscription` не створюється.

**REQ-04 (IF…THEN).** IF `User.status` не дорівнює `"active"`, THEN система
SHALL відхилити оформлення підписки: `Subscription` і `Payment` не
створюються, запит до Платіжного шлюзу не надсилається, а користувач
отримує помилку `user_not_active`.

**REQ-05 (WHEN).** WHEN Платіжний шлюз повторно повідомляє про успішну
авторизацію з `external_txn_id`, який уже є в `Payment`, система SHALL не
створювати другий `Payment`, не змінювати `Subscription.expires_at` і
повернути шлюзу той самий `payment_id`, що й під час першої обробки.

## Gherkin

User story вливаю в Feature, окремого розділу історій немає. Чотири
сценарії закривають успіх, відмову, неактивного користувача і повторну
транзакцію. Числа в Then беруться з Background або з кроку When, а не
з'являються вперше в перевірці.

```gherkin
Feature: Оформити підписку
  Як читач
  Я хочу оформити підписку на обраний тариф
  Щоб читати книги без очікування

  Background:
    Given у каталозі є Plan з code = "standard": price_cents = 49900, currency = "UAH", duration_days = 30, max_concurrent_loans = 5
    And читач "olya@example.com" має User зі status = "active"

  Scenario: успішна авторизація активує підписку (REQ-01, REQ-02)
    Given для "olya@example.com" створено Subscription зі status = "pending" для Plan "standard"
    When Платіжний шлюз повертає успішну авторизацію списання 49900 UAH з external_txn_id = "gw-8842"
    Then Payment створено: amount_cents = 49900, currency = "UAH", status = "success", external_txn_id = "gw-8842", paid_at = поточний момент
    And Subscription.status = "active", started_at = Payment.paid_at, expires_at = started_at + 30 днів

  Scenario: відмова авторизації скасовує підписку (REQ-03)
    Given для "olya@example.com" створено Subscription зі status = "pending" для Plan "standard"
    When Платіжний шлюз повертає відмову авторизації списання
    Then Subscription.status = "canceled"
    And кількість Payment зі status = "success" для цієї Subscription = 0

  Scenario: неактивний користувач не доходить до оплати (REQ-04)
    Given користувач "ihor@example.com" має User зі status = "pending"
    When "ihor@example.com" обирає Plan "standard" і підтверджує оформлення
    Then Subscription для "ihor@example.com" не створено
    And запит до Платіжного шлюзу не надіслано
    And система повертає помилку "user_not_active"

  Scenario: повторна транзакція не створює другий Payment (REQ-05)
    Given Subscription "olya@example.com" вже активна після Payment з payment_id = "pay-01" та external_txn_id = "gw-8842"
    When Платіжний шлюз повторно повідомляє про успіх з external_txn_id = "gw-8842"
    Then новий Payment не створено
    And кількість Payment для цієї Subscription = 1
    And Subscription.expires_at не змінився
    And шлюзу повернуто payment_id = "pay-01"
```

## Нефункціональні вимоги

NFR не описують крок актора, тому окремих Gherkin-сценаріїв для них немає.
Їхня перевірка — тести якості, а не сценарій UC.

| ID | Категорія | Вимога |
|----|-----------|--------|
| NFR-01 | Продуктивність | Пошук і фільтрація в каталозі: p95 ≤ 2 с при 500 одночасних читачах |
| NFR-02 | Продуктивність | Підтвердження оплати в інтерфейсі: p95 ≤ 5 с від натискання «Оплатити» до результату |
| NFR-03 | Доступність | 99,5% на місяць, тобто ≤ 3 год 36 хв простою на 30-денний місяць |
| NFR-04 | Безпека | Увесь трафік — HTTPS/TLS 1.2+; паролі зберігаються як bcrypt з cost ≥ 12 у `User.password_hash` |
| NFR-05 | Локалізація | Інтерфейс UA/EN; мова береться з `User.locale`, для нового користувача — з `Accept-Language` |
| NFR-06 | Сумісність | Chrome, Firefox і Safari останніх двох версій на дату релізу |
| NFR-07 | Інтеграція | Листи підтвердження й нагадування передаються зовнішньому поштовому сервісу; система фіксує спробу й не гарантує доставку |

Ті самі NFR-01…NFR-07 стоять у `spec.md` зі способом перевірки. У матриці
вони позначені як нетрасовані: прив'язка до одного UC зробила б вигляд, що
вимога належить лише йому, хоча вона діє на всю систему.