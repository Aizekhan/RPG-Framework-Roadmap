# Audit — Entities/Templates

## 1. Чи потрібна будь-якій RPG?
Окремої папки/структури Templates у Entities не існує. Template functionality фактично реалізована всередині Entities/Archetypes через ArchetypeTemplateRegistry, ArchetypeTemplateBuilder та відповідні interfaces.

## 2. Чи залежить від конкретної гри?
Частина template mechanism універсальна, але поточна реалізація містить MythHunter-specific default components та Item template.

## 3. Чи можна перевикористати без змін?
Ні. Template abstraction потребує відокремлення generic template infrastructure від конкретних game definitions.

## 4. Framework чи Game Layer?
Mixed.

## 5. Чи є архітектурні проблеми?
- Немає окремої canonical Templates структури: вона захована в Archetypes.
- Це ускладнює розмежування archetype, template і semantic classification.
- Template registry hardcodes MythHunter components and default Item.
- Template overrides use Dictionary<Type, object>.
- Builder mutates registry directly, mixing definition construction and registry ownership.
- Component delegate cache has manual component bootstrap plus reflection fallback.
- Template matching and entity creation are combined in one registry.
- Concrete builder type leaks through IArchetypeTemplateRegistry.

## 6. Від чого залежить?
Core.ECS, concrete components, logging, DI, reflection.

## 7. Хто залежить від цього?
ArchetypeSystem, EntityFactory and entity creation flow.

## 8. Чи є зайві залежності?
Так: concrete MythHunter component knowledge and reflection fallback are implementation concerns that should not define a generic template API.

## 9. Що потребує рефакторингу?
- Виділити окремий generic EntityTemplate module.
- Визначити Template як immutable definition/configuration.
- Відокремити template definition, template matching та entity instantiation.
- Перенести Item/Hero/Enemy templates у Game Layer.
- Замінити weakly typed component dictionaries більш чітким component-definition API.
- Після визначення canonical Archetype model вирішити, чи Template взагалі має належати до Entities/Archetypes, чи бути окремим module.

## Висновок:
☑ Framework
☑ Game Layer
☑ Technical Debt

Категорія Templates не має окремої реалізації; її поточна логіка є частиною ArchetypeTemplateRegistry і потребує виділення як самостійного поняття.
