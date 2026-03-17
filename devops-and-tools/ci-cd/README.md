# CI/CD ⚙️

Continuous Integration (CI) и Continuous Deployment (CD) — практики автоматизации проверки качества кода и его доставки в Production.

## Содержание

### Концепции

* **CI (Continuous Integration)** — Разработчик отправляет код → автоматически запускаются тесты и сборка. Ошибки обнаруживаются немедленно, а не перед релизом.
* **CD (Continuous Delivery)** — После успешного CI, артефакт (JAR, Docker-образ) готов к деплою. Деплой запускается вручную.
* **CD (Continuous Deployment)** — Автоматический деплой в Production после прохождения всех проверок.

### GitHub Actions

Это встроенный CI/CD инструмент GitHub, использующий YAML-файлы для описания пайплайна.

#### Структура файла (`.github/workflows/ci.yml`)
```yaml
name: Java CI Pipeline # Название пайплайна

on: # Когда запускать
  push:
    branches: [ "main", "develop" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build-and-test: # Название задания
    runs-on: ubuntu-latest # Среда выполнения (виртуальная машина)

    steps:
      - name: Checkout code # 1. Получить код из репозитория
        uses: actions/checkout@v4

      - name: Set up JDK 17 # 2. Установить Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build with Gradle # 3. Собрать и протестировать
        run: ./gradlew build

      - name: Build Docker image # 4. Собрать Docker-образ
        run: docker build -t my-app:latest .

      - name: Push to Docker Hub # 5. Загрузить образ в Registry
        if: github.ref == 'refs/heads/main' # Только из main-ветки
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push my-app:latest
```

#### Ключевые концепции GitHub Actions
* **Workflow** — Автоматизированный процесс (файл `.yml`).
* **Job** — Набор шагов (steps), выполняемых на одном Runner.
* **Step** — Одно действие (команда оболочки или `uses:` actions).
* **Action** — Переиспользуемый модуль (напр., `actions/checkout@v4`).
* **Secrets** — Зашифрованные переменные (`${{ secrets.API_KEY }}`). Задаются в настройках репозитория.
* **Artifacts** — Файлы (`.jar`, отчёты тестов), сохраняемые между заданиями (`upload-artifact`).

### Другие CI/CD платформы
* **Jenkins** — Мощный self-hosted CI/CD сервер (Jenkinsfile, Groovy-скрипты). Требует инфраструктуры.
* **GitLab CI** — Встроен в GitLab. Конфигурация через `.gitlab-ci.yml`.
* **CircleCI / TeamCity** — Облачные и гибридные решения.

---

[⬅️ Назад к Linux](../linux/README.md) | [Вернуться к DevOps Index ➡️](../README.md)