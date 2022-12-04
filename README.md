## Шпаргалка по оконным функциям

Оконная функция выполняет вычисления для набора записей, объединённых по какому-либо признаку. 
Такой набор называют *окном* — отсюда и название функции.
Вместе с текущей записью оконная функция обрабатывает остальные записи, которые входят в то же окно. 

Для каждой записи функция выводит одно значение. 
Этим оконная функция отличается от агрегирующей: и та и другая вычисляются для набора записей, но оконная функция не объединяет записи в одну, как агрегирующая, сохраняя независимость записей.

![about img for win func](https://pictures.s3.yandex.net/resources/2.1.2_2880border_1637138247.png)

Внутри выражения OVER находится оператор PARTITION BY. Он разделяет записи на группы, или разделы, в зависимости от значения в поле user_id. Записи с одинаковым user_id окажутся в одном окне. Для каждого из окон будет рассчитан результат оконной функции.

![about img for win func](https://pictures.s3.yandex.net/resources/2.1.1_2880border_1637138278.png)

### Функции ранжирования:

- **ROW_NUMBER()** Взвращает порядковый номер записи в окне.
- **RANK()** присваивает одинаковым значениям одинаковый ранг. 
- **DENSE_RANK()** не учитывает количество записей и назначает ранги последовательно. 
- **NTILE()** позволяет назначать записям фиксированное количество рангов — в зависимости от аргумента, который передают функции. 

```
SELECT *,
         NTILE(3) OVER (ORDER BY revenue)
FROM online_store.orders 
```
Функция NTILE() с аргументом 3 разделит записи на три группы в зависимости от значения revenue

### Агрегирующие оконные функции:
- **AVG()**
- **MIN()** 
- **MAX()**
```
SELECT *,
       MAX(event_dt) OVER (PARTITION BY user_id) AS last_order_dt
FROM online_store.orders; 
```

- **SUM()**
```
SELECT *,
       SUM(revenue) OVER (PARTITION BY event_dt) AS daily_rev
FROM online_store.orders; 
```
- **COUNT()**

### Функции смещения: 
- **LEAD()** Позволяет возвращать предыдущие записи.
- **LAG()** Следующие.

```
LEAD/LAG(<поле>, <смещение>, <значение по умолчанию>) OVER (<определение окна>)
```


### Ранжирование с присвоением одинакого ранга одинаковым значениям без прерывания ранга:
```
select created_at::date, costs, dense_rank() over(order by costs desc ) from 
tools_shop.costs
```

### Подсчет значений с накоплением.
```
with t as (select sum(costs) costs, date_trunc('month', created_at)::date as dt
from tools_shop.costs
group by date_trunc('month', created_at)::date)
select dt, sum(costs) over(order by dt) from t
```



### Вынос окна в переменную.

В основном запросе конструкцию WINDOW указывают после оператора WHERE и до оператора ORDER BY

```
SELECT *,
       ROW_NUMBER() OVER my_window,
       RANK() OVER my_window,
       DENSE_RANK() OVER one_more_window
FROM online_store.orders
WINDOW my_window AS (ORDER BY revenue),
       one_more_window AS (PARTITION BY user_id)
```

### Ограничения
С оконными функциями ```DISTINCT``` не сработает. 
```
DISTINCT is not implemented for window functions
SELECT COUNT(DISTINCT user_id) OVER (ORDER BY event_dt)
      ^^^
FROM online_store.orders; 
```

Оконные функции нельзя сочетать с группировкой.
```
window functions are not allowed in GROUP BY
SELECT event_dt, 
       COUNT(*) OVER (PARTITION BY event_dt)
FROM online_store.orders
GROUP BY event_dt, COUNT(*) OVER (PARTITION BY event_dt);
                     ^^^ 
```

 Оконные функции нельзя использовать в условиях после WHERE, как и агрегирующие функции. Вычисление агрегирующих и оконных функций происходит после срезов
 
 ```
 window functions are not allowed in WHERE

SELECT event_dt, 
       COUNT(*) OVER (PARTITION BY event_dt)
FROM online_store.orders
WHERE COUNT(*) OVER (PARTITION BY event_dt) = 100;
        ^^^ 
 ```
 
