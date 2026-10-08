# Audit — Components/Movement

## 1. Чи потрібна будь-якій RPG?
Так. Position, movement, path and visibility are common RPG runtime concepts.

## 2. Чи залежить від конкретної гри?
Не повністю. Концепції універсальні, але поточна реалізація прив'язана до Unity через Vector3/Quaternion і містить конкретну модель movement points.

## 3. Чи можна перевикористати без змін?
Ні. Потрібно відокремити engine representation від універсальних domain/runtime contracts.

## 4. Framework чи Game Layer?
Framework candidate з Technical Debt.

## 5. Чи є архітектурні проблеми?
- Пряма залежність усіх компонентів від UnityEngine.
- PositionComponent змішує position, previous position, rotation і scale.
- MovementComponent змішує фізичні параметри руху, direction, runtime state та movement-points economy.
- PathComponent поєднує path data та runtime traversal state.
- VisibilityComponent змішує perception configuration та current visibility state.
- Serialization methods у всіх компонентах є placeholder та фактично не реалізовані.
- MovementPoints/MaxMovementPoints можуть бути gameplay resource, а не universal movement concern.
- Quaternion/Vector3 роблять компоненти непридатними для non-Unity/server/headless reuse.

## 6. Від чого залежить?
Core.ECS / ISerializableComponent, UnityEngine, System.Collections.Generic.

## 7. Хто залежить від цього?
Рух, navigation/pathfinding, rendering/view, AI та gameplay systems.

## 8. Чи є зайві залежності?
UnityEngine dependency є архітектурним обмеженням для universal Framework. Serialization implementation також знаходиться не на своєму рівні.

## 9. Що потребує рефакторингу?
- Ввести engine-neutral position/rotation types або adapter layer.
- Розділити transform state, movement configuration і movement runtime state.
- Відокремити path definition від path traversal state.
- Винести visibility/perception rules із component data.
- Вирішити, чи movement points належать generic movement, stamina або gameplay resource.
- Перенести serialization у dedicated infrastructure.
- Реалізувати або прибрати placeholder serialization.

## Висновок:
☑ Framework
☐ Game Layer
☑ Technical Debt

Загальна класифікація: Framework candidate, але поточні компоненти потребують суттєвої декомпозиції та engine decoupling.