# Миграции Базы Данных (Flyway / Liquibase)

До этого момента в наших заметках мы упоминали настройку `spring.jpa.hibernate.ddl-auto=update`.  
Она говорит Hibernate: «Эй, посмотри на мои Java `@Entity`. Если в базе нет таких таблиц или колонок — создай их сам».

**Почему это УЖАСНО в реальном Продакшене?**
Представьте: вы решили переименовать поле `username` в `login`. Hibernate увидит это и просто... **удалит колонку `username` вместе с миллионом данных всех ваших пользователей**, и затем создаст пустую колонку `login`. Ваш бизнес уничтожен.

В реальной разработке таблицы в БД **никогда не изменяются автоматически кодом Java**. 
Для этого используются системы управления Миграциями (Flyway или Liquibase).

---

### 1. Что такое Миграции?
Миграции — это "Система контроля версий (Git) для Базы Данных".
Вы пишете SQL-скрипты описывающие структурные изменения (например, создание новой таблицы или переименование колонки). Инструмент (Flyway или Liquibase) выполняет эти скрипты строго по порядку.

---

### 2. Как работает Flyway?

Мы создаем папку в ресурсах проекта `src/main/resources/db/migration`. 
Внутри нее мы кладем файлы с очень строгими названиями:

- `V1__init_schema.sql` (Создаем таблицы users и roles)
- `V2__add_avatar_to_users.sql` (Добавляем колонку)
- `V3__insert_admin.sql` (Заполняем первого админа)

**Что делает Flyway, когда поднимается Spring Boot?**
1. Он идет в вашу базу данных (PostgreSQL) и создает там техническую табличку `flyway_schema_history`. В ней он хранит список скриптов, которые он уже когда-либо запускал.
2. Flyway заходит в папку `db/migration` и сканирует файлы.
3. Он сверяет файлы с таблицей в базе. 
4. Ага! Файл `V1` уже был запущен год назад. Файл `V2` был месяц назад. А вот файл `V3` — новый!
5. Flyway выполняет SQL внутри `V3` и ставит галочку в `flyway_schema_history`, что скрипт успешно отработал.

*С этого момента вы не можете изменять файл `V1` или `V2`. Если Flyway увидит, что текст внутри старого файла поменялся (хэш-сумма не совпадает), он остановит запуск сервера с ошибкой "Validation failed". База Данных неприкосновенна!*

---

### 3. Как подключить Flyway

В классном мире Spring Boot это безумно просто.

**1. Добавляем плагин/зависимость в `pom.xml`:**
```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

**2. Насильно выключаем Hibernate-магию в `application.yml`:**
```yaml
spring:
  # Отключаем авто-генерацию таблиц из Entity. Теперь структура БД 100% ручная!
  jpa:
    hibernate:
      ddl-auto: validate # validate означает: "Только сверяет, совпадают ли Java классы с реальными таблицами"
    show-sql: true
    
  # Настроим Flyway
  flyway:
    enabled: true
    baseline-on-migrate: true # Если запускаем на уже существующей базе
```

**3. Пишем скрипт-миграцию SQL:**
Создаем файл: `src/main/resources/db/migration/V1__Create_user_table.sql`:

```sql
-- Прям чистый, настоящий SQL в зависимости от вашей БД (Postgres)
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL
);

-- Индекс для быстрого поиска при логине
CREATE INDEX idx_user_email ON users(email);
```

Файл `V2__Add_active_flag.sql` (через месяц):
```sql
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
```

---

### 5. Практический пример: Liquibase (XML подход)

Liquibase считается стандартом для больших Enterprise-проектов. Его главная сила в **XML-чейнджлогах**, которые позволяют описывать изменения базы данных в виде тегов.

#### Структура папок в `src/main/resources`:
```text
📂 db/changelog
 ├── db.changelog-master.xml   # Главный входной файл (Инжекс)
 └── 📁 changes                # Папка с конкретными изменениями
      ├── 01-create-users.xml
      └── 02-add-role-column.xml
```

**1. Подключение в `pom.xml`:**
```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

**2. Главный файл (db.changelog-master.xml):**
Этот файл просто собирает (импортирует) все остальные файлы с миграциями.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
    http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.9.xsd">

    <include file="db/changelog/changes/01-create-users.xml"/>
    <include file="db/changelog/changes/02-add-role-column.xml"/>

</databaseChangeLog>
```

**3. Пример миграции (01-create-users.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog ...>
    <changeSet id="1" author="dmitri">
        <createTable tableName="users">
            <column name="id" type="BIGINT" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="username" type="VARCHAR(100)">
                <constraints unique="true" nullable="false"/>
            </column>
            <column name="password" type="VARCHAR(255)">
                <constraints nullable="false"/>
            </column>
        </createTable>
    </changeSet>
</databaseChangeLog>
```

#### Почему XML лучше SQL в Liquibase?
1.  **Абстракция**: Вы пишете `<createTable>`, и Liquibase сам поймет, какой SQL сгенерировать для PostgreSQL, а какой для Oracle.
2.  **Валидация**: XML схема проверяет, что вы не допустили ошибок в названиях атрибутов.
3.  **Авто-откат (Pre-conditions)**: Liquibase может проверить условия (например, «существует ли колонка?») перед запуском.

**4. Настройка в `application.yml`:**
```yaml
spring:
  liquibase:
    enabled: true
    change-log: classpath:db/changelog/db.changelog-master.xml
```

---

### Итог: Что выбрать?

*   Выбирайте **Flyway**, если ваша команда любит старый добрый **SQL** и вы не планируете менять базу данных. Это быстро, просто и наглядно.
*   Выбирайте **Liquibase (XML)**, если вам нужна **максимальная мощь**, вы работаете в большой команде, где важна строгая структура XML, или если проект должен поддерживать разные типы СУБД.

**Помните главное:** Какую бы систему вы ни выбрали, всегда отключайте `spring.jpa.hibernate.ddl-auto=update` на реальных проектах! 🚀
