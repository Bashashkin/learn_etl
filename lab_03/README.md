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
    rating,
    TIMESTAMPDIFF(MINUTE, order_date, delivery_date) AS delivery_minutes
FROM service_quality
WHERE rating IS NOT NULL;
```
Итоговое представление

<img width="398" height="210" alt="image" src="https://github.com/user-attachments/assets/fed24ea4-1d80-49a8-aa9b-765c6afa5314" />

Корреляция между временем и оценки доставки 
```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression

X = df[['delivery_minutes']]
y = df['rating']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

model = LinearRegression()
model.fit(X_train, y_train)

print("R2:", model.score(X_test, y_test))
```
Результат:
R2: -0.001946379607057347

Время доставки не влияет на рейтинг, может быть из-за того, что данные были сгенерированы.

<img width="567" height="455" alt="image" src="https://github.com/user-attachments/assets/8f3b0a60-913d-44b4-b595-83ae2c83d49f" />

[Анализ доставок в ipynb](https://github.com/Bashashkin/learn_etl/blob/main/lab_03/lab3_etl.ipynb)
