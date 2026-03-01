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

#### 📄 Шаг 4: `DTO` (Data Transfer Object)
Создаем объекты для передачи данных.
*   **Зачем:** Мы никогда не отдаем пароли или технические поля базы наружу. DTO — это "витрина" ваших данных.

```java
// UserResponse - только те поля, которые безопасно показывать клиенту
public record UserResponse(Long id, String username, String role) {}

// UserRequest - данные, которые приходят при регистрации
public record UserRequest(String username, String password) {}
```

---

#### 🔄 Шаг 5: `Mapper` (Конвертация)
Инструмент для автоматического превращения Entity в DTO и наоборот.
*   **Где:** `UserMapper.java` (обычно через библиотеку MapStruct).
*   **Зачем:** Чтобы вручную не писать `dto.setName(user.getName())` для 20 полей.

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    // Entity -> DTO
    UserResponse toResponse(User user);
    
    // DTO -> Entity
    User toEntity(UserRequest request);
}
```

---

#### 🧠 Шаг 6: `Service` (Логика)
Здесь происходит вся магия: проверка паролей, поиск в базе, маппинг.

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final PasswordEncoder passwordEncoder;

    public UserResponse register(UserRequest request) {
        User user = userMapper.toEntity(request); // Используем маппер
        user.setPassword(passwordEncoder.encode(request.password())); // Хешируем!
        user.setRole("USER");
        
        User saved = userRepository.save(user);
        return userMapper.toResponse(saved);
    }
}
```

---

#### 🚪 Шаг 7: `Controller` (REST API)
Принимает запросы и отдает ответы.

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

#### 🛡 Шаг 8: `Security` (Защита)
Настраиваем доступ к эндпоинтам.

```java
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/users/**").permitAll()
                .anyRequest().authenticated()
            ).build();
    }
}
```

---

### Как всё взаимодействует (Итог):
1.  **Config** подгружает секреты и настройки.
2.  **Model** определяет таблицу в БД.
3.  **Repository** дает методы для сохранения/поиска.
4.  **DTO** описывает формат обмена данными.
5.  **Mapper** переводит данные между форматами.
6.  **Service** управляет бизнес-процессом.
7.  **Controller** связывает бэкенд с интернетом.
8.  **Security** проверяет права доступа на входе.
---

---

## Типичная структура папок проекта (от А до Я)

В реальном Spring Boot проекте файлы раскладываются по пакетам (папкам) в соответствии с их ролью. Ниже представлен стандарт именования и организации:

```text
📂 project-root
 ├── 📁 config             (Шаг 1: Глобальные настройки)
 │    ├── SecurityConfig.java         # Имя технологии + Config
 │    └── 📁 properties
 │         └── AppProperties.java      # Хранение @ConfigurationProperties
 │
 ├── 📁 entity             (Шаг 2: Модели БД)
 │    └── User.java                   # Существительное в единственном числе
 │
 ├── 📁 repository         (Шаг 3: Доступ к данным)
 │    └── UserRepository.java         # Имя сущности + Repository
 │
 ├── 📁 dto                (Шаг 4: Объекты передачи)
 │    ├── 📁 request
 │    │    └── UserCreateRequest.java  # Имя сущности + Request
 │    └── 📁 response
 │         └── UserResponse.java       # Имя сущности + Response
 │
 ├── 📁 mapper             (Шаг 5: Конвертация)
 │    └── UserMapper.java             # Имя сущности + Mapper
 │
 ├── 📁 service            (Шаг 6: Бизнес-логика)
 │    ├── UserService.java            # Имя сущности + Service
 │    └── 📁 impl                      # (Опционально) Реализация интерфейса
 │         └── UserServiceImpl.java    # Имя сервиса + Impl
 │
 ├── 📁 controller         (Шаг 7: API Эндпоинты)
 │    └── UserController.java         # Имя сущности + Controller
 │
 └── 📁 security           (Шаг 8: Фильтры и защита)
      ├── JwtFilter.java              # Техническое имя фильтра
      └── CustomUserDetailsService.java # Имя интерфейса + Impl/Custom
```

### Подробный разбор именования (Шпаргалка):

*   **`Config`**: Настройки Spring, бины и фильтры.
*   **`Entity`**: Отражение таблицы в БД. Только поля и связи.
*   **`Repository`**: Наследники `JpaRepository`. Только методы поиска.
*   **`Request / Response`**: Слой DTO. Разделяйте вход и выход!
*   **`Mapper`**: Чистые функции перевода данных.
*   **`Service`**: Логика. Вызывает репозиторий, делает расчеты.
*   **`Controller`**: Валидация входа и HTTP статусы ответа.

---

*Удачи в изучении Spring Boot!* 🚀
