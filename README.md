# ANALYZE

- Создадим базу данных людей и заполним её данными

  ```roomsql
  CREATE DATABASE test_people;
  \c test_people;

  CREATE TABLE people (
      id SERIAL PRIMARY KEY,
      age INTEGER,
      gender BOOLEAN, -- TRUE = male, FALSE = female
      married BOOLEAN -- TRUE = married, FALSE = not married
  );
  
  CREATE OR REPLACE FUNCTION random_bool() RETURNS BOOLEAN AS $$
  BEGIN
    RETURN random() < 0.5;
  END;
  $$ LANGUAGE plpgsql;
  
  INSERT INTO people (age, gender, married)
  SELECT
      (random() * 70 + 18)::INTEGER, -- Случайный возраст от 18 до 88
      random_bool(),                 -- Случайный пол
      random_bool()                  -- Случайное семейное положение
  FROM generate_series(1, 100);
  ```

- Выполним ANALYZE

  ```roomsql
  ANALYZE people;
  ```

- Придумаем SQL запрос к базе

  ```roomsql
  SELECT COUNT(*)      -- Количество неженатых мужчин старше 50
  FROM people
  WHERE gender = TRUE
    AND married = FALSE
    AND age > 50;
  ```

- Замерим производительность

  ```roomsql
  EXPLAIN ANALYZE SELECT COUNT(*)
  FROM people
  WHERE gender = TRUE
    AND married = FALSE
    AND age > 50;
  ```
  
  Получим вывод:

  ```
  QUERY PLAN       
                                          
  ---------------------------------------------------------------
  ----------------------------------------
   Aggregate  (cost=2.27..2.28 rows=1 width=8) (actual time=0.027
  ..0.027 rows=1 loops=1)
     ->  Seq Scan on people  (cost=0.00..2.25 rows=9 width=0) (ac
  tual time=0.015..0.023 rows=10 loops=1)
           Filter: (gender AND (NOT married) AND (age > 50))
           Rows Removed by Filter: 90
   Planning Time: 0.093 ms
   Execution Time: 0.051 ms
  (6 rows)
  ```

- Отключим автоматическое переанализироание

  ```roomsql
  ALTER SYSTEM SET autovacuum = 'off';
  ```

- Поменяем содержимое базы, не меняя её объем, но меняя распределение

  Применим ряд изменений:

  ```roomsql
    UPDATE people
    SET married = TRUE
    WHERE gender = FALSE AND age > 30;
    
    UPDATE people
    SET age = 35, married = TRUE
    WHERE id = 5;
    
    UPDATE people
    SET age = age - 1
    WHERE married = TRUE;
  ```

- Повторно произведем замеры и получим ответ:

  ```
  QUERY PLAN                                   
               
  ----------------------------------------------
  ----------------------------------------------
  -------------
   Aggregate  (cost=37.25..37.26 rows=1 width=8)
   (actual time=0.069..0.070 rows=1 loops=1)
     ->  Seq Scan on people  (cost=0.00..36.75 r
  ows=199 width=0) (actual time=0.064..0.065 row
  s=0 loops=1)
           Filter: (gender AND (NOT married) AND
   (age > 50))
           Rows Removed by Filter: 100
   Planning Time: 0.150 ms
   Execution Time: 0.108 ms
  (6 rows)
  ```

**Вывод**: при исопльзовании ANALYZE время выполнения запроса к бд было 0.051 ms а стало 0.108 ms - время увеличилось примерно в два раза

