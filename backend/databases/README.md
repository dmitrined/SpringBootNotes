# Databases & ORM 💾

Этот раздел посвящён хранению, управлению и доступу к данным. Здесь разбираются основные подходы, технологии, а также продвинутые темы (индексы, транзакции, оптимизация).

## Содержание

### Реляционные базы данных (SQL)
* **PostgreSQL / MySQL** — Архитектура и основные отличия, типы данных, партицирование, репликация (Master-Slave / Master-Master).
* **Нормализация БД** — Зачем нужна, формы (1NF, 2NF, 3NF, BCNF) и денормализация в угоду производительности.
* **Индексы (Indexes)** — B-Tree, Hash, GIN / GiST, как работают, композитные индексы, сканирование индекса, "покрывающие" индексы.
* **Транзакции (Transactions) и ACID**
  * **Atomicity** — Атомарность (всё или ничего)
  * **Consistency** — Согласованность
  * **Isolation** — Изолированность
  * **Durability** — Надежность
* **Уровни изоляции транзакций**:
  1. Read Uncommitted (Грязные чтения / Dirty reads)
  2. Read Committed (Неповторяемые чтения / Non-repeatable reads)
  3. Repeatable Read (Фантомные чтения / Phantom reads)
  4. Serializable
* **Блокировки и Concurrency Control**:
  * Пул соединений (HikariCP).
  * Оптимистичные блокировки (`@Version`) vs Пессимистичные блокировки (`FOR UPDATE`).

### Нереляционные базы данных (NoSQL)
* **MongoDB (Document-oriented)** — Коллекции, документы, BSON, агрегации.
* **Redis (Key-Value) / Memcached** — In-memory кэширование, TTL, структуры данных (Strings, Hashes, Lists, Sets, Sorted Sets), Pub/Sub.
* **Cassandra (Column-family)** — Распределенная архитектура, Masterless, CAP-теорема.
* **Elasticsearch / Solr** — Полнотекстовый поиск.

### Object-Relational Mapping (ORM)
* **Hibernate / JPA Frameworks**
  * **Entity Lifecycle** — Состояния сущности: Transient, Persistent / Managed, Detached, Removed.
  * **N+1 Запросов** — Главная проблема ORM. Решения: `JOIN FETCH`, `@EntityGraph`, `@BatchSize`.
  * **Lazy vs Eager Loading** — Ленивая и жадная загрузка связанных сущностей.
  * **Кэширование Hibernate**
    * *First Level Cache* (Кеш сессии/EntityManager — включен по умолчанию).
    * *Second Level Cache* (Общий кэш фабрики сессий — Ehcache, Hazelcast, Redis).
    * *Query Cache* (Кэширование самих результатов запроса).

### Data Access Data (Spring Data)
* **Spring Data JPA** — Репозитории, автоматическая генерация запросов (`findByUsername`), `@Query` для кастомного JPQL / Native SQL, Specifications / Criteria API.
* **Spring Data Redis** и кэш.

### Миграции Баз Данных Schema Evolution
* Почему не стоит использовать `spring.jpa.hibernate.ddl-auto=update` в Production.
* **Flyway** — Миграции кодом SQL, таблица `flyway_schema_history`, версионирование (`V1__...`, `U2__...`).
* **Liquibase** — Миграции на основе XML / YAML / SQL, `databaseChangeLog`, Rollbacks.

---

[⬅️ Назад к Architecture](../architecture/README.md) | [Вперед к Java ➡️](../java/README.md)