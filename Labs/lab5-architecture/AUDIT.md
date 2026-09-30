# Аудит: Завдання 5

Перевірка `model/components.puml` (v1) проти `spec.md`, трасування до
Lab1..Lab4.

## 1. Платіжний шлюз намальований всередині межі системи

У v1 `cloud "Payment Gateway"` стоїть у package "Backend". Це зовнішня
система-актор з Lab2 і роль swimlane з Lab4 — за критерієм «межа системи
одним прямокутом: зовнішні системи поза нею» вона мусить бути ззовні, а
середині має бути лише адаптер `PaymentModule`. Як залишити — Lab5
суперечила б Lab2.

→ Фікс: винести `PG` за package; в `PaymentModule` визначити
required-інтерфейс `IPaymentProvider` і окрему реалізацію-адаптер, який
дивиться назовні.
→ Commit: `lab5: fix - move payment gateway out of boundary, add adapter`

## 2. Цикл LoanModule <-> SubscriptionModule

У v1 є обидві стрілки: `Loan --> Subscription` і `Subscription --> Loan`.
За Lab3 перевірку підписки ініціює тільки позика (`LoanService` питає
`SubscriptionService`), зворотного виклику в жодній моделі ланцюга немає.
Критерій «жодного циклу» порушений, і трасування до Lab3 — теж.

→ Фікс: прибрати `Subscription --> Loan`; потреба показати число активних
позик закривається тим, що LoanModule питає `SubscriptionService.getMax
ConcurrentLoans()` — напрямок той самий.
→ Commit: `lab5: fix - remove circular dependency loan-subscription`

## 3. DataAccess не під'єднаний, і загалом немає жодного інтерфейсу

В v1 рядок `modулі --> [DataAccess]` — пуста назва замість вузла:
PlantUML створює фантомний вузол, репозиторій фактично ні з чим не
з'єднаний. І друге: усі зв'язки — голі стрілки, а критерій вимагає
provided/required хоча б на межах (шлюз, пошта, OAuth).

→ Фікс: замість фантомного рядка — DataAccess розпакується на йменні
репозиторії (UserRepository, BookRepository, SubscriptionRepository,
PaymentRepository, LoanRepository, ReviewRepository, ProgressRepository),
кожен модуль залежить від свого, усі вони впираються в Database; зовнішнім
точкам контакту — порти `IPaymentProvider`, `IEmailSender`, `IOAuthProvider`
з адаптерами всередині межі (ports-and-adapters).
→ Commit: `lab5: fix - wire DataAccess via repositories, add interfaces`

## Узгодженість ланцюга (перевірено, окрім #1/#2)

- `CatalogModule -> DB`: Book/Author/Genre/BookAuthor — Lab1.
- `ProgressModule`: REQ-05, ReadingProgress (унікальна пара user+book —
  ключ з аудиту Lab1).
- `ReviewModule`: REQ-06/10/12, UC-09 (відгук) і UC-11 (модерація, Бібліотекар)
  з Lab2 — нумерація по виправленій діаграмі.
- `AuthModule`: REQ-01 + повторний лист REQ-11 (той самий непокритий UC —
  тут він покритий компонентом, бо фонові процеси живуть у коді, а не в
  варіантах використання).