# Generics, лямбды и Optional

## Generics

Generics позволяют описывать типизированные классы и методы без дублирования кода.

```java
List<String> names = new ArrayList<>();
names.add("Nikita");
// names.add(42); // ошибка компиляции
```

В Java generics в основном реализованы через type erasure: большая часть информации о параметре типа стирается при компиляции. Поэтому нельзя обычным способом написать `new T()` или `new T[]`.

## Инвариантность и PECS

`List<Integer>` не является подтипом `List<Number>`, хотя `Integer` — подтип `Number`.

- Producer Extends: если структура отдаёт значения типа T, использую `? extends T`.
- Consumer Super: если структура принимает значения типа T, использую `? super T`.

```java
static double sum(List<? extends Number> values) {
    return values.stream().mapToDouble(Number::doubleValue).sum();
}
static void addDefaults(List<? super Integer> values) {
    values.add(1);
    values.add(2);
}
```

## Lambda и functional interface

Лямбда — реализация функционального интерфейса, то есть интерфейса с одним абстрактным методом.

```java
Predicate<String> notEmpty = value -> !value.isEmpty();
Function<String, Integer> length = String::length;
```

- `Predicate<T>` — принимает T, возвращает boolean;
- `Function<T, R>` — преобразует T в R;
- `Consumer<T>` — принимает T, ничего не возвращает;
- `Supplier<T>` — поставляет значение;
- `UnaryOperator<T>` — преобразует T в T;
- `Comparator<T>` — задаёт порядок сравнения.

## Optional

`Optional<T>` представляет возможное отсутствие значения. Чаще всего применяю как возвращаемый тип, когда отсутствие результата — нормальный сценарий.

```java
User user = repository.findById(id)
    .orElseThrow(() -> new UserNotFoundException(id));
```

`orElse(value)` вычисляет аргумент сразу, а `orElseGet(supplier)` — только если Optional пустой. Optional не является универсальной заменой всем nullable-полям и параметрам.

## Практика

1. Пишу generic-метод поиска максимального элемента.
2. Объясняю `? extends` и `? super`.
3. Преобразую цикл в Stream pipeline.
4. Показываю разницу между `orElse` и `orElseGet`.
