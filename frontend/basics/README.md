# HTML & CSS Basics 🧱

Фундамент фронтенд-разработки. Без твёрдых знаний HTML и CSS невозможно построить качественный интерфейс.

## Содержание

### HTML5
* **Семантическая разметка** — `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`. Почему это важно для SEO и доступности (a11y).
* **Формы** — `<input>` types (text, email, number, range, file), `<label>`, `<select>`, `<textarea>`, атрибуты валидации (`required`, `pattern`, `min/max`).
* **Мета-теги** — `<meta charset>`, `<meta viewport>`, `<meta description>`, Open Graph (`og:`).
* **Доступность (a11y)** — ARIA-атрибуты (`role`, `aria-label`, `aria-hidden`), семантика для скринридеров.

### CSS3
* **Flexbox** — Сетка в одном измерении.
  - `justify-content`: выравнивание по главной оси.
  - `align-items`: выравнивание по поперечной оси.
* **CSS Grid** — Двумерная сетка.

#### Примеры CSS верстки

**Центрирование элемента (Flexbox):**
```css
.container {
  display: flex;
  justify-content: center; /* по горизонтали */
  align-items: center;     /* по вертикали */
  height: 100vh;
}
```

**Адаптивная сетка (Grid):**
```css
.grid-container {
  display: grid;
  /* Колонки минимум 200px, заполняют всё пространство */
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
}
```

**CSS Переменные:**
```css
:root {
  --primary-color: #3498db;
}

.button {
  background-color: var(--primary-color);
}
```

### Препроцессоры и методологии
* **SCSS / SASS** — Переменные, вложенность, миксины (`@mixin`), наследование (`@extend`).
* **Методологии именования** — BEM (`block__element--modifier`).
* **CSS Modules** — Изоляция стилей в компонентах (React, Next.js).

### Адаптивный и отзывчивый дизайн
* **Единицы измерения** — `px`, `em`, `rem`, `%`, `vw/vh`, `clamp()`.
* **Responsive Images** — `srcset`, `<picture>`, `object-fit`.

---

[⬅️ Назад к Frontend](../README.md) | [Вперед к Web Fundamentals ➡️](Web_Fundamentals.md)