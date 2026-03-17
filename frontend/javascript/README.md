# JavaScript 🟡

Язык веба. Понимание JS — ключ к любому современному фронтенду. Этот раздел охватывает современный JavaScript (ES6+) и его ключевые концепции.

## Содержание

### Основы языка (Core Concepts)
* **Типы данных** — Примитивы (`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`) и Объекты.
* **`var` vs `let` vs `const`** — Область видимости (Scope), поднятие (Hoisting), Temporal Dead Zone.
* **Преобразование типов (Type Coercion)** — Явное (`Number()`, `String()`) и неявное, `==` vs `===`.
* **Прототипное наследование** — Цепочка прототипов `[[Prototype]]`, `Object.create()`, `class` как синтаксический сахар.

### Функции и замыкания
* **Функции** — Function Declaration vs Expression vs Arrow Function (`=>`), их отличия в контексте `this`.
* **`this`** — Контекст выполнения в разных ситуациях, `bind`, `call`, `apply`.
* **Замыкания (Closures)** — Доступ к переменным из внешнего скоупа. Практические применения: фабрики, модули.
* **IIFE (Immediately Invoked Function Expression)** — Паттерн изоляции кода.
* **Currying (Каррирование)** — Преобразование функции с несколькими аргументами.

### Современный ES6+ синтаксис
* **Деструктуризация** — `const { name, age } = user;`, `const [first, ...rest] = arr;`
* **Spread / Rest операторы** — `...` для копирования объектов/массивов и агрегирования аргументов.
* **Optional Chaining `?.`** — Безопасный доступ к вложенным свойствам.
* **Nullish Coalescing `??`** — Значение по умолчанию только при `null`/`undefined`.
* **Template Literals** — Строки в обратных кавычках с `${expression}`.
* **Modules (ESM)** — `import` / `export`, статические и динамические (`import()`) импорты.

### Асинхронность (Async JavaScript)
* **Event Loop** — Как JS (однопоточный) выполняет асинхронный код: Call Stack, Web APIs, Callback Queue, Microtask Queue.
* **Callbacks** — Колбэки и проблема "Callback Hell".
* **Promises** — Состояния (`pending`, `fulfilled`, `rejected`), `.then()`, `.catch()`, `.finally()`. `Promise.all` vs `Promise.allSettled` vs `Promise.race`.
* **`async/await`** — Синтаксический сахар над Promise для читаемого асинхронного кода.

### Работа с DOM
* **Выборка элементов** — `querySelector`, `querySelectorAll`, `getElementById`.
* **Манипуляция** — `createElement`, `appendChild`, `innerHTML`, `textContent`, `classList`.
* **События (Events)** — `addEventListener`, Event Object, всплытие (`bubbling`) и перехват (`capturing`), делегирование.
* **Fetch API** — Выполнение HTTP-запросов напрямую из браузера.

### Паттерны и продвинутые техники
* **Иммутабельность** — Почему важно не мутировать данные напрямую (`map`, `filter`, `reduce`).
* **Паттерны** — Module, Observer/PubSub, Singleton.
* **Производительность** — `debounce`, `throttle`, Memoization.

---

[⬅️ Назад к HTML & CSS](../basics/README.md) | [Вперед к TypeScript ➡️](../typescript/README.md)