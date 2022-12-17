## Шпаргалка по оконным функциям

# Оглавление
1. [Основы](#introduction)
2. [Функции ранжирования](#rage)
3. [Агрегирующие оконные функции](#agr)
4. [Функции смещения](#lead)
5. [Подсчет значений с накоплением](#sun)
6. [Вынос окна в переменную](#win)
7. [Временные рамки](#time)
8. [Ограничения](#corr)
9. [Прочее](#other)
 
### Основы <a name="introduction"></a>

Оконная функция выполняет вычисления для набора записей, объединённых по какому-либо признаку. 
Такой набор называют *окном* — отсюда и название функции.
Вместе с текущей записью оконная функция обрабатывает остальные записи, которые входят в то же окно. 

Для каждой записи функция выводит одно значение. 
Этим оконная функция отличается от агрегирующей: и та и другая вычисляются для набора записей, но оконная функция не объединяет записи в одну, как агрегирующая, сохраняя независимость записей.

![about img for win func](https://pictures.s3.yandex.net/resources/2.1.2_2880border_1637138247.png)

Внутри выражения OVER находится оператор PARTITION BY. Он разделяет записи на группы, или разделы, в зависимости от значения в поле user_id. Записи с одинаковым user_id окажутся в одном окне. Для каждого из окон будет рассчитан результат оконной функции.

![about img for win func](https://pictures.s3.yandex.net/resources/2.1.1_2880border_1637138278.png)

### Функции ранжирования <a name="rage"></a>: 

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

### Агрегирующие оконные функции<a name="agr"></a>:
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

### Функции смещения<a name="lead"></a>: 
- **LEAD()** Позволяет возвращать предыдущие записи.
- **LAG()** Следующие.

```
LEAD/LAG(<поле>, <смещение>, <значение по умолчанию>) OVER (<определение окна>)
```


### Подсчет значений с накоплением<a name="sun"></a>.
```
with t as (select sum(costs) costs, date_trunc('month', created_at)::date as dt
from tools_shop.costs
group by date_trunc('month', created_at)::date)
select dt, sum(costs) over(order by dt) from t
```

### Вынос окна в переменную <a name="win"></a>.

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

### Временные рамки <a name="time"></a>.

Режим **ROWS**:

Записи, которые попадут в рамку, — до и после текущей: ROWS BETWEEN <начало рамки> AND <конец рамки>. 
_Начало рамки_ задают выражением **N PRECEDING** , где **N** — это количество записей до текущей. 
_Конец рамки_ задают выражением **N FOLLOWING**, где **N** — это количество записей после текущей.

```
SELECT *,
       AVG(revenue) OVER (PARTITION BY user_id ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING) AS avg_revenue
FROM online_store.orders
```
![about img for win func](https://pictures.s3.yandex.net/resources/5.1.3_2880border_1639758883.png)

Режим **RANGE**:
В отличие от **ROWS**, режим **RANGE** позволяет задавать диапазоны на основе значений в записях, а не на основе их количества

```
SELECT *,
       SUM(revenue) OVER (ORDER BY event_dt RANGE BETWEEN '3 day' PRECEDING AND CURRENT ROW) AS sum_revenue
FROM online_store.orders 
```

![about img for win func](https://pictures.s3.yandex.net/resources/5.2_2880border_1639759221.png)

В режиме RANGE действуют дополнительные правила:

- В выражении ```OVER``` обязательно указывают оператор ```ORDER BY```, а без него запрос выдаст ошибку.
- После ```ORDER BY``` можно указать только одно поле для сортировки. Будьте внимательны с сортировкой записей в обратном порядке — в этом случае логика указания границ рамок изменится.
- Указываемый диапазон должен соответствовать типу данных поля для сортировки. Если после ```ORDER BY``` указано поле типа ```timestamp``` или ```date```, то диапазон должен быть в виде значения типа ```interval```. Если после ```ORDER BY``` указано поле типа ```integer```, ```float``` или другого численного типа данных, то диапазон тоже должен быть выражен числом.


### Ограничения <a name="corr"></a>.
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
 
 ### ПРочее <a name="other"></a>.
 Ранжирование с присвоением одинакого ранга одинаковым значениям без прерывания ранга:
```
select created_at::date, costs, dense_rank() over(order by costs desc ) from 
tools_shop.costs
```

