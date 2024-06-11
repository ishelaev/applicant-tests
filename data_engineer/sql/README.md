# Тест SQL

На основе таблиц базы данных, напишите SQL код, который возвращает необходимые результаты
Пример: 

Общее количество товаров
```sql
select count (*) from items
```

## Структура данных

Используемый синтаксис: Oracle SQL или другой

| Сustomer       | Description           |
| -------------- | --------------------- |
| customer\_id   | customer unique id    |
| customer\_name | customer name         |
| country\_code  | country code ISO 3166 |

| Items             | Description       |
| ----------------- | ----------------- |
| item\_id          | item unique id    |
| item\_name        | item name         |
| item\_description | item description  |
| item\_price       | item price in USD |

| Orders       | Description                 |
| ------------ | --------------------------- |
| date\_time   | date and time of the orders |
| item\_id     | item unique id              |
| customer\_id | user unique id              |
| quantity     | number of items in order    |

| Countries     | Description           |
| ------------- | --------------------- |
| country\_code | country code          |
| country\_name | country name          |
| country\_zone | AMER, APJ, LATAM etc. |


| Сonnection\_log         | Description                           |
| ----------------------- | ------------------------------------- |
| customer\_id            | customer unique id                    |
| first\_connection\_time | date and time of the first connection |
| last\_connection\_time  | date and time of the last connection  |

## Задания

### 1) Количество покупателей из Италии и Франции

| **Country_name** | **CustomerCountDistinct** |
| ------------------------- | ----------------------------- |
| France                    | #                             |
| Italy                     | #                             |

```sql
SELECT 
    c.country_name AS Country_name,
    COUNT(DISTINCT cu.customer_id) AS CustomerCountDistinct
FROM 
    Customer cu
JOIN 
    Countries c ON cu.country_code = c.country_code
WHERE 
    c.country_name IN ('France', 'Italy')
GROUP BY 
    c.country_name;
```

### 2) ТОП 10 покупателей по расходам

| **Customer_name** | **Revenue** |
| ---------------------- | ----------- |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |

```sql
SELECT 
    cu.customer_name AS Customer_name,
    SUM(o.quantity * i.item_price) AS Revenue
FROM 
    Orders o
JOIN 
    Customer cu ON o.customer_id = cu.customer_id
JOIN 
    Items i ON o.item_id = i.item_id
GROUP BY 
    cu.customer_name
ORDER BY 
    Revenue DESC
LIMIT 10;
```

### 3) Общая выручка USD по странам, если нет дохода, вернуть NULL

| **Country_name** | **RevenuePerCountry** |
| ------------------------- | --------------------- |
| Italy                     | #                     |
| France                    | NULL                  |
| Mexico                    | #                     |
| Germany                   | #                     |
| Tanzania                  | #                     |

```sql
SELECT 
    c.country_name AS Country_name,
    CASE 
        WHEN SUM(o.quantity * i.item_price) IS NULL THEN NULL
        ELSE SUM(o.quantity * i.item_price)
    END AS RevenuePerCountry
FROM 
    Countries c
LEFT JOIN 
    Customer cu ON c.country_code = cu.country_code
LEFT JOIN 
    Orders o ON cu.customer_id = o.customer_id
LEFT JOIN 
    Items i ON o.item_id = i.item_id
GROUP BY 
    c.country_name;
```

### 4) Самый дорогой товар, купленный одним покупателем

| **Customer\_id** | **Customer\_name** | **MostExpensiveItemName** |
| ---------------- | ------------------ | ------------------------- |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |

```sql
WITH CustomerPurchases AS (
    SELECT 
        cu.customer_id,
        cu.customer_name,
        i.item_name,
        i.item_price,
        ROW_NUMBER() OVER (PARTITION BY cu.customer_id ORDER BY i.item_price DESC) AS rn
    FROM 
        Orders o
    JOIN 
        Customer cu ON o.customer_id = cu.customer_id
    JOIN 
        Items i ON o.item_id = i.item_id
)
SELECT 
    customer_id,
    customer_name,
    item_name AS MostExpensiveItemName
FROM 
    CustomerPurchases
WHERE 
    rn = 1;
```

