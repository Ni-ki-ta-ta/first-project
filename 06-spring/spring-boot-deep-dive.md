# Spring Boot: подробный конспект

## Что добавляет Spring Boot

Spring Framework предоставляет контейнер и инфраструктуру. Spring Boot ускоряет сборку приложения: предлагает starters, автоконфигурацию, удобную конфигурацию и встроенный сервер. Boot не отменяет необходимости понимать Spring Core.

## `@SpringBootApplication`

Составная аннотация включает:
- `@SpringBootConfiguration` — специализированный вариант `@Configuration`;
- `@EnableAutoConfiguration` — механизм автоконфигурации;
- `@ComponentScan` — поиск компонентов в пакете главного класса и подпакетах.

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## Auto-configuration

Автоконфигурация настраивает приложение на основании classpath, существующих bean и свойств. Например, web starter позволяет Boot настроить MVC и встроенный сервер.

Я могу изучають condition evaluation report, исключить конкретную автоконфигурацию или переопределить поведение собственным bean. Автоконфигурация не угадывает архитектуру приложения: она применяет набор условных конфигураций.

## Starters

Starter — согласованный набор зависимостей для типового сценария:
- `spring-boot-starter-web` — Spring MVC и web-инфраструктура;
- `spring-boot-starter-data-jpa` — Spring Data JPA и ORM-интеграция;
- `spring-boot-starter-validation` — Bean Validation;
- `spring-boot-starter-security` — Spring Security;
- `spring-boot-starter-test` — тестовые библиотеки и интеграции.

Версии обычно управляются Spring Boot dependency management. Я избегаю случайного смешивания несовместимых версий.

## Конфигурация

```yaml
server:
  port: 8080

spring:
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/tasks}
    username: ${DB_USER:tasks}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate

app:
  page-size: 20
```

`${DB_URL:...}` означает: взять переменную окружения, а если её нет — значение после двоеточия. Реальные пароли я не храню в Git.

## Profiles и properties

Профили позволяют разделять настройки окружений: `application-dev.yml`, `application-test.yml`, `application-prod.yml`. Активный профиль можно задать через `spring.profiles.active`.

Для группы настроек использую `@ConfigurationProperties`: это типобезопаснее множества разрозненных `@Value`.

```java
@ConfigurationProperties(prefix = "app")
public record AppProperties(int pageSize) {}
```

Класс нужно зарегистрировать через `@ConfigurationPropertiesScan` или `@EnableConfigurationProperties`.

## Встроенный сервер и запуск

Boot обычно запускает встроенный servlet container, например Tomcat. Приложение можно собрать в исполняемый JAR:

```bash
./mvnw spring-boot:run
./mvnw package
java -jar target/app.jar
```

Первая команда запускает через Maven plugin, вторая собирает артефакт, третья запускает собранный JAR.

## Actuator

Actuator предоставляет endpoints для диагностики и мониторинга: health, info, metrics и другие. В production я открываю только необходимые endpoints и защищаю чувствительные сведения. Нельзя бездумно публиковать environment, beans и конфигурационные данные.

## Логирование

- `TRACE` — очень подробная диагностика;
- `DEBUG` — отладочная информация;
- `INFO` — значимые штатные события;
- `WARN` — потенциальная проблема;
- `ERROR` — ошибка, требующая внимания.

В логах не должно быть паролей, токенов, персональных данных и секретных payload. Полезно сохранять correlation ID и контекст операции.

## Практика

1. Создаю Boot-приложение с Web, Validation и Data JPA.
2. Настраиваю порт и datasource через YAML.
3. Делаю профили dev и test.
4. Переношу настройки в `@ConfigurationProperties`.
5. Добавляю Actuator health.
6. Изучаю condition evaluation report.
7. Собираю JAR и запускаю его отдельно от IDE.
