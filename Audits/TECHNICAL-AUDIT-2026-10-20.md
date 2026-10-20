# Повний технічний аудит RPG Framework / MythHunter
**Дата:** 2026-10-20  
**Перевірений source HEAD:** `22d3a98219ec53a604862e86a6e09b8e1e1b0bd9` (гілка `dev`)  
**Roadmap:** `Aizekhan/RPG-Framework-Roadmap`, гілка `main`  
**Тип аудиту:** віддалений аудит репозиторію, документації, конфігурації CI та доступних GitHub workflow-результатів. Unity Editor локально не запускався.

## 1. Executive summary

Проєкт має значний обсяг повторно використовуваної RPG-логіки та сформовані архітектурні правила. Водночас доказова база для заяв «готово», «усі тести зелені» та «готово для використання в довільному проєкті Unity» нерівномірна.

**Загальний статус: AMBER / потрібна стабілізація та валідація перед нарощуванням нових систем.**

Найважливіші висновки:
1. Загальний workflow `.github/workflows/ci.yml` є no-op: виконує лише `echo`, не збирає і не тестує проєкт.
2. Є спеціалізовані модульні workflow-и; останній перевірений Progression workflow на SHA `22d3a98` завершився успішно, включно з Build Runtime, Build Tests і Run tests. Це підтверджує саме Progression pipeline, а не весь Unity-проєкт.
3. `CLAUDE.md` прямо фіксує, що CI не компілює `Assets/_MythHunter`; зелений CI не доводить справність Game Layer.
4. Звіт від 19 жовтня фіксує 120 активних framework tests, але окремо позначає виконання 120 тестів та 100% pass rate як pending. Твердження про 512/512 у handoff від 20 жовтня не підтверджено доступними доказами на точному SHA.
5. Стара `CrossSystemIntegrationTests.cs` видалена через 91 помилку компіляції після змін API. Це прибрало блокування компіляції, але залишило інтеграційне покриття як окремий ризик. Дерево гілки все ще містить integration test assembly definition, але в каталозі видно лише `.asmdef` та `.meta`, не самі інтеграційні тести.
6. Документація Roadmap відстає від source branch: `STATUS.md` і `HANDOFF_NEXT_CHAT.md` досі вказують HEAD `6d6fa9b` та EPIC 16–19, тоді як source вже на `22d3a98` і handoff описує EPIC 20.
7. У `ARCHITECTURE.md` уже задокументовано архітектурний борг: EntityManager і ComponentCache частково дублюють сховище/кешування; CombatSystem має надто багато відповідальностей.

## 2. Scope і джерела

Переглянуто:
- Source repository: https://github.com/Aizekhan/MythHunter/tree/dev
- Roadmap repository: https://github.com/Aizekhan/RPG-Framework-Roadmap
- `CLAUDE.md`, `ARCHITECTURE.md`, `ARCHITECTURE-QUICK-REFERENCE.md`
- `.github/workflows/ci.yml`, `.github/workflows/architecture-validation.yml`
- останній доступний workflow Progression для SHA `22d3a982...`
- `FINAL-SUCCESS-REPORT.md`, Roadmap `STATUS.md`, `HANDOFF_NEXT_CHAT.md`
- дерево `Assets/_Framework` і `Assets/_MythHunter/Tests/Integration`
- відкриті PR у source та Roadmap.

**Не виконано в цьому середовищі:** локальне відкриття Unity, повна компіляція Editor, запуск усіх EditMode tests, Player Build, PlayMode/smoke tests, профілювання runtime, перевірка локального незакоміченого стану. Будь-які висновки про них позначені як непідтверджені.

## 3. Критичні та високі знахідки

### AUD-01 — Критичний: загальний CI не забезпечує перевірку коду
**Доказ:** `.github/workflows/ci.yml` має job `noop`, який виконує лише `echo "No CI actions needed"`.  
**Ризик:** звичайний push/PR може бути позначений успішним без компіляції чи тестування. Це створює хибну впевненість у якості.  
**Рекомендація:** зробити обов'язковими окремі jobs для Framework .NET build/tests, architecture tests, Unity EditMode tests і, де доступно, Game Layer compilation. Якщо Unity licensing/runner недоступний, статус має бути явно `not run/blocked`, не green.  
**Критерій закриття:** тестовий workflow падає на навмисно зламаному compile/test sample та проходить для валідного SHA.

