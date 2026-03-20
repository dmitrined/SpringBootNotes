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
* **Promises** — Состояния (`pending`, `fulfilled`, `rejected`).
* **`async/await`** — Синтаксический сахар над Promise.

#### Пример: Fetch данных с async/await
```javascript
async function fetchData(url) {
  try {
    const response = await fetch(url);
    if (!response.ok) throw new Error('Ошибка сети');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка:', error);
  }
}
```

### Паттерны и производительность
* **Производительность** — `debounce`, `throttle`.

#### Пример: Debounce (Защита от слишком частых вызовов)
```javascript
function debounce(func, delay) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), delay);
  };
}

// Применение: поиск при вводе
const handleSearch = debounce((query) => console.log('Searching:', query), 500);
```

---

[⬅️ Назад к HTML & CSS](../basics/README.md) | [Вперед к Advanced JS ➡️](Advanced_JS.md)