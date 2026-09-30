# AI-слід: Завдання 3

Інструмент: Claude Code. Шлях «З AI» (RULES.md).

## Промпти

**Промпт 1.** "Візьми UC-05 Позичити книгу з Lab2 і зроби Mermaid
sequenceDiagram: учасники LoanService, SubscriptionService, Repository;
додай alt на відмову позики."
Результат: `model/borrow-book.mmd` v1 (BorrowController замість LoanService,
без помилкових гілок).

**Промпт 2.** (аудит) "Звір учасників діаграми з таблицею в spec.md,
перевір що кожен alt має else, і що ліміт читається з Plan."

## Топ-3 помилки AI

1. **Назва учасника не узгоджена зі spec** — `BorrowController` замість
   `LoanService` (AUDIT #1). Фідбек AI: учасники діаграми = таблиця spec,
   один до одного.
   → Commit: `lab3: fix - rename controller to LoanService, add CatalogService`
2. **Згенерував «щасливий шлях» без else і без opt** — `isActive` константне
   true; умова прямо вимагає alt/opt/loop (AUDIT #2).
   → Commit: `lab3: fix - add alt branches for subscription, overdue, limit`
3. **Перевірку ліміту не прив'язав до Plan** — `countActive` без порівняння
   з `max_concurrent_loans` (суперечність з Lab1, AUDIT #3).
   → Commit: `lab3: fix - read max_concurrent_loans from Plan`