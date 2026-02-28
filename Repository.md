# Слой Репозиторий (Repository)

### Роль
**Репозиторий** — это слой доступа к данным (DAO). Он инкапсулирует в себе всю работу с базой данных (SQL-запросы, поиск, сохранение).

**За что отвечает:**
- Выполнение CRUD операций (Create, Read, Update, Delete).
- Генерация SQL запросов на основе имен методов (Spring Data JPA).
- Выполнение сложных запросов с помощью `@Query`.

**Что делать КАТЕГОРИЧЕСКИ нельзя:**
- Писать бизнес-логику.
- Вызывать другие репозитории (репозиторий должен знать только о своей таблице).

---

### Аннотации

| Аннотация | Описание |
| `@Repository` | Указывает, что класс работает с БД. (Хотя при использовании JpaRepository её можно не писать, она ставится автоматически). |
| `@Query` | Позволяет написать свой SQL или JPQL запрос вручную, если возможностей стандартных методов не хватает. |
| `@Param` | Связывает переменную в методе с параметром в `@Query`. |
| `@Modifying` | Обязательная аннотация вместе с `@Query`, если ваш запрос изменяет данные (UPDATE или DELETE). |

---

### Связи
- Вызывается только из **слоя Service**.
- Работает напрямую с **базой данных** и **Entity (Model)**.

---

### Примеры кода
```java
package com.example.repository;

import com.example.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository // Эта аннотация опциональна для интерфейсов, наследующих JpaRepository, но полезна для читаемости
public interface UserRepository extends JpaRepository<User, Long> {

    // ПРИМЕР 1: Автоматическая генерация запроса Спрингом (Найдет юзера по email)
    Optional<User> findByEmail(String email);

    // ПРИМЕР 2: Автоматический запрос на существование (Вернет true/false)
    boolean existsByEmail(String email);

    // ПРИМЕР 3: Сложный кастомный запрос на JPQL (поиск по фрагменту email'а)
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")
    List<User> findUsersByEmailDomain(@Param("domain") String domain);

    // ПРИМЕР 4: Запрос, который ИЗМЕНЯЕТ данные (UPDATE/DELETE). Обязательна аннотация @Modifying!
    @Modifying
    @Query("UPDATE User u SET u.status = 'BANNED' WHERE u.id = :id")
    void banUser(@Param("id") Long id);
}
```

---

### Best Practices (Чистый код)
1. **Имена методов**: Используйте возможности Spring Data JPA (findBy..., existsBy..., countBy...). Это избавляет от написания лишнего SQL.
2. **Optional**: Всегда возвращайте `Optional<T>` для методов поиска по ID или уникальному полю, чтобы избежать `NullPointerException`.
3. **Не перегружайте**: Если запрос слишком сложный, возможно, его стоит вынести в отдельный `CustomRepository`.
