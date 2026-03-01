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
Прежде чем писать бизнес-логику, мы задаем настройки приложения.
*   **Зачем:** Чтобы внешние параметры (ключи, пути к папкам) не были «зашиты» в коде.

```java
@Data // Lombok: создает геттеры, сеттеры, equals, hashCode и toString
@Configuration // Spring: помечает класс как источник определений бинов
@ConfigurationProperties(prefix = "app.security") // Привязывает поля к свойствам из application.yml (app.security.*)
public class SecurityProperties {
    
    // @Value используется для одиночных значений, если не нужен целый объект настроек
    @Value("${app.token.expiration:3600}") // Инъекция значения из конфига с дефолтным значением 3600
    private long expiration;

    private String jwtSecret; // Будет заполнено из app.security.jwt-secret
}
```

---

#### 📦 Шаг 2: `Model` (Сущность и Enum)
Описываем структуру базы данных.
*   **Зачем:** Это фундамент данных, с которыми работает приложение.

```java
@Entity // Указывает, что этот класс является JPA сущностью и привязан к таблице
@Table(name = "users") // Явное имя таблицы в базе данных
@Data // Генерирует геттеры, сеттеры и стандартные методы
@Builder // Позволяет создавать объекты через User.builder().username("...").build()
@NoArgsConstructor // Нужен Hibernate для создания объекта через рефлексию
@AllArgsConstructor // Нужен для работы @Builder
public class User {
    @Id // Помечает поле как первичный ключ (Primary Key)
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Автоинкремент (1, 2, 3...) на стороне БД
    private Long id;

    @Column(unique = true, nullable = false) // Уникальное поле, не может быть null
    private String username;

    private String password;

    @Enumerated(EnumType.STRING) // Сохраняет Enum в базу как строку ("ADMIN"), а не как число
    private Role role;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL) // Связь «Один ко многим». Один юзер - много заказов.
    private List<Order> orders;
}

/** [enum] Роли пользователей */
public enum Role {
    USER, ADMIN
}
```

---

#### 🌉 Шаг 3: `Repository` (Интерфейс базы данных)
Прослойка для выполнения SQL-запросов.
*   **Зачем:** Избавляет от написания SQL вручную для простых операций.

```java
@Repository // Помечает класс как компонент доступа к данным
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Spring Data сам напишет SQL на основе имени метода!
    Optional<User> findByUsername(String username);

    // Пример кастомного запроса на языке JPQL
    @Query("SELECT u FROM User u WHERE u.role = :role")
    List<User> findAllByRole(@Param("role") Role role);
}
```

---

#### 📄 Шаг 4: `DTO` (Record — Объекты передачи данных)
Формат данных, который мы показываем миру.
*   **Зачем:** Безопасность (скрываем пароли) и стабильность API.

```java
@Schema(description = "Информация о пользователе для ответа") // Swagger: описание для документации
public record UserResponse(
    @Schema(description = "ID пользователя") Long id,
    @Schema(description = "Логин") String username,
    @Schema(description = "Роль") Role role
) {}

@Schema(description = "Данные для регистрации")
public record UserRequest(
    @NotBlank // Валидация: строка не должна быть пустой
    @Size(min = 4) // Валидация: минимум 4 символа
    String username,

    @NotBlank
    @Size(min = 8)
    String password
) {}
```

---

#### 🔄 Шаг 5: `Mapper` (Конвертация)
Перевод между Entity и DTO.
*   **Зачем:** Чтобы не писать вручную `user.set...` в сервисе.

```java
@Mapper(componentModel = "spring") // Помечает интерфейс для работы MapStruct
public interface UserMapper {
    
    // Из Entity в DTO
    UserResponse toResponse(User user);
    
    // Из DTO в Entity (игнорируя ID при создании)
    @Mapping(target = "id", ignore = true)
    User toEntity(UserRequest request);
}
```

---

#### 🧠 Шаг 6: `Service` (Бизнес-логика)
«Мозг» приложения, где живут правила.
*   **Зачем:** Здесь принимаются решения, обрабатываются ошибки и управляются транзакции.

```java
@Slf4j // Создает логгер: позволяет писать log.info("...")
@Service // Помечает класс как сервис (бизнес-логика)
@RequiredArgsConstructor // Создает конструктор для всех final полей (DI через конструктор)
@Transactional // Гарантирует, что все действия в методе либо выполнятся успешно, либо всё откатится
public class UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final PasswordEncoder passwordEncoder;

    public UserResponse register(UserRequest request) {
        log.info("Регистрация нового пользователя: {}", request.username());

        if (userRepository.findByUsername(request.username()).isPresent()) {
            throw new RuntimeException("Пользователь уже существует");
        }

        User user = userMapper.toEntity(request);
        user.setPassword(passwordEncoder.encode(request.password()));
        user.setRole(Role.USER); // По умолчанию - обычный юзер
        
        User saved = userRepository.save(user);
        return userMapper.toResponse(saved);
    }
}
```