### AUD-02 — Критичний: недостатній наскрізний доказ справності Game Layer
**Доказ:** `CLAUDE.md` каже, що CI не компілює `Assets/_MythHunter`; повна валідація передбачає Unity EditMode, чисту повну компіляцію, Player Build і smoke tests.  
**Ризик:** модулі можуть компілюватися ізольовано, а ігрові adapters, installers, сцени або інтеграція — ні.  
**Рекомендація:** запустити повну Unity-перевірку на точному SHA `22d3a982...` у Unity 6000.0.45f1; зберегти XML test report, Editor.log, build output і smoke-test checklist.  
**Критерій закриття:** зафіксовані SHA + фактичні результати кожного gate; не використовувати звіт для іншого коміту.

### AUD-03 — Високий: інтеграційні тести були видалені через несумісний API
**Доказ:** commit `7a5c30e` видалив `Assets/_MythHunter/Tests/Integration/CrossSystemIntegrationTests.cs` після 91 помилки через несумісні підписи Quest/Crafting/Trading. Раніше commit `a39de5c` описує 11 integration tests у 570 рядках, але кінцевий стан гілки вже не містить цього test source; integration asmdef лишився.  
**Ризик:** збірка могла очиститися від застарілого тесту, але відновлення наскрізного покриття відкладене; залишений asmdef може бути порожньою тестовою збіркою або приховувати незавершену інфраструктуру.  
**Рекомендація:** перевірити фактичний вміст integration каталогу в робочій копії, видалити asmdef лише якщо підтверджено, що він зайвий, або додати новий набір тестів з актуальними API. Не повертати видалений файл без повного переписування під чинні контракти.  
**Критерій закриття:** є тести Quest → Dialogue, Crafting → Inventory → Trading, Item → Loot → Inventory, Faction → Dialogue/Trading (де залежність справді реалізована); усі запускаються в Unity на тому самому SHA.

### AUD-04 — Високий: Roadmap-документи відстають від source
**Доказ:** Roadmap `STATUS.md` досі каже Last reconciled 2026-10-18 та source HEAD `6d6fa9b`; `HANDOFF_NEXT_CHAT.md` описує EPIC 16–19. Source head — `22d3a982...`; є окремий звіт про виправлення компіляції й handoff від 20 жовтня повідомляє про EPIC 20 та 512 tests.  
**Ризик:** наступний агент виконає роботу від неправильного SHA, повторить завершені задачі або пропустить відкриті quality gates.  
**Рекомендація:** відкрити документаційний PR у Roadmap: актуальний HEAD, EPIC 20 evidence, точні відомі тести, unresolved gates, один ACTIVE item. Розвести «код написано», «зібрано», «тести виконані», «тести пройшли», «інтеграцію підтверджено».  
**Критерій закриття:** Roadmap точно посилається на tested SHA й не називає pending gates завершеними.

### AUD-05 — Високий: тестові цифри суперечать одна одній
**Доказ:** `FINAL-SUCCESS-REPORT.md` описує 120 активних framework tests, але в checklist позначає «120 tests executed» та «100% pass rate» як pending. Handoff повідомляє 512/512. На GitHub видно успішний окремий Progression workflow для SHA `22d3a98`, не доказ 512 Unity tests для всього проєкту.  
**Ризик:** метрики можуть означати різні набори тестів (Framework .NET tests, Unity EditMode, architecture, integration) і вводити в оману.  
**Рекомендація:** опублікувати один машинно згенерований test summary з колонками: suite, target, commit SHA, discovered, executed, passed, failed, skipped, runner, report path. Не агрегувати різні набори в одне число без визначення.  
**Критерій закриття:** цифра 512 має XML/CI evidence із точним SHA або замінена на справжню, розкладену за наборами.

