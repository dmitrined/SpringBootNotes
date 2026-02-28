# Пагинация и Сортировка (Pagination & Sorting)

У нас есть таблица `users` в базе данных. Вы делаете запрос `SELECT * FROM users;` (в Java это `userRepository.findAll()`).

**Проблема:** Если в вашей базе 10 миллионов пользователей, этот метод попытается достать их всех, заполнит всю оперативную память (RAM) вашего сервера, и сервер упадет с ошибкой `OutOfMemoryError`. 

Более того, фронтенд не сможет и не должен отрисовывать сразу 10 миллионов строк в браузере.

**Решение:** Отдавать данные **по кусочкам (страницам)**. Это и есть Пагинация. 

---

### 1. Как это работает в Spring Boot?

Spring Data JPA предоставляет гениально простой интерфейс — **`Pageable`**.
Как только мы передадим этот интерфейс в запрос, Hibernate автоматически сгенерирует SQL-запросы с операторами `LIMIT` (сколько записей брать) и `OFFSET` (сколько пропустить).

---

### 2. Изменения в слое `Repository`

В обычном `JpaRepository` уже встроена поддержка пагинации. Но если мы пишем свои кастомные методы, мы просто добавляем параметр `Pageable` в конец метода, и возвращаем не `List`, а `Page` (чтобы знать общее количество элементов).

```java
package com.example.repository;

import com.example.model.User;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // 1. Стандартный поиск: вернуть всех -> Плохо
    // List<User> findAll(); 
    
    // 2. Поиск со страницами -> УЖЕ встроен в JpaRepository!
    // Page<User> findAll(Pageable pageable); 

    // 3. Кастомный поиск с пагинацией (например, найти всех админов по страницам)
    Page<User> findByRole(String role, Pageable pageable);
}
```

---

### 3. Изменения в слое `Service`

В сервисе нет ничего сложного. Он просто пробрасывает `Pageable` от Контроллера до Репозитория. И, конечно же, вынимает из объекта `Page<User>` чистые данные `List<UserResponse>`, преобразуя внутренние Entity в DTO-шки.

```java
package com.example.service;

import com.example.dto.UserResponse;
import com.example.model.User;
import com.example.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    // Метод возвращает не List, а Page
    public Page<UserResponse> getAllUsersPaged(Pageable pageable) {
        
        // 1. Делаем умный запрос в базу
        Page<User> usersPage = userRepository.findAll(pageable);
        
        // 2. Превращаем каждую Entity в DTO (Spring внутри себя сделает .map() для списка)
        return usersPage.map(user -> new UserResponse(user.getEmail(), user.getUsername()));
    }
}
```

---

### 4. Изменения в слое `Controller` (Откуда берется Pageable?)

Контроллер получает параметры от клиента прямо в URL запроса (Query Parameters). 
Например: `GET /api/users?page=0&size=5&sort=username,asc`
`page=0` (Спринг считает страницы с нуля)
`size=5` (брать по 5 штук)
`sort=username,asc` (Сортировать по полю username по алфавиту).

Контроллер магическим образом превратит эти параметры в объект `Pageable`!

```java
package com.example.controller;

import com.example.dto.UserResponse;
import com.example.service.UserService;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<Page<UserResponse>> getAllUsers(
            // Аннотация @PageableDefault страхует нас: если фронтенд забудет передать size и page,
            // Спринг сам подставит: 1-ю страницу (page = 0) по 10 элементов, отсортированную по ID по убыванию.
            @PageableDefault(size = 10, page = 0, sort = "id", direction = org.springframework.data.domain.Sort.Direction.DESC) 
            Pageable pageable
    ) {
        // Прокидываем объект в сервис
        return ResponseEntity.ok(userService.getAllUsersPaged(pageable));
    }
}
```

---

### 5. Что увидит Фронтенд (JSON)?
Фронтенд получит не просто массив, а очень "богатый" объект страницы. В нем лежат не только сами юзеры, но и **мета-данные**, необходимые клиентскому программисту для отрисовки кнопочек "След. страница", "Пред. страница", "Всего страниц".

```json
{
    "content": [
        {
            "email": "ivan@gmail.com",
            "username": "Ivan"
        },
        {
            "email": "olga@mail.ru",
            "username": "Olga"
        }
        // ... еще 3 юзера
    ],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 5,
        "sort": {
            "empty": false,
            "sorted": true,
            "unsorted": false
        },
        "offset": 0,
        "paged": true,
        "unpaged": false
    },
    // ВАЖНЫЕ МЕТА-ДАННЫЕ:
    "last": false,          // Это последняя страница? (чтоб скрыть кнопку "Вперед")
    "totalElements": 150,   // Всего пользователей в базе
    "totalPages": 30,       // Значит всего страниц (если по 5 штук): 30
    "first": true,          // Это первая страница?
    "size": 5,
    "number": 0,            // Текущий номер страницы
    "empty": false
}
```
