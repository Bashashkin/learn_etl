# Лабораторная работа 4.1 Анализ данных с помощью Dask и визуализация графов (DAG)

### Вариант 5

### Цель работы:
Получить практические навыки работы с библиотекой Dask для построения базовых ETL-конвейеров (Extract, Transform, Load)
при обработке больших массивов данных, не помещающихся в оперативную память.
Изучить принципы «ленивых вычислений» (lazy evaluation), управление памятью и визуализацию ориентированных ациклических графов (DAG).



## Ход работы

[Исходные данные](https://disk.yandex.ru/d/fbPE3VNKYocd7g/UK%20Property%20Price%20official%20data%201995-202304.zip)

В работе использовался датасет, содержащий сведения о сделках с недвижимостью в Великобритании. Данные включают информацию о ценах, датах сделок, типах объектов недвижимости и их географическом расположении. Каждая строка датасета соответствует отдельной сделке купли-продажи объекта недвижимости.

Структура датасета

- transaction_id - уникальный идентификатор транзакции
- price - стоимость объекта недвижимости
- date - дата совершения сделки
- postcode - почтовый индекс
- property_type - тип недвижимости
- new_build_flag - признак новостройки (Y/N)
- tenure_type - тип владения (freehold / leasehold)
- primary_addressable_object_name - основной номер объекта
- secondary_addressable_object_name - дополнительный номер (если есть)
- street - улица
- locality - локальная область
- town_city - город
- district - район
- county - округ
- ppd_category_type - категория сделки
- record_status - статус записи

Поле property_type принимает следующие значения:

- D - отдельно стоящий дом (Detached)
- S - дом на две семьи (Semi-Detached)
- T - таунхаус (Terraced)
- F - квартира (Flat)
- O - другой тип недвижимости

### Извелечение данных
```python
import dask.dataframe as dd
from dask.distributed import Client
from dask.diagnostics import ProgressBar
import pandas as pd

client = Client(n_workers=2, threads_per_worker=2, processes=True)
client

columns = [
    "transaction_id",
    "price",
    "date",
    "postcode",
    "property_type",
    "new_build_flag",
    "tenure_type",
    "primary_addressable_object_name",
    "secondary_addressable_object_name",
    "street",
    "locality",
    "town_city",
    "district",
    "county",
    "ppd_category_type",
    "record_status"
]

df = dd.read_csv(
    '202304.csv',
    header=None,              
    names=columns,            
    dtype=str,                
    blocksize="64MB"
)
```

### Трансформация и очистка данных

```python
missing_values = df.isnull().sum()
mysize = df.index.size
missing_count = (missing_values / mysize) * 100

with ProgressBar():
    missing_count_percent = missing_count.compute()

print(missing_count_percent)
```

Вывод:
<img width="457" height="417" alt="image" src="https://github.com/user-attachments/assets/b37d790b-4b35-4166-9c73-162af490e4f7" />

```python
columns_to_drop = list(
    missing_count_percent[missing_count_percent > 60].index
)

print("Удаляемые столбцы:", columns_to_drop)

df_dropped = df.drop(columns=columns_to_drop)

df_dropped.head()
```

Вывод:
<img width="1517" height="589" alt="image" src="https://github.com/user-attachments/assets/a0669ffe-c0ee-4e19-b2dc-c446fc3df725" />

### Загрузка / Сохранение результатов

```python
df_dropped.to_parquet(
    'cleaned_dataset.parquet',
    engine='pyarrow'
)
```

### Визуализация направленных ациклических графов (DAG)

```python
df_dropped.visualize(filename="etl_dag", format="png")
```
<img width="573" height="251" alt="etl_dag" src="https://github.com/user-attachments/assets/7b013535-5b5c-4114-9518-833ed2b7e0d2" />

```python
df["price"].astype(float).mean().visualize(filename="mean_dag")
```
<img width="573" height="443" alt="mean_dag" src="https://github.com/user-attachments/assets/4706431f-6be3-48c9-b478-6f13b96b2543" />

### Визуализация данных

<img width="649" height="377" alt="visualization" src="https://github.com/user-attachments/assets/a268034c-17ce-46d1-9b18-83239b8be238" />

<img width="435" height="242" alt="123" src="https://github.com/user-attachments/assets/dcc8fcac-ec56-4fa5-b774-b86a3c4a352b" />

<img width="411" height="259" alt="city_with_most_sales" src="https://github.com/user-attachments/assets/6cbefe7f-0135-4621-b7ed-f828cdb87c4c" />

<img width="298" height="362" alt="propety_type_price_distribution" src="https://github.com/user-attachments/assets/8a0088ce-a83d-496c-a5a6-63d3cf1f1c82" />

## Вывод

В ходе выполнения лабораторной работы был реализован процесс анализа большого набора данных о сделках с недвижимостью в Великобритании с использованием библиотеки Dask. Также была изучена концепция направленных ациклических графов. Для визуалиазии данных была использована библиотека Altair

[Файл .ipynb](https://disk.yandex.ru/d/7TZ3DG34FlY1Xw)
