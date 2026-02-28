# Маппинг Сущностей (MapStruct)

Вы уже знаете, что ни в коем случае нельзя возвращать `Entity` (Модели) из контроллера клиенту. 
Мы используем **DTO** — объекты для передачи чистых данных (без паролей, системных дат и циклических ссылок).

**В чем боль?** Начинающие программисты (а часто и сеньоры) пишут тонны ручного и скучного кода:

```java
// Представьте, что у юзера 20 полей! (имя, фамилия, адреса, телефоны, инн)
User entity = new User();
entity.setFirstName(dto.getFirstName());
entity.setLastName(dto.getLastName());
entity.setEmail(dto.getEmail());
entity.setAge(dto.getAge());
entity.setAvatar(dto.getAvatar());
// ... еще 15 строк
```
Каждый такой "перекладчик" ломается, если вы в Entity поменяли название одного поля, а в DTO забыли.

---

### Топовое решение: MapStruct

**MapStruct** — это сумасшедше быстрый кодогенератор для Java (работает по принципу Lombok, но не про геттеры/сеттеры, а про маппер сущностей). 

Он сам напишет все `dto.get...()` и `entity.set...()`.

---

### 1. Как подключить?
Нужно добавить в `pom.xml` зависимость и процессор аннотаций (чтобы он генерировал код до компиляции проекта):

```xml
<dependencies>
    <!-- Сама библиотека -->
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>1.5.5.Final</version>
    </dependency>
</dependencies>

<!-- А это в конце pom.xml в секции <build><plugins>... -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.5.5.Final</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

---

### 2. Как пользоваться (Магия в три строчки)

Создаем интерфейс (Spring сам сделает реализацию под капотом):

```java
package com.example.mapper;

import com.example.dto.RegisterRequest;
import com.example.dto.UserResponse;
import com.example.model.User;
import org.mapstruct.Mapper;

// "componentModel = spring" говорит Спрингу сделать из интерфейса @Bean, чтобы мы могли использовать @Autowired
@Mapper(componentModel = "spring")
public interface UserMapper {

    // 1. Из DTO создать новенького Юзера (Entity) перед сохранением
    User toEntity(RegisterRequest request);

    // 2. Из загруженного из базы Юзера сделать безопасный DTO для фронтенда
    UserResponse toResponse(User user);
}
```

Всё! Больше ничего писать не нужно. MapStruct сам определит, что поле `email` в `RegisterRequest` нужно переложить в поле `email` объекта `User`.

---

### 3. Использование в Сервисе (Наслаждаемся чистотой кода)

Посмотрите, в насколько изящный код превратился наш Сервис:

```java
package com.example.service;

import com.example.dto.RegisterRequest;
import com.example.dto.UserResponse;
import com.example.model.User;
import com.example.mapper.UserMapper;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor // Lombok внедрит mapper через авто-генерацию конструктора
public class UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper; // Внедряем маппер как обычный сервис

    public UserResponse register(RegisterRequest request) {
        
        // 1. ОДНОЙ строкой переложили 20 полей! DTO -> Entity
        User newUser = userMapper.toEntity(request);
        
        // Тут можно задать технические системные поля:
        newUser.setRegistrationDate(LocalDateTime.now());
        
        // 2. Сохранили через репозиторий в БД (сгенерит id)
        User savedUser = userRepository.save(newUser);

        // 3. Снова ОДНОЙ строкой убрали всё лишнее и запаковали пароли. Entity -> DTO
        return userMapper.toResponse(savedUser);
    }
}
```

---

### 4. Сложный случай: Разные названия полей

Иногда поля называются по-разному. Например, Фронтенд присылает `firstName`, а в БД у вас поле называется `name`. MapStruct умеет перекладывать и такое:

```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    // Скажем MapStruct: "Возьми поле firstName из объекта DTO и положи в поле name объекта User"
    @Mapping(source = "firstName", target = "name")
    User toEntity(RegisterRequest request);
}
```

### 5. Интеграция с Lombok
Если вы используете `Lombok` (и мы настоятельно рекомендуем это делать), то в `pom.xml` в блоке `<annotationProcessorPaths>` вам придется добавить еще и процессор Ломбока: `lombok-mapstruct-binding`. Тогда MapStruct сможет видеть геттеры и сеттеры, сгенерированные Ломбоком за секунду до самого МапСтракта.
