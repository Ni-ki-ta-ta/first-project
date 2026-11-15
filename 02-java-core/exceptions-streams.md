# Исключения, Lambda и Stream API

## Исключения
Исключение сообщает, что обычный сценарий выполнения не может продолжаться.

`Throwable` делится на `Error` и `Exception`; `RuntimeException` — разновидность unchecked Exception.
- Checked исключение компилятор требует обработать или объявить в `throws`.
- Unchecked исключение не требует этого на этапе компиляции.
- `Error` обычно обозначает серьёзную проблему среды выполнения.

```java
try {
    int result = 10 / value;
} catch (ArithmeticException e) {
    System.out.println("Делить на ноль нельзя");
}
```

Я не перехватываю исключение только ради того, чтобы программа «не падала». Я обрабатываю исключение там, где могу принять решение. Если оборачиваю ошибку, сохраняю исходную причину (`cause`).

### try-with-resources
Автоматически закрывает ресурсы, реализующие AutoCloseable.

```java
try (var reader = Files.newBufferedReader(path)) {
    return reader.readLine();
} catch (IOException e) {
    throw new UncheckedIOException("Не удалось прочитать " + path, e);
}
```

## Lambda
Лямбда — короткая реализация функционального интерфейса (интерфейса с одним абстрактным методом).

```java
Comparator<String> byLength =
    (a, b) -> Integer.compare(a.length(), b.length());
```

Method reference — ссылка на уже существующий метод:
```java
tasks.sort(Comparator.comparing(Task::getTitle));
```

## Stream API
Stream описывает обработку последовательности элементов.
1. Источник: `list.stream()`.
2. Промежуточные операции: `filter`, `map`, `sorted`, `distinct`.
3. Терминальная операция: `toList`, `collect`, `count`, `findFirst`, `forEach`.

```java
List<String> titles = tasks.stream()
    .filter(Task::isActive)
    .map(Task::getTitle)
    .sorted()
    .toList();
```

Промежуточные операции ленивые: выполняются при терминальной операции.
- `filter` оставляет подходящие элементы.
- `map` преобразует каждый элемент.
- `flatMap` объединяет потоки из элементов.
- `reduce` сводит к одному значению.
- `collect` собирает результат.

Я не меняю внешнее состояние внутри stream без необходимости. Я не использую parallelStream автоматически: параллельность может добавить накладные расходы.

## Иерархия исключений

`Throwable` — корень. Основные ветви: `Error` (обычно серьёзные проблемы JVM/среды) и `Exception`. `RuntimeException` — unchecked-подкласс `Exception`. Checked исключения нужно обработать или указать в `throws`; unchecked компилятор этого не требует.

## Собственные исключения

Создавай доменное исключение, когда оно помогает выразить понятную ошибку предметной области. Не создавай десятки типов без пользы. Не глотай причину:
```java
throw new IllegalStateException("Не удалось сохранить задачу", cause);
```

## Lambda и функциональные интерфейсы

Часто встречаются `Predicate<T>` (проверка true/false), `Function<T,R>` (преобразование), `Consumer<T>` (действие без результата), `Supplier<T>` (поставщик значения), `Comparator<T>` (сравнение).

## Основные Stream-операции

- `filter` — оставить элементы, прошедшие условие.
- `map` — преобразовать каждый элемент.
- `flatMap` — преобразовать элемент в поток и объединить потоки.
- `distinct` — убрать дубликаты по `equals`.
- `sorted` — отсортировать.
- `limit` / `skip` — ограничить/пропустить элементы.
- `anyMatch` / `allMatch` / `noneMatch` — проверить условие.
- `findFirst` — вернуть первый элемент как `Optional`.
- `collect` — собрать результат; `toList()` доступен с Java 16 и возвращает неизменяемый список.

Поток нельзя повторно использовать после терминальной операции. Stream сам по себе не хранит данные и обычно не меняет источник.

## Optional

`Optional<T>` явно показывает, что результата может не быть. Полезны `orElse`, `orElseGet`, `orElseThrow`, `map`, `ifPresent`. `orElse` вычисляет аргумент сразу; `orElseGet` вызывает Supplier только при пустом Optional. Я не использую Optional автоматически для каждого поля класса или параметра.

