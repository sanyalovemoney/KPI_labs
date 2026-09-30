# Lab2 — Вимоги та варіанти використання

Домен той самий, що в [Lab1](../lab1-er/README.md) — онлайн-бібліотека з
підпискою. Тут модель даних перекладається на мову вимог: EARS, user stories,
Gherkin і діаграма варіантів використання.

## Файли

- [spec.md](spec.md) — актори, цілі, критерії прийняття
- [requirements.md](requirements.md) — EARS-вимоги + stories + Gherkin
- [traceability.md](traceability.md) — матриця трасування REQ ↔ UC
- [model/usecase.puml](model/usecase.puml) — Use Case Diagram (PlantUML)
- [AUDIT.md](AUDIT.md) — топ-3 розбіжності
- [DEFENSE.md](DEFENSE.md) — точка входу для рев'ю

## Чому PlantUML, а не Mermaid

Mermaid не має нативної use-case-діаграми. Умова це допускає («PlantUML /
Mermaid»), тому для цієї діаграми обрано PlantUML — див. ADR-001.