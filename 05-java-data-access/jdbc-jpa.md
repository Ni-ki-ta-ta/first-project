# JDBC, JPA и Hibernate

## JDBC
JDBC — низкоуровневый API Java для выполнения SQL через соединение с БД.

```java
String sql = "SELECT id, name FROM users WHERE id = ?";
try (PreparedStatement ps = connection.prepareStatement(sql)) {
    ps.setLong(1, userId);
    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) {
            String name = rs.getString("name");
        }
    }
}
```
`?` — параметр. PreparedStatement отделяет SQL-код от значений и помогает защититься от SQL injection.

## Основные объекты
- Connection — соединение с БД.
- PreparedStatement — подготовленный SQL с параметрами.
- ResultSet — результат SELECT.
- SQLException — ошибка взаимодействия с БД.

Закрывай ресурсы через try-with-resources.

## Connection Pool
Пул (например, HikariCP) хранит ограниченное количество открытых соединений и выдаёт их приложению. Слишком большой пул может перегрузить БД.

## JPA и Hibernate
JPA — спецификация ORM для Java. Hibernate — популярная реализация JPA.
- Entity — объект, связанный с записью таблицы.
- Persistence Context — контекст управляемых сущностей.
- Dirty Checking — обнаружение изменений managed entity.
- Lazy Loading — связь загружается при обращении.
- Eager Loading — связь загружается сразу по выбранной стратегии.

## N+1
Один запрос получает список из 100 заказов, затем ORM выполняет отдельный запрос для клиента каждого заказа. Получается 101 запрос.

Обнаружение: SQL-логи и профилирование. Возможные решения: fetch join, EntityGraph, DTO projection — выбирай по сценарию.

## Правила
Я не отдаю Entity напрямую из REST-контроллера; я использую DTO. Я не делаю все связи EAGER. Не ставь CascadeType.ALL автоматически. Понимай границы транзакции и последствия Lazy Loading.

## SQL injection и PreparedStatement

Не склеивай пользовательский ввод в SQL:
```java
// Плохо: пользовательский ввод меняет текст SQL
String sql = "SELECT * FROM users WHERE name = '" + name + "'";
```
Я использую параметры `?` и `PreparedStatement`. Параметры подходят для значений, но не для имён таблиц/столбцов или направления сортировки: такие части выбирай из белого списка.

## JDBC: execute методы

- `executeQuery()` — для SELECT, возвращает `ResultSet`.
- `executeUpdate()` — для INSERT/UPDATE/DELETE и DDL; возвращает число затронутых строк (для DDL обычно 0).
- `execute()` — универсальный вариант, когда заранее неизвестен тип результата.

## JPA: жизненный цикл entity

Упрощённо entity может быть new/transient (ещё не управляется контекстом), managed (отслеживается), detached (отсоединена) или removed (помечена на удаление). Изменения managed entity в транзакции могут сохранится через dirty checking при flush/commit.

## Lazy Loading и транзакции

Lazy-связь может потребовать дополнительный SQL при чтении. Если обратиться к ней после закрытия persistence context, можно получить `LazyInitializationException` (в Hibernate). Планируй загрузку данных в транзакции и формируй DTO нужным запросом, а не включай EAGER повсюду.

## Cascade и orphanRemoval

Cascade передаёт операции от родителя связанным объектам. `orphanRemoval = true` удаляет зависимую сущность, когда она удалена из связи. Это мощные настройки: применяй только если жизненный цикл дочерней сущности действительно зависит от родителя.

## Транзакции и flush

`flush` синхронизирует изменения persistence context с БД, но сам по себе не означает COMMIT. Окончательный commit завершает транзакцию. Ошибка может проявиться при flush/commit, а не в строке `save`.

