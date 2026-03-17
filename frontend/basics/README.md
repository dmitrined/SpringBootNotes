# HTML & CSS Basics 🧱

Фундамент фронтенд-разработки. Без твёрдых знаний HTML и CSS невозможно построить качественный интерфейс.

## Содержание

### HTML5
* **Семантическая разметка** — `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`. Почему это важно для SEO и доступности (a11y).
* **Формы** — `<input>` types (text, email, number, range, file), `<label>`, `<select>`, `<textarea>`, атрибуты валидации (`required`, `pattern`, `min/max`).
* **Мета-теги** — `<meta charset>`, `<meta viewport>`, `<meta description>`, Open Graph (`og:`).
* **Доступность (a11y)** — ARIA-атрибуты (`role`, `aria-label`, `aria-hidden`), семантика для скринридеров.

### CSS3
* **Box Model** — `margin`, `padding`, `border`, `box-sizing: border-box`.
* **Cascading & Specificity** — Как браузер решает, какой стиль применить (специфичность: `id > class > tag`).
* **Flexbox** — `display: flex`, оси (`flex-direction`), `justify-content`, `align-items`, `flex-wrap`, `gap`.
* **CSS Grid** — `display: grid`, `grid-template-columns/rows`, `grid-area`, `auto-fill/auto-fit`, `fr` единица.
* **Позиционирование** — `static`, `relative`, `absolute`, `fixed`, `sticky`.
* **CSS-переменные (Custom Properties)** — `--color-primary: #...`, `var(--color-primary)`.
* **Анимации** — `transition` (плавные переходы), `@keyframes` + `animation` (кастомные анимации).
* **Медиазапросы (Media Queries)** — `@media (max-width: 768px)`, Mobile-First vs Desktop-First подход.

### Препроцессоры и методологии
* **SCSS / SASS** — Переменные, вложенность, миксины (`@mixin`), наследование (`@extend`).
* **Методологии именования** — BEM (`block__element--modifier`).
* **CSS Modules** — Изоляция стилей в компонентах (React, Next.js).

### Адаптивный и отзывчивый дизайн
* **Единицы измерения** — `px`, `em`, `rem`, `%`, `vw/vh`, `clamp()`.
* **Responsive Images** — `srcset`, `<picture>`, `object-fit`.

---

[⬅️ Назад к Frontend](../README.md) | [Вперед к JavaScript ➡️](../javascript/README.md)