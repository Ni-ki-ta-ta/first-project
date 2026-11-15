# Spring Framework и Spring Boot

## IoC и DI
IoC (Inversion of Control) — создание и управление объектами передаётся контейнеру Spring.
DI (Dependency Injection) — зависимости передаются объекту извне, а не создаются им самостоятельно.

В большинстве случаев предпочитаю constructor injection:

```java
@Service
public class TaskService {
    private final TaskRepository repository;

    public TaskService(TaskRepository repository) {
        this.repository = repository;
    }
}
```
Так зависимости видны, а объект проще тестировать.

## Bean
Bean — объект, которым управляет Spring Container. Он создаётся через component scanning (`@Component`, `@Service`, `@Repository`, `@Controller`) или метод `@Bean` в конфигурации.

## Spring Boot
Boot упрощает настройку приложения:
- starters подключают согласованные группы зависимостей;
- auto-configuration настраивает компоненты по classpath и конфигурации;
- встроенный сервер позволяет запускать приложение без отдельной установки Tomcat.

Аннотации:
- `@SpringBootApplication` — точка входа и базовая конфигурация.
- `@RestController` — HTTP-контроллер, возвращающий тело ответа.
- `@Service` — сервисный компонент.
- `@Repository` — компонент доступа к данным.
- `@Configuration` / `@Bean` — явная конфигурация.
- `@ConfigurationProperties` — типобезопасное чтение настроек.

## Слои
Controller — HTTP-вход; Service — бизнес-правила; Repository — доступ к данным; Entity — модель хранения; DTO — контракт API. Это распространённый подход, а не обязательная структура любого приложения.

## @Transactional
Аннотация задаёт транзакционное поведение. В стандартной proxy-модели вызов метода из другого bean проходит через proxy; внутренний вызов метода того же объекта может обойти proxy. Поэтому self-invocation может не включить ожидаемую транзакцию.

Изучаю propagation, isolation и rollback rules. Секреты не коммить; настройки разделяй по профилям.

## Как проходит HTTP-запрос

В типичном MVC-приложении запрос проходит через servlet filters, затем DispatcherServlet, handler mapping, controller, service, repository и обратно через сериализацию ответа. Реальный путь зависит от конфигурации и подключённых фильтров.

## Валидация входных данных

Я использую Bean Validation: например, `@NotBlank`, `@Size`, `@Positive` на DTO и `@Valid` у параметра контроллера. Валидация на сервере обязательна, даже если клиент уже проверяет форму.

## Обработка ошибок

`@RestControllerAdvice` и `@ExceptionHandler` позволяют централизованно преобразовывать исключения в HTTP-ответы. Не возвращай клиенту stack trace, SQL или секреты. Сохраняй диагностический контекст в логах.

## Конфигурация

`application.properties` / `application.yml` задают настройки. Профили (`application-dev.yml`, `application-prod.yml`) позволяют различать окружения. Пароли и токены передавай через переменные окружения или secret store, я не храню реальные значения в Git.

## Bean scopes и lifecycle

По умолчанию Spring bean singleton в пределах ApplicationContext: один экземпляр на контейнер. Это не означает «один объект на всю JVM». Я не храню в singleton bean изменяемое состояние конкретного HTTP-запроса.

## @Transactional — частые ловушки

В proxy-режиме транзакция обычно включается при вызове через Spring proxy. Self-invocation (вызов метода того же объекта через `this`) может обойти proxy. По умолчанию runtime exceptions и errors приводят к rollback; checked exceptions по умолчанию обычно не приводят, если не настроить `rollbackFor`. Проверяй правила конкретной версии и конфигурации.


## Spring Data Repository

Spring Data JPA позволяет объявить интерфейс репозитория, например `JpaRepository<User, Long>`, и получить типовые операции без ручного написания SQL для каждого CRUD-сценария. Методы вида `findByEmail(...)` могут выводить запрос из имени метода. Для сложных запросов я использую `@Query`, DTO projection или отдельный custom repository — я не пытаюсь уместить сложную бизнес-логику в длинное имя метода.

## DTO и маппинг

Входной DTO (request) и выходной DTO (response) могут отличаться. Не принимай Entity напрямую из JSON: клиент не должен управлять служебными полями и связями persistence-модели. Маппинг можно делать вручную или библиотекой, но правила преобразования должны оставаться понятными.
