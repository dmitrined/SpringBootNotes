# Validation (Валидация данных)

**Bean Validation** — это стандарт проверки данных в Java. Мы должны проверять данные сразу, как только они попали в приложение (в контроллере).

### Основные аннотации

| Аннотация | Что проверяет |
| :--- | :--- |
| `@NotNull` | Поле не может быть `null`. |
| `@NotBlank` | Строка не может быть пустой (`""`) или состоять только из пробелов. |
| `@Size(min=.., max=..)` | Ограничение длины строки или размера списка. |
| `@Email` | Проверяет, что строка похожа на email адрес. |
| `@Min` / `@Max` | Ограничение числовых значений. |
| `@Positive` / `@Negative` | Число должно быть больше/меньше нуля. |
| `@FutureOrPresent` | Дата должна быть в будущем или сегодня (удобно для бронирования). |

---

### Как это работает?

1. **В DTO**: вешаем аннотации на поля.
```java
@Data
public class RegisterRequest {
    @NotBlank(message = "Username is mandatory")
    @Size(min = 3, max = 20)
    private String username;

    @Email(message = "Invalid email format")
    private String email;

    @NotBlank
    @Size(min = 6)
    private String password;
}
```

2. **В Контроллере**: добавляем аннотацию `@Valid`.
```java
@PostMapping("/register")
public ResponseEntity<?> register(@Valid @RequestBody RegisterRequest request) {
    // Если данные не пройдут проверку, метод даже не начнет работу!
    // Spring сам вернет 400 Bad Request.
    return ResponseEntity.ok().build();
}
```

---

### Почему это важно?
- Защита базы данных от "мусора".
- Информативные ошибки для пользователя (через `message`).
- Автоматизация: вам не нужно писать десятки `if (name == null)`.
