# Lab4 - Process: «Оформлення підписки»

## Що це за робота

Четверта лаба зшиває UC з Lab2 і sequence Lab3 в один бізнес-процес у формі
activity зі swimlanes. Показую сигналізацію шлюзу (send signal
`authorizePayment`, accept event `onGatewayCallback`), fork/join на
повідомленнях успіху і три розгалуження: статус користувача (REQ-04), дубль
транзакції (REQ-05), успіх/відмова (REQ-02/03).

Домен той самий, учасники звірені з [ER-моделі Lab1](../lab1-er/model/er.mmd).
Шлюз — тригер події, не виконавець ([ADR-001](adr/adr-001-gateway-as-event-trigger.md));
поштового сервісу немає — [ADR-002 lab2](../lab2-usecases/adr/adr-002-mail-not-an-actor.md).

## Як читати артефакти

[spec.md](spec.md) — процес, форма, ролі, повідомлення.
[model/](model/) — activity-діаграма `subscription-process.puml`.
[renders/](renders/) — рендер. [AUDIT.md](AUDIT.md) — знахідки аудиту.
[adr/](adr/) — рішення про роль шлюзу. [ai/](ai/) — запит на аналіз і фідбек.
[DEFENSE.md](DEFENSE.md) — точка входу.

## Що змінилося порівняно з Lab3

Кроки послідовностей зшиті в один процес; асинхронне вікно шлюзу —
send signal + detach + accept event (окрема точка входу); fork/join
на повідомленнях успіху.