---

#### 🚪 Шаг 7: `Controller` (Входная точка)
REST-интерфейс для внешних вызовов.
*   **Зачем:** Принимать запросы, проверять валидность и возвращать HTTP статусы.

```java
@Tag(name = "Пользователи", description = "Управление аккаунтами") // Swagger: группировка в UI
@RestController // @Controller + @ResponseBody (всегда возвращает JSON)
@RequestMapping("/api/users") // Базовый URL для всех методов внутри
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @Operation(summary = "Регистрация") // Swagger: описание метода
    @PostMapping("/register") // Обработка POST запроса
    public ResponseEntity<UserResponse> register(@Valid @RequestBody UserRequest request) {
        // @Valid запускает проверку @NotBlank, @Size и т.д.
        // @RequestBody говорит Spring достать данные из JSON в теле запроса
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.register(request));
    }
}
```

---

#### 🛡 Шаг 8: `Security` (Защита)
Настройка прав доступа.
*   **Зачем:** Определить, кто и куда может заходить.

```java
@Configuration
@EnableWebSecurity // Включает поддержку безопасности Spring Security
public class SecurityConfig {

    @Bean // Регистрирует объект как бин в контексте Spring
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable()) // Отключаем защиту CSRF для REST API
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // Доступ только админам
                .requestMatchers("/api/users/register").permitAll() // Доступно всем
                .anyRequest().authenticated() // Всё остальное требует логина
            ).build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // Алгоритм шифрования паролей
    }
}
```

---

### Как всё взаимодействует (Итог):
1.  **Config**: Приложение стартует и подгружает настройки.
2.  **Controller**: Получает JSON, валидирует его (`@Valid`) и отдает в Сервис.
3.  **Service**: Делает проверки, через **Mapper** переводит DTO в **Model**, шифрует пароль и зовет **Repository**.
4.  **Repository**: Сохраняет данные в БД.
5.  **Security**: Незаметно стоит "на входе" и проверяет, есть ли у пользователя роль `ADMIN` перед доступом к секретным папкам.
---

---

---

## Типичная структура папок проекта (от А до Я)

В реальном Spring Boot проекте файлы раскладываются по пакетам (папкам) в соответствии с их ролью. Ниже представлен стандарт именования и ТИПЫ файлов:

```text
📂 project-root
 ├── 📁 config             (Шаг 1: Конфигурация)
 │    ├── [class] SecurityConfig.java         # Настройки безопасности
 │    └── 📁 properties
 │         └── [class] AppProperties.java      # Свойства @ConfigurationProperties
 │
 ├── 📁 model              (Шаг 2: Модели БД)
 │    ├── [class] User.java                   # Сущность (Entity)
 │    └── [enum]  Role.java                   # Перечисление ролей
 │
 ├── 📁 repository         (Шаг 3: Доступ к данным)
 │    └── [interface] UserRepository.java     # Интерфейс базы данных
 │
 ├── 📁 dto                (Шаг 4: Объекты передачи)
 │    ├── 📁 request
 │    │    └── [record] UserCreateRequest.java # Входящие данные (неизменяемые)
 │    └── 📁 response
 │         └── [record] UserResponse.java      # Исходящие данные (неизменяемые)
 │
 ├── 📁 mapper             (Шаг 5: Конвертация)
 │    └── [interface] UserMapper.java          # Описание логики маппинга
 │
 ├── 📁 service            (Шаг 6: Бизнес-логика)
 │    ├── [class] UserService.java            # Основная логика приложения
 │    └── 📁 impl                             # (Опционально)
 │         └── [class] UserServiceImpl.java    # Реализация интерфейса
 │
 ├── 📁 controller         (Шаг 7: API Эндпоинты)
 │    └── [class] UserController.java         # Обработка HTTP запросов
 │
 └── 📁 security           (Шаг 8: Фильтры и защита)
      ├── [class] JwtFilter.java              # Технический фильтр
      └── [class] CustomUserDetailsService.java # Загрузка пользователя
```

### Подробный разбор именования и типов:

*   **`Config` [class]**: Глобальные настройки.
*   **`Entity` [class]**: Отражение таблицы в БД.
*   **`Role` [enum]**: Набор фиксированных значений (ADMIN, USER).
*   **`Repository` [interface]**: Описываем методы, Spring реализует их сам.
*   **`Request / Response` [record]**: Компактные и безопасные DTO.
*   **`Mapper` [interface]**: Описание правил перевода для MapStruct.
*   **`Service` [class]**: Реализация бизнес-логики.
*   **`Controller` [class]**: Входная точка для внешнего мира.

---

*Удачи в изучении Spring Boot!* 🚀
