# Лабораторная работа 2.1 Динамические соединения с базами данных

## Цель работы. Получить практические навыки создания сложного ETL-процесса, включающего динамическую загрузку файлов по HTTP, нормализацию базы данных, обработку дубликатов и настройку обработки ошибок с использованием Pentaho Data Integration (PDI).

## Вариант 5

- Основной фильтр для загрузки в БД: Quantity > 3
- Аналитика: Статистика по регионам и анализ прибыльности

### Ход работы 

Job

<img width="766" height="326" alt="image" src="https://github.com/user-attachments/assets/9e630f3c-549a-474d-bd98-d50d39175b9b" />

Установка переменной CSV_FILE_PATH с ссылкой на файл с данными
<img width="1189" height="548" alt="image" src="https://github.com/user-attachments/assets/e5441ebb-b0db-4109-8e16-ad2f1099f813" />


File exists проверяет наличие файла, в данном случае из ранее установленной переменной CSV_FILE_PATH
<img width="409" height="165" alt="image" src="https://github.com/user-attachments/assets/9da78491-882b-4896-b4be-7499ba90e931" />


Настройка HTTP
<img width="1211" height="765" alt="image" src="https://github.com/user-attachments/assets/7628516e-6b8e-4a22-b81d-178a7b97f9ec" />


#### Загрузка таблицы orders

<img width="720" height="303" alt="image" src="https://github.com/user-attachments/assets/ab049ef0-b3ba-4bb5-b148-da028a2f94c7" />

Изменение названий столбцов и определение типов данных
<img width="1197" height="725" alt="image" src="https://github.com/user-attachments/assets/6e530cf7-3288-4aa6-af77-d3e85d706549" />

Дедупликация

<img width="759" height="769" alt="image" src="https://github.com/user-attachments/assets/9e506350-63a7-4e37-8c01-9e328c41f85f" />

Фильтрация

<img width="739" height="380" alt="image" src="https://github.com/user-attachments/assets/1a93d615-371d-4833-a776-70d2c8d18387" />


Загрузка таблицы products

<img width="745" height="278" alt="image" src="https://github.com/user-attachments/assets/29956dc4-55a3-4688-85f2-4d272b3e220d" />

Загрузка таблицы customers


<img width="696" height="281" alt="image" src="https://github.com/user-attachments/assets/33fc77d1-4071-4c6c-8719-4bd3549e7c84" />

## Проверка

Подсчет количества строк в orders
<img width="977" height="345" alt="image" src="https://github.com/user-attachments/assets/4cc32f88-078c-4856-b5f3-7a47504dfb9d" />

Проверка работы фильтра (quantity > 3 по условию задачи)
<img width="970" height="328" alt="image" src="https://github.com/user-attachments/assets/5c44a0c6-b451-49a3-aeee-0fa5aec06ce6" />

Пример данных orders
<img width="1223" height="575" alt="image" src="https://github.com/user-attachments/assets/62800cf7-7ea3-40a4-bafd-0459202b762a" />

Пример данных customers
<img width="1302" height="348" alt="image" src="https://github.com/user-attachments/assets/fbf4e9ac-21e1-4122-af33-a7acd9af9904" />

Пример данных products
<img width="1419" height="363" alt="image" src="https://github.com/user-attachments/assets/b191501d-ff21-45ec-b29f-925651b16c79" />



## Анализ данных

### Статистика по регионам

<img width="1658" height="442" alt="image" src="https://github.com/user-attachments/assets/6d9e4dc0-d59f-4d92-811d-bf0ea158410d" />

<img width="1292" height="1024" alt="image" src="https://github.com/user-attachments/assets/79f3b172-0af7-4405-817b-c5b63a9c67f0" />

Выводы из анализа: Западный и Восточный регионы показывают наибольший объем продаж и прибыли, Южный регион имеет самую высокую маржинальность, несмотря на меньший объем продаж.
Центральный регион показывает низкую рентабельность и высокую долю возвратов. Самый высокий процент возвратов в Западном регионе, что требует анализа качества продукции

Города лидеры:
- Нью-Йорк
- Лос-Анджелес
- Чикаго


### Анализ прибыльности

<img width="1790" height="1181" alt="image" src="https://github.com/user-attachments/assets/84f96cf2-b5d8-4daf-8a91-53f59ad0fb4d" />

Выводы из анализы: Техника и мебель приносят основную прибыль, но канцтовары имеют низкую маржинальность. Корпоративные клиенты наиболее прибыльны, потребительский сегмент - наименее. Скидки более 30% делают продажи убыточными. Пик продаж приходится на ноябрь-декабрь, спад - на январь-февраль

Вывод: Были получены практические навыки создания сложного ETL-процесса, включающего динамическую загрузку файлов по HTTP, нормализацию базы данных, обработку дубликатов и настройку обработки ошибок с использованием Pentaho Data Integration. Был проведен подробный анализ данных в Python с подключением к СУБД MySQL
