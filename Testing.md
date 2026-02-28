# Тестирование (Testing)

Тесты — это гарантия того, что ваш код работает правильно и не сломается завтра при добавлении новых функций.

---

### 1. Типы тестов

#### Unit-тесты (Модульные)
- Тестируют **один маленький кусочек** кода (обычно один метод в сервисе).
- Работают очень быстро.
- Не запускают всё приложение и не подключаются к базе данных.
- Все зависимости "подменяются" заглушками (Mocks).

#### Integration-tests (Интеграционные)
- Тестируют **взаимодействие** разных частей приложения.
- Например: Контроллер -> Сервис -> Репозиторий -> База данных.
- Работают медленнее, так как запускают контекст Spring.

---

### 2. Главные аннотации

| Аннотация | Описание |
| :--- | :--- |
| `@Test` | Помечает метод как тестовый. |
| `@SpringBootTest` | Запускает полный контекст приложения для интеграционных тестов. |
| `@MockBean` / `@Mock` | Создает "пустышку" вместо реального объекта. Вы можете диктовать ей, что возвращать. |
| `@InjectMocks` | Создает реальный объект и вставляет в него созданные `@Mock`. |
| `@MockMvc` | Утилита для имитации HTTP-запросов к вашим контроллерам без запуска сервера. |
| `@Spy` | Позволяет следить за реальным объектом, подменяя только часть его методов. |
| `@ExtendWith` | Расширяет поведение тестов (например, с `MockitoExtension.class` для активации моков). |
| `@AutoConfigureMockMvc` | Автоматически настраивает `MockMvc` для интеграционных тестов. |
| `@Autowired` | Внедряет зависимость (bean) из контекста Spring прямо в тестовый класс. |

---

### 3. Пример Unit-теста (Mockito)
```java
@ExtendWith(MockitoExtension.class)
class AuthServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private AuthService authService;

    @Test
    void register_ShouldFail_WhenUserExists() {
        // Обучаем Mock: "Если спросят про этот email, ответь true"
        when(userRepository.existsByEmail("test@test.com")).thenReturn(true);

        // Проверяем, что кидается исключение
        assertThrows(AlreadyExistsException.class, () -> {
            authService.register(new RegisterRequest("user", "test@test.com", "123456"));
        });
    }
}
```

---

### 4. Пример Integration-теста (@MockMvc)
```java
@SpringBootTest
@AutoConfigureMockMvc
class AuthControllerIT {

    @Autowired
    private MockMvc mockMvc;

    @MockBean // ПРИМЕР: Подменяем реальный сервис на "пустышку" (чтобы не отправлять реальные email)
    private EmailService emailService;

    @Test
    void login_ShouldReturn200() throws Exception {
        mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"username\":\"admin\", \"password\":\"123456\"}"))
                .andExpect(status().isOk());
    }
}
```
---

### Best Practices
1. **Given-When-Then**: Пишите тесты по этой структуре: Дано (создаем данные) -> Когда (вызываем метод) -> Тогда (проверяем результат).
2. **Изоляция**: Тесты не должны зависеть друг от друга. Один тест упал — остальные должны пройти.
3. **Названия**: Называйте тесты так, чтобы было понятно, что именно сломалось: `saveUser_shouldSaveToDatabase()`.
