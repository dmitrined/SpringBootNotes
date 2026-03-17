# Security 🛡️

Раздел о безопасности приложений: аутентификация, авторизация и защита от распространенных атак.

## Содержание

### Базовые концепции
1. **[Security Overview](./Security.md)** — Подробный разбор Spring Security (настройки, фильтры, компоненты).
2. **Аутентификация (Authentication)** — Проверка личности (Basic, Session-based, OAuth2).
3. **Авторизация (Authorization)** — Управление доступом (RBAC — Role-Based, ABAC — Attribute-Based).

### Стандарты и протоколы
* **JWT (JSON Web Token)** — Структура токена (Header, Payload, Signature), хранение (Header vs Cookie), Refresh Tokens.
* **OAuth2 / OpenID Connect (OIDC)** — Вход через Google/GitHub, Flow (Authorization Code, Client Credentials).

### Безопасность API
* **CORS (Cross-Origin Resource Sharing)** — Как настроить доступ для фронтенда.
* **CSRF (Cross-Site Request Forgery)** — Защита форм и API.
* **XSS (Cross-Site Scripting)** — Валидация и санитизация данных.
* **SQL Injection** — Почему PreparedStatement — это важно.
* **Rate Limiting** — Защита от Brute-force и DDoS атак.

### Криптография
* **Hashing** — BCrypt, SCrypt, PBKDF2 (почему нельзя использовать MD5/SHA-256 для паролей).
* **Encryption** — Симметричное (AES) и асимметричное (RSA) шифрование.

---

[⬅️ Назад к Java Core](../java/README.md) | [Вперед к Spring Boot ➡️](../spring-boot/README.md)
