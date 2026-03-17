# Thymeleaf (Шаблонизатор)

**Thymeleaf** — это шаблонизатор (template engine) для Java, который чаще всего используется в Spring MVC для генерации HTML-страниц прямо на сервере.

Если в случае с REST API (и React/Vue на клиенте) сервер просто отдает JSON, то в случае с Thymeleaf сервер **самостоятельно строит HTML** и отдает клиенту (браузеру) уже готовую страницу. 

Идея взаимодействия:
1. Браузер запрашивает страницу `/todos`.
2. Spring `@Controller` получает данные (например, из БД), кладет их в объект `Model` и возвращает **имя шаблона** (например, `todos`).
3. Thymeleaf берёт HTML файл `resources/templates/todos.html`, подставляет туда данные из `Model`.
4. Сервер возвращает готовый, красивый HTML клиенту.

---

### SSR (Thymeleaf) vs SPA (React/Vue/Angular)

| Свойство | SSR (Server-Side Rendering / Thymeleaf) | SPA (Single Page Application / React, Vue) |
| :--- | :--- | :--- |
| **Кто строит UI?** | Сервер (Java) генерирует весь HTML. | Браузер (JS) скачивает пустую страницу и сам строит UI, получая данные по JSON. |
| **Навигация** | При каждом клике по ссылке/кнопке страница перезагружается целиком. | Страница не перезагружается, контент меняется динамически. |
| **Сложность** | Низкая. Всё на Java, не нужно настраивать Node.js, Webpack, NPM. | Высокая. Нужно держать два отдельных проекта (Backend + Frontend). |
| **Где лучше применять?** | Админки, внутренние панели (B2B), прототипы, учебные проекты. | Сложные интерактивные интерфейсы, публичные порталы с богатым UI. |

---

### Основные возможности (Атрибуты Thymeleaf)

В HTML-файлах Thymeleaf добавляет свои атрибуты к стандартным тегам (начинаются с `th:`):

| Атрибут | Зачем нужен | Пример |
| :--- | :--- | :--- |
| `th:text` | Вставляет текст из переменной. | `<span th:text="${user.name}">Имя</span>` |
| `th:each` | Цикл (рендер списка элементов). | `<tr th:each="todo : ${todos}">...</tr>` |
| `th:if` / `th:unless` | Условия (if/else). | `<div th:if="${todos.isEmpty()}">Пусто!</div>` |
| `th:href` | Ссылки (автоматически добавляет контекст приложения). | `<a th:href="@{/login}">Войти</a>` |
| `th:action` | Указывает URL для отправки формы (POST/GET). | `<form th:action="@{/todos/add}" method="post">` |
| `th:replace` | Вставка фрагментов (шапка, футер) из других файлов. | `<div th:replace="fragments/header :: header"></div>` |

---

### Плюсы и минусы

**Преимущества:**
1. Естественные HTML-шаблоны (файл `.html` с Thymeleaf можно открыть в браузере локально и он не сломается).
2. Выдающаяся интеграция со Spring MVC и Spring Security (например, логин через форму).
3. Не нужно отдельно собирать фронтенд (ускоряет разработку простых проектов).

**Недостатки:**
1. Для сложного интерактивного UI (drag&drop, чаты в реальном времени) SSR не подходит — страница будет постоянно "мигать" из-за перезагрузок.
2. Вся визуальная логика перемешивается с HTML внутри тегов `th:*`, что со временем может ухудшить читаемость.

---

### Пример в коде: "ToDo List" (Список задач)

Здесь мы пишем полноценный **MVC Controller** (не REST), который обрабатывает запросы и возвращает HTML-страницы.

**Контроллер (TodoController.java):**
```java
package com.example.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.ArrayList;
import java.util.List;

@Controller // ВАЖНО: @Controller, а не @RestController, так как мы возвращаем HTML!
public class TodoController {

    private final List<String> todos = new ArrayList<>(List.of("Изучить Spring Boot", "Понять Thymeleaf"));

    // Отображение страницы со списком
    @GetMapping("/todos")
    public String getTodos(Model model) {
        model.addAttribute("todos", todos); // Передаем данные в шаблон
        return "todos"; // Spring найдет файл src/main/resources/templates/todos.html
    }

    // Обработка отправки формы
    @PostMapping("/todos/add")
    public String addTodo(@RequestParam("task") String task) {
        todos.add(task);
        return "redirect:/todos"; // Редирект предотвращает повторную отправку формы при обновлении (Pattern: Post-Redirect-Get)
    }
}
```

**Шаблон (src/main/resources/templates/todos.html):**
```html
<!DOCTYPE html>
<!-- Обязательно добавляем пространство имен th для IDE -->
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>ToDo List</title>
    <!-- Подключаем Bootstrap для красоты UI -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="container mt-5">

    <h2>Список задач</h2>

    <!-- Форма для добавления задачи -->
    <form th:action="@{/todos/add}" method="post" class="mb-4">
        <div class="input-group">
            <input type="text" name="task" class="form-control" placeholder="Новая задача..." required>
            <button class="btn btn-primary" type="submit">Добавить</button>
        </div>
    </form>

    <!-- Вывод списка задач через th:each -->
    <ul class="list-group">
        <li th:each="todo : ${todos}" class="list-group-item">
            <span th:text="${todo}">Пример задачи (этот текст заменится реальным)</span>
        </li>
    </ul>

    <!-- Условие th:if (покажется только если список пуст) -->
    <div th:if="${#lists.isEmpty(todos)}" class="alert alert-info mt-3">
        Задач пока нет! Напишите что-нибудь.
    </div>

</body>
</html>
```
