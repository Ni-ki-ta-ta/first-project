# Spring Data JPA и Hibernate

## Что чем является

- JPA — спецификация Java для ORM.
- Hibernate — популярная реализация JPA.
- Spring Data JPA — Spring-проект, упрощающий работу с JPA repository.
- Entity — объект, отображаемый на запись/таблицу БД.
- Persistence Context — контекст управления entity, обычно связанный с EntityManager.

## Entity и Repository

```java
@Entity
@Table(name = "tasks")
public class Task {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String name;
}

public interface TaskRepository extends JpaRepository<Task, Long> {
    List<Task> findByStatus(TaskStatus status);
    boolean existsByNameIgnoreCase(String name);
}
```

Для сложных запросов использую `@Query`, projections или custom repository. Схему production-БД обычно контролирую миграциями, а не автоматическим созданием Hibernate.

## Persistence Context и dirty checking

Persistence Context отслеживает загруженные entity. Если managed entity изменить внутри транзакции, Hibernate может обнаружить изменения и отправить UPDATE при flush/commit. Это dirty checking.

## Состояния entity

- transient — новый объект, ещё не управляется persistence context;
- managed — отслеживается текущим контекстом;
- detached — ранее управляемый объект, сейчас вне контекста;
- removed — помечен на удаление.

## Связи

- `@ManyToOne` — многие записи ссылаются на одну;
- `@OneToMany` — одна запись связана со многими;
- `@OneToOne` — один к одному;
- `@ManyToMany` — многие ко многим, часто через join table.

В двунаправленной связи важно определить owning side — сторону, которая управляет внешним ключом. Изменение только inverse side может не обновить связь в БД.

## LAZY и EAGER

`LAZY` откладывает загрузку связи до обращения, если это поддерживается провайдером. `EAGER` требует загрузить связь, но не гарантирует один конкретный SQL JOIN.

Я не делаю все связи EAGER. Вместо этого выбираю данные под сценарий: fetch join, entity graph, projection или отдельный запрос.

## N+1

N+1 возникает, когда сначала выполняется запрос списка, а затем дополнительные запросы для каждой связанной записи.

Исправления могут включать `JOIN FETCH`, `@EntityGraph`, DTO projection, batch fetching или изменение сценария чтения. Нельзя бездумно добавлять fetch join коллекций к пагинации: это может привести к дубликатам и проблемам с пагинацией.

## Транзакции

Границу транзакции обычно задаю на сервисном use case через `@Transactional`, чтобы несколько операций репозитория были атомарными.

Изучаю propagation, isolation, readOnly и rollback rules. `readOnly` — подсказка инфраструктуре, а не универсальная гарантия запрета записи.

## Миграции

Для контролируемого изменения схемы использую Flyway или Liquibase. Миграции хранятся в Git и применяются последовательно. Уже применённую миграцию не редактирую без согласованной процедуры.

## Практика

1. Создаю User и Task со связью Many-to-One.
2. Пишу repository и derived query methods.
3. Добавляю пагинацию и сортировку.
4. Нахожу N+1 по SQL-логам.
5. Исправляю запрос через EntityGraph или projection.
6. Добавляю Flyway migration.
7. Тестирую rollback и ограничения БД.
