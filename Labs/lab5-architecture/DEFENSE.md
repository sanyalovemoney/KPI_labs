# Defense: Завдання 5 — Компонентна модель

## 1. Намір і критерії

Модульний моноліт: Web UI -> REST API -> сім модулів домену ->
DataAccess -> Database, за межею — Платіжний шлюз, Поштовий сервіс,
OAuth-провайдер через required-інтерфейси й адаптери. Критерії в `spec.md`:
трасування кожного компонента до Lab1..4, чітка межа, інтерфейси, відсутність
циклів і зворотних залежностей.

## 2. Топ-3 розбіжності (знайдені й виправлені)

1. **Payment Gateway всередині межі системи** — суперечить актору Lab2 і
   ролі Lab4; всередині має бути тільки адаптер.
   → Commit: `lab5: fix - move payment gateway out of boundary, add adapter`
2. **Цикл LoanModule <-> SubscriptionModule** — Lab3 знає виклик лише в один
   бік; критерій «жодного циклу» порушений.
   → Commit: `lab5: fix - remove circular dependency loan-subscription`
3. **DataAccess не під'єднаний (зірваний рядок у вихіднику) і нуль
   інтерфейсів на межах**
   → Commit: `lab5: fix - wire DataAccess via repositories, add interfaces`

## 3. Ключове рішення

Модульний моноліт проти мікросервісів і проти безструктурного моноліту:
Lab3 малює синхронні виклики в межах однієї схеми — мікросервіси зробили б
їх розподіленими транзакціями, тобто Lab5 сперечалась би з Lab3. Без модулів
нереальний аудит трасування.

→ `adr/adr-001-modular-monolith.md`

## 4. Перевірка узгодженості з ланцюгом моделей

- **Lab1:** кожна сутність ER має репозиторій у DataAccess; Book/Author/
  Genre -> CatalogModule, Subscription/Plan -> SubscriptionModule,
  Loan -> LoanModule, Review -> ReviewModule, ReadingProgress ->
  ProgressModule, Payment -> PaymentModule, User -> AuthModule.
- **Lab2:** зовнішні актори шлюзу й пошти = компоненти за межею з
  required-інтерфейсами; Бібліотекар (модерація) — у ReviewModule.
- **Lab3:** послідовність Loan -> Subscription -> Repository — ребра
  графа без зворотних; цикл прибрано (фікс #2).
- **Lab4:** обидві гілки fork (активація і лист) — у різних компонентах
  (SubscriptionModule і адаптер `IEmailSender`), з'єднаних саме через
  цей інтерфейс.
- REQ-11 (повторний лист, непокритий у Lab2) має носія — фоновий процес
  AuthModule; ланцюг замикається без прорух.