### AUD-06 — Високий: відкриті PR і source/roadmap можуть вводити в оману
**Доказ:** відкритий MythHunter PR #4 `UnifiedMythHunterWizard` має head `dev`, його metadata вказує на merged commit, проте PR усе ще open; відкритий MythHunter PR #5 `Feature/phases` датований 2025 роком. Roadmap PR #22 для EPIC 11 Phase 5 лишається відкритим, хоча source уже пішов далі.  
**Ризик:** PR-и застарілі, незрозумілий статус джерела істини, можливі повторні або невірні злиття.  
**Рекомендація:** вручну звірити цілі PR, base/head refs, diff і merge history. Закривати застарілі PR лише після підтвердження, що вони не містять незлитих потрібних змін.  
**Критерій закриття:** кожен відкритий PR має власника, реальну мету й актуальну base/head; зайві закриті з коротким поясненням.

## 4. Архітектура

### Позитивні сторони
- Є явна межа `Assets/_Framework` і `Assets/_MythHunter`.
- `RPGFramework.ECS.Runtime.asmdef` має `noEngineReferences: true` і не має явних assembly references — сильний напрям для engine-independent ядра.
- `CLAUDE.md` задає напрям залежностей, Git-guardrails і вимоги до доказів.
- Є окремі assembly definitions для нових систем і спеціалізовані workflow-и для частини модулів.
- `ARCHITECTURE.md` формулює важливі правила: компоненти — дані; системи — поведінка; події — факти; реплікується authoritative state; persistence повинна мати версіонування/міграції.

### AUD-07 — Середній/високий: потенційне дублювання зберігання стану
**Доказ:** `ARCHITECTURE.md` прямо описує `EntityManager` як `EntityId -> Dictionary<Type,IComponent>` і `Type -> HashSet<EntityId>`, та зазначає, що `ComponentCache` дублює частину storage/cache behavior.  
**Ризик:** подвійне джерело істини, додаткові dirty/update правила, зайва складність та майбутні помилки узгодження.  
**Рекомендація:** не робити великий перепис зараз. Спершу визначити відповідальність EntityManager і ComponentCache, контракт актуальності cache, правила інвалідації та вимірювані performance cases. Додати characterization tests перш ніж змінювати реалізацію.

### AUD-08 — Середній/високий: CombatSystem має забагато відповідальностей
**Доказ:** `ARCHITECTURE.md` вказує, що CombatSystem веде active combat, lifecycle, ресурси/rage/concentration, phase handling, async combat loop та events.  
**Ризик:** складне тестування, багато причин для змін, важкий повторний reuse в іншій RPG.  
**Рекомендація:** після відновлення інтеграційних тестів спроєктувати межі окремо; не дробити без тестів і вимірюваної потреби. Потенційні кандидати: CombatLifecycle/Encounter, Resource/Cost, Turn/Phase orchestration та event publishing.

### AUD-09 — Середній: архітектурні правила частково перевіряються простими grep
**Доказ:** `architecture-validation.yml` має grep для assembly references; path glob `Assets/_Framework/**/*.asmdef` залежить від поведінки shell globbing. Архітектурні тести окремо запускуються в Unity.  
**Ризик:** прості grep можуть пропустити непрямі залежності або бути чутливими до структури каталогів; повідомлення workflow може звучати ширше за фактичну перевірку.  
**Рекомендація:** використовувати рекурсивний `find`/скрипт JSON для всіх .asmdef та перевіряти reference graph; компілювати Framework assemblies окремо; залишити Unity architecture tests як додатковий шар.

### AUD-10 — Середній: pure .NET та Unity-specific code потрібно валідовувати двічі
Заявлена архітектура — pure .NET 8.0 для Framework modules і Unity adapters в game layer. Це корисно, однак різні середовища можуть приховувати несумісності: .NET test runner не дорівнює Unity compiler/runtime.  
**Рекомендація:** підтримувати дві явні валідації — .NET module tests і Unity EditMode/full compile. Підписувати результати на конкретний SHA.

## 5. CI/CD та якість тестування

