# Обработка ошибок (@ControllerAdvice)

В реальном мире приложение постоянно сталкивается с ошибками: пользователь ввел занятый `email`, не нашел `ID` в базе данных или забыл передать пароль. 

Если в Spring Boot просто выкинуть `new RuntimeException("Пользователь не найден")`, сервер вернет клиенту страшную **500 Internal Server Error** с огромным куском Java-кода (Stacktrace). Это ломает фронтенд (React/Vue/Mobile) и выглядит непрофессионально.

**✅ Правильное решение:** Использовать глобальный перехватчик ошибок `@ControllerAdvice`.

---

### 1. Как работает @ControllerAdvice?

`@ControllerAdvice` (или `@RestControllerAdvice` для REST API) — это "Страж", который стоит перед всеми вашими Контроллерами. 

Когда любой Контроллер или Сервис внезапно выбрасывает `Exception` (ошибку), этот Страж ловит её на лету, сам генерирует красивый и понятный JSON-ответ и отдает его клиенту (вместо падения сервера).

---

### 2. Создаем собственные классы ошибок (Business Exceptions)

Не стоит использовать стандартный `RuntimeException`. Лучше создать свои классы ошибок, чтобы чётко понимать, что пошло не так.

Создадим папку `exception` и пару классов:

**Ошибка: Ресурс не найден (Например, юзер с ID=99)**
```java
package com.example.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

**Ошибка: Конфликт данных (Например, Email уже занят)**
```java
package com.example.exception;

public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}
```

---

### 3. Единая структура ответа об ошибке (ErrorResponse)

Фронтендеры скажут вам "Спасибо", если все ошибки от сервера будут приходить в одинаковом формате JSON. 
Сделаем для этого простой DTO (в пакете `dto`):

```java
package com.example.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;
import java.time.LocalDateTime;

@Getter
@AllArgsConstructor
public class ErrorResponse {
    private LocalDateTime timestamp; // Когда случилась ошибка
    private int status;              // HTTP код (404, 400, 409...)
    private String error;            // Краткий тип (Not Found)
    private String message;          // Понятный текст для пользователя ("Юзер не найден")
    private String path;             // По какому URL произошла беда
}
```

---

### 4. Пишем Глобальный Перехватчик (GlobalExceptionHandler)

Создадим класс, который будет "ловить" наши кастомные ошибки из пункта 2 и упаковывать их в формат из пункта 3.

```java
package com.example.exception;

import com.example.dto.ErrorResponse;
import jakarta.servlet.http.HttpServletRequest;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;

@Slf4j
@RestControllerAdvice // Аннотация означает "Я перехватываю ошибки со всех @RestController"
public class GlobalExceptionHandler {

    // 1. Ловим ошибку "Не найдено" -> Отдаем HTTP 404
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex, 
            HttpServletRequest request) {
        
        log.warn("Resource not found: {}", ex.getMessage()); // Обязательно логируем
        
        ErrorResponse error = new ErrorResponse(
                LocalDateTime.now(),
                HttpStatus.NOT_FOUND.value(), // 404
                HttpStatus.NOT_FOUND.getReasonPhrase(),
                ex.getMessage(),
                request.getRequestURI() // Получаем URL из запроса (/api/users/99)
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    // 2. Ловим ошибку "Конфликт" (email занят) -> Отдаем HTTP 409
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleDuplicate(
            DuplicateResourceException ex, 
            HttpServletRequest request) {
        
        log.warn("Duplicate resource: {}", ex.getMessage());
        
        ErrorResponse error = new ErrorResponse(
                LocalDateTime.now(),
                HttpStatus.CONFLICT.value(), // 409
                HttpStatus.CONFLICT.getReasonPhrase(),
                ex.getMessage(),
                request.getRequestURI()
        );
        return new ResponseEntity<>(error, HttpStatus.CONFLICT);
    }

    // 3. Последняя линия обороны: ловим абсолютно ВСЕ непредвиденные сбои (NullPointerException и т.д.)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
            Exception ex, 
            HttpServletRequest request) {
        
        // Тут пишем log.error, потому что это реальная бага в коде!
        log.error("Unhandled exception occurred at {}: ", request.getRequestURI(), ex);
        
        ErrorResponse error = new ErrorResponse(
                LocalDateTime.now(),
                HttpStatus.INTERNAL_SERVER_ERROR.value(), // 500
                "Internal Server Error",
                "Что-то пошло не так на сервере. Обратитесь в поддержку.", // Скрываем реальную причину от юзера
                request.getRequestURI()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

---

### 5. Как это выглядит в Сервисе (`UserService`)

Теперь, если в бизнес-логике что-то идет не по плану, мы просто кидаем `throw new ...`. Не нужно задумываться о JSON'ах и HTTP статусах — Глобальный Перехватчик сам сделает всю грязную работу.

```java
package com.example.service;

import com.example.exception.DuplicateResourceException;
import com.example.exception.ResourceNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    // ... repository

    public void registerUser(String email) {
        if (userRepository.existsByEmail(email)) {
            // Кидаем нашу понятную ошибку. Сервис тут же прерывает работу.
            throw new DuplicateResourceException("Пользователь с email " + email + " уже существует!");
        }
        // ... сохранение
    }

    public User getUserById(Long id) {
        // Красивый интерфейс Optional из Java 8
        return userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Пользователь с ID " + id + " не найден в базе данных."));
    }
}
```

### ⚡ Итог (Что в итоге увидит Фронтендер в Postman):

Клиент делает: `GET /api/users/999` (id которого нет)
Ответ сервера `404 Not Found`:
```json
{
    "timestamp": "2024-03-15T10:30:15.123",
    "status": 404,
    "error": "Not Found",
    "message": "Пользователь с ID 999 не найден в базе данных.",
    "path": "/api/users/999"
}
```
*Красиво, безопасно (не "палит" внутренний код) и профессионально!*
