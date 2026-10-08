Назва: Events / Event Types

1. Чи потрібна будь-якій RPG?
Так. Типізовані події є базовим способом повідомлення про зміни стану та взаємодії між модулями без жорсткого прямого зв’язку.

2. Чи залежить від конкретної гри?
Інтерфейс IEvent та механізм generic-публікації є загальними. Конкретні Domain-події та події Lobby/Game — Game Layer.

3. Чи можна перевикористати без змін?
Generic event contract — так. Конкретні події — ні, вони описують конкретну гру/домен.

4. Framework чи Game Layer?
Змішано: IEvent/EventPriority — Framework; конкретні Domain/Game/Lobby events — Game Layer.

5. Чи є архітектурні проблеми?
- IEvent є мінімальним контрактом, але EventBus очікує додаткові extension-методи GetEventId/GetPriority.
- Це фактично прихований обов’язковий контракт, який не видно з IEvent.
- Події реалізовані як struct, що добре для allocation-free value events, але створює обмеження для polymorphism, inheritance та queued reference semantics.
- EventPriority знаходиться разом із generic events, хоча priority є політикою dispatch і не обов’язково властивістю самої події.
- Частина подій містить явні game/domain concepts, які не повинні потрапляти у Framework.
- Події та handlers можуть легко стати implicit API між багатьма модулями, тому потрібні стабільні правила ownership/dependency.

6. Від чого залежить?
Generic contracts залежать переважно від System/type infrastructure. Конкретні event structs залежать від відповідних Game Layer components/services/Unity types.

7. Хто залежить від цього?
EventBus, systems, services, UI, networking та інші publishers/subscribers.

8. Чи є зайві залежності?
Так: частина concrete events тягне Unity/game-specific dependencies; priority та event metadata змішані з event contract.

9. Що потребує рефакторингу?
- Залишити Framework event contract мінімальним.
- Визначити explicit event metadata/policy замість прихованої залежності від extension-методів.
- Відокремити Framework events від Game Layer events структурно.
- Зафіксувати правило ownership: event описує occurrence, а не handler/system behavior.
- Переглянути необхідність EventPriority у самому event contract.

Висновок:
☑ Framework
☑ Game Layer
☑ Technical Debt