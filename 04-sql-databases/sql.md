# SQL: запросы, JOIN, группировки и оконные функции

## SELECT и логический порядок

```sql
SELECT name, price
FROM products
WHERE price > 100
ORDER BY price DESC
LIMIT 10;
```

- `SELECT` — какие выражения вывести.
- `FROM` — источник строк.
- `WHERE` — фильтрация строк до группировки.
- `GROUP BY` — формирование групп.
- `HAVING` — фильтрация групп.
- `ORDER BY` — сортировка результата.
- `LIMIT` — ограничение количества строк.

Упрощённый логический порядок: `FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`. Это логика обработки, а не обязательный физический порядок исполнения.

## JOIN

```sql
SELECT u.name, o.id
FROM users u
JOIN orders o ON o.user_id = u.id;
```

- `INNER JOIN` — только строки с совпадением.
- `LEFT JOIN` — все строки слева; если пары справа нет, правые столбцы равны NULL.
- `RIGHT JOIN` — сохраняет все строки справа.
- `FULL JOIN` — сохраняет строки обеих сторон.
- `CROSS JOIN` — декартово произведение.

Важно: условие в `WHERE` по правой таблице может превратить результат `LEFT JOIN` в поведение, похожее на INNER JOIN.

```sql
-- Сохранить пользователей без заказов:
SELECT u.id, o.id
FROM users u
LEFT JOIN orders o
  ON o.user_id = u.id
 AND o.status = 'PAID';
```

## WHERE и HAVING

`WHERE` фильтрует строки до группировки. `HAVING` фильтрует группы после `GROUP BY`. В HAVING можно использовать агрегаты и выражения по группировочным столбцам — не только агрегатные функции.

```sql
SELECT customer_id, COUNT(*) AS orders_count
FROM orders
WHERE status = 'PAID'
GROUP BY customer_id
HAVING COUNT(*) > 3;
```

Нельзя написать `WHERE COUNT(*) > 3`, потому что агрегат формируется после этапа WHERE.

## Агрегаты и NULL

- `COUNT(*)` считает строки.
- `COUNT(column)` считает ненулевые значения.
- `SUM`, `AVG`, `MIN`, `MAX` работают с выражениями; NULL обычно игнорируется агрегатами.
- `NULL` означает неизвестное/отсутствующее значение. Для проверки использую `IS NULL`, не `= NULL`.
- `COALESCE(a, b)` возвращает первое не-NULL значение.

## GROUP BY

Каждое выражение в SELECT, которое не агрегируется, обычно должно быть частью GROUP BY (с учётом функциональных зависимостей и правил конкретной СУБД).

```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 100000;
```

## Подзапросы и EXISTS

`EXISTS` проверяет, существует ли хотя бы одна строка, удовлетворяющая подзапросу.

```sql
SELECT u.id
FROM users u
WHERE EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.user_id = u.id
);
```

Для проверки отсутствия связанных строк использую `NOT EXISTS`. С `NOT IN` нужно быть осторожным, если подзапрос может вернуть NULL.

## CTE

`WITH` задаёт имя промежуточному результату и помогает структурировать запрос.

```sql
WITH paid_orders AS (
  SELECT * FROM orders WHERE status = 'PAID'
)
SELECT customer_id, COUNT(*)
FROM paid_orders
GROUP BY customer_id;
```

CTE не обязательно материализуется: поведение оптимизатора зависит от СУБД и версии.

## Оконные функции

Оконная функция считает значение по набору строк, не схлопывая строки в одну группу.

```sql
SELECT customer_id, created_at,
       ROW_NUMBER() OVER (
         PARTITION BY customer_id
         ORDER BY created_at DESC
       ) AS row_num
FROM orders;
```

- `PARTITION BY` делит строки на группы окна.
- `ORDER BY` задаёт порядок внутри окна.
- `ROW_NUMBER` нумерует строки.
- `RANK` оставляет пропуски после одинаковых рангов.
- `DENSE_RANK` не оставляет пропуски.

## UNION

`UNION` объединяет результаты совместимых SELECT и удаляет дубликаты. `UNION ALL` сохраняет дубликаты и обычно дешевле. Количество и типы столбцов должны быть совместимы.

## Практика

1. Найти пользователей без заказов через LEFT JOIN.
2. Решить ту же задачу через NOT EXISTS.
3. Найти клиентов с более чем тремя оплаченными заказами.
4. Вывести последний заказ каждого клиента через ROW_NUMBER.
5. Сравнить UNION и UNION ALL.
6. Найти запрос, в котором WHERE по правой таблице ломает ожидаемый LEFT JOIN.
