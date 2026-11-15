# Тестирование Spring-приложений

## Уровни тестов

- Unit test — тестирую класс изолированно, зависимости заменяю mock/fake.
- Slice test — поднимаю ограниченную часть Spring-контекста, например MVC.
- Integration test — проверяю взаимодействие нескольких компонентов и инфраструктуры.
- End-to-end test — проверяю систему через внешний интерфейс, близкий к пользовательскому сценарию.

Не каждый тест должен поднимать полный Spring Context: это медленнее и усложняет диагностику.

## Unit test сервиса

```java
@ExtendWith(MockitoExtension.class)
class TaskServiceTest {
    @Mock TaskRepository repository;
    @InjectMocks TaskService service;

    @Test
    void returnsTaskWhenItExists() {
        var task = new Task(1L, "Study");
        when(repository.findById(1L)).thenReturn(Optional.of(task));

        var result = service.getById(1L);

        assertEquals("Study", result.name());
        verify(repository).findById(1L);
    }
}
```

Я проверяю поведение, а не внутреннее устройство класса. Не делаю `verify` каждого вызова без необходимости.

## MVC slice test

`@WebMvcTest` поднимает ограниченный MVC-контекст. Через `MockMvc` проверяю маршруты, сериализацию, валидацию и статусы без запуска полноценного сервера и БД.

## Integration test и Testcontainers

`@SpringBootTest` поднимает контекст приложения. Для реальной БД использую Testcontainers, чтобы тестировать на той же СУБД, что и приложение.

H2 не всегда повторяет PostgreSQL. Для SQL, миграций и особенностей диалекта лучше использовать PostgreSQL-контейнер.

Testcontainers запускает зависимости в контейнерах: PostgreSQL, Kafka, Redis и другие. Это приближает среду к реальной, но требует Docker и увеличивает время тестов.

## Что проверяю

- успешный сценарий;
- отсутствующие данные;
- некорректный ввод;
- граничные значения;
- конфликт и дубликаты;
- права доступа;
- транзакционные эффекты;
- взаимодействие с реальной схемой БД.

## Практика

1. Пишу unit-тесты сервисного слоя.
2. Пишу MVC-тесты контроллера.
3. Проверяю валидацию и ErrorResponse.
4. Поднимаю PostgreSQL через Testcontainers.
5. Запускаю миграции в интеграционном тесте.
