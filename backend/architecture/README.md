# Architecture & Patterns 🏗️

В этом разделе собраны заметки по построению надежной, масштабируемой и поддерживаемой архитектуры на бэкенде.

## Содержание

### Паттерны проектирования
- [**Design Patterns (Шаблоны)**](./Design_Patterns.md) 🧩 — Порождающие, структурные и поведенческие паттерны.

### Микросервисная архитектура
- [**Microservices (Микросервисы)**](./Microservices.md) 🔗 — Как строить распределенные системы.

### Архитектурные стили и API
1. **RESTful API** — Принципы построения (Stateless, Cacheable, Uniform Interface), идемпотентность методов (GET, PUT, DELETE), версионирование.
2. **GraphQL** — Особенности, `Query` (получение данных) и `Mutation` (изменение данных), решение проблемы Over-fetching и Under-fetching.
3. **gRPC** — Бинарный протокол от Google поверх HTTP/2, Protobuf, сценарии применения (межсервисное взаимодействие).

### Паттерны проектирования (Design Patterns)
* **Creational (Порождающие)**: Singleton, Builder, Factory Method, Abstract Factory.
* **Structural (Структурные)**: Adapter, Facade, Proxy, Decorator.
* **Behavioral (Поведенческие)**: Strategy, Observer, Command, Chain of Responsibility.

### Микросервисная архитектура (Microservices)
* **API Gateway** — Единая точка входа для клиентов (маршрутизация, аутентификация, rate limiting).
* **Service Discovery** — Обнаружение сервисов (Consul, Eureka).
* **Circuit Breaker** — Предотвращение каскадных сбоев (Resilience4j).
* **SAGA Pattern** — Распределенные транзакции (Choreography vs Orchestration).
* **CQRS (Command Query Responsibility Segregation)** — Разделение моделей чтения и записи.
* **Event Sourcing** — Хранение состояний приложения в виде последовательности событий.

### Брокеры сообщений (Message Brokers)
* **Apache Kafka** — Event streaming, Topics, Partitions, Consumer Groups, гарантии доставки (At-most-once, At-least-once, Exactly-once).
* **RabbitMQ** — Exchanges (Direct, Topic, Fanout), Queues, Bindings.
* Отличия и сценарии применения (Message Queue vs Event Stream).

### Монолит vs Микросервисы
* Преимущества и недостатки каждого подхода.
* Когда стоит начинать с монолита и как правильно выделять микросервисы.
* Модульный монолит (Modular Monolith) как золотая середина.

---

[⬅️ Назад к Backend](../README.md) | [Вперед к Databases ➡️](../databases/README.md)