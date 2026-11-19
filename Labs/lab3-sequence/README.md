# Lab3 — Sequence: «Оформити підписку»

## Що це за робота

Третя лаба розгортає UC з Lab2 у послідовності викликів. Обраний
«Оформити підписку»: у нього зовнішній актор (шлюз) і чотири
перевірювані сценарії з Lab2 Gherkin. На виході — чотири діаграми,
по одній на сценарій, з фрагментами alt/opt і коректними activation.

Домен той самий, учасники звірені з [ER-моделі Lab1](../lab1-er/model/er.mmd):
Сервіс підписок (Subscription, Plan), Сервіс платежів (Payment),
Платіжний шлюз. Поштового сервісу немає — [ADR-002 lab2](../lab2-usecases/adr/adr-002-mail-not-an-actor.md).

## Як читати артефакти

[spec.md](spec.md) — UC, сценарії, учасники, кроки, критерії.
[model/](model/) — чотири діаграми: seq-01-success, seq-02-rejection,
seq-03-inactive, seq-04-duplicate. [renders/](renders/) — їхні рендери.
[AUDIT.md](AUDIT.md) — знахідки аудиту. [ai/](ai/) — запит на аналіз і
фідбек. [adr/adr-001-gateway-callback-recipient.md](adr/adr-001-gateway-callback-recipient.md) —
маршрут callback. [DEFENSE.md](DEFENSE.md) — точка входу.

## Що змінилося порівняно з Lab2

Кроки Gherkin розгорнуті в повідомлення: додано ініціацію оплати
(authorizePayment з Сервісу підписок) і адресата callback
(onGatewayCallback до Сервісу платежів) — це рішення ADR-001.
Callback асинхронний, тому activation Сервісу підписок розділена.