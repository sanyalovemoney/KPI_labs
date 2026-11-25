# Специфікація Lab4: процес «Оформлення підписки»

## Навіщо це

Lab3 показала послідовності кроків у чотирьох сценаріях; тепер показую цілий
процес — що система робить паралельно, де розгалужується і як чекає/не чекає
на зовнішній шлюз. Вхід — UC «Оформити підписку» з Lab2. Ролі беру з учасників
Lab3, дії — з повідомлень sequence-діаграм, числа й поля — з `er.mmd` Lab1 і
Gherkin Lab2. Нових сутностей і полів не вигадую.

## Форма (і чому)

Activity-діаграма в PlantUML зі swimlanes за ролями. BPMN не беру: у нього
своя семантика подій і шлюзів, якої курс не вимагає, а мої lab2 (use case) і
lab3 (sequence) уже в PlantUML — тримати весь ланцюг в одному інструменті
чесніше. Машину станів не беру: процес тут важливіший за життєвий цикл однієї
сутності — у мене кілька ролей і зовнішній шлюз, а переходи
`Subscription.status` і так видно в кроках.

Callback-потік — окрема точка входу процесу (accept event), тому
дубль-транзакція досяжна без дії читача. Fork на успішному шляху — свідома
абстракція: у sequence `returnPaymentId` і `subscriptionActivated` йдуть
послідовно, у процесі вони паралельні, бо залежності за даними між ними немає.

## Ролі (учасники з Lab3)

Читач, Веб-UI, Сервіс підписок, Сервіс платежів, Платіжний шлюз. Пошти немає
(ADR-002 lab2: не актор, деталь реалізації в NFR-07). Callback асинхронний
(ADR-001 lab3).

## Кроки, розгалуження, повідомлення

Числа ті самі, що в Gherkin Lab2: Plan "standard" — 49900, "UAH", 30 днів.
Процес — два потоки: оформлення (запуск читачем) і обробка callback (окрема
точка входу).

### Потік A — оформлення (запуск читачем)

1. Читач через Веб-UI надсилає `submitSubscription("standard")`.
2. Сервіс підписок перевіряє `User.status` (`checkUserActive`).
   - **Розгалуження [status != "active"]:** `Subscription` і `Payment` не
     створюються, запит до шлюзу не йде; Сервіс підписок повертає Веб-UI
     `userNotActive()`, Веб-UI показує `showError("user_not_active")`
     → кінець (REQ-04).
   - **[active]** → далі.
3. Веб-UI викликає `createSubscription(userId, "standard")`; Сервіс підписок
   читає `Plan` (`getPlanByCode`) і створює `Subscription` зі
   `status = "pending"` (REQ-01).
4. Сервіс підписок надсилає шлюзу `authorizePayment` (send signal) і detach:
   синхронний потік завершується — Веб-UI повертає читачу
   `subscriptionPending()`. Процес не чекає шлюз (ADR-001 lab3). Callback
   приймає Сервіс платежів (accept event) — це окрема точка входу: Потік B.

### Потік B — обробка callback (accept event, окрема точка входу)

5. Точка входу: accept event `onGatewayCallback(external_txn_id, result)`.
   Сюди входить і повторна транзакція (REQ-05) — без дії читача. Сервіс
   платежів шукає `Payment` за `external_txn_id` (`findPaymentByTxn`):
   - **[Payment уже існує]:** шлюзу повертається той самий `payment_id`
     (`returnPaymentId("pay-01")`); `expires_at` не змінюється → кінець
     (REQ-05).
   - **[немає]** → далі.
6. **Розгалуження [результат авторизації]:**
   - **[відмова]:** `cancelSubscription`; Веб-UI повідомляється
     (`subscriptionDeclined`) і показує `showError("authorization_declined")`
     → кінець (REQ-03).
   - **[успіх]:** `createPayment` (self, `status = "success"`, `paid_at`);
     активування `activateSubscription` → `status = "active"`,
     `started_at = Payment.paid_at`, `expires_at = started_at + 30 днів`
     (REQ-02); fork: гілка 1 — `returnPaymentId` шлюзу; гілка 2 —
     `subscriptionActivated` → Веб-UI → `showResult("active")`; join → кінець.

Повідомлення процесу — ті самі, що в sequence Lab3:

| Повідомлення | Напрям | Тип | Реакція процесу |
|--------------|--------|-----|-----------------|
| `submitSubscription` | Читач → Веб-UI | виклик | передати в Сервіс підписок |
| `createSubscription` | Веб-UI → Сервіс підписок | виклик | checkUserActive, getPlanByCode, створення Subscription |
| `checkUserActive` | Сервіс підписок (self) | виклик | розгалуження REQ-04 |
| `userNotActive` | Сервіс підписок → Веб-UI | return | showError("user_not_active") |
| `getPlanByCode` | Сервіс підписок (self) | виклик | джерело 49900 / "UAH" / 30 днів |
| `authorizePayment` | Сервіс підписок → Шлюз | **send signal** | detach: синхронний потік завершується після `subscriptionPending()` |
| `subscriptionPending` | Веб-UI → Читач | return | читач поінформований, процес триває без нього |
| `onGatewayCallback` | Шлюз → Сервіс платежів | **accept event** | окрема точка входу (Потік B): успіх / відмова / дубль |
| `findPaymentByTxn` | Сервіс платежів (self) | виклик | розгалуження «дубль чи ні» (REQ-05) |
| `returnPaymentId` | Сервіс платежів → Шлюз | виклик | шлюз отримує той самий payment_id |
| `createPayment` | Сервіс платежів (self) | виклик | Payment success, paid_at |
| `activateSubscription` | Сервіс платежів → Сервіс підписок | виклик | status = "active" (REQ-02) |
| `cancelSubscription` | Сервіс платежів → Сервіс підписок | виклик | status = "canceled" (REQ-03) |
| `subscriptionActivated` / `subscriptionDeclined` | Сервіс платежів → Веб-UI | виклик | showResult / showError читачу |
| `showResult` / `showError` | Веб-UI → Читач | виклик | фінальний стан для читача |

## Критерії прийняття

- [ ] рівно одна форма (activity), без другої діаграми
- [ ] ≥1 розгалуження if/else (статус користувача, результат шлюзу)
- [ ] ≥1 fork/join на реальній синхронізації (успішний шлях:
      returnPaymentId шлюзу | subscriptionActivated → showResult);
      асинхронне вікно шлюзу = send signal + detach + accept event
      як окрема точка входу, без fork
- [ ] ≥1 повідомлення з показаною реакцією: send signal шлюзу, accept event
      callback, продовження без очікування показано
- [ ] повідомлення процесу = повідомлення sequence Lab3 (authorizePayment,
      onGatewayCallback, activate/cancel, findPaymentByTxn)
- [ ] без недосяжних кроків і дір; кожен шлях веде до кінця
- [ ] рендер синхронний з джерелом

## Поза обсягом

Інші UC (позика, читання, відгук) лишаються поза цим процесом. Retry
планувальника і вітальні листи — предмет Lab5 (архітектура). Пошта — деталь
реалізації за ADR-002 lab2, серед ролей її немає.

Розбіжність сигнатури `onGatewayCallback` усередині lab3 (seq-02 передає
`subscriptionId`, seq-01/04 — `external_txn_id`): lab3 не переробляю (зданий
вчасно), для lab4 обрано єдину сигнатуру за `external_txn_id`.
