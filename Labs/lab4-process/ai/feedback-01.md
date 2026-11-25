# Фідбек 01: аналіз процесу (25.11)

## Що взяв із відповіді ШІ
1-5: у spec бракувало імен повідомлень (createSubscription, userNotActive,
subscriptionDeclined, subscriptionActivated, returnPaymentId) - процес
не дорівнював sequence за критерієм. Додав.
6-8: структура була хибна. Лінійний потік не досягає REQ-05 (дубль -
повторний callback без дії читача), а join блокував би читача
(UML join = AND-синхронізація). Перебудував: асинхронне вікно =
send signal + detach (sync-потік завершується) + accept event як
окрема точка входу (туди входить і дубль). Fork/join переніс на
успішний шлях: fork (returnPaymentId шлюзу | subscriptionActivated
-> showResult читачу), join перед кінцем.
12-14: таблицю виправив (createPayment - self; showResult/showError -
виклики; доповнив checkUserActive/getPlanByCode/subscriptionPending).

## Спірні місця - мої рішення
- Сигнатура onGatewayCallback: за external_txn_id (є розбіжність усередині
  lab3: seq-02 передає subscriptionId; lab3 не чіпаю - він зданий, для
  lab4 обрано єдину сигнатуру, фіксую в spec).
- Порядок returnPaymentId і subscriptionActivated у fork: у sequence
  послідовний, у процесі - паралельний. Це свідома абстракція
  (немає залежності за даними), зафіксовано в spec "Форма (і чому)".

## Що відхилив
〈наприклад:Gateway Adapter як учасник - не з ER; нічого, якщо все взяв〉

## Моя помилка, яку зловив аналіз
Я спершу запропонував join на асинхронному вікні. Аналіз показав:
join чекає ВСІ гілки - читач був би заблокований до відповіді шлюзу,
супереч ADR-001 lab3. Прийняв виправлення до будь-якого коміта діаграми.