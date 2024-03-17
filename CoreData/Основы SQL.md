# Основны SQL

# Основные операторы SQL

## Операторы работы с базой данных
### SHOW DATABASES
SQL-команда, которая отвечает за просмотр доступных баз данных.

### CREATE DATABASE
Команда для создания новой базы данных.

### USE
С помощью этой SQL-команды USE <database_name> выбирается база данных, необходимая для дальнейшей работы с ней.

### SOURCE
А SOURCE <file.sql> позволит выполнить сразу несколько SQL-команд, содержащихся в файле с расширением .sql.

### DROP DATABASE
Стандартная SQL-команда для удаления целой базы данных.

## Операторы работы с таблицей
### SHOW TABLES
С помощью этой несложной команды можно увидеть все таблицы, которые доступны в базе данных.

### CREATE TABLE
SQL-команда для создания новой таблицы.
Пример:
Создайте таблицу «instructor»:
```sql
CREATE TABLE instructor (
  ID CHAR(5),
  name VARCHAR(20) NOT NULL,
  dept_name VARCHAR(20),
  salary NUMERIC(8,2),
  PRIMARY KEY (ID),
  FOREIGN KEY (dept_name) REFERENCES department(dept_name)
);
```
Ограничения:
- ячейка таблицы не может иметь значение NULL;
- первичный ключ — PRIMARY KEY(col_name1, col_name2, …);
- внешний ключ — FOREIGN KEY(col_namex1, …, col_namexn) REFERENCES table_name(col_namex1, …, col_namexn).

### DESCRIBE
С помощью DESCRIBE <table_name> можно просмотреть различные сведения (тип значений, является ключом или нет) о столбцах таблицы.

### INSERT
Команда INSERT INTO <table_name> в SQL отвечает за добавление данных в таблицу:
```sql
INSERT INTO  (, , , …)
  VALUES (, , , …);
```

### UPDATE
SQL-команда для обновления данных таблицы:
```sql
UPDATE 
  SET  = ,  = , ...
  WHERE ;
```

### DELETE
SQL-команда DELETE FROM <table_name> используется для удаления данных из таблицы.

### DROP TABLE
А так можно удалить всю таблицу целиком.

## Операторы работы с данными
### SELECT
Для получения данных из выбранной таблицы:
```sql
SELECT , , …
  FROM ;
```

### SELECT DISTINCT
В столбцах таблицы могут содержаться повторяющиеся данные. Используйте SELECT DISTINCT для получения только неповторяющихся данных.
```sql
SELECT , , …
  FROM 
  WHERE ;
```

### WHERE
Можно использовать ключевое слово WHERE в SELECT для указания условий в запросе:
В запросе можно задавать следующие условия:
- сравнение текста;
- сравнение численных значений;
- логические операции AND (и), OR (или) и NOT (отрицание).

Пример
Попробуйте выполнить следующие команды. Обратите внимание на условия, заданные в WHERE:
```sql
SELECT * FROM course WHERE dept_name=’Comp. Sci.’;
SELECT * FROM course WHERE credits>3;
SELECT * FROM course WHERE dept_name='Comp. Sci.' AND credits>3;
```

### GROUP BY
Оператор GROUP BY часто используется с агрегатными функциями, такими как COUNT, MAX, MIN, SUM и AVG, для группировки выходных значений.
Пример
Выведем количество курсов для каждого факультета:
```sql
SELECT COUNT(course_id), dept_name
  FROM course
  GROUP BY dept_name;
```

### ORDER BY
ORDER BY используется для сортировки результатов запроса по убыванию или возрастанию. ORDER BY отсортирует по возрастанию, если не будет указан способ сортировки ASC или DESC.
```sql
SELECT , , …
  FROM 
  ORDER BY , , … ASC|DESC;
```
Пример
Выведем список курсов по возрастанию и убыванию количества кредитов:
```sql
SELECT * FROM course ORDER BY credits;
SELECT * FROM course ORDER BY credits DESC;
```

### BETWEEN
BETWEEN используется для выбора значений данных из определённого промежутка. Могут быть использованы числовые и текстовые значения, а также даты.
Пример
Выведем список инструкторов, чья зарплата больше 50 000, но меньше 100 000:
```sql
SELECT * FROM instructor
  WHERE salary BETWEEN 50000 AND 100000;
```

### LIKE
Оператор LIKE используется в WHERE, чтобы задать шаблон поиска похожего значения.
Есть два свободных оператора, которые используются в LIKE:
- % (ни одного, один или несколько символов);
- _ (один символ).

Пример
Выведем список курсов, в имени которых содержится «to», и список курсов, название которых начинается с «CS-»:
```sql
SELECT * FROM course WHERE title LIKE ‘%to%’;
SELECT * FROM course WHERE course_id LIKE 'CS-___';
```

### IN
С помощью IN можно указать несколько значений для оператора WHERE.
Пример
Выведем список студентов с направлений Comp. Sci., Physics и Elec. Eng.:
```sql
SELECT * FROM student
  WHERE dept_name IN (‘Comp. Sci.’, ‘Physics’, ‘Elec. Eng.’);
```

### JOIN
JOIN используется для связи двух или более таблиц с помощью общих атрибутов внутри них. На изображении ниже показаны различные способы объединения в SQL. Обратите внимание на разницу между левым внешним объединением и правым внешним объединением:

![13](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/4288859e-c102-4d0b-8b84-bca3207c67f7)

Пример
Выведем список всех обязательных курсов и детали о них:
```sql
SELECT prereq.course_id, title, dept_name, credits, prereq_id
  FROM prereq
  LEFT OUTER JOIN course
  ON prereq.course_id=course.course_id;
```

## Агрегатные функции
Это не совсем основные команды SQL, однако знать их тоже желательно. Агрегатные функции используются для получения совокупного результата, относящегося к рассматриваемым данным:
- COUNT(col_name) — возвращает количество строк;
- SUM(col_name) — возвращает сумму значений в данном столбце;
- AVG(col_name) — возвращает среднее значение данного столбца;
- MIN(col_name) — возвращает наименьшее значение данного столбца;
- MAX(col_name) — возвращает наибольшее значение данного столбца.


## Вопросы
### В чем отличия SQLite и SQL
SQL — язык запросов, с помощью которого специалисты отдают команды для управления базой данных.

SQLite — СУБД (Система управления базами данных), программное обеспечение, которое поддерживает этот язык. Человек, работающий с SQLite, будет использовать для обращения к базе язык запросов SQL. Но сама по себе СУБД намного шире, чем просто обертка для языка, и предоставляет множество других функций.

## Источники
- [Основные команды SQL, которые должен знать каждый программист](https://tproger.ru/translations/sql-recap)
- [Типы соединения](https://learndb.ru/articles/article/30)
- [Все, что необходимо знать про индексы MS SQL](https://otus.ru/journal/vse-chto-neobhodimo-znat-pro-indeksy-ms-sql/)
- [Связи между таблицами базы данных](https://habr.com/ru/articles/488054/)
- [Основные данные и Swift: параллелизм](https://coderlessons.com/articles/mobilnaia-razrabotka-articles/osnovnye-dannye-i-swift-parallelizm)
