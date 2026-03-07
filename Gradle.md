# Gradle — Сборщик проекта (Build Tool) 🐘

**Gradle** — это современная система автоматизации сборки, которая сочетает в себе гибкость Ant и мощь управления зависимостями Maven. В Spring Boot он является стандартом де-факто вместе с Maven.

---

## 🛠 Ключевые файлы

| Файл | Описание |
| :--- | :--- |
| `build.gradle` | Главный скрипт сборки. Здесь описываются зависимости, плагины и задачи. |
| `settings.gradle` | Настройки структуры проекта (например, имя проекта и перечисление модулей). |
| `gradlew` / `gradlew.bat` | **Gradle Wrapper**. Скрипт для запуска Gradle без его предварительной установки на ПК. |
| `gradle/wrapper/` | Папка с самим исполняемым файлом (jar) и настройками версии Gradle. |

---

## 🚀 Gradle Wrapper (Почему это важно?)

Вы всегда должны использовать `./gradlew` вместо просто `gradle`.
*   **Консистентность**: Все разработчики и CI/CD сервер используют одну и ту же версию Gradle.
*   **Автоматизация**: Если Gradle не установлен, Wrapper скачает нужную версию сам.

```bash
# Пример запуска сборки через Wrapper
./gradlew build
```

---

## 🔄 Жизненный цикл сборки (Build Lifecycle)

Gradle работает в три этапа:

1.  **Initialization**: Определение, какие проекты участвуют в сборке (чтение `settings.gradle`).
2.  **Configuration**: Выполнение скриптов `build.gradle`, построение дерева задач (Task Graph).
3.  **Execution**: Выполнение только тех задач, которые нужны для текущего запроса (например, `compileJava`).

---

## 📦 Управление зависимостями (Dependencies)

В Gradle зависимости разделены по "конфигурациям" (областям видимости):

```groovy
dependencies {
    // Нужна для компиляции и работы приложения
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // Только во время выполнения (например, драйвер БД)
    runtimeOnly 'org.postgresql:postgresql'
    
    // Только для компиляции (библиотеки типа Lombok)
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    
    // Только для тестов
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

## 🔌 Плагины (Plugins)

Плагины расширяют возможности Gradle, добавляя новые задачи и конфигурации.

```groovy
plugins {
    id 'java' // Базовая поддержка Java
    id 'org.springframework.boot' version '3.2.2' // Плагин для Spring Boot (bootRun, bootJar)
    id 'io.spring.dependency-management' version '1.1.4' // Управление версиями без указания version в dependencies
}
```

---

## 🏛 Репозитории (Repositories)

Где Gradle ищет библиотеки. По умолчанию это Maven Central.

```groovy
repositories {
    mavenCentral() // Основной репозиторий Java сообщества
    mavenLocal()   // Ваш локальный кэш (~/.m2/repository)
}
```

---

## 📂 Пример стандартного build.gradle (Kotlin DSL)

Сейчас всё чаще используется `build.gradle.kts` (на базе Kotlin), что дает автодополнение и строгую типизацию.

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

---

## ⌨️ Популярные команды

*   `./gradlew clean` — Удаляет папку `build` (полная очистка).
*   `./gradlew build` — Полная сборка проекта + прогон тестов.
*   `./gradlew bootRun` — Запуск Spring Boot приложения.
*   `./gradlew test` — Запуск только тестов.
*   `./gradlew dependencies` — Показать дерево зависимостей (полезно при конфликтах).

---

### 💡 Maven vs Gradle: Что выбрать?
*   **Maven**: Строгий XML, декларативный подход, "стандарт" для больших консервативных компаний.
*   **Gradle**: Гибкий код (Groovy/Kotlin), работает **быстрее** за счет кэширования и инкрементальной сборки.
