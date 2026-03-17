# Model (Entity / Сущность)

**Entity** — это точное отражение таблицы в базе данных. Каждый объект этого класса — это строка в таблице.

---

### Аннотации Entity
| Аннотация | Зачем нужна |
| :--- | :--- |
| `@Entity` | Говорит JPA, что этот класс нужно привязать к таблице в БД. |
| `@Table(name = "...")` | Указывает точное имя таблицы (лучше писать во множественном числе). |
| `@Id` | Помечает первичный ключ (primary key). |
| `@GeneratedValue` | Настраивает автоинкремент (как будет создаваться ID). |
| `@Column` | Настройки колонки (имя, обязательность — nullable, уникальность). |
| `@ManyToOne`, `@OneToMany` | Настройка связей между таблицами. |
| `@Enumerated` | Используется для сохранения `Enum` (обязательно с `EnumType.STRING`, чтобы в базу писался текст, а не цифра). |

---

### Пример в коде

**Entity (User.java):**
```java
package com.example.model;

import jakarta.persistence.*;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.Builder;
import java.util.List;

@Entity
@Table(name = "users")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password; // Это поле НИКОГДА не должно попасть в DTO

    // ПРИМЕР: Использование перечислений (Enum)
    @Enumerated(EnumType.STRING) // Обязательно STRING, иначе в базе будет цифра
    @Column(nullable = false)
    private UserStatus status; // Например: ACTIVE, BANNED, DELETED

    // ПРИМЕР: Связь "Много к Одному" (Много пользователей могут иметь одну роль)
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "role_id")
    private Role role;

    // ПРИМЕР: Связь "Один ко Многим" (У одного пользователя много постов)
    @OneToMany(mappedBy = "user")
    private List<Post> posts;
}
```
