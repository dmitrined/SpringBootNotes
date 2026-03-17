# Java Core ☕

Этот раздел посвящен основам языка Java — фундаменту, на котором строится Spring Boot и любой современный корпоративный софт.

## Содержание

### Основы языка (Core Java)
* **Память: Stack vs Heap** — Как JVM управляет ресурсами.
* **Передача параметров** — Всегда по значению (Pass-by-value), а не по ссылке.
* **String Pool** — Особенности строк, `String`, `StringBuilder`, `StringBuffer`.
* **Equals & HashCode** — Контракт и корректная реализация.
* **Exceptions (Исключения)** — Hierarchy, Checked vs Unchecked, `try-with-resources`.

### Объектно-Ориентированное Программирование (OOP)
* **Инкапсуляция** — Скрытие реализации.
* **Наследование** — Повторное использование кода.
* **Полиморфизм** — Один интерфейс, множество реализаций.
* **Абстракция** — Абстрактные классы vs Интерфейсы (default/static методы).
* **Принципы SOLID**:
  1. **S**ingle Responsibility (Один класс — одна ответственность).
  2. **O**pen/Closed (Открыт для расширения, закрыт для изменения).
  3. **L**iskov Substitution (Принцип подстановки Барбары Лисков).
  4. **I**nterface Segregation (Разделение интерфейсов).
  5. **D**ependency Inversion (Инверсия зависимостей).

### Java Collections Framework
* **List** — `ArrayList`, `LinkedList` (Array vs Node-based).
* **Set** — `HashSet`, `LinkedHashSet`, `TreeSet` (Уникальность).
* **Map** — `HashMap` (Как работает внутри: Bucket, Collision, Treeify), `LinkedHashMap`, `TreeMap`.
* **Queue / Deque** — Очереди и двусторонние очереди.

### Generics (Обобщения)
* Type Erasure (Стирание типов).
* Wildcards: `? extends T` (Upper Bounded) vs `? super T` (Lower Bounded) — PECS rule.

### Многопоточность (Multithreading)
* **Thread & Runnable** — Создание потоков.
* **Синхронизация** — `synchronized`, `volatile`, `ReentrantLock`.
* **Concurrency Utils** — `Semaphore`, `CountDownLatch`, `CyclicBarrier`.
* **ThreadPools** — `ExecutorService`, `ScheduledExecutorService`.
* **Concurrent Collections** — `ConcurrentHashMap`, `CopyOnWriteArrayList`.
* **Virtual Threads (Java 21)** — Проект Loom и новая эра масштабируемости.

### Функциональное программирование (Java 8+)
* **Lambda Expressions** — Функциональные интерфейсы (`Consumer`, `Function`, `Predicate`, `Supplier`).
* **Stream API** — Промежуточные (`filter`, `map`, `flatmap`) и терминальные (`collect`, `foreach`, `findAny`) операции.
* **Optional** — Безопасная работа с `null`.

### Reflection & Annotations
* Как работают аннотации Spring (@Service, @Autowired) под капотом.
* Динамическое управление объектами.

---

[⬅️ Назад к Databases](../databases/README.md) | [Вперед к Security ➡️](../security/README.md)