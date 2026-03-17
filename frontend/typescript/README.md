# TypeScript 🔷

TypeScript — это строго типизированный надмножество JavaScript. Он помогает находить ошибки на этапе разработки и делает код самодокументируемым.

## Содержание

### Базовые типы
* **Примитивы** — `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.
* **Специальные типы** — `any` (избегать!), `unknown` (безопасный any), `never`, `void`.
* **Массивы и Кортежи** — `string[]` / `Array<string>`, Tuple: `[string, number]`.
* **Объединения и Пересечения** — Union `string | number`, Intersection `TypeA & TypeB`.
* **Enum** — `enum Direction { Up, Down, Left, Right }`, `const enum` (не генерирует JS-объект).

### Интерфейсы и Типы
* **`interface` vs `type`** — Ключевые сходства и отличия. Когда что использовать.
  * `interface` — Лучше для описания форм объектов и классов (поддерживает расширение `extends`).
  * `type` — Лучше для Union, Тuple, примитивных алиасов и сложных трансформаций.
* **Опциональные поля** — `name?: string` (поле может отсутствовать).
* **Readonly** — `readonly id: number` (защита от изменений).

### Generics (Обобщения)
* **Базовый синтаксис** — `function identity<T>(arg: T): T { return arg; }`
* **Ограничения** — `<T extends object>` — T должен быть объектом.
* **Использование в интерфейсах** — `interface ApiResponse<T> { data: T; status: number; }`
* **Утилитарные типы (Utility Types)**:
  - `Partial<T>` — Сделать все поля опциональными.
  - `Required<T>` — Сделать все поля обязательными.
  - `Readonly<T>` — Запретить изменение полей.
  - `Pick<T, K>` — Выбрать подмножество полей.
  - `Omit<T, K>` — Исключить поля.
  - `Record<K, V>` — Строит объект с ключами K и значениями V.
  - `ReturnType<T>` — Получить тип возвращаемого значения функции.

### Продвинутые типы
* **Type Guards** — `typeof x === 'string'`, `instanceof`, пользовательские гарды (`is`).
* **Discriminated Unions** — Объединение типов с общим полем-дискриминатором (`kind`).
* **Mapped Types** — `{ [K in keyof T]: boolean }` — создание новых типов на основе существующих.
* **Conditional Types** — `T extends U ? X : Y` — условные типы.
* **Template Literal Types** — `` `on${Capitalize<string>}` `` — строковые типы.

### TypeScript и React
* **Типизация компонентов** — `React.FC<Props>` vs `function Component(props: Props)`.
* **Типизация хуков** — `useState<User | null>(null)`, `useRef<HTMLDivElement>(null)`.
* **Типизация событий** — `React.ChangeEvent<HTMLInputElement>`, `React.MouseEvent<HTMLButtonElement>`.

### Конфигурация
* **`tsconfig.json`** — Ключевые флаги: `strict`, `target`, `module`, `paths`, `baseUrl`.

---

[⬅️ Назад к JavaScript](../javascript/README.md) | [Вперед к Framework ➡️](../framework/README.md)