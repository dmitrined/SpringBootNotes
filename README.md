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

```java
@Data // Lombok: создает геттеры, сеттеры, equals, hashCode и toString
@Configuration // Spring: помечает класс как источник определений бинов
@ConfigurationProperties(prefix = "app.security") // Привязывает поля к свойствам из application.yml (app.security.*)
public class SecurityProperties {
    
    @Value("${app.token.expiration:3600}") // Инъекция одиночного значения
    private long expiration;

    private String jwtSecret;
}
```

---

#### 🐘 Шаг 2: `Migrations` (Схема базы данных)
Создаем структуру таблиц через Flyway или Liquibase.
*   **Зачем:** Чтобы структура БД была под контролем Git и не зависела от авто-магии Hibernate.

```sql
-- Пример V1__create_users_table.sql (Flyway - Чистый SQL)
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL
);
```

```xml
<!-- Пример db.changelog-master.xml (Liquibase - XML подход) -->
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.24.xsd">

    <changeSet id="001-create-users-table" author="Dmitri Nedioglo">
        <createTable tableName="users">
            <column name="id" type="BIGINT" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="username" type="VARCHAR(100)">
                <constraints unique="true" nullable="false"/>
            </column>
            <column name="password" type="VARCHAR(255)">
                <constraints nullable="false"/>
            </column>
            <column name="role" type="VARCHAR(20)">
                <constraints nullable="false"/>
            </column>
        </createTable>
    </changeSet>

</databaseChangeLog>
```

---

#### 📦 Шаг 3: `Model` (Сущность и Enum)
Описываем Java-объекты, которые маппятся на таблицы.

```java
@Entity // Помечает класс как JPA сущность
@Table(name = "users") // Имя таблицы в БД
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String username;

    private String password;

    @Enumerated(EnumType.STRING) // Роль как строка
    private Role role;
}
```

---

#### 🌉 Шаг 4: `Repository` (Интерфейс базы данных)

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

---

#### 📄 Шаг 5: `DTO` (Record — Объекты передачи данных)

```java
@Schema(description = "Ответ пользователю")
public record UserResponse(Long id, String username, Role role) {}

@Schema(description = "Запрос на регистрацию")
public record UserRequest(
    @NotBlank @Size(min = 4) String username,
    @NotBlank @Size(min = 8) String password
) {}
```

---

#### 🔄 Шаг 6: `Mapper` (Конвертация)

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserResponse toResponse(User user);
    
    @Mapping(target = "id", ignore = true)
    User toEntity(UserRequest request);
}
```

---

#### 🧠 Шаг 7: `Service` (Бизнес-логика)

```java
@Slf4j
@Service
@RequiredArgsConstructor
@Transactional
public class UserService {
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final PasswordEncoder passwordEncoder;

    public UserResponse register(UserRequest request) {
        log.info("Регистрация пользователя: {}", request.username());
        User user = userMapper.toEntity(request);
        user.setPassword(passwordEncoder.encode(request.password()));
        user.setRole(Role.USER);
        return userMapper.toResponse(userRepository.save(user));
    }
}
```

---

#### 🚪 Шаг 8: `Controller` (Входная точка)

```java
@Tag(name = "Users")
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @Operation(summary = "Регистрация")
    @PostMapping("/register")
    public ResponseEntity<UserResponse> register(@Valid @RequestBody UserRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.register(request));
    }
}
```

---

#### 🛡 Шаг 9: `Security` (Защита)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/users/register").permitAll()
                .anyRequest().authenticated()
            ).build();
    }
}
```

---

### Как всё взаимодействует (Итог):
1.  **Config**: Загрузка настроек.
2.  **Migrations**: Создание/обновление таблиц в БД.
3.  **Model**: Отражение таблиц в коде.
4.  **Repository**: Доступ к данным.
5.  **DTO**: Правила обмена данными.
6.  **Mapper**: Трансформация данных.
7.  **Service**: Выполнение бизнес-задач.
8.  **Controller**: Обработка HTTP запросов.
9.  **Security**: Проверка прав на входе.

---

## Типичная структура папок проекта (от А до Я)

В реальном Spring Boot проекте файлы раскладываются по пакетам и ресурсам в соответствии с их ролью. Ниже представлен самый подробный стандарт организации:

```text
📂 project-root
 ├── 📁 src/main/java/com/example
 │    ├── 📁 config             (Шаг 1: Конфигурация)
 │    │    ├── [class] SecurityConfig.java         # Настройки бинов и фильтров
 │    │    └── 📁 properties
 │    │         └── [class] AppProperties.java      # Класс для @ConfigurationProperties
 │    │
 │    ├── 📁 model              (Шаг 3: Сущности)
 │    │    ├── [class] User.java                   # JPA Сущность
 │    │    └── [enum]  Role.java                   # Роли (ADMIN, USER)
 │    │
 │    ├── 📁 repository         (Шаг 4: Репозитории)
 │    │    └── [interface] UserRepository.java     # Интерфейс базы данных
 │    │
 │    ├── 📁 dto                (Шаг 5: Передача данных)
 │    │    ├── 📁 request
 │    │    │    └── [record] UserCreateRequest.java # Входящий JSON
 │    │    └── 📁 response
 │    │         └── [record] UserResponse.java      # Исходящий JSON
 │    │
 │    ├── 📁 mapper             (Шаг 6: Конвертация)
 │    │    └── [interface] UserMapper.java          # Маппинг Entity <-> DTO
 │    │
 │    ├── 📁 service            (Шаг 7: Бизнес-логика)
 │    │    ├── [interface] UserService.java         # Описание бизнес-функций
 │    │    └── 📁 impl
 │    │         └── [class] UserServiceImpl.java    # Логика реализации
 │    │
 │    ├── 📁 controller         (Шаг 8: API Эндпоинты)
 │    │    └── [class] UserController.java         # REST Контроллер
 │    │
 │    └── 📁 security           (Шаг 9: Защита)
 │         ├── [class] JwtFilter.java              # Проверка JWT токена
 │         └── [class] UserDetailsServiceImpl.java # Загрузка юзера для Security
 │
 └── 📁 src/main/resources
      ├── 📁 db
      │    ├── 📁 migration     (Шаг 2: Flyway)
      │    │    └── V1__init.sql                    # SQL скрипт создания таблиц
      │    └── 📁 changelog     (Шаг 2: Liquibase)
      │         └── db.changelog-master.xml         # Индекс миграций (XML)
      │
      ├── application.yml       # Базовые настройки
      ├── application-dev.yml   # Профиль для разработки (DB: H2/Localhost)
      └── application-prod.yml  # Профиль для сервера (DB: Postgres/Docker)
```

### Подробный разбор именования и типов:

*   **`Config` [class]**: Глобальные настройки.
*   **`Migrations` [.sql/.yaml]**: История изменений БД (Шаг 2).
*   **`Application Profiles`**: Файлы `application-{profile}.yml` для разделения сред.
*   **`Entity` [class]**: Отражение таблицы в БД.
*   **`Repository` [interface]**: Слой доступа к данным.
*   **`DTO` [record]**: Объекты обмена «из интернета».
*   **`Mapper` [interface]**: Переводчики между слоями.
*   **`Service` [class/interface]**: Главные мозги с `@Transactional`.
*   **`Controller` [class]**: Дверь в приложение.

---

*Удачи в изучении Spring Boot!* 🚀

*Удачи в изучении Spring Boot!* 🚀
