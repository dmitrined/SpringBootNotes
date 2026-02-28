# Lombok: Наш помощник

**Lombok** — это библиотека, которая автоматически генерирует скучный код (геттеры, сеттеры, конструкторы) прямо во время компиляции. Она делает код чище.

### Основные аннотации

| Аннотация | Что она создает |
| :--- | :--- |
| `@Getter` / `@Setter` | Создает методы `get...()` и `set...()` для всех полей класса. |
| `@ToString` | Генерирует красивый метод для печати объекта в логах. |
| `@EqualsAndHashCode` | Нужна для корректного сравнения объектов. |
| `@NoArgsConstructor` | Создает пустой конструктор (обязательно для @Entity). |
| `@AllArgsConstructor` | Создает конструктор со всеми полями. |
| `@RequiredArgsConstructor` | Создает конструктор только для `final` полей (идеально для DI в сервисах). |
| `@Data` | "Всё в одном": включает Getter, Setter, ToString, EqualsAndHashCode и RequiredArgsConstructor. |
| `@Builder` | Позволяет создавать объекты красиво: `User.builder().name("A").build()`. |

---

### Пример без Lombok (Плохо):
```java
public class User {
    private String name;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    // + еще 50 строк конструкторов, toString и т.д.
}
```

### Пример с Lombok (Хорошо):
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String name;
    private String email;
}
// Всё! Код чистый и понятный.
```

---

### Почему это экономит время?
- Если вы добавите новое поле в класс, вам не нужно пересоздавать конструкторы и геттеры. Lombok всё сделает за вас.
- Меньше кода = меньше мест, где можно сделать опечатку.
