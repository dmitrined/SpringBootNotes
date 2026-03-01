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
@Configuration // Spring: помечает класс как источник бинов (объектов, которыми управляет Spring)
@ConfigurationProperties(prefix = "app.security") // Привязывает поля к свойствам из application.yml (все, что начинается с app.security)
public class SecurityProperties {
    
    @Value("${app.token.expiration:3600}") // Инъекция одиночного значения из конфига (3600 - значение по умолчанию)
    private long expiration; // Время жизни токена в секундах

    private String jwtSecret; // Секрет для подписи JWT (подтягивается из app.security.jwt-secret)
}
```

---

#### 🐘 Шаг 2: `Migrations` (Схема базы данных)
Создаем структуру таблиц через Flyway или Liquibase. Это гарантирует, что у всех разработчиков база будет одинаковой.

```sql
-- Пример V1__create_users_table.sql (Flyway - Чистый SQL)
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY, -- Автоинкрементный ID (для Postgres)
    username VARCHAR(100) NOT NULL UNIQUE, -- Логин, обязателен и уникален
    password VARCHAR(255) NOT NULL, -- Хеш пароля
    role VARCHAR(20) NOT NULL -- Роль пользователя (USER/ADMIN)
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

    <!-- changeSet - одна атомарная единица изменений -->
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
Описываем Java-объекты, которые связаны с таблицами БД через Hibernate (JPA).

```java
@Entity // JPA: Помечает класс как сущность (объект БД)
@Table(name = "users") // JPA: Явное имя таблицы в базе
@Data // Lombok: Создает геттеры, сеттеры, toString, equals, hashCode
@Builder // Lombok: Добавляет паттерн "Строитель" (удобное создание объектов)
@NoArgsConstructor // Lombok: Конструктор без аргументов (обязателен для Hibernate)
@AllArgsConstructor // Lombok: Конструктор со всеми полями (нужен для @Builder)
public class User {
    @Id // JPA: Помечает поле как первичный ключ (Primary Key)
    @GeneratedValue(strategy = GenerationType.IDENTITY) // JPA: Стратегия автоинкремента (1, 2, 3...)
    private Long id;

    @Column(unique = true, nullable = false) // JPA: Доп. правила для колонки (уникальность, не null)
    private String username;

    private String password;

    @Enumerated(EnumType.STRING) // JPA: Сохранять Enum как строку ("ADMIN"), а не как число (0)
    private Role role;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL) // JPA: Связь "Один ко многим" (один юзер - много постов/заказов)
    // mappedBy - указывает на поле в дочерней сущности, которое владеет связью
    private List<Order> orders;
}

/** 
 * Пример дочерней сущности для демонстрации ManyToOne
 * @ManyToOne - Связь "Многие к одному" (много заказов - к одному юзеру)
 * @JoinColumn - Указывает имя колонки внешнего ключа (foreign key) в таблице заказов
 */

/** [enum] Список доступных ролей в системе */
public enum Role {
    USER, ADMIN
}
```

---

#### 🌉 Шаг 4: `Repository` (Слой доступа к данным)
Интерфейс, который берет на себя всю работу с SQL запросами.

```java
@Repository // Spring: Помечает интерфейс как DAO (Data Access Object)
public interface UserRepository extends JpaRepository<User, Long> { // <Сущность, Тип ID>
    
    /** Находит пользователя по имени. Spring сам сгенерирует SQL! */
    Optional<User> findByUsername(String username);
}
```

---

#### 📄 Шаг 5: `DTO` (Record — Объекты передачи данных)
Специальные классы-контейнеры для API. Мы никогда не отдаем пароль из `User` наружу, для этого есть DTO.

```java
@Schema(description = "Ответ с данными пользователя") // OpenAPI: Описание для Swagger
public record UserResponse(
    @Schema(description = "ID пользователя") Long id,
    @Schema(description = "Логин (никнейм)") String username,
    @Schema(description = "Текущая роль") Role role
) {}

@Schema(description = "Запрос на создание аккаунта")
public record UserRequest(
    @NotBlank(message = "Логин не может быть пустым") // Validation: Проверка на пустоту
    @Size(min = 4, max = 20) // Validation: Ограничение длины строки
    String username,

    @NotBlank(message = "Пароль обязателен")
    @Size(min = 8) // Пароль минимум 8 символов
    String password
) {}
```

---

#### 🔄 Шаг 6: `Mapper` (Конвертация слоев)
Автоматизирует процесс перекладывания данных из Entity в DTO и обратно.

