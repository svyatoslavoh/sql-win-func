## Шпаргалка по оконным функциям

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
 
