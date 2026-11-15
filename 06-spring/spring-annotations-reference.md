# Справочник аннотаций Spring и Spring Boot

Аннотация сама по себе не выполняет магию: её должен обработать Spring, Hibernate, Jackson, Bean Validation или другой механизм.

## Регистрация компонентов и конфигурация

| Аннотация | Назначение |
|---|---|
| `@Component` | Регистрирует класс как bean при component scanning |
| `@Service` | Стереотип сервисного слоя |
| `@Repository` | Стереотип доступа к данным; участвует в переводе persistence-исключений |
| `@Controller` | MVC-контроллер |
| `@RestController` | `@Controller` + `@ResponseBody` |
| `@Configuration` | Класс конфигурации |
| `@Bean` | Регистрирует результат метода как bean |
| `@ComponentScan` | Настраивает поиск компонентов |
| `@Import` | Подключает конфигурацию |
| `@Profile` | Ограничивает регистрацию активным профилем |
| `@Primary` | Делает bean предпочтительным кандидатом |
| `@Qualifier` | Уточняет, какой bean внедрить |
| `@Lazy` | Откладывает создание bean до первого обращения, если применимо |

## Внедрение зависимостей и настройки

| Аннотация | Назначение |
|---|---|
| `@Autowired` | Внедрение зависимости; на единственном конструкторе обычно необязательна |
| `@Value` | Внедрение значения из property или expression |
| `@ConfigurationProperties` | Типобезопасная привязка группы настроек |
| `@PropertySource` | Подключение дополнительного property-файла |
| `@ConfigurationPropertiesScan` | Поиск классов настроек |
| `@EnableConfigurationProperties` | Регистрация указанных properties-классов |

## Жизненный цикл

| Аннотация | Назначение |
|---|---|
| `@PostConstruct` | Метод после внедрения зависимостей |
| `@PreDestroy` | Метод перед уничтожением bean |
| `@Scope` | Настройка scope |
| `@DependsOn` | Явное указание порядка инициализации |
| `@EventListener` | Обработчик события приложения |

## Spring MVC и REST

| Аннотация | Назначение |
|---|---|
| `@RequestMapping` | Общая привязка URL и параметров запроса |
| `@GetMapping` | Обработка GET |
| `@PostMapping` | Обработка POST |
| `@PutMapping` | Обработка PUT |
| `@PatchMapping` | Обработка PATCH |
| `@DeleteMapping` | Обработка DELETE |
| `@PathVariable` | Значение из пути |
| `@RequestParam` | Query-параметр |
| `@RequestBody` | Десериализация тела запроса |
| `@ResponseBody` | Запись результата в тело ответа |
| `@ResponseStatus` | HTTP-статус |
| `@RequestHeader` | Чтение HTTP-заголовка |
| `@CookieValue` | Чтение cookie |
| `@CrossOrigin` | Настройка CORS |
| `@RestControllerAdvice` | Глобальные советы для REST-контроллеров |
| `@ExceptionHandler` | Обработка выбранных исключений |

## Jakarta Bean Validation

| Аннотация | Назначение |
|---|---|
| `@Valid` | Запускает проверку ограничений объекта в поддерживаемом контексте |
| `@NotNull` | Значение не null |
| `@NotBlank` | Строка не null и содержит непробельный символ |
| `@NotEmpty` | Строка/коллекция/массив не пустые и не null |
| `@Size` | Ограничение размера |
| `@Min`, `@Max` | Числовые границы |
| `@Positive`, `@PositiveOrZero` | Положительное число или ноль и выше |
| `@Email` | Базовая проверка формата email |
| `@Pattern` | Проверка по регулярному выражению |

Ограничения обычно импортируются из `jakarta.validation.constraints`, а `@Valid` — из `jakarta.validation.Valid`. В старых проектах могут встречаться `javax.validation`.

## Транзакции и persistence

| Аннотация | Назначение |
|---|---|
| `@Transactional` | Декларативное управление транзакцией |
| `@Entity` | JPA-сущность |
| `@Table` | Настройка таблицы |
| `@Id` | Первичный ключ |
| `@GeneratedValue` | Стратегия генерации ключа |
| `@Column` | Настройка столбца |
| `@ManyToOne`, `@OneToMany` | Связи сущностей |
| `@OneToOne`, `@ManyToMany` | Другие типы связей |
| `@JoinColumn` | Настройка внешнего ключа |
| `@Enumerated` | Сохранение enum |
| `@Query` | Явный запрос Spring Data |
| `@Modifying` | Изменяющий запрос Spring Data JPA |

`@Transactional` часто работает через proxy. Self-invocation может обойти proxy. По умолчанию rollback обычно выполняется для unchecked exceptions; checked exceptions требуют отдельной настройки, например `rollbackFor`.

## Spring Security

| Аннотация | Назначение |
|---|---|
| `@EnableWebSecurity` | Подключение web security configuration в соответствующих конфигурациях |
| `@EnableMethodSecurity` | Включение method security |
| `@PreAuthorize` | Проверка доступа до метода |
| `@PostAuthorize` | Проверка доступа после метода |
| `@Secured` | Проверка authority/roles при включённой поддержке |
| `@WithMockUser` | Тестовый пользователь в Spring Security Test |

## Примеры

```java
@RestController
@RequestMapping("/tasks")
public class TaskController {
    @GetMapping("/{id}")
    public TaskDto get(@PathVariable long id) {
        return taskService.getById(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public TaskDto create(@Valid @RequestBody CreateTaskRequest request) {
        return taskService.create(request);
    }
}
```

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(long id) {
    // Операция доступна ADMIN.
}
```

## Важные оговорки

- Импорты важны: `jakarta.*` и старые `javax.*` не взаимозаменяемы.
- Часть аннотаций работает через proxy, поэтому важны правила вызова.
- Я не добавляю аннотации «на всякий случай»: сначала понимаю, какой механизм их обрабатывает.
- Поведение сверяю с документацией версии, используемой в проекте.
