# Загрузка файлов (File Upload)

В веб-приложениях часто нужно загружать файлы (картинки, документы, PDF). В Spring Boot для этого есть мощный и удобный встроенный инструмент — интерфейс `MultipartFile`.

---

### 1. Как "летит" файл по HTTP

Почему файл обычно **не отправляют** как JSON?
Файл — это набор байтов (binary data), а JSON — текстовый формат. Перевод файла в строку (Base64) для отправки внутри JSON возможен, но он увеличивает размер файла примерно на 30%, замедляет работу и считается "антипаттерном" для больших файлов.

**Стандартный способ** — это тип запроса `multipart/form-data`.
В таком запросе тело (body) делится на отдельные части (parts). Одна часть может быть самим файлом, а другая — обычным текстом (например, описание файла или ID пользователя).

---

### 2. Что такое MultipartFile?

Когда клиент отправляет `multipart/form-data`, Spring автоматически перехватывает его и дает вам удобный объект `MultipartFile`.

**Важно:** `MultipartFile` — это не файл на вашем жестком диске. Это объект, который просто хранит информацию, переданную в текущем запросе (в оперативной памяти или во временной папке сервера).

| Основные методы `MultipartFile` | Что делает |
| :--- | :--- |
| `getOriginalFilename()` | Возвращает имя файла, которое было у клиента (например, `photo.png`). |
| `getContentType()` | Тип файла, например `image/png` или `application/pdf`. |
| `getSize()` | Размер файла в байтах. |
| `getInputStream()` | Поток байтов (Идеально для сохранения больших файлов, не забивая RAM). |
| `getBytes()` | Читает весь файл в массив `byte[]` (Опасно для гигантских файлов — может упасть с `OutOfMemoryError`). |

---

### 3. Где хранить файлы: 2 подхода

| Подход | Суть | Плюсы | Минусы | Примеры |
| :--- | :--- | :--- | :--- | :--- |
| **В ОС (File System / S3)** | Файл лежит в папке на сервере (`./uploads`), а в БД лежит только строка с путем к нему. | База не раздувается. Быстрая отдача файлов сервером (Nginx). | Нужно бэкапить и БД, и папку. Сложности с Docker volume. | Аватарки, фото товаров, большие PDF-договоры. |
| **В Базе Данных (BLOB)** | Файл конвертируется в массив байтов (`byte[]`) и ложится прямо в ячейку таблицы. | Всё в одном месте (удобный бэкап). Легкий контроль прав доступа. | База данных раздувается до гигантских размеров. Падает скорость. | Мелкие конфиденциальные сканы (паспорт). |

> [!TIP]
> **Золотое правило:** Большие файлы (фото, документы) — **всегда в файловую систему** (или облако S3). В БД храним только метаданные (путь, размер, владелец).

---

### 4. Важные правила безопасности (ОБЯЗАТЕЛЬНО!)

1. **Не доверяйте имени файла.** Хакер может назвать файл `../../etc/passwd` и перезаписать системные файлы ОС! Генерируйте уникальные имена сами (например, через `java.util.UUID`).
2. **Проверяйте тип (White-list).** Если ждете картинку, проверяйте что `getContentType()` равен `image/jpeg` или `image/png`. Если не проверить, вам могут загрузить скрипт `.sh` или `.exe`.
3. **Ограничьте размер!** Иначе сервер "упадет", если кто-то загрузит файл на 10 ГБ.
   Добавьте в `application.properties` (или `application.yml`):
   ```properties
   # Максимальный размер одного файла (например, 10 Мегабайт)
   spring.servlet.multipart.max-file-size=10MB
   # Максимальный размер всего запроса (если грузят сразу несколько файлов)
   spring.servlet.multipart.max-request-size=10MB
   ```

---

### 5. Как тестировать загрузку (в Postman)

