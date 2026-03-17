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
package com.example.service;

import com.example.dto.RegisterRequest;
import com.example.dto.AuthResponse;
import com.example.dto.UserResponse;
import com.example.exception.AlreadyExistsException;
import com.example.model.User;
import com.example.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Slf4j // ПРИМЕР: Добавляем логгер в сервис
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder; // Зашифрует пароль перед сохранением

    // ПРИМЕР 1: Запись в базу. Если будет ошибка, изменения откатятся.
    @Transactional 
    public AuthResponse register(RegisterRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            log.warn("Registration failed. Email {} is already in use", request.getEmail()); // Логируем бизнес-ошибку
            throw new AlreadyExistsException("Email already taken");
        }

        User user = User.builder()
                .email(request.getEmail())
                .password(passwordEncoder.encode(request.getPassword()))
                .build();

        userRepository.save(user);
        log.info("Successfully created new user with email: {}", user.getEmail()); // Логируем успех
        
        return new AuthResponse("Success", user.getEmail());
    }

    // ПРИМЕР 2: Чтение из базы. @Transactional(readOnly = true) ускоряет работу
    @Transactional(readOnly = true)
    public UserResponse findById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("User not found"));
        return new UserResponse(user.getEmail(), user.getEmail());
    }
}
```

---

### Best Practices (Чистый код)
1. **Один сервис — одна задача**: Старайтесь не создавать "Божественные сервисы" (Giant Services), которые делают всё. Лучше разделить их по функционалу (UserService, BookingService).
2. **Программируйте на интерфейсах**: Если логика сложная или может меняться, создайте интерфейс `AuthService` и его реализацию `AuthServiceImpl`.
3. **Без побочных эффектов**: Сервис не должен зависеть от того, откуда его вызвали (из контроллера, из шедулера или из теста).
