# Spring Boot Notes 🍃

Этот репозиторий содержит краткие и понятные заметки по основным слоям и механизмам Spring Boot. Идеально подходит для повторения и быстрого старта.

## Содержание (Table of Contents)

### Основы слоев
1.  [**Controller (Контроллер)**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Controller.md) — Входная точка приложения, обработка запросов.
2.  [**Service (Сервис)**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Service.md) — Бизнес-логика и координация.
3.  [**Repository (Репозиторий)**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Repository.md) — Работа с базой данных (CRUD).
4.  [**Model**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Model.md) — Сущности базы данных (Entities).
5.  [**DTO**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/DTO.md) — Объекты передачи данных.

### Инструменты и механизмы
*   [**Validation**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Validation.md) — Проверка входящих данных.
*   [**Lombok**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Lombok.md) — Генерация шаблонного кода (Getters/Setters).
*   [**Security**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Security.md) — Аутентификация и авторизация.
*   [**Testing**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Testing.md) — Unit и интеграционные тесты.
*   [**Logging**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Logging.md) — Как настроить логи (Slf4j).
*   [**Swagger / OpenAPI**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Swagger.md) — Документация API (`@Operation`, `@Tag`).
*   [**Thymeleaf**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Thymeleaf.md) — Шаблонизатор для генерации HTML на сервере.
*   [**File Upload**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/FileUpload.md) — Загрузка и отдача файлов (MultipartFile).
*   [**E-mail**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Email.md) — Отправка почты (SMTP, JavaMailSender).
*   [**Docker**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Docker.md) — Упаковка приложения и БД в контейнеры.
*   [**Maven**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Maven.md) — Система сборки и управления зависимостями (`pom.xml`).
*   [**Exception Handling**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/ExceptionHandling.md) — Глобальная обработка ошибок (`@ControllerAdvice`).
*   [**Configuration**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Configuration.md) — Настройки, профили, `application.yml` и `@Value`.
*   [**Pagination**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Pagination.md) — Пагинация и сортировка данных (`Pageable`).
*   [**Migrations**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Migrations.md) — Управление структурой БД, отказ от `ddl-auto=update` (Flyway / Liquibase).
*   [**MapStruct**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/MapStruct.md) — Автоматический маппинг `DTO` <-> `Entity`.
*   [**Utils**](file:///Users/dmitrinedioglo/Documents/GitHub/SpringBootNotes/Utils.md) — Вспомогательные классы.

---

## Архитектура: Порядок создания слоев (Снизу вверх — от А до Я)

В Spring Boot принято разделять код на слои (Layered Architecture). Чтобы всё работало правильно, мы начинаем с **фундамента** (БД и Настройки) и постепенно поднимаемся к **фасаду** (Контроллеру).

### Пошаговый гайд на примере сущности `User` и системы ролей (`ADMIN`/`USER`)

---

#### 🛠 Шаг 1: `Configuration` (Основание)
Прежде чем писать код, мы определяем настройки. Например, где хранить аватарки или секретный ключ JWT.
*   **Где:** `application.yml` и класс `@ConfigurationProperties`.
*   **Зачем:** Чтобы менять параметры без пересборки кода.

```java
@Data
@Configuration
@ConfigurationProperties(prefix = "app.security")
public class SecurityProperties {
    private String jwtSecret; // Секрет для подписи токенов
    private long expiration;  // Время жизни токена
}
```

---

#### 📦 Шаг 2: `Model` (Сущность)
Создаем Java-объект, который будет "жить" в базе данных.
*   **Где:** `User.java` (Entity).
*   **Зачем:** Описываем структуру таблицы (id, name, роль).

```java
@Entity
@Table(name = "users")
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private String password;
    private String role; // Например: 'ADMIN' или 'USER'
}
```

---

#### 🌉 Шаг 3: `Repository` (Доступ к данным)
Создаем "мост" для работы с БД.
*   **Где:** `UserRepository.java` (Interface).

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username); // Поиск по имени
}
```

---

#### 📄 Шаг 4: `DTO` и `Mapper` (Передача данных)
Создаем объекты для общения с внешним миром и инструмент для их превращения в Entity.
*   **DTO:** `UserResponse.java` (без пароля!).
*   **Mapper:** `UserMapper.java` (MapStruct).

```java
// DTO - то, что увидит клиент
public record UserResponse(Long id, String username, String role) {}

// Mapper - автоматический перевод Entity -> DTO
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserResponse toResponse(User user);
}
```

---

#### 🧠 Шаг 5: `Service` (Логика)
Здесь происходит вся магия: проверка паролей, поиск в базе, маппинг.
*   **Где:** `UserService.java`.

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final PasswordEncoder passwordEncoder;

    public UserResponse register(UserRequest request) {
        User user = new User();
        user.setUsername(request.username());
        user.setPassword(passwordEncoder.encode(request.password())); // Хешируем!
        user.setRole("USER"); // По умолчанию даем роль USER
        
        User saved = userRepository.save(user);
        return userMapper.toResponse(saved);
    }
}
```

---

#### 🚪 Шаг 6: `Controller` (REST API)
Принимает запросы и отдает ответы.
*   **Где:** `UserController.java`.

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @PostMapping("/register")
    public ResponseEntity<UserResponse> register(@RequestBody UserRequest request) {
        return ResponseEntity.status(201).body(userService.register(request));
    }
}
```

---

#### 🛡 Шаг 7: `Security` (Защита и Доступ)
Настраиваем, кто может обращаться к эндпоинтам.
*   **Где:** `SecurityConfig.java`.

```java
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // Только админам
                .requestMatchers("/api/users/**").permitAll()     // Всем (регистрация)
                .anyRequest().authenticated()
            ).build();
    }
}
```

---

### Как всё взаимодействует (Итог):
1.  **Config** подгружает секреты.
2.  **Controller** ловит HTTP запрос.
3.  **Service** обрабатывает данные через **Config** и **Repository**.
4.  **Repository** сохраняет **Model** в базу.
5.  **Mapper** превращает **Model** в **DTO** и возвращает его через **Controller** клиенту.

*Удачи в изучении Spring Boot!* 🚀
