# Spring MVC, валидация и обработка ошибок

## DispatcherServlet

Spring MVC использует front controller. Центральный компонент `DispatcherServlet` принимает запрос и организует поиск подходящего обработчика, связывание параметров и формирование ответа.

## Controller и REST Controller

`@Controller` часто используется с представлениями. `@RestController` подходит для REST API: результат метода записывается в тело ответа.

```java
@RestController
@RequestMapping("/api/tasks")
public class TaskController {
    private final TaskService service;

    public TaskController(TaskService service) {
        this.service = service;
    }

    @GetMapping("/{id}")
    public TaskDto get(@PathVariable long id) {
        return service.getById(id);
    }
}
```

## Источники параметров

- `@PathVariable` — часть URL, например `/tasks/42`.
- `@RequestParam` — query string, например `/tasks?status=DONE`.
- `@RequestBody` — тело запроса, например JSON.
- `@RequestHeader` — HTTP-заголовок.
- `@CookieValue` — cookie.

## DTO

DTO — объект передачи данных через границу приложения. Я не отдаю JPA Entity напрямую клиенту: в entity могут быть внутренние поля, lazy-связи и детали хранения.

```java
public record CreateTaskRequest(
    @NotBlank @Size(max = 200) String name,
    @Positive long assigneeId
) {}

public record TaskDto(long id, String name, String status) {}
```

## Валидация

`spring-boot-starter-validation` подключает Jakarta Bean Validation.

```java
@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public TaskDto create(@Valid @RequestBody CreateTaskRequest request) {
    return service.create(request);
}
```

Для вложенных объектов я использую `@Valid` на соответствующем поле. Проверка на клиенте не заменяет серверную валидацию. Бизнес-правила, зависящие от БД, требуют отдельной проверки и ограничений БД.

## Обработка ошибок

Часто применяемые статусы:
- 400 — некорректный ввод;
- 404 — ресурс не найден;
- 409 — конфликт состояния или уникальности;
- 401 — отсутствует/неверна аутентификация;
- 403 — недостаточно прав;
- 500 — неожиданная внутренняя ошибка.

```java
@RestControllerAdvice
public class ApiExceptionHandler {
    @ExceptionHandler(TaskNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(TaskNotFoundException ex) {
        return new ErrorResponse("TASK_NOT_FOUND", ex.getMessage());
    }
}

public record ErrorResponse(String code, String message) {}
```

Я не возвращаю клиенту stack trace, SQL, пути файлов и секреты. Подробности остаются в логах.

## Filters и Interceptors

Servlet Filter работает на уровне servlet container до и после DispatcherServlet. HandlerInterceptor интегрирован со Spring MVC и работает вокруг вызова controller handler. Они не взаимозаменяемы.

Для аутентификации и авторизации я использую Spring Security, а не самописный interceptor.

## HTTP-методы

- GET — получить ресурс; безопасный и идемпотентный.
- POST — создать ресурс или выполнить действие; обычно не идемпотентен.
- PUT — заменить ресурс; по смыслу идемпотентен.
- PATCH — частично изменить ресурс; идемпотентность зависит от семантики.
- DELETE — удалить ресурс; по смыслу идемпотентен.

## Практика

1. Создаю CRUD API задач.
2. Выношу request/response в DTO.
3. Добавляю валидацию.
4. Создаю исключения для 404 и 409.
5. Добавляю единый формат ошибок.
6. Проверяю статусы через Postman и интеграционные тесты.
