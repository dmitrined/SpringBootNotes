# Security (Безопасность)

Spring Security — это мощный (и сложный) механизм для защиты вашего приложения.

### Роль
- **Аутентификация (Authentication)**: Кто ты? (Проверка логина/пароля).
- **Авторизация (Authorization)**: Что тебе можно делать? (Проверка ролей: ROLE_ADMIN, ROLE_TENANT).

---

### Основные компоненты

| Компонент | Роль |
| :--- | :--- |
| `SecurityFilterChain` | Сердце настроек. Здесь мы пишем, какие URL закрыты, а какие открыты. |
| `UserDetails` | Интерфейс, который должен реализовать ваш класс `User`, чтобы Spring Security его понимал. |
| `UserDetailsService` | Сервис, который ищет пользователя в базе по его имени (username). |
| `PasswordEncoder` | Утилита для хеширования паролей (обычно `BCryptPasswordEncoder`). |

---

### Аннотации

| Аннотация | Описание |
| :--- | :--- |
| `@EnableWebSecurity` | Включает поддержку безопасности в проекте. |
| `@Configuration` | Помечает класс как настроечный. |
| `@PreAuthorize` | Позволяет проверять права прямо над методом (например, `@PreAuthorize("hasRole('ADMIN')")`). |

---

### Логика работы цепочки
1. Запрос приходит на сервер.
2. Spring Security проверяет, есть ли токен или куки.
3. Если это `/api/auth/register`, запрос пропускается (permitall).
4. Если это закрытый путь, Security проверяет права пользователя.

---

### Пример настройки
```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

// ПРИМЕР КОНФИГУРАЦИИ
@Configuration
@EnableWebSecurity
@EnableMethodSecurity // Включает аннотации вроде @PreAuthorize
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable) // Отключаем CSRF (часто делают для REST API)
            // Настраиваем доступ к URL
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll() // Сюда можно всем (регистрация/логин)
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // Сюда только админам
                .anyRequest().authenticated() // На остальные нужны права (токен)
            )
            // JWT обычно означает REST API, поэтому отключаем сессии
            .sessionManagement(sess -> sess.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }

    // Бин для хеширования паролей, чтобы мы могли делать passwordEncoder.encode() в сервисах
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

// ПРИМЕР ИСПОЛЬЗОВАНИЯ В СЕРВИСЕ (Method Security)
@Slf4j
@Service
public class AdminService {

    // Выполнится только если у пользователя есть роль ADMIN
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        log.warn("Admin is attempting to delete user with id: {}", id); // Логируем важное действие!
        // логика удаления
    }
}

---

### Best Practices
1. **BCrypt**: Никогда не храните пароли в открытом виде! Пользуйтесь `BCrypt`.
2. **Stateless**: Для современных приложений (React/Angular) используйте JWT токены вместо сессий.
3. **Метод безопасности**: Используйте `@PreAuthorize` в сервисах для дополнительной защиты "на лету".
