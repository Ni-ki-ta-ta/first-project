# Spring Core: подробный конспект

## IoC и DI

IoC (Inversion of Control) — принцип, при котором управление созданием и связыванием объектов передаётся контейнеру. DI (Dependency Injection) — способ реализации IoC: объект получает зависимости извне.

```java
@Service
public class TaskService {
    private final TaskRepository repository;

    public TaskService(TaskRepository repository) {
        this.repository = repository;
    }
}
```

Я предпочитаю constructor injection: обязательные зависимости видны, поля можно сделать `final`, а в тесте легко передать mock или fake.

## Bean и контейнер

Bean — объект, которым управляет Spring IoC Container. `BeanFactory` — базовый контейнерный интерфейс; `ApplicationContext` добавляет события, интернационализацию и интеграцию с инфраструктурой приложения.

Bean регистрируется через component scanning (`@Component`, `@Service`, `@Repository`, `@Controller`) или метод `@Bean` в конфигурации.

```java
@Configuration
public class AppConfig {
    @Bean
    Clock clock() {
        return Clock.systemUTC();
    }
}
```

`@Bean` особенно полезна для объектов сторонних библиотек, которые я не могу пометить `@Component`.

## Component scanning

`@ComponentScan` задаёт пакеты для поиска компонентов. `@SpringBootApplication` включает поиск от пакета главного класса и его подпакетов. Поэтому главный класс приложения обычно располагаю в корневом пакете.

## Bean scopes

- `singleton` — один экземпляр на Spring-контейнер; scope по умолчанию.
- `prototype` — новый экземпляр при каждом запросе bean из контейнера.
- `request` — экземпляр на HTTP-запрос.
- `session` — экземпляр на HTTP-сессию.
- `application` — экземпляр на ServletContext.

Singleton — не «один объект на всю JVM». Я не храню в singleton-сервисе изменяемое состояние конкретного запроса: это может вызвать гонки между потоками.

Если prototype внедрить в singleton обычным способом, зависимость обычно будет получена при создании singleton. Для получения нового prototype при каждом обращении можно использовать `ObjectProvider`.

## Жизненный цикл bean

Упрощённо:
1. Spring создаёт экземпляр.
2. Внедряет зависимости.
3. Вызывает aware-callbacks, если они реализованы.
4. Вызывает BeanPostProcessor до и после инициализации.
5. Выполняет init-callback (`@PostConstruct`, `InitializingBean` или initMethod).
6. Bean готов к использованию.
7. При закрытии контекста выполняется destroy-callback (`@PreDestroy`, `DisposableBean` или destroyMethod).

## AOP и proxy

AOP позволяет вынести повторяющееся поведение: транзакции, безопасность, логирование и метрики. Spring часто реализует AOP через proxy: JDK dynamic proxy или CGLIB.

Вызов метода через proxy может запустить advice. Внутренний вызов `this.someMethod()` обходит внешний proxy, поэтому `@Transactional` на таком внутреннем вызове может не сработать ожидаемым образом. Ограничения зависят от вида proxy; например, CGLIB не может переопределить final-метод.

## События

События через `ApplicationEventPublisher` и `@EventListener` помогают ослабить связанность компонентов. Но обычное событие внутри приложения не заменяет надёжную очередь: оно не гарантирует доставку после падения процесса.

## Частые ошибки

- Создавать зависимости через `new` внутри Spring-сервисов и обходить контейнер.
- Использовать field injection там, где лучше подходит конструктор.
- Хранить изменяемое состояние запроса в singleton.
- Размещать главный класс Boot вне корневого пакета компонентов.
- Ожидать, что self-invocation пройдёт через proxy.
- Считать все стереотипы полностью одинаковыми: они регистрируют компоненты, но отражают разные роли; `@Repository` также участвует в переводе persistence-исключений.

## Практика

1. Создаю интерфейс `TaskRepository` и две реализации.
2. Регистрирую одну через `@Component`, вторую через `@Bean`.
3. Выбираю реализацию через `@Primary` и `@Qualifier`.
4. Получаю несколько prototype-bean через `ObjectProvider`.
5. Проверяю порядок lifecycle-callbacks.
6. Объясняю, почему self-invocation может обойти транзакционный proxy.
