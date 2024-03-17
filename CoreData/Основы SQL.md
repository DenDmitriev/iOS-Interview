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
