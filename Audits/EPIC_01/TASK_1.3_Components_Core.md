# Audit — Components/Core

## 1. Чи потрібна будь-якій RPG?
Так. Базові компоненти ідентичності, назви, опису та значення корисні в багатьох RPG та можуть бути частиною Framework.

## 2. Чи залежить від конкретної гри?
Ні. За поточними файлами залежність лише від Core/ECS; game-specific gameplay semantics відсутні.

## 3. Чи можна перевикористати без змін?
Майже. Структури прості й універсальні, але окремі контракти варто уточнити.

## 4. Framework чи Game Layer?
Framework candidate.

## 5. Чи є архітектурні проблеми?
- Компоненти напряму маркуються ISerializableComponent, тому serialization policy вбудована в data layer.
- IdComponent з int не визначає, чи це Entity ID, persistent ID або domain ID.
- ValueComponent надто загальний і має слабку семантику.
- Mutable public fields прості для ECS, але контракт ownership не формалізований.
- Name/Description використовують string; потрібна загальна memory/serialization policy.

## 6. Від чого залежить?
Від MythHunter.Core.ECS.ISerializableComponent.

## 7. Хто залежить від цього?
Потрібно окремо визначити usages у системах та ECS queries. Самі компоненти не мають залежностей на Game Layer.

## 8. Чи є зайві залежності?
Явних зайвих залежностей у цих файлах не виявлено.

## 9. Що потребує рефакторингу?
- Уточнити універсальний контракт ID.
- Переглянути, чи serialization marker має залишатися на компонентах.
- Уточнити семантику ValueComponent.
- Визначити правила для string-heavy components.

## Висновок:
☑ Framework
☐ Game Layer
☑ Technical Debt

Загальна класифікація: Framework, з невеликим технічним боргом у контрактах і семантиці.
