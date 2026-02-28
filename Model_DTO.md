# Model (Entity) и DTO

Эти два типа объектов часто путают, но у них абсолютно разные задачи.

---

## 1. Model (Entity / Сущность)
**Entity** — это точное отражение таблицы в базе данных. Каждый объект этого класса — это строка в таблице.

### Аннотации Entity
| Аннотация | Зачем нужна |
| :--- | :--- |
| `@Entity` | Говорит JPA, что этот класс нужно привязать к таблице в БД. |
| `@Table(name = "...")` | Указывает точное имя таблицы (лучше писать во множественном числе). |
| `@Id` | Помечает первичный ключ (primary key). |
| `@GeneratedValue` | Настраивает автоинкремент (как будет создаваться ID). |
| `@Column` | Настройки колонки (имя, обязательность — nullable, уникальность). |
| `@ManyToOne`, `@OneToMany` | Настройка связей между таблицами. |

---

## 2. DTO (Data Transfer Object)
**DTO** — это объект для перевозки данных. Он нужен только для того, чтобы передать данные от клиента к серверу (Request) или от сервера к клиенту (Response).

### Зачем нужен DTO?
1. **Безопасность**: Нельзя показывать клиенту пароли, хеши или секретные поля из Entity.
2. **Гибкость**: На фронтенде данные могут быть нужны в другом формате, чем они лежат в базе.
3. **Уменьшение трафика**: Передаем только те поля, которые реально нужны.

---

### Сравнение в коде

**Entity (User.java):**
```java
@Entity
@Table(name = "users")
@Data
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password; // Это поле НИКОГДА не должно попасть в DTO

    private String firstName;
    private String lastName;
}
```

**DTO (UserResponse.java):**
```java
@Data
public class UserResponse {
    private String email;
    private String fullName; // Мы объединили имя и фамилию для фронтенда
}
```

---

### Best Practices
1. **Разделяйте**: Никогда не используйте Entity в параметрах методов контроллера. Только DTO.
2. **Mapper**: Используйте библиотеки вроде MapStruct или просто создавайте статические методы `fromEntity` в DTO для конвертации.
3. **Именование**: Называйте их понятно: `UserCreateRequest`, `UserUpdateDto`, `UserResponse`.
