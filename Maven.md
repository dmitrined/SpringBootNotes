# Maven (Сборщик проекта)

**Maven** — это система автоматизации сборки и управления зависимостями для Java-проектов. Без него пришлось бы скачивать `.jar` файлы сторонних библиотек (Spring, Lombok, базы данных) вручную и закидывать их в проект. Maven делает это автоматически.

---

### 1. Как работает Maven?

1. Вы указываете нужную библиотеку (зависимость) в файле `pom.xml`.
2. При сборке проекта Maven смотрит в этот файл и ищет библиотеку на вашем компьютере (в папке `~/.m2/repository`).
3. Если её там нет, он скачивает её из глобального интернета (Maven Central).
4. После скачивания библиотек, Maven компилирует ваш код, прогоняет тесты и упаковывает всё в один готовый файл (`.jar` или `.war`).

---

### 2. Сердце проекта: pom.xml

Файл `pom.xml` (Project Object Model) лежит в самом корне проекта. В нем описано вообще всё о вашем приложении.

**Типичный пример для нашего Spring Boot проекта:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Spring Boot как "родительский" проект (отсюда берутся версии всех библиотек!) -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.5</version> <!-- Версия Spring Boot -->
        <relativePath/> 
    </parent>

    <!-- Информация о нашем проекте -->
    <groupId>com.example</groupId>
    <artifactId>user-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>user-api</name>
    <description>Demo project for Spring Boot User Management</description>

    <!-- Настройка версии Java -->
    <properties>
        <java.version>17</java.version>
    </properties>

    <!-- Зависимости (Библиотеки), которые мы подключаем -->
    <dependencies>
        <!-- Для создания REST контроллеров (@RestController, @GetMapping) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Для работы с базой данных (JPA / Hibernate) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- Драйвер для PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Спойлер: Lombok (чтобы не писать геттеры и сеттеры) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Библиотека для тестов (JUnit, Mockito) -->
        <!-- scope "test" означает, что библиотека не попадет в финальный собранный проект (продакшен) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Плагин, который запакует проект в исполняемый .jar файл -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

> [!TIP]
> **Где брать эти блоки XML?**  
> Вам не нужно писать их вручную. Вы всегда можете зайти на сайт **[MvnRepository](https://mvnrepository.com/)**, найти нужную библиотеку (например, `jjwt` для токенов) и просто скопировать оттуда блок `<dependency>`.

---

### 3. Жизненный цикл сборки (Lifecycle)

Когда вы запускаете сборку, Maven проходит несколько строгих этапов (фаз). Вы не можете запустить стадию `package` без запуска предыдущей `compile`.

1. **`validate`** — проверка, всё ли правильно в `pom.xml`.
2. **`compile`** — компиляция исходного Java-кода в байт-код (`.class` файлы).
3. **`test`** — прогон Unit-тестов (если тесты упадут, сборка остановится!).
4. **`package`** — упаковка скомпилированного кода в `.jar` архив.
5. **`verify`** — прогон интеграционных тестов и дополнительные проверки.
6. **`install`** — копирование вашего `.jar` в локальный репозиторий (`~/.m2`), чтобы его могли использовать другие проекты на вашем ПК.
7. **`deploy`** — отправка готового файла на удаленный сервер (Nexus/Artifactory) для других программистов.

Также есть отдельный цикл (не зависящий от сборки):
- **`clean`** — удаляет папку `target/` с результатами предыдущей сборки. Всегда делайте это перед новой сборкой!

---

### 4. Шпаргалка: Самые частые команды

Выполняются в терминале в корне проекта.

| Команда | Что делает | Когда использовать |
| :--- | :--- | :--- |
| `mvn clean` | Удаляет кэш и прошлые компиляции (папку `target`). | Каждый раз перед новой сборкой образа Docker. |
| `mvn compile` | Компилирует код. | Проверить, что нет синтаксических ошибок, без сборки `.jar`. |
| `mvn test` | Только запускает тесты. | Перед коммитом в Git (чтобы тесты не упали в Pipeline). |
| `mvn clean package` | Очищает старое, компилирует, тестирует и собирает `.jar`. | Самая частая команда! Создает готовый файл для запуска. |
| `mvn clean package -DskipTests` | Собирает `.jar`, но **игнорирует** выполнение тестов. | Когда нужно срочно собрать код (или если тесты падают), а проверить функционал хочется. |
| `mvn spring-boot:run` | Запускает само приложение прямо из терминала. | Альтернатива зеленой кнопке "Run" в IntelliJ IDEA. |

---

### 5. Что такое Maven Wrapper (`mvnw`)?

В корне современных Spring Boot проектов вы можете заметить файлы `mvnw` (для Linux/Mac) и `mvnw.cmd` (для Windows).

**Maven Wrapper** (Обертка Maven) — это скрипт. Он решает одну из главных проблем:
Вам (или другому разработчику в вашей команде) **вообще не обязательно** скачивать и устанавливать Maven на свой компьютер. 

Если вы напишете в терминале:
`./mvnw clean package`

Скрипт `mvnw` сам посмотрит, есть ли на ПК нужная версия Maven. Если нет — он в фоновом режиме её скачает, и через неё соберет проект. Это гарантирует, что Петя и Вася будут собирать проект **одинаковой версией Maven**.
