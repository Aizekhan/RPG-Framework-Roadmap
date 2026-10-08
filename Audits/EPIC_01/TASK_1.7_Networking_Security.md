# Audit — Networking/Security

## 1. Чи потрібна будь-якій RPG?
Для multiplayer RPG — так. Для single-player — optional module.

## 2. Чи залежить від конкретної гри?
Концепція reusable. Поточна реалізація має лише logger dependency, але security model надто спрощена для production networking.

## 3. Чи можна перевикористати без змін?
Ні.

## 4. Framework чи Game Layer?
Framework candidate + Critical Technical Debt.

## 5. Чи є архітектурні проблеми?
- NetworkSecurityProvider використовує HMAC-SHA256 з локально згенерованим ключем.
- Ключ створюється при construction і ніде не передається peer/server/client, тому цей механізм не встановлює реального shared secret protocol.
- Немає key exchange, provisioning, rotation або secure key storage.
- HMAC забезпечує integrity/authentication за наявності секретного ключа, але не confidentiality; пакет не шифрується.
- Немає nonce/sequence number/timestamp, тому відсутній replay protection.
- VerifyAndExtractPacket порівнює MAC вручну байт-за-байтом замість constant-time comparison API.
- Немає maximum packet size / allocation guard до виділення memory.
- Security provider працює тільки на byte[] пакетах і не має protocol-level identity/session binding.
- Немає rate limiting, abuse protection, disconnect policy або authentication/authorization layer.
- Logger є прямою dependency security primitive.
- SecurePacket не описує wire format/version, лише HMAC prefix + plaintext.
- Відсутній чіткий security boundary між transport security, message authenticity та gameplay authorization.

## 6. Від чого залежить?
System.Security.Cryptography та IMythLogger.

## 7. Хто залежить від цього?
Майбутній transport/network pipeline, client/server protocol.

## 8. Чи є зайві залежності?
Так:
- logger можна винести через diagnostics abstraction.
- Security provider не повинен одноосібно вирішувати authentication/authorization.

## 9. Що потребує рефакторингу?
- Визначити threat model.
- Відокремити cryptographic primitives від session/authentication/security policy.
- Впровадити реальне key provisioning/exchange та rotation.
- Додати nonce/sequence/replay protection.
- Використовувати constant-time MAC verification.
- Додати packet size limits та malformed input handling.
- Визначити encryption requirements окремо від integrity.
- Додати authentication/authorization/session identity.
- Додати abuse/rate limiting policy.
- Зробити security optional/configurable module для single-player.

## Висновок:
- [x] Framework
- [ ] Game Layer
- [x] Technical Debt
