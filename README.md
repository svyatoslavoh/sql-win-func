## Шпаргалка по оконным функциям

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
 
