# Быстрые шпаргалки Java Backend

Это краткая памятка для повторения. Если забыл смысл команды или ограничения — я открываю подробный файл по ссылке.

## Git: обычная работа

```bash
git status                         # что изменено?
git switch main                   # перейти на main
git pull --ff-only                 # обновить main без merge-коммита
git switch -c feature/my-task      # создать ветку и перейти в неё
git diff                           # посмотреть незастейдженные изменения
git add src/                       # подготовить изменения из src
git diff --staged                  # проверить то, что попадёт в коммит
git commit -m "Add task search"    # сохранить staged-изменения
git push -u origin feature/my-task # отправить ветку и задать upstream
```

`add` подготавливает, `commit` сохраняет локально, `push` отправляет коммиты на сервер. Подробности: [Git](01-foundations/git.md).

## Git: если что-то пошло не так

| Ситуация | Команда | Эффект |
|---|---|---|
| Убрать файл из staging | `git restore --staged file` | Файл остаётся изменённым локально |
| Отменить локальные изменения файла | `git restore file` | Незакоммиченные изменения файла теряются |
| Отменить опубликованный коммит | `git revert HASH` | Создаётся новый отменяющий коммит |
| Посмотреть историю | `git log --oneline --graph` | Компактная история веток |
| Временно убрать работу | `git stash push -m "wip"` | Сохранит незакоммиченные изменения |
| Посмотреть удалённые обновления | `git fetch origin` | Получить данные без слияния |

Перед опасной командой я сначала выполняю `git status` и я проверяю, что именно будет изменено.

## Maven

```bash
mvn clean          # удалить результаты предыдущей сборки
mvn compile        # скомпилировать основной код
mvn test           # запустить тесты
mvn package        # собрать JAR/WAR
mvn verify         # выполнить проверки жизненного цикла до verify
mvn dependency:tree # показать дерево зависимостей
```

Подробности: [Maven, Docker и CI/CD](10-infrastructure/maven-docker-ci.md).

## Docker

```bash
docker ps                    # запущенные контейнеры
docker ps -a                 # все контейнеры
docker logs -f app            # смотреть логи контейнера
docker compose up -d          # запустить compose-сервисы в фоне
docker compose down           # остановить и удалить контейнеры/сеть
docker compose down -v        # то же + удалить volumes (можно потерять данные!)
```

## SQL: базовый порядок запроса

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
WHERE active = true
GROUP BY department_id
HAVING COUNT(*) > 5
ORDER BY employee_count DESC;
```

- `WHERE` фильтрует строки до группировки.
- `GROUP BY` формирует группы.
- `HAVING` фильтрует группы; там можно использовать агрегаты и группировочные выражения.
- `ORDER BY` сортирует итог.

Помни: `COUNT(*)` считает строки, `COUNT(column)` не считает NULL. Для NULL я использую `IS NULL`, а не `= NULL`.

## Java: что я повторяють перед собеседованием

- `==` для примитивов сравнивает значения; для ссылок — идентичность объектов.
- `equals` — логическое равенство; контракт требует согласованного `hashCode`.
- `ArrayList` — динамический массив; `HashMap` — отображение ключ-значение; `HashSet` — уникальные значения.
- `? extends T` — чаще чтение; `? super T` — чаще запись (PECS).
- Checked exception нужно обработать или объявить; unchecked — нет.
- `volatile` обеспечивает видимость, но не делает `count++` атомарным.
- Stream промежуточные операции ленивые; после terminal operation поток нельзя использовать повторно.

## HTTP: основные статусы

| Код | Коротко |
|---|---|
| 200 | Успешный ответ |
| 201 | Ресурс создан |
| 204 | Успех без тела ответа |
| 400 | Некорректный запрос |
| 401 | Нет корректной аутентификации |
| 403 | Доступ запрещён |
| 404 | Не найдено |
| 409 | Конфликт состояния |
| 500 | Внутренняя ошибка сервера |

## Spring: типичный поток

`HTTP request → Filters → Controller → Service → Repository → Database`

Контроллер принимает/возвращает DTO, сервис содержит бизнес-правила, репозиторий работает с данными. Это типичная схема, а не единственно возможная архитектура.

Подробные темы: [Spring Boot](06-spring/spring-boot.md), [JPA](05-java-data-access/jdbc-jpa.md), [HTTP](07-web-api/http-rest.md).


---

## Spring: быстрые ориентиры

| Задача | Инструмент |
|---|---|
| Зарегистрировать компонент | `@Component`, `@Service`, `@Repository`, `@Controller` |
| Создать bean явно | `@Configuration` + `@Bean` |
| Внедрить зависимость | Constructor injection |
| Выбрать bean | `@Qualifier`, `@Primary` |
| Настроить свойства | `application.yml`, `@ConfigurationProperties` |
| Выбрать окружение | `spring.profiles.active` |
| Создать REST endpoint | `@RestController` + mapping-аннотации |
| Прочитать id из URL | `@PathVariable` |
| Прочитать query-параметр | `@RequestParam` |
| Прочитать JSON | `@RequestBody` |
| Проверить DTO | `@Valid` + Jakarta constraints |
| Обработать ошибки | `@RestControllerAdvice` + `@ExceptionHandler` |
| Управлять транзакцией | `@Transactional` |
| Создать JPA repository | `JpaRepository<Entity, IdType>` |
| Ограничить доступ | `SecurityFilterChain`, method security |
| Проверить health | Spring Boot Actuator |

### Maven Wrapper

```bash
./mvnw clean test
./mvnw spring-boot:run
./mvnw package
java -jar target/app.jar
```

- `clean` удаляет результаты предыдущей сборки из `target`.
- `test` компилирует и запускает тесты.
- `spring-boot:run` запускает Boot-приложение через Maven plugin.
- `package` собирает артефакт.
- `java -jar` запускает собранный JAR.


---

## Java concurrency: коротко

| Инструмент | Для чего |
|---|---|
| `synchronized` | Взаимное исключение + happens-before для монитора |
| `volatile` | Видимость и порядок, но не атомарность `x++` |
| `AtomicInteger` | Атомарные операции над числом |
| `ReentrantLock` | Явная блокировка, `tryLock`, interruptible wait |
| `ExecutorService` | Управление пулом потоков |
| `Future` | Получение результата асинхронной задачи |
| `CompletableFuture` | Цепочки асинхронных вычислений |
| `BlockingQueue` | Producer-consumer |
| `ConcurrentHashMap` | Конкурентная map |

## JVM: диагностика

```bash
jcmd -l
jcmd <pid> VM.version
jcmd <pid> GC.heap_info
jstack <pid>
```

## SQL: что помнить

- `WHERE` фильтрует строки до группировки; `HAVING` — группы.
- `COUNT(*)` считает строки; `COUNT(column)` не считает NULL.
- Для отсутствующих связанных строк часто подходит `NOT EXISTS`.
- `NOT IN` опасен, если подзапрос может вернуть NULL.
- Условие в `WHERE` по правой таблице может разрушить ожидаемое поведение `LEFT JOIN`.
- `UNION` удаляет дубликаты; `UNION ALL` сохраняет.

## PostgreSQL: перед оптимизацией

1. Воспроизвожу медленный запрос.
2. Смотрю `EXPLAIN (ANALYZE, BUFFERS)`.
3. Сравниваю estimated rows и actual rows.
4. Проверяю индексы, сортировки, join strategy и buffers.
5. Меняю одну вещь и сравниваю на сопоставимых данных.
