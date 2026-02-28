# Слой Controller (Контроллер)

### Роль
**Контроллер** — это "входная дверь" вашего приложения. Его главная задача — принять HTTP-запрос от клиента (браузера, мобильного приложения), проверить корректность входных данных и передать выполнение бизнес-логике (слою Service).

**За что отвечает:**
- Определение путей (URL) и методов (GET, POST, PUT, DELETE).
- Валидация входящих данных (@Valid).
- Возврат HTTP-статуса (200 OK, 201 Created, 404 Not Found и т.д.).
- Преобразование данных из JSON в Java-объекты и наоборот.

**Что делать КАТЕГОРИЧЕСКИ нельзя:**
- Писать бизнес-логику (расчеты, условия, работу с базой данных).
- Обращаться напрямую к Repository.
- Содержать состояние (контроллеры должны быть stateless).

---

### Аннотации

| Аннотация | Описание |
| :--- | :--- |
| `@RestController` | Помечает класс как контроллер, где каждый метод возвращает данные (обычно JSON) напрямую в тело ответа. |
| `@RequestMapping("/api/...")` | Указывает базовый URL для всех методов в этом классе. |
| `@PostMapping`, `@GetMapping` | Определяют тип HTTP-запроса для конкретного метода. |
| `@RequestBody` | Говорит Spring взять данные из тела запроса (JSON) и превратить их в Java-объект. |
| `@PathVariable` | Используется для извлечения данных из самого URL (например, `/api/users/{id}`). |
| `@RequestParam` | Используется для параметров в конце URL после знака `?` (например, `/api/search?name=Ivan`). |
| `@Valid` | Запускает проверку данных (валидацию) перед тем, как метод начнет работу. |

---

### Связи
Контроллер взаимодействует только со **слоем Service**.
1. Клиент посылает запрос.
2. Контроллер принимает его.
3. Контроллер вызывает метод `Service`.
4. Получает результат от `Service`.
5. Оборачивает результат в `ResponseEntity` и отправляет клиенту.

---

### Примеры кода
```java
@Slf4j // ПРИМЕР: Добавляем логгер в контроллер
@Tag(name = "User API", description = "Управление пользователями системы") // ПРИМЕР: Группировка для Swagger
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor // Создает конструктор для внедрения зависимостей
public class UserController {

    private final UserService userService; // Внедряем сервис

    // ПРИМЕР 1: POST запрос с телом (JSON) и валидацией
    @Operation(summary = "Регистрация нового пользователя", description = "Создает пользователя и возвращает его данные")
    @PostMapping("/register")
    public ResponseEntity<UserResponse> register(@Valid @RequestBody RegisterRequest request) {
        UserResponse response = userService.register(request);
        // Возвращаем статус 201 Created
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    // ПРИМЕР 2: GET запрос с переменной в URL (@PathVariable)
    // URL ожидается: /api/users/123
    @Operation(summary = "Получить пользователя по ID")
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUserById(
            @Parameter(description = "ID пользователя базы данных") @PathVariable("id") Long id) {
        log.info("Received request to fetch user with id: {}", id); // Логируем входящий запрос
        UserResponse response = userService.findById(id);
        return ResponseEntity.ok(response);
    }

    // ПРИМЕР 3: GET запрос с параметром в строке запроса (@RequestParam)
    // URL ожидается: /api/users/search?name=Ivan
    @Operation(summary = "Поиск пользователей по имени")
    @GetMapping("/search")
    public ResponseEntity<List<UserResponse>> searchUsers(
            @Parameter(description = "Имя для поиска (например, Ivan)") @RequestParam("name") String name) {
        List<UserResponse> users = userService.findByName(name);
        return ResponseEntity.ok(users);
    }
}
```

---

### Best Practices (Чистый код)
1. **Тонкие контроллеры**: В методе контроллера должно быть 2-3 строки кода. Всё остальное — в сервисе.
2. **Используйте ResponseEntity**: Всегда возвращайте `ResponseEntity`, чтобы явно контролировать HTTP-статусы.
3. **Единый префикс**: Начинайте все пути с `/api`, чтобы отделить API от статических ресурсов или фронтенда.
