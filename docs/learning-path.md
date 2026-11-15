# Порядок изучения

Я не пытаюсь учить всё одновременно. Для трудоустройства Java Backend сначала нужны крепкие основы Java, SQL, HTTP, Spring и тестирования.

## Этап 1. Инструменты и основы
Git, терминал, Java Core, ООП, исключения, коллекции, Generics, алгоритмы, Maven, отладчик IntelliJ.

**Результат:** самостоятельно написать небольшую Java-программу, покрыть её тестами, собрать Maven и отправить изменения через Git.

## Этап 2. Базы данных и backend
SQL, PostgreSQL, JOIN, группировки, оконные функции, индексы, транзакции, JDBC, HTTP, REST, Spring Boot, JPA/Hibernate.

**Результат:** REST API с PostgreSQL, валидацией, обработкой ошибок и тестами.

## Этап 3. Качество и эксплуатация
JUnit, Mockito, интеграционные тесты, Testcontainers, Docker, Linux, CI/CD, логи, метрики, безопасность.

**Результат:** сервис можно собрать, запустить, проверить и диагностировать.

## Этап 4. Углубление
JVM, Garbage Collector, многопоточность, архитектура, Redis, Kafka, распределённые системы, Kubernetes.

**Результат:** понимать компромиссы сложных решений и не добавлять технологии без необходимости.

## Приоритет для Junior Java Backend
1. Java Core и ООП.
2. Коллекции, Generics, Exceptions, Stream API.
3. SQL и PostgreSQL.
4. HTTP и REST.
5. Spring Boot и JPA.
6. JUnit и Mockito.
7. Git, Maven, Docker.
8. Алгоритмы, отладка и чтение чужого кода.

Я не перехожу к Kafka и Kubernetes, пока не могу уверенно написать обычный CRUD-сервис.

## Как распределять время

Для начального трудоустройства большую часть времени отдавай Java Core, SQL, Spring Boot, тестам и одному законченному проекту. Условный ориентир: 60–70% — писать код и решать задачи, 20–30% — разбирать теорию, остальное — повторять и готовиться объяснять решения. Это ориентир, а не обязательная норма.

Я перехожу дальше, когда могу самостоятельно решить небольшую практическую задачу по текущей теме, объяснить решение и отладить типичную ошибку.



## Отдельный маршрут по Spring

1. Spring Core: IoC, DI, bean, ApplicationContext, component scanning.
2. Жизненный цикл bean: scopes, init/destroy callbacks, BeanPostProcessor.
3. AOP: proxy, advice, self-invocation, транзакции.
4. Spring Boot: starters, auto-configuration, YAML, profiles, properties.
5. Spring MVC: DispatcherServlet, controller, DTO, HTTP mapping.
6. Validation и errors: Jakarta Validation, ControllerAdvice, ErrorResponse.
7. Data JPA: entity states, persistence context, relations, fetch, N+1, transactions.
8. Security: filter chain, authentication, authorization, password hashing, JWT/OAuth2.
9. Testing: unit, MVC slice, integration, Testcontainers.
10. Production: Actuator, logs, metrics, health checks, secrets.

Для каждого этапа я делаю небольшой кодовый пример, тесты и краткое объяснение своими словами. Этап считаю завершённым только после самостоятельной практики.
