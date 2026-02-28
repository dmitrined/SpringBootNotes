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
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Отключаем защиту для API
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll() // Публичные пути
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // Только для админов
                .anyRequest().authenticated() // Всё остальное — только после входа
            );
        return http.build();
    }
}

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
```

---

### Best Practices
1. **BCrypt**: Никогда не храните пароли в открытом виде! Пользуйтесь `BCrypt`.
2. **Stateless**: Для современных приложений (React/Angular) используйте JWT токены вместо сессий.
3. **Метод безопасности**: Используйте `@PreAuthorize` в сервисах для дополнительной защиты "на лету".