### Поточна картина
- Загальний `ci.yml`: no-op.
- Є `architecture-validation.yml` з Unity EditMode Architecture category і assembly definition checks.
- Є окремі module workflows, включно з Progression. Перевірений run для HEAD `22d3a98` пройшов Validate module boundaries, Restore dependencies, Build Runtime, Build Tests і Run tests.
- У `CLAUDE.md` задокументовано правильну вимогу: Unity EditMode → full compilation → Player Build → smoke tests. Але доступні поточні докази не закривають усі ці кроки для HEAD `22d3a98`.

### Рекомендовані quality gates
1. Framework build + unit tests (.NET 8).
2. Assembly/dependency boundary validation.
3. Unity EditMode tests, включно з architecture та gameplay integration.
4. Повна Unity-компіляція з чистою Console.
5. Player Build.
6. Smoke test найважливішого gameplay loop.
7. Збереження тестових артефактів і прив'язка звіту до SHA.

## 6. Документація та готовність до AI-агентів

### Позитивне
`CLAUDE.md` уже задає стабільні робочі правила, включно з тим, що зміни лише через branches/PR, не зливати без підтвердження, перевіряти точний SHA та не видаляти файли без аналізу.

### AUD-11 — Високий для цілі «AI збирає RPG»: інструкція є, але єдиного перевіреного сценарію AI-bootstrap не показано
**Ризик:** агент може прочитати правила й API, але все ще не знати, як із чистого Unity-проєкту ініціалізувати framework, підключити модулі, створити контент та запустити весь цикл тестування. Наявність `CLAUDE.md` не доводить автоматичне складання гри.  
**Рекомендація:** створити окремий чистий demo/test project або reproducible sample, з AI quickstart, каталогом модулів (підтримувані функції, залежності, приклади), командами валідації та acceptance test. Перевірити його Claude Code від чистого checkout.  
**Критерій закриття:** агент у чистому проєкті створює working slice (entity/character, combat interaction, loot/inventory та quest reward) і проходить визначені автоматичні checks без ручних інструкцій автора, окрім задокументованого prompt.

## 7. Продуктові межі: що ще не можна заявляти

Не можна вважати доказаними лише за наявністю runtime modules:
- готовий 3D UI для всіх систем;
- повний playable gameplay loop;
- production-ready networking/multiplayer, хоча в репозиторії є networking layers;
- повноцінне save/load із version migration на кожному gameplay module;
- автоматичне створення повної RPG одним запитом AI;
- сумісність усіх модулів у чистому Unity-проєкті;
- 512/512 тести без прикріпленого звіту з точним SHA.

## 8. Пріоритетний план виправлень

### P0 — перед новими gameplay EPIC
1. Запустити й зберегти Unity повну компіляцію та EditMode на exact HEAD `22d3a982...`.
2. Встановити точний фактичний стан 120 tests / 512 tests та розвести набори в звіті.
3. Відновити інтеграційне тестування під актуальні API.
4. Оновити Roadmap через PR: STATUS, HANDOFF, active item, tested SHA і відкриті gates.
5. Розібрати відкриті/застарілі PR #4, #5, Roadmap #22.

### P1 — зробити якість відтворюваною
6. Замінити no-op CI на справжні gates або чітко позначити його як status-only.
7. Автоматизувати перевірку всіх asmdef dependencies.
8. Додати згенерований тестовий summary з artifacts.
9. Підтвердити integration assembly: або містить актуальні тестові файли, або її видалити/переробити.

### P2 — стабілізація архітектури та AI-цілі
10. Визначити межі EntityManager/ComponentCache та characterization tests.
11. Спроєктувати декомпозицію CombatSystem після тестів.
12. Побудувати AI-bootstrap sample та провести reproducibility test у чистому Unity-проєкті.

## 9. Висновок

Фреймворк має міцну основу й суттєвий набір систем, але зараз головний ризик — не відсутність чергового модуля, а розрив між задокументованою готовністю та доказовою валідацією всього продукту. Спочатку треба підтвердити compile/test/integration на одному SHA, повернути міжсистемне покриття та синхронізувати документацію. Лише після цього є сенс оцінювати, наскільки Framework готовий до передачі AI-агентам та повторного використання в іншій RPG.

**Статус аудиту:** віддалений аудит завершено в межах доступних GitHub-даних; Unity runtime/build verification залишається відкритим gate.