### 5) Ежемесячный доход

| **Month (MM format)** | **Total Revenue** |
| --------------------- | ----------------- |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |

```sql
SELECT 
    DATE_FORMAT(o.date_time, '%m') AS Month,
    SUM(o.quantity * i.item_price) AS TotalRevenue
FROM 
    Orders o
JOIN 
    Items i ON o.item_id = i.item_id
GROUP BY 
    DATE_FORMAT(o.date_time, '%m');
```

### 6) Найти дубликаты

Во время передачи данных произошел сбой, в таблице orders появилось несколько 
дубликатов (несколько результатов возвращаются для date_time + customer_id + item_id). 
Вы должны их найти и вернуть количество дубликатов.

```sql
SELECT 
    date_time,
    customer_id,
    item_id,
    COUNT(*) AS DuplicateCount
FROM 
    Orders
GROUP BY 
    date_time,
    customer_id,
    item_id
HAVING 
    COUNT(*) > 1;
```

### 7) Найти "важных" покупателей

Создать запрос, который найдет всех "важных" покупателей,
т.е. тех, кто совершил наибольшее количество покупок после своего первого заказа.

| **Customer\_id** | **Total Orders Count** |
| --------------------- |-------------------------------|
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |

```sql
WITH FirstOrderDate AS (
    SELECT 
        customer_id,
        MIN(date_time) AS first_order_date
    FROM 
        Orders
    GROUP BY 
        customer_id
),
CustomerOrdersCount AS (
    SELECT 
        o.customer_id,
        COUNT(*) AS orders_count
    FROM 
        Orders o
    JOIN 
        FirstOrderDate fod ON o.customer_id = fod.customer_id
    WHERE 
        o.date_time > fod.first_order_date
    GROUP BY 
        o.customer_id
)
SELECT 
    fod.customer_id AS Customer_id,
    COALESCE(coc.orders_count, 0) AS Total_Orders_Count
FROM 
    FirstOrderDate fod
LEFT JOIN 
    CustomerOrdersCount coc ON fod.customer_id = coc.customer_id
ORDER BY 
    Total_Orders_Count DESC;
```

### 8) Найти покупателей с "ростом" за последний месяц

Написать запрос, который найдет всех клиентов,
у которых суммарная выручка за последний месяц
превышает среднюю выручку за все месяцы.

| **Customer\_id** | **Total Revenue** |
| --------------------- |-------------------|
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |

```sql
WITH MonthlyRevenue AS (
    SELECT 
        DATE(date_time) AS month,
        customer_id,
        SUM(quantity * item_price) AS revenue
    FROM 
        Orders
    GROUP BY 
        DATE(date_time), customer_id
),
AverageRevenue AS (
    SELECT 
        customer_id,
        AVG(revenue) AS avg_revenue
    FROM 
        MonthlyRevenue
    GROUP BY 
        customer_id
),
LastMonthRevenue AS (
    SELECT 
        customer_id,
        SUM(revenue) AS last_month_revenue
    FROM 
        MonthlyRevenue
    WHERE 
        month = DATE_FORMAT(DATE_SUB(NOW(), INTERVAL 1 MONTH), '%Y-%m')
    GROUP BY 
        customer_id
)
SELECT 
    mr.customer_id AS Customer_id,
    IFNULL(lmr.last_month_revenue, 0) AS "Total Revenue"
FROM 
    MonthlyRevenue mr
LEFT JOIN 
    LastMonthRevenue lmr ON mr.customer_id = lmr.customer_id
JOIN 
    AverageRevenue ar ON mr.customer_id = ar.customer_id
WHERE 
    IFNULL(lmr.last_month_revenue, 0) > ar.avg_revenue;
```
