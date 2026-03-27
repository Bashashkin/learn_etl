# Лабораторная работа 3.1 Интеграция данных из нескольких источников. Обработка и согласование данных из разных источников

## Цель работы.
Разработать комплексное ETL-решение для интеграции данных из локальной СУБД PostgreSQL и файловых источников (CSV/Excel) в целевое хранилище MySQL. Спроектировать верхнеуровневую архитектуру аналитического решения.

## Вариант 5
- Тема: Клиентсикй сервис
- Задача: Создать отчет по качеству обслуживания: сопоставить время доставки с оценкой клиента в отзыве.

## Ход работы

Архитектура решения

<img width="1037" height="994" alt="image" src="https://github.com/user-attachments/assets/a74db2f3-3731-401d-b6a8-37a84e5b8aeb" />

Создание таблицы заказов
```SQL
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    order_date TIMESTAMP,
    delivery_date TIMESTAMP,
    order_amount NUMERIC(10,2)
);
```
Генерация данных
```SQL
INSERT INTO orders (customer_id, order_date, delivery_date, order_amount)
SELECT
    (random() * 10000)::INT,
    order_date,
    order_date + (random() * INTERVAL '10 days'), -- доставка через 0–10 дней
    ROUND((random() * 500 + 10)::numeric, 2)
FROM (
    SELECT NOW() - (random() * INTERVAL '365 days') AS order_date
    FROM generate_series(1, 1000000)
) t;
```
Создание таблицы доставок
```python
import csv
import random

statuses = ["delivered", "in_transit", "cancelled", "delayed"]

with open("delivery_status.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["order_id", "status"])

    for _ in range(50000):
        order_id = random.randint(1, 1000000)
        status = random.choice(statuses)
        writer.writerow([order_id, status])
```
Создание таблицы отзывов
```python
import pandas as pd
import random

data = []

for _ in range(50000):
    order_id = random.randint(1, 1000000)

    # оценка зависит от "условного качества"
    rating = random.choices(
        [1, 2, 3, 4, 5],
        weights=[5, 10, 20, 30, 35]
    )[0]

    data.append([order_id, rating])

df = pd.DataFrame(data, columns=["order_id", "rating"])
df.to_excel("reviews.xlsx", index=False)
```

Трансформация данных

<img width="734" height="231" alt="image" src="https://github.com/user-attachments/assets/eaf84c17-520f-4da9-aadb-153c98e2eaee" />

Подключение PostgreSQL в Pentaho DI

<img width="1193" height="722" alt="image" src="https://github.com/user-attachments/assets/4a65d8f2-61ea-4e2f-9418-e8f3cd48ffe9" />

Объединение таблиц по ключу order_id

<img width="420" height="498" alt="image" src="https://github.com/user-attachments/assets/19a99582-ff0c-4b63-8969-a422c38c6437" />

Подсчет дней и часов доставки

<img width="888" height="425" alt="image" src="https://github.com/user-attachments/assets/b2c4767f-d52c-4612-937f-1c7876671bc3" />

Фильтр по рейтингу и статусу доставки

<img width="746" height="384" alt="image" src="https://github.com/user-attachments/assets/f79426b3-c7e5-4640-a52b-8e5fa9c971c8" />

Подключение MySQL 

<img width="897" height="613" alt="image" src="https://github.com/user-attachments/assets/92d3b60d-716f-4c9c-967a-8add64c5555c" />

Создание витрины данных

```SQL
CREATE VIEW service_quality_report AS
SELECT 
    order_id,
    customer_id,
    order_date,
    delivery_date,
    status,
    rating,
    order_amount,
    delivery_days,
    delivery_hours,
    CASE 
        WHEN delivery_days = 0 THEN 'Доставка день-в-день'
        WHEN delivery_days = 1 THEN 'Доставка на следующий день'
        WHEN delivery_days BETWEEN 2 AND 3 THEN 'Быстрая доставка (2-3 дня)'
        WHEN delivery_days BETWEEN 4 AND 5 THEN 'Обычная доставка (4-5 дней)'
        WHEN delivery_days BETWEEN 6 AND 7 THEN 'Медленная доставка (6-7 дней)'
        WHEN delivery_days >= 8 THEN 'Очень медленная доставка (8+ дней)'
        ELSE 'Нет данных'
    END AS delivery_speed_category,
    CASE 
        WHEN rating = 5 THEN 'Идеально'
        WHEN rating = 4 THEN 'Хорошо'
        WHEN rating = 3 THEN 'Нормально'
        WHEN rating = 2 THEN 'Плохо'
        WHEN rating = 1 THEN 'Очень плохо'
        ELSE 'Нет оценки'
    END AS rating_category,
    -- Соответствие времени доставки и оценки
    CASE 
        WHEN delivery_days <= 3 AND rating >= 4 THEN 'Отличный сервис - быстрая доставка с высокой оценкой'
        WHEN delivery_days <= 3 AND rating <= 3 THEN 'Быстрая доставка, но низкая оценка - проверить качество'
        WHEN delivery_days >= 4 AND rating >= 4 THEN 'Хорошая оценка несмотря на медленную доставку'
        WHEN delivery_days >= 4 AND rating <= 3 THEN 'Плохой сервис - медленная доставка с низкой оценкой'
        WHEN delivery_days = 0 AND rating = 5 THEN 'Идеально - доставка день в день и высокая оценка'
        ELSE 'Смешанная оценка качества сервиса'
    END AS service_quality_assessment
FROM 
	service_quality;
```
