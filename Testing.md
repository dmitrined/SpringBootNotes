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
package com.example.service;

import com.example.repository.UserRepository;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class) // Включаем поддержку Mockito
class UserServiceTest {

    @Mock
    private UserRepository userRepository; // Делаем "куклу" БД

    @InjectMocks
    private UserService userService; // Сюда Mockito подставит userRepository

    @Test
    void shouldReturnTrueWhenEmailExists() {
        // Указываем: "когда спросят ivan@gmail.com, отвечай true"
        when(userRepository.existsByEmail("ivan@gmail.com")).thenReturn(true);

        // Вызываем реальный метод сервиса
        boolean exists = userService.checkUserExists("ivan@gmail.com");

        // Проверяем результат
        assertTrue(exists);
        verify(userRepository).existsByEmail("ivan@gmail.com"); // Убеждаемся, что метод БД реально вызывался
    }
}
```

---

### 2. Integration Tests (Интеграционные тесты)
Тестируем, как разные слои работают вместе. Здесь поднимается реальный Spring Context.

```java
package com.example.controller;

import com.example.dto.RegisterRequest;
import com.example.service.UserService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;

@SpringBootTest 
@AutoConfigureMockMvc // Позволяет отправлять фейковые HTTP запросы
class UserControllerIT {

    @Autowired
    private MockMvc mockMvc; 

    @Autowired
    private ObjectMapper objectMapper; // Для превращения объектов в JSON (и обратно)

    @MockBean
    private UserService userService; // Замокаем сервис, чтобы не писать в реальную БД при тесте контроллера

    @Test
    void shouldReturn201OnSuccessfulRegistration() throws Exception {
        RegisterRequest request = new RegisterRequest("Ivan", "ivan@gmail.com", "password");
        
        // Настраиваем Mock, чтобы сервис не ругался
        when(userService.register(any(RegisterRequest.class))).thenReturn(null);

        // Имитируем POST запрос
        mockMvc.perform(post("/api/users/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated()); // Ожидаем ответ 201 Created
    }
}
```
---

### Best Practices
1. **Given-When-Then**: Пишите тесты по этой структуре: Дано (создаем данные) -> Когда (вызываем метод) -> Тогда (проверяем результат).
2. **Изоляция**: Тесты не должны зависеть друг от друга. Один тест упал — остальные должны пройти.
3. **Названия**: Называйте тесты так, чтобы было понятно, что именно сломалось: `saveUser_shouldSaveToDatabase()`.
