# Слой Service (Сервис)

### Роль
**Сервис** — это "мозг" вашего приложения. Здесь живет вся бизнес-логика: расчеты, проверки прав доступа, сложные условия и координация работы между разными репозиториями.

**За что отвечает:**
- Реализация бизнес-логики.
- Обработка исключений (Exceptions).
- Транзакционность (если нужно сохранить данные в несколько таблиц сразу).
- Преобразование данных из Entity в DTO и наоборот (иногда с помощью Mapper).

**Что делать КАТЕГОРИЧЕСКИ нельзя:**
- Получать напрямую `HttpServletRequest` или работать с HTTP-статусами (это работа контроллера).
- Возвращать Entity напрямую в контроллер (лучше использовать DTO).

---

### Аннотации

| Аннотация | Описание |
| :--- | :--- |
| `@Service` | Помечает класс как сервис. Spring создаст экземпляр этого класса (Bean) и будет им управлять. |
| `@Transactional` | Гарантирует, что все действия внутри метода либо выполнятся успешно вместе, либо, если произойдет ошибка, все изменения в базе откатятся. |
| `@RequiredArgsConstructor` | (Lombok) Удобно использовать для внедрения зависимостей (репозиториев) через конструктор. |

---

### Связи
1. **Входящая**: Вызывается из `Controller`.
2. **Исходящая**: Вызывает методы `Repository` для работы с базой данных.
3. **Вспомогательная**: Может вызывать другие сервисы или утилиты.

---

### Примеры кода
```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository; // Репозиторий
    private final PasswordEncoder passwordEncoder; // Утилита

    // ПРИМЕР 1: Запись в базу. Если будет ошибка, изменения откатятся.
    @Transactional 
    public AuthResponse register(RegisterRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new AlreadyExistsException("Email already taken");
        }

        User user = User.builder()
                .email(request.getEmail())
                .password(passwordEncoder.encode(request.getPassword()))
                .build();

        userRepository.save(user);
        return new AuthResponse("Success", user.getEmail());
    }

    // ПРИМЕР 2: Только чтение. readOnly = true улучшает производительность!
    @Transactional(readOnly = true)
    public UserResponse getUserInfo(Long id) {
        User user = userRepository.findById(id).orElseThrow();
        return new UserResponse(user.getEmail(), user.getFirstName());
    }
}
```

---

### Best Practices (Чистый код)
1. **Один сервис — одна задача**: Старайтесь не создавать "Божественные сервисы" (Giant Services), которые делают всё. Лучше разделить их по функционалу (UserService, BookingService).
2. **Программируйте на интерфейсах**: Если логика сложная или может меняться, создайте интерфейс `AuthService` и его реализацию `AuthServiceImpl`.
3. **Без побочных эффектов**: Сервис не должен зависеть от того, откуда его вызвали (из контроллера, из шедулера или из теста).
