# Audit — Networking

Source project: Aizekhan/MythHunter
Branch: dev
Path: Assets/_MythHunter/Code/Networking

## Name

Networking

## 1. Is it needed by any RPG?

Not every RPG needs networking, but a universal Framework intended to support online/co-op/multiplayer RPGs should provide networking abstractions. Networking is therefore Framework scope, while specific transports can be optional.

## 2. Does it depend on a specific game?

Conceptually mostly no, but the current implementation is coupled to MythHunter infrastructure and Events.

Examples include MythHunter DI/logging, NetworkEventBus inheritance from EventBus, and NetworkEventMessage coupling networking to event serialization.

## 3. Can it be reused without changes?

No, not as production-ready universal networking.

Interfaces and layering concepts are reusable. Current client/server implementations are largely placeholders that simulate connections and sending instead of performing real transport.

## 4. Framework or Game Layer?

Framework candidate / infrastructure.

Transport contracts, protocol messages, serialization abstractions and security interfaces belong in Framework infrastructure. Concrete game messages/events should remain outside the generic networking core.

## 5. Are there architectural problems?

- NetworkSystem overlaps with ClientNetworkSystem and ServerNetworkSystem.
- Client/Server system wrappers overlap with NetworkClient/NetworkServer.
- NetworkClient and NetworkServer are transport stubs.
- Serialization uses assembly scanning and runtime type names.
- NetworkEventMessage relies on reflection/runtime type creation.
- NetworkEventBus couples networking to EventBus inheritance.
- Security generates a process-local HMAC key, unsuitable for real client/server sessions.
- HMAC comparison is not constant-time.
- No visible real transport/protocol/retry/timeout/authority/replication layer.
- Message serialization and entity/component replication are not clearly separated.
- Higher-level NetworkSystem drops the source clientId when forwarding server messages.

## 6. What does it depend on?

- Cysharp UniTask
- MythHunter.Core.DI
- MythHunter.Utils.Logging
- MythHunter.Networking.Messages
- MythHunter.Networking.Serialization
- MythHunter.Events / Events.Network
- MythHunter.Data.Serialization
- System.Security.Cryptography

## 7. Who depends on it?

Likely consumers include networking event integration, gameplay replication/commands, client/server bootstrap, multiplayer systems and future persistence/serialization integration.

## 8. Are there unnecessary dependencies?

Likely yes:
- Networking -> EventBus inheritance
- networking messages -> concrete MythHunter event infrastructure
- overlapping Client/Server/Core orchestration
- runtime assembly scanning for message registration

## 9. What needs refactoring?

No implementation changes in this audit.

Candidates:
- Separate transport, protocol/messages, serialization, replication and security.
- Remove NetworkEventBus inheritance coupling; use composition/adapters.
- Replace runtime type-name scanning with explicit stable message registration/IDs.
- Clarify the primary client/server orchestration boundary.
- Define real authority, commands and replication in later networking epics.
- Replace placeholder transports with explicit adapter implementations.
- Redesign security around proper session/key management and constant-time authentication checks.

## Conclusion

- [x] Framework
- [ ] Game Layer
- [x] Technical Debt

Classification: Networking is fundamentally Framework infrastructure, but the current implementation is incomplete and has architectural overlap and coupling that must be resolved before it is reusable production infrastructure.
