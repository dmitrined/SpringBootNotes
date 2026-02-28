# Отправка E-mail (JavaMailSender)

Во многих реальных системах отправка email используется для уведомлений (подтверждение регистрации, смена статуса заказа, сброс пароля). Это важная **интеграция с внешним миром**, а значит — потенциальный источник задержек и ошибок.

---

### 1. Как бэкенд отправляет письма (SMTP)
Бэкенд-приложение (Spring Boot) **не рассылает** письма самостоятельно. Оно работает как клиент и подключается к **SMTP-серверу** (Simple Mail Transfer Protocol) — например, к серверам Google (Gmail) или корпоративному почтовику.

Процесс:
1. Приложение открывает соединение с SMTP-сервером.
2. Проходит аутентификацию (логин/пароль из `application.properties`).
3. Передаёт письмо (от кого, кому, тема, тело).
4. SMTP-сервер уже доставляет письмо адресату.

---

### 2. Ключевые инструменты Spring Boot

В Spring Boot отправка реализуется через интерфейс **`JavaMailSender`**. Spring сам читает настройки SMTP из `application.properties` и создает этот бин.

**Почему логика писем всегда в `@Service`?**
Отправка email — это бизнес-операция (и внешний побочный эффект). Ей **не место в Контроллере**.
- Контроллер принимает HTTP-запрос, валидирует DTO и отдает ответ.
- Сервис содержит сложную логику, обращается к базе данных и отправляет письмо. Это делает код тестируемым и чистым.

---

### 3. Plain Text vs HTML (Зачем нужен Thymeleaf?)

Можно отправить письмо простым текстом (Plain Text). Но это выглядит некрасиво и устаревше (нельзя добавить кнопки, жирный шрифт, стили).
В реальных проектах стандартом является **HTML-письмо**.

**❌ Как не надо делать (писать HTML строками в Java):**
```java
String html = "<html><body><h1>Привет, " + name + "!</h1></body></html>";
```
Это нечитаемо и невозможно поддерживать.

**✅ Правильный подход (Использование шаблонизатора Thymeleaf):**
Мы создаем отдельный `.html` файл в ресурсах. В Java мы передаем данные в объект `Context`, а **Thymeleaf** сам подставляет эти данные в HTML-разметку (через `th:text`, `th:href` и т.д.).

Для отправки HTML писем (или писем со вложениями) обычного текста недостаточно. Здесь на помощь приходит **`MimeMessage`** и утилита **`MimeMessageHelper`**, которая позволяет удобно задать кодировку (UTF-8) и сказать почтовому клиенту, что внутри именно HTML.

---

### 4. Золотые правила работы с Email (Архитектура и Тесты)

1. **Обязательно логируйте процессы!** SMTP может быть недоступен, пароль может измениться. Без логов (`log.info` и `log.error`) вы никогда не узнаете, почему пользователь не получил письмо сброса пароля.
2. **Принимайте DTO, а не Entity.** В API для отправки почты нужно передавать DTO (например, `EmailRequest`), а не тянуть сущности из базы.
3. **Не шлите реальные письма в авто-тестах!** Тесты должны работать быстро и без интернета. В интеграционных тестах мы используем фейковые SMTP-серверы (например, GreenMail) или мокаем (`@MockBean`) сам `EmailService`. Тест должен проверить только то, что *попытка* отправки была вызвана.

---

### 5. Полный практический пример (Регистрация + Welcome Email)

Допустим, после создания учетной записи (как мы делали в предыдущих примерах), мы хотим отправить пользователю красивое приветственное письмо.

#### 5.1. Шаблон письма (src/main/resources/templates/emails/welcome.html)
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Добро пожаловать!</title>
</head>
<body style="font-family: Arial, sans-serif; color: #333; line-height: 1.6;">
    <h2 style="color: #0056b3;">Привет, <span th:text="${userName}">Имя</span>!</h2>
    <p>Спасибо за регистрацию в нашей системе. Ваш логин:</p>
    <p><strong th:text="${userEmail}">email@example.com</strong></p>
    <br>
    <a href="https://example.com/login" style="background: #28a745; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px;">
        Войти в аккаунт
    </a>
</body>
</html>
```

#### 5.2. Сервис для писем (EmailService.java)
```java
package com.example.service;

import jakarta.mail.MessagingException;
import jakarta.mail.internet.MimeMessage;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@Slf4j // Обязательно для внешних интеграций
@Service
@RequiredArgsConstructor
public class EmailService {

    private final JavaMailSender mailSender; // Работает с SMTP
    private final TemplateEngine templateEngine; // Thymeleaf для рендера HTML

    // ПРИМЕР: Метод отправки приветственного письма (Вызывается после регистрации)
    public void sendWelcomeEmail(String toEmail, String userName) {
        log.info("Начало отправки приветственного письма на адрес: {}", toEmail);

        try {
            // 1. Подготавливаем данные для шаблона
            Context context = new Context();
            context.setVariable("userName", userName);
            context.setVariable("userEmail", toEmail);

            // 2. Рендерим HTML из файла welcome.html
            String processHtml = templateEngine.process("emails/welcome", context);

            // 3. Создаем "сложное" MIME сообщение (поддерживает HTML)
            MimeMessage mimeMessage = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true, "UTF-8");

            helper.setTo(toEmail);
            helper.setSubject("Добро пожаловать в нашу систему!");
            helper.setText(processHtml, true); // true означает, что это HTML, а не plain text

            // 4. Отправляем через SMTP
            mailSender.send(mimeMessage);
            log.info("Письмо успешно отправлено на адрес: {}", toEmail);

        } catch (MessagingException e) {
            // Если SMTP лежит или логин неверный - перехватываем и логируем как ERROR!
            log.error("Ошибка при отправке письма на {}: {}", toEmail, e.getMessage(), e);
            throw new RuntimeException("Не удалось отправить email", e);
        }
    }
}
```

#### 5.3. Использование в Основном Сервисе (UserService.java)
```java
package com.example.service;

import com.example.dto.RegisterRequest;
import com.example.dto.UserResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserService {

    // ... Репозитории и шифраторы паролей (как в предыдущих примерах)
    private final EmailService emailService; // Внедряем наш сервис писем

    public UserResponse register(RegisterRequest request) {
        // 1. Валидация, сохранение в базу данных
        // userRepository.save(user);

        // 2. Отправка email как побочный эффект бизнес-логики
        // (Лучше делать это асинхронно через @Async или очереди очередей вроде RabbitMQ, 
        // чтобы пользователь не ждал 3 секунды, пока SMTP сервер ответит)
        emailService.sendWelcomeEmail(request.getEmail(), request.getUsername());

        return new UserResponse(request.getEmail(), request.getUsername());
    }
}
```
