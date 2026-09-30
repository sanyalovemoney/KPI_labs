# Аудит: Завдання 3

Перевірка `model/borrow-book.mmd` (v1) проти `spec.md`, UC-05/06 із Lab2 і
ER-моделі Lab1.

## 1. Учасники не збігаються зі spec

На діаграмі є `BorrowController`, хоча в `spec.md` сервіс названий
`LoanService`, і зовсім зник `CatalogService` — той, що перевіряє наявність
книги в каталозі (Book із Lab1). Дві назви для одного компонента — пряма
розбіжність «модель ↔ spec».

→ Фікс: перейменувати `BorrowController` на `LoanService`, додати
`CatalogService` з `opt` «книги немає в каталозі».
→ Commit: `lab3: fix - rename controller to LoanService, add CatalogService`

## 2. Немає помилкових гілок, попри вимогу завдання

В умові прямо потрібні альтернативні/умовні гілки (opt/alt/loop). У v1 єдиний
`alt` без `else`, а `isActive` повертає константне `true`. Відсутні: відмова
при неактивній підписці (REQ-04, через `Subscription.expires_at` Lab1),
блокування через прострочення (REQ-07), перевищення ліміту, і `loop`
нагадувань по активних позиках.

→ Фікс: перебудувати `alt` з чотирма гілками (успіх / підписка / прострочення
/ ліміт), додати `opt` і `loop`.
→ Commit: `lab3: fix - add alt branches for subscription, overdue, limit`

## 3. Ліміт не звіряється з Plan — суперечить Lab1

`countActive` повертає 2, але порівняння ні з чим: у моделі Lab1
`max_concurrent_loans` лежить у `Plan`, а на діаграмі цей виклик відсутній —
тобто правило REQ-04 не спирається на дані, які для нього визначені.
Суперечність між моделями даних і взаємодії.

→ Фікс: додати `LoanService->>SubscriptionService: getMaxConcurrentLoans()`
(через Subscription→Plan) і порівняння в умові `alt`.
→ Commit: `lab3: fix - read max_concurrent_loans from Plan`

## Узгодженість з Lab1/Lab2 (перевірено, окрім #3 — проблем немає)

- `due_at = now + 30 days` з REQ-04 Lab2 = поле `Loan.due_at` у Lab1; статус
  позики не зберігається — виводиться з `returned_at`/`due_at`, як у фіксах
  аудиту Lab1.
- «Підписка неактивна» перевіряється по `Subscription.expires_at` — поле є
  в Lab1.