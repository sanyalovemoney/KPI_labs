# Аудит: Завдання 4. Процес «Оформлення підписки»

## Цикл lab4 (25.11): діаграма проти spec, sequence Lab3 і ER

Перевіряв activity-діаграму проти `spec.md`, sequence Lab3
(`model/seq-01..04.puml`) і `er.mmd`. Запит на аналіз — `ai/01.md`,
рішення — `ai/feedback-01.md`.

### 1. Діра досяжності REQ-05 і хибний fork/join

Лінійний потік від `submitSubscription` не досягав повторної транзакції
(вона — callback без дії читача, seq-04 починається одразу з шлюзу),
а fork/join на асинхронному вікні блокував би читача (UML join чекає
ВСІ гілки). Виправлення: асинхронне вікно = send signal + detach +
accept event як окрема точка входу; fork/join перенесено на успішний
шлях (повернення `payment_id` | повідомлення читачу), обидві гілки
реально завершуються.

→ Коміт: `f8d66ba` lab4: spec (два потоки) + `ee13ea5` lab4: модель

### 2. Імена повідомлень процесу не збігалися з sequence Lab3

У spec бракувало `createSubscription`, `userNotActive`,
`subscriptionDeclined`, `subscriptionActivated`, `returnPaymentId`; таблиця
маркувала `createPayment` як "pay<->subs" (він self) і
`showResult`/`showError` як return (це виклики). Виправлено до дослівної
звірки з seq-01..04.

→ Коміт: `f8d66ba` lab4: spec

### 3. Нотаційні факти PlantUML activity (перевірені рендером, не на око)

(а) перший токен міток фрагментів (opt/alt/if) з'їдається
автолінком у PlantUML 1.2026.x — обхід: ведучий маркер "•";
(б) комбінація detach-в-if + плаваюча accept event + bare-identifier
аргументи падає 400-м на kroki-збірці, plantuml.com рендерить;
(в) перевизначення поточної доріжки всередині гілки + note ламає парс.
Кожен факт — окремим тестовим рендером.

→ Коміт: `ee13ea5` lab4: модель