```java
@Mapper(componentModel = "spring") // MapStruct: Генерирует реализацию маппера как бин Spring
public interface UserMapper {
    
    /** Превращает Entity (из БД) в DTO (для клиента) */
    UserResponse toResponse(User user);
    
    /** Превращает DTO (от клиента) в Entity (для БД), игнорируя ID при создании */
    @Mapping(target = "id", ignore = true)
    User toEntity(UserRequest request);
}
```

---

#### 🧠 Шаг 7: `Service` (Бизнес-логика)
Здесь живет основная логика вашего приложения.

```java
@Slf4j // Lombok: Добавляет логгер (через переменную 'log')
@Service // Spring: Помечает класс как сервис (слой бизнес-логики)
@RequiredArgsConstructor // Lombok: Создает конструктор для всех 'final' полей (Dependency Injection)
@Transactional // Spring: Все действия в методе либо выполнятся вместе, либо откатятся при ошибке
public class UserService {
    private final UserRepository userRepository; // Внедряем репозиторий
    private final UserMapper userMapper; // Внедряем маппер
    private final PasswordEncoder passwordEncoder; // Внедряем кодировщик паролей

    public UserResponse register(UserRequest request) {
        log.info("Попытка регистрации пользователя: {}", request.username()); // Логирование
        
        // 1. Проверяем существование
        if (userRepository.findByUsername(request.username()).isPresent()) {
            throw new RuntimeException("Пользователь уже существует!");
        }

        // 2. Маппим DTO в Entity
        User user = userMapper.toEntity(request);
        
        // 3. Шифруем пароль
        user.setPassword(passwordEncoder.encode(request.password()));
        
        // 4. Назначаем роль по умолчанию
        user.setRole(Role.USER);
        
        // 5. Сохраняем и возвращаем ответ
        return userMapper.toResponse(userRepository.save(user));
    }
}
```

---

#### 🚪 Шаг 8: `Controller` (REST Эндпоинты)
Лицо вашего приложения. Принимает HTTP запросы и возвращает ответы.

```java
@Tag(name = "Пользователи", description = "Управление аккаунтами и регистрация") // Swagger: Группа API
@RestController // Spring: @Controller + @ResponseBody (всегда возвращает JSON)
@RequestMapping("/api/users") // Spring: Базовый путь (URL) для этого контроллера
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @Operation(summary = "Создать нового пользователя") // Swagger: Описание метода
    @PostMapping("/register") // Spring: Обработка HTTP POST запроса
    public ResponseEntity<UserResponse> register(
        @Valid @RequestBody UserRequest request // @Valid: Включает проверку @NotBlank и @Size
    ) {
        // @RequestBody: Превращает входящий JSON в Java объект UserRequest
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.register(request));
    }
}
```

---

#### 🛡 Шаг 9: `Security` (Защита и доступ)
Настраивает права доступа к вашим методам.

```java
@Configuration // Spring: Класс с настройками
@EnableWebSecurity // Security: Включает стандартные фильтры безопасности веб-слоя
@EnableMethodSecurity // Security: Включает проверку прав через @PreAuthorize в коде
public class SecurityConfig {

    @Bean // Spring: Объект будет создан и управляем контейнером Spring
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable()) // Отключаем CSRF для REST (т.к. работаем через токены)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/users/register").permitAll() // Разрешаем регистрацию всем
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // Только для админов
                .anyRequest().authenticated() // Остальные запросы - только после логина
            ).build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // Алгоритм хеширования (производная от BCrypt)
    }
}
```

---

### Как всё взаимодействует (Итог):
1.  **Config**: Загружает настройки (тайм-ауты, секреты) из файлов YAML.
2.  **Migrations**: Создает необходимые таблицы в базе данных при первом запуске.
3.  **Model**: Описывает эти таблицы в виде Java классов для Hibernate.
4.  **Repository**: Дает готовые методы (save, findById) для работы с БД.
5.  **DTO**: Определяет, какие поля клиент может присылать и видеть в JSON.
6.  **Mapper**: Быстро «переливает» данные из DTO в Entity и обратно.
7.  **Service**: Делает реальную работу (шифрует, проверяет логику, считает).
8.  **Controller**: Открывает «дверь» (URL) для клиентов, проверяет JSON.
9.  **Security**: Проверяет, есть ли у зашедшего в «дверь» пропуск (токен) и нужная роль.

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
