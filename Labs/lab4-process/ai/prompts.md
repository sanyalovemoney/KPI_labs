# AI-слід: Завдання 4

Інструмент: Claude Code. Шлях «З AI».

## Промпти

**Промпт 1.** "Для домену online-biblioneka з Laboratorії 1-2, процес
підписки: activity-діаграма Mermaid flowchart зі swimlanes читач/система/
платіжний шлюз, fork на активацію+лист, alt на прострочення."
Результат: `model/subscription-lifecycle.mmd` v1 — замість fork вийшли
послідовні G --> H --> I, а K без вихідного переходу.

**Промпт 2.** (аудит) "Перевір кожен вузол на вхід/вихід, guard-і на
дослівність із REQ, лічильник спроб."

## Топ-3 помилки AI

1. **Проглотив fork/join попри пряму вимогу в промпті** — написав
   послідовно (AUDIT #1). Фідбек: «паралельна ділянка = явно fork і join
   ноди, не просто дві стрілки з одного вузла».
   → Commit: `lab4: fix - add fork/join for activation and welcome email`
2. **Створив термінальний-подібний вузол без виходу** (блокування позики)
   — «діра» в графі (AUDIT #2). Це та ж хвороба, що Lab3 #2: AI малює
   щасливий шлях + один тупік; фідбек — «кінець кожної гілки: або
   повернення в цикл, або завершальний стан».
   → Commit: `lab4: fix - add return-book flow, unblock and terminal states`
3. **Guard написав «так/ні»** замість дослівних умов REQ-03/REQ-09 і
   згенерував необмежений цикл ретраїв (AUDIT #3).
   → Commit: `lab4: fix - label payment guards with REQ refs, cap retries at 3`