# Конфигурация, Профили и `@Value`

В программировании есть одно главное правило безопасности: **НИКОГДА не хардкодить пароли, ключи и пути внутри Java-кода**.

Если у вас в `Service.java` написан `String password = "secret123";`, этот пароль уйдет в Github навсегда. Для управления настройками в Spring Boot используются файлы `application.properties` (или более современный `application.yml`).

---

### 1. `application.yml` вместо `.properties`

По умолчанию Spring создает текстовый файл `application.properties`. Но в реальных (enterprise) проектах почти 100% используют формат **YAML** (`.yml`). Он позволяет делать удобные "деревья" вложенности без постоянного повторения первых слов (как в properties).

**❌ Как выглядит .properties:**
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/user_db
spring.datasource.username=postgres
spring.datasource.password=1234
server.port=8080
```

**✅ Как выглядит .yml (нужно просто переименовать файл):**
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/user_db
    username: postgres
    password: 1234 # Пароли часто передают через переменные окружения ОС

server:
  port: 8080
```

---

### 2. Забираем настройки в код через `@Value`

Помимо системных настроек (БД, порт), вы можете создавать **свои собственные настройки**.
Например, в нашем FileUpload-сервисе мы писали путь папке `uploads/avatars` прямо в Java-классе. Давайте исправим это на профессиональный подход.

**Шаг 1: Добавляем кастомную настройку в `application.yml`**
```yaml
app:
  file:
    # Директория, куда мы будем загружать аватарки
    upload-dir: "/var/www/uploads/avatars"
    
  security:
    # Секретный ключ для подписи JWT-токенов
    jwt-secret: "my-super-secret-key-that-nobody-knows"
```

**Шаг 2: Внедряем эти значения в Java Класс через `@Value`**

Аннотация `@Value("${путь.в.yml}")` заставляет Spring на этапе запуска найти нужную строку в файле конфигурации и подставить её значение в переменную `String`.

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import jakarta.annotation.PostConstruct;

@Service
public class FileService {

    // Spring сам прочитает путь из .yml и подставит сюда строку
    // Если в yml настройки не будет, сервер не запустится! (и это хорошо)
    @Value("${app.file.upload-dir}")
    private String avatarUploadDirectory;
    
    // Но вы можете задать значение "по-умолчанию" через двоеточие:
    @Value("${app.file.max-size:5MB}") 
    private String maxFileSize; // Если настройки max-size нет в yml, возьмет '5MB'

    @PostConstruct // Вызовется сразу после того, как все @Value будут заполнены (после конструктора)
    public void init() {
        System.out.println("Сервис загрузки готов. Папка: " + avatarUploadDirectory);
    }
}
```

---

### 3. Профили (Profiles) — Dev, Test, Prod

Проблема: Когда вы пишете код дома, ваша база данных висит на `localhost:5432`. Когда вы деплоите код на рабочий сервер (Production), база данных висит на сервере `10.0.0.5` с совершенно другим паролем! 

*Вам что, перед каждым коммитом в Github переписывать пароли в `.yml`?* **Нет!**
Именно для этого существуют **Профили (Profiles).**

Spring позволяет создать несколько версий yml-файла со специальным суффиксом.

Обычно делают 3 файла:
1. `application.yml` (Глобальные настройки, общие для всех сред. Он же "главный", который говорит, какой профиль запустить).
2. `application-dev.yml` (Для вас, разработка на локальном компьютере, localhost).
3. `application-prod.yml` (Для боевого сервера).

#### Как это работает на практике

Создаем `application-dev.yml` (Домашняя среда):
```yaml
server:
  port: 8080 # У себя на компьютере запускаем на 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/my_pc_db
    username: user
    password: 123
  jpa:
    show-sql: true # Разработчику дома полезно видеть SQL в консоли
```

Создаем `application-prod.yml` (Боевая среда):
```yaml
server:
  port: 80 # На реальном сервере сайт висит на порту 80 (стандартный HTTP)

spring:
  datasource:
    url: jdbc:postgresql://192.168.1.10:5432/prod_db
    username: admin
    password: VeryStrongPassword123!
  jpa:
    show-sql: false # На боевом сервере нельзя "спамить" в логи запросами DDL/SQL
```

#### Как переключить?
В "главном" `application.yml` (или при запуске `.jar` через консоль) вы просто говорите Спрингу, в каком "режиме" (профиле) включиться.

**В главном `application.yml`:**
```yaml
spring:
  profiles:
    active: dev # Говорим запустить приложение в режиме DEV
```
Теперь Spring объединит настройки из `application.yml` и `application-dev.yml` (с приоритетом у `dev`). Когда вы отправите проект на сервер, DevOps инженер или Docker-скрипт просто запустит его командой:

`java -jar myapp.jar --spring.profiles.active=prod`

И приложение "прочитает" все боевые пароли и порты из `prod.yml`, не прикасаясь к `dev`-версии. Потрясающе удобно!
