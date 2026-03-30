# Лабораторная работа 4.1 Анализ данных с помощью Dask и визуализация графов (DAG)

### Вариант 5

### Цель работы:
Получить практические навыки работы с библиотекой Dask для построения базовых ETL-конвейеров (Extract, Transform, Load)
при обработке больших массивов данных, не помещающихся в оперативную память.
Изучить принципы «ленивых вычислений» (lazy evaluation), управление памятью и визуализацию ориентированных ациклических графов (DAG).



## Ход работы

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




