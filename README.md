# Уровни изоляции

## Шаг 1: создание таблицы и заполнение её данными

Создать бд bank и подключиться к ней:

![img_2.png](img/img_2.png)

Создать таблицу:

![img_3.png](img/img_3.png)

Наполнить её данными, как в статье:

![img_4.png](img/img_4.png)

## Шаг 2: Read uncommitted

- Запустить 2 параллельные транзакции. Выполнить `SELECT * FROM`.

![img_8.png](img/img_8.png)

- Выполнить `INSERT`, `DELETE`, `UPDATE` в одной транзакции. Выполнить `SELECT * FROM` в другой транзакции. Выполнить запрос `SUM(balance)`

![img_9.png](img/img_9.png)

- Выполнить `ROLLBACK` в первой транзакции. Выполнить `SELECT SUM(balance) FROM accounts` во второй транзакции

![img_10.png](img/img_10.png)

Вывод: вторая транзакция не видит не закомиченные изменения первой, в постгресе Read uncommitted отсутствует

## Шаг 3: Read committed

- Запустить 2 параллельные транзакции. Выполнить `SELECT * FROM`.

  ![img_8.png](img/img_8.png)

- Выполнить `INSERT`, `DELETE`, `UPDATE` в одной транзакции. Просмотреть данные таблицы в обеих транзакциях
![img_12.png](img/img_12.png)
- Выполнить COMMIT в первой транзакции и снова просмотреть содержание таблицы
![img_13.png](img/img_13.png)

Вывод: вторая транзакция видит только закомиченные изменения первой

[//]: # (![Read uncommitted]&#40;./resource/Read%20committed/read_commited.png&#41;)

## Шаг 4: Repeatable read

- Запустить 2 параллельные транзакции. Выполнить `SELECT * FROM`.

[//]: # (  ![img_6.png]&#40;img_6.png&#41; )

[//]: # (![img_7.png]&#40;img_7.png&#41;)

[//]: # (  ![img_8.png]&#40;img_8.png&#41;)
  ![img_14.png](img/img_14.png)
- Выполнить `INSERT`, `DELETE`, `UPDATE` в одной транзакции. Выполнить `UPDATE` той же строки во второй транзакции

[//]: # (![img_15.png]&#40;img_15.png&#41;)

[//]: # (  ![Read uncommitted]&#40;./resource/Repeatable%20read/repeatable_read_1.png&#41;)
[//]: # (  ![img_16.png]&#40;img_16.png&#41;)
![img_17.png](img/img_17.png)
- Закоммитить изменения первой транзакции
![img_18.png](img/img_18.png)

[//]: # (![Read uncommitted]&#40;./resource/Repeatable%20read/repeatable_read_2.png&#41;)
- Выполнить `SELECT * FROM` во второй транзакции.
![img_19.png](img/img_19.png)  
Вывод: нельзя изменить данные в одной транзакции, не закоммитив изменения в другой

[//]: # (![Read uncommitted]&#40;./resource/Repeatable%20read/repeatable_read_3.png&#41;)

## Шаг 5: Serializable

- Запустить 2 параллельные транзакции. Выполнить `SELECT * FROM`.
![img_20.png](img/img_20.png)

- Выполнить `UPDATE`, `INSERT`, `DELETE` в первой транзакции. Выполнить `SELECT * FROM` в обоих транзакциях.

(переименование Пети в Ивана) 

![img_21.png](img/img_21.png)

(внесены ещё изменения)  

![img_22.png](img/img_22.png)

Вывод: каждая транзакция видит только свои изменения