1. Выберите HTTP-метод: **POST**
2. Перейдите во вкладку: **Body** -> **form-data**
3. В колонке `KEY` напишите `file` (название переменной). Сбоку от поля переключите тип с `Text` на `File`.
4. В колонке `VALUE` появится кнопка "Select Files" — выберите файл с компьютера.

---

### 6. Полный пример на Java (Сохранение в Файловую систему)

Продолжая нашу логику из системы **Пользователей**, давайте сделаем функционал загрузки аватарки профиля.

#### Сервис (FileService.java)
```java
package com.example.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;

import java.io.IOException;
import java.net.MalformedURLException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.UUID;

@Slf4j
@Service
public class FileService {

    // Папка, куда мы сохраняем (можно задать в application.properties)
    private final Path uploadDir = Paths.get("uploads/avatars");

    public FileService() {
        try {
            // Создаем папку, если её нет при старте
            Files.createDirectories(uploadDir);
        } catch (IOException e) {
            throw new RuntimeException("Could not create upload directory!");
        }
    }

    // Метод 1: ЗАГРУЗКА на сервер
    public String saveAvatar(MultipartFile file) {
        if (file.isEmpty()) {
            throw new IllegalArgumentException("Cannot upload empty file");
        }
        
        // 1. Генерируем безопасное уникальное имя (чтобы избежать конфликтов и взломов)
        String originalName = file.getOriginalFilename();
        String extension = originalName.substring(originalName.lastIndexOf("."));
        String uniqueFileName = UUID.randomUUID() + extension;

        try {
            // 2. Указываем финальный путь на диске
            Path destination = uploadDir.resolve(uniqueFileName);
            
            // 3. Копируем поток из памяти на жесткий диск
            Files.copy(file.getInputStream(), destination, StandardCopyOption.REPLACE_EXISTING);
            log.info("File saved beautifully to: {}", destination);
            
            // В реальном проекте мы сохранили бы uniqueFileName в сущность User (user.setAvatar(uniqueFileName))
            return uniqueFileName;
        } catch (IOException e) {
            throw new RuntimeException("Failed to store file", e);
        }
    }

    // Метод 2: СКАЧИВАНИЕ с сервера (отдача файла браузеру)
    public Resource loadAvatar(String filename) {
        try {
            Path file = uploadDir.resolve(filename);
            Resource resource = new UrlResource(file.toUri());

            if (resource.exists() || resource.isReadable()) {
                return resource;
            } else {
                throw new RuntimeException("Could not read the file!");
            }
        } catch (MalformedURLException e) {
            throw new RuntimeException("Error: " + e.getMessage());
        }
    }
}
```

#### Контроллер (FileController.java)
```java
package com.example.controller;

import com.example.service.FileService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@Tag(name = "File API", description = "Загрузка и скачивание файлов")
@RestController
@RequestMapping("/api/files")
@RequiredArgsConstructor
public class FileController {

    private final FileService fileService;

    // ПРИМЕР: Получаем файл по ключу "file" из form-data
    @Operation(summary = "Загрузить аватар пользователя")
    @PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<String> uploadFile(
            @RequestParam("file") MultipartFile file,
            @RequestParam("userId") Long userId // Можно принимать и обычные поля рядом с файлом!
    ) {
        // Здесь мы могли бы сначала достать User по userId и обновить его поле avatar...
        String savedFileName = fileService.saveAvatar(file);
        return ResponseEntity.ok("Файл успешно загружен. Имя: " + savedFileName);
    }

    // ПРИМЕР: Отдаем файл клиенту (например, для тега <img src="..."> на фронте)
    @Operation(summary = "Скачать/посмотреть аватар")
    @GetMapping("/download/{filename}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String filename) {
        Resource file = fileService.loadAvatar(filename);

        return ResponseEntity.ok()
                // Указываем браузеру, что это файл для скачивания (или inline для показа картинки)
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + file.getFilename() + "\"")
                // Либо contentType(MediaType.IMAGE_JPEG) чтобы браузер просто показал картинку
                .body(file);
    }
}
```
