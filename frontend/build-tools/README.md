# Build Tools 🔧

Инструменты, которые превращают ваш исходный код в оптимизированный бандл, готовый к деплою в браузер.

## Содержание

### Пакетные менеджеры
* **npm** — Встроен в Node.js. `package.json`, `node_modules`, `package-lock.json`.
* **yarn** — Альтернатива с улучшенной скоростью и воспроизводимостью (Workspaces).
* **pnpm** — Самый быстрый, экономит место на диске за счёт жёстких ссылок.
* **Ключевые команды**: `install`, `add <pkg>`, `remove <pkg>`, `run <script>`, `update`.

### Сборщики (Bundlers)

#### Vite ⚡ (Современный стандарт)
* Использует **ES Modules** в development для мгновенного HMR (Hot Module Replacement).
* Для production сборки использует **Rollup** под капотом.
* **`vite.config.ts`** — Конфигурация: плагины (`@vitejs/plugin-react`), `resolve.alias`, `server.proxy`.
* Шаблоны: `npm create vite@latest`.

#### Webpack (Классика)
* Точка входа (`entry`), выходной бандл (`output`), трансформации (`loaders`), плагины (`plugins`).
* **Loaders** — Обрабатывают не-JS файлы: `babel-loader`, `css-loader`, `file-loader`.
* **Code Splitting** — Автоматическое разбиение на чанки для отложенной загрузки.

### Транспиляция и компиляция

* **Babel** — Транспиляция современного JS (ES2015+) в синтаксис, понятный старым браузерам.
* **SWC / esbuild** — Сверхбыстрые альтернативы Babel, написанные на Rust/Go. Используются внутри Vite и Next.js.
* **PostCSS** — Трансформации CSS (добавление вендорных префиксов через Autoprefixer, TailwindCSS).

### TypeScript Compilation
* **`tsc`** — Официальный компилятор. `tsconfig.json` управляет его поведением.
* **Режим `noEmit: true`** — Использовать TS только для проверки типов, сборку оставить Vite/Babel.

### Оптимизация для Production
* **Minification** — Удаление пробелов и сокращение имён переменных (Terser, esbuild).
* **Tree Shaking** — Удаление неиспользуемого кода из бандла (работает с ES Modules).
* **Code Splitting** — Разбивка на части (`chunks`) для параллельной загрузки.
* **Asset Hashing** — Добавление хэша к имени файла `app.a1b2c3.js` для инвалидации кэша браузера.
* **Compression** — Gzip / Brotli на уровне сервера (Nginx, CDN).

---

[⬅️ Назад к Framework](../framework/README.md) | [Вернуться к Frontend Index ➡️](../README.md)