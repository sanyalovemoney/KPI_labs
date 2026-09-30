# Defense: Завдання 3 — Sequence позики книги

## 1. Намір і критерії

Деталізую UC-05 «Позичити книгу» (Lab2) як sequence: Читач → LoanService →
SubscriptionService/CatalogService → Repository, з alt по правилах
REQ-04/REQ-07 і ліміту з Plan. Критерії — в `spec.md`.

## 2. Топ-3 розбіжності (знайдені й виправлені)

1. **Учасник `BorrowController` ≠ `LoanService` зі spec, зник CatalogService**
   → проти критерію «учасники узгоджені з ER/сервісами».
   → Commit: `lab3: fix - rename controller to LoanService, add CatalogService`
2. **Жодної помилкової гілки: isActive = true константою, alt без else**
   → проти вимоги завдання (opt/alt/loop).
   → Commit: `lab3: fix - add alt branches for subscription, overdue, limit`
3. **Ліміт позики не звірявся з `Plan.max_concurrent_loans`** — правило
   REQ-04 не спиралося на поле з Lab1.
   → Commit: `lab3: fix - read max_concurrent_loans from Plan`

## 3. Ключове рішення

Який UC брати: позика замість підписки — бо в неї 3 правила-відмови, а
умова оцінює саме альтернативні гілки. Зовнішні системи при цьому лишаю
Lab4/Lab5.

→ `adr/adr-001-scenario-choice.md`

## 4. Перевірка узгодженості з попередніми моделями

- Кожен учасник діаграми прив'язаний до сутності Lab1: LoanService→Loan,
  SubscriptionService→Subscription/Plan, CatalogService→Book; звірено
  переліком.
- `due_at = now+30d` взято з формулювання REQ-04 (Lab2), а не вигадувано;
  прострочення визначене через `due_at`/`returned_at` — як у фіксах аудиту
  Lab1 (статус не зберігаємо).
- Кроки діаграми покроково відповідають сценарію UC-05 + include UC-06
  з Lab2; жодного крока, якого в UC немає.