<br><a name="T-1"></a>

# 2   Запросы SQL к связанным таблицам

## 2.1 Связи между таблицами

Связь «**один ко многим**» имеет место, когда одной записи главной таблицы соответствует несколько записей связанной таблицы, а каждой записи связанной таблицы соответствует только одна запись главной таблицы.

Связь «**многие ко многим**» имеет место когда каждой записи одной таблицы соответствует несколько записей во второй, и наоборот, каждой записи второй таблицы соответствует несколько записей в первой. 

![пример связи](/images/svaz.png)

### Создание таблицы с внешними ключами
пример: создать таблицу book по  логической схеме
![пример связи](/images/2.1.jpg)
```sql
CREATE TABLE book (
    book_id INT PRIMARY KEY AUTO_INCREMENT, 
    title VARCHAR(50), 
    author_id INT NOT NULL, 
    genre_id INT, 
    FOREIGN KEY (author_id)  REFERENCES author (author_id),
    FOREIGN KEY (genre_id)  REFERENCES genre (genre_id),
    price DECIMAL(8,2), 
    amount INT
);
```

### Действия при удалении записи главной таблицы

С помощью выражения **```ON DELETE```** можно установить действия, которые выполняются для записей подчиненной таблицы при удалении связанной строки из главной таблицы. При удалении можно установить следующие опции:

* **```CASCADE```**: автоматически удаляет строки из зависимой таблицы при удалении  связанных строк в главной таблице.
* **```SET NULL```**: при удалении  связанной строки из главной таблицы устанавливает для столбца внешнего ключа значение NULL. (В этом случае столбец внешнего ключа должен поддерживать установку NULL).
* **```SET DEFAULT```** похоже на SET NULL за тем исключением, что значение  внешнего ключа устанавливается не в NULL, а в значение по умолчанию для данного столбца.
* **```RESTRICT```**: отклоняет удаление строк в главной таблице при наличии связанных строк в зависимой таблице.

***Важно!*** Если для столбца установлена опция **``SET NULL``**, то при его описании нельзя задать ограничение на пустое значение.

**Пример**
Будем считать, что при удалении автора из таблицы author, необходимо удалить все записи о книгах из таблицы book, написанные этим автором. Данное действие необходимо прописать при создании таблицы.

Запрос:
```sql
CREATE TABLE book (
    book_id INT PRIMARY KEY AUTO_INCREMENT, 
    title VARCHAR(50), 
    author_id INT NOT NULL, 
    price DECIMAL(8,2), 
    amount INT, 
    FOREIGN KEY (author_id)  REFERENCES author (author_id) ON DELETE CASCADE
);
```

## 2.2 Запросы на выборку, соединение таблиц

### Соединение INNER JOIN

Оператор внутреннего соединения **``INNER JOIN``** соединяет две таблицы. Порядок таблиц для оператора неважен, поскольку оператор является симметричным.

Результат запроса формируется так:

* каждая строка одной таблицы сопоставляется с каждой строкой второй таблицы;
* для полученной «соединённой» строки проверяется условие соединения;
* если условие истинно, в таблицу результата добавляется соответствующая «соединённая» строка;

```sql
SELECT
 ...
FROM
    таблица_1 INNER JOIN  таблица_2
    ON условие
...
```
**Пример**
Вывести название книг и их авторов.
Запрос:
```sql
SELECT title, name_author
FROM 
    author INNER JOIN book
    ON author.author_id = book.author_id;
```
### Внешнее соединение LEFT и RIGHT OUTER JOIN

Оператор внешнего соединения **``LEFT OUTER JOIN``**  (можно использовать LEFT JOIN) соединяет две таблицы. Порядок таблиц для оператора важен, поскольку оператор не является симметричным.

```sql
SELECT
 ...
FROM
    таблица_1 LEFT JOIN  таблица_2
    ON условие
...
```
Результат запроса формируется так:

* в результат включается внутреннее соединение (INNER JOIN) первой и второй таблицы в соответствии с условием;
* затем в результат добавляются те записи первой таблицы, которые не вошли во внутреннее соединение на шаге 1, для таких записей соответствующие поля второй таблицы заполняются значениями NULL.

Пример:
Вывести все жанры, которые не представлены в книгах на складе.
```sql
select name_genre
from genre left join book
        on genre.genre_id = book.genre_id
where book.amount is null
```


### Перекрестное соединение CROSS JOIN
Оператор перекрёстного соединения, или декартова произведения CROSS JOIN (в запросе вместо ключевых слов можно поставить запятую между таблицами) соединяет две таблицы. Порядок таблиц для оператора неважен, поскольку оператор является симметричным. Его структура:
```sql
SELECT
 ...
FROM
    таблица_1 CROSS JOIN  таблица_2
...
```
или
```sql
SELECT
 ...
FROM
    таблица_1, таблица_2
...
```
* Результат запроса формируется так: каждая строка одной таблицы соединяется с каждой строкой другой таблицы, формируя  в результате все возможные сочетания строк двух таблиц.

пример
Необходимо в каждом городе провести выставку книг каждого автора в течение 2020 года. Дату проведения выставки выбрать случайным образом. Создать запрос, который выведет город, автора и дату проведения выставки. Последний столбец назвать Дата. Информацию вывести, отсортировав сначала в алфавитном порядке по названиям городов, а потом по убыванию дат проведения выставок.
```sql
Таблица genre:
+----------+-------------+
| genre_id | name_genre  |
+----------+-------------+
| 1        | Роман       |
| 2        | Поэзия      |
| 3        | Приключения |
+----------+-------------+

Таблица author:
+-----------+------------------+
| author_id | name_author      |
+-----------+------------------+
| 1         | Булгаков М.А.    |
| 2         | Достоевский Ф.М. |
| 3         | Есенин С.А.      |
| 4         | Пастернак Б.Л.   |
| 5         | Лермонтов М.Ю.   |
+-----------+------------------+

Таблица city:
+---------+-----------------+
| city_id | name_city       |
+---------+-----------------+
| 1       | Москва          |
| 2       | Санкт-Петербург |
| 3       | Владивосток     |
+---------+-----------------+
```

```sql
select name_city, name_author, date_add('2020-01-01', interval FLOOR(RAND() * 365 ) day) as Дата 
from city cross join author                       
order by  city.name_city, Дата desc;
```
### Запросы на выборку из нескольких таблиц

Пусть таблицы связаны между собой следующим образом:
![пример связи](/images/2.2.1.jpg)
тогда запрос на выборку для этих таблиц будет иметь вид:
```sql
SELECT
 ...
FROM
    first 
    INNER JOIN  second ON first.first_id = second.first_id
    INNER JOIN  third  ON second.second_id = third.second_id
...
```

Если же таблицы связаны так:
![пример связи](/images/2.2.2.jpg)
то запрос на выборку выглядит следующим образом:
```sql
SELECT
 ...
FROM
    first 
    INNER JOIN  third ON first.first_id = third.first_id
    INNER JOIN second ON third.second_id = second.second_id 
...
```
**Пример**:  Вывести информацию о книгах (жанр, книга, автор), относящихся к жанру, включающему слово «роман» в отсортированном по названиям книг виде.
Решение:
```sql
select name_genre, title, name_author  
from 
    genre
    inner join book on book.genre_id = genre.genre_id
    inner join author on book.author_id = author.author_id
where genre.name_genre like '%Роман%'
order by book.title;
```

### Запросы для нескольких таблиц с группировкой
В запросах с групповыми функциями могут использоваться несколько таблиц, между которыми используются различные типы соединений.

**Пример**
Посчитать количество экземпляров  книг каждого автора из таблицы author.  Вывести тех авторов,  количество книг которых меньше 10, в отсортированном по возрастанию количества виде. Последний столбец назвать Количество.

Запрос:
```sql
select name_author, sum(amount) as Количество
from
    author left join book
    on book.author_id  = author.author_id
group by author.name_author
having Количество  < 10 or Количество is null
order by Количество
```

### Запросы для нескольких таблиц со вложенными запросами

**пример**: Вывести в алфавитном порядке всех авторов, которые пишут только в одном жанре. Поскольку у нас в таблицах так занесены данные, что у каждого автора книги только в одном жанре,  для этого запроса внесем изменения в таблицу book. Пусть у нас  книга Есенина «Черный человек» относится к жанру «Роман», а книга Булгакова «Белая гвардия» к «Приключениям» (эти изменения в таблицы уже внесены).

```sql
select author.name_author
from book inner join author
on book.author_id = author.author_id
group by book.author_id
having count(distinct(genre_id)) = 1
```

### Вложенные запросы в операторах соединения
Вложенные запросы могут использоваться в операторах соединения JOIN.  При этом им необходимо присваивать имя, которое записывается сразу после закрывающей скобки вложенного запроса.
```sql
SELECT
 ...
FROM
    таблица ... JOIN  
       (
        SELECT ...
       ) имя_вложенного_запроса
    ON условие
...
```
**пример**: Вывести информацию о книгах (название книги, фамилию и инициалы автора, название жанра, цену и количество экземпляров книги), написанных в самых популярных жанрах, в отсортированном в алфавитном порядке по названию книг виде. Самым популярным считать жанр, общее количество экземпляров книг которого на складе максимально.
```sql


select title, name_author, name_genre, price, amount
from 
    author
    inner join book on book.author_id = author.author_id
    inner join genre on book.genre_id = genre.genre_id
where book.genre_id in 
    (
    select q1.genre_id
    from 
        ( -- код книги  и количество экземпляров кнгиг которые отностятся к жанру этой книги
            select genre_id, sum(amount) as sum_amount
            from  book
            group by genre_id
        )q1
        inner join
        ( -- максимальное кол-во экземляров книг у одного жанра 
            select genre_id, sum(amount) as sum_amount
            from book
            group by genre_id
            order by  sum_amount desc
            limit 1
        )q2
        on q1.sum_amount = q2.sum_amount
    )
order by title 
```
### Операция соединение, использование USING()

При описании соединения таблиц с помощью **```JOIN```** в некоторых случаях вместо **```ON```** и следующего за ним условия можно использовать оператор **```USING()```**.

**```USING()```** позволяет указать набор столбцов, которые есть в обеих объединяемых таблицах. Если база данных хорошо спроектирована, а каждый внешний ключ имеет такое же имя, как и соответствующий первичный ключ (например, genre.genre_id = book.genre_id), тогда можно использовать предложение **```USING()```** для реализации операции **```JOIN```**. 

При этом после SELECT, при использовании столбцов из USING(), необязательно указывать, из какой именно таблицы берется столбец.

**Пример:** Вывести название книг, фамилии и id их авторов.
Запрос:
```sql
Вариант с ON

SELECT title, name_author, author.author_id /* явно указать таблицу - обязательно */
FROM 
    author INNER JOIN book
    ON author.author_id = book.author_id;
Вариант с USING

SELECT title, name_author, author_id /* имя таблицы, из которой берется author_id, указывать не обязательно*/
FROM 
    author INNER JOIN book
    USING(author_id);
```
## 2.3 Запросы корректировки, соединение таблиц

### Запросы на обновление, связанные таблицы
В запросах на обновление можно использовать связанные таблицы:
```sql
UPDATE таблица_1
     ... JOIN таблица_2
     ON выражение
     ...
SET ...   
WHERE ...;
```
***Пример***
Для книг, которые уже есть на складе (в таблице book) по той же цене, что и в поставке (supply), увеличить количество на значение, указанное в поставке, а также обнулить количество этих книг в поставке.

Этот запрос должен отобрать строки из таблиц bookи supply такие, что у них совпадают и автор, и название книги. Но в таблице supply фамилия автора записана не числом (id), а текстом. Следовательно, чтобы выполнить сравнение по фамилии автора нужно "подтянуть" таблицу author,  которая связана с bookпо столбцу author_id.  И в логическом выражении, описывающем соединение таблиц, можно будет использовать столбцы из таблиц book, authorи supply. 

Если таблицы логически связаны по двум и более столбцам (на рисунке связи обозначены линиями), возможно через другие таблицы, условие соединение будет включать связи по нужным столбцам через логический оператор AND. Например, для следующих таблиц логическую связь по названию и автору:
![пример связи](/images/2.3.1.png)

условие соединения можно записать в виде:
```sql
book INNER JOIN author ON author.author_id = book.author_id
     INNER JOIN supply ON book.title = supply.title 
                          and supply.author = author.name_author
```
Запрос:
```sql
UPDATE book 
     INNER JOIN author ON author.author_id = book.author_id
     INNER JOIN supply ON book.title = supply.title 
                         and supply.author = author.name_author
SET book.amount = book.amount + supply.amount,
    supply.amount = 0   
WHERE book.price = supply.price;

SELECT * FROM book;
SELECT * FROM supply
```

### Запросы на добавление, связанные таблицы

Запросом на добавление можно добавить записи, отобранные с помощью запроса на выборку, который включает несколько таблиц:
```sql
INSERT INTO таблица (список_полей)
SELECT список_полей_из_других_таблиц
FROM 
    таблица_1 
    ... JOIN таблица_2 ON ...
    ...
```
**Пример** Включить новых авторов в таблицу author с помощью запроса на добавление, а затем вывести все данные из таблицы author.  Новыми считаются авторы, которые есть в таблице supply, но нет в таблице author.
```sql
insert into author (name_author)
SELECT supply.author
FROM 
    author 
    RIGHT JOIN supply on author.name_author = supply.author
WHERE name_author IS Null;

select * from author
```

### Запрос на добавление, связанные таблицы(2)

Пример:Добавить новые книги из таблицы supply в таблицу book на основе сформированного выше запроса. Затем вывести для просмотра таблицу book.
```sql
insert into book (title, author_id, price, amount)
SELECT title, author_id, price, amount
FROM 
    author 
    INNER JOIN supply ON author.name_author = supply.author
WHERE amount <> 0;
select * from book
```
### Запрос на обновление, вложенные запросы

**Пример:**Задать для книги Пастернака «Доктор Живаго»  жанр «Роман».
Если мы знаем код этой книги в таблице book (в нашем случае это 9) и код жанра «Роман» в таблице genre (это 1), запрос будет очень простым.
Более сложным будет запрос, если известно только название жанра (результат будет точно таким же):
```sql
UPDATE book
SET genre_id = 
      (
       SELECT genre_id 
       FROM genre 
       WHERE name_genre = 'Роман'
      )
WHERE book_id = 9;

SELECT * FROM book;
```

### Каскадное удаление записей связанных таблиц
При создании таблицы для внешних ключей с помощью **```ON DELETE```** устанавливаются опции, которые определяют действия , выполняемые при удалении связанной строки из главной таблицы.
В частности, **```ON DELETE CASCADE```** автоматически удаляет строки из зависимой таблицы при удалении  связанных строк в главной таблице.
В таблице book эта опция установлена для поля author_id.
**Пример** Удалить всех авторов и все их книги, общее количество книг которых меньше 20.


Запрос:
```sql
-- select * from author;
delete from  author
where author.author_id in
    (
    select author_id
    from book
    group by author_id
    having sum(amount)  < 20
);

-- select * from author;
```
### Удаление записей, использование связанных таблиц
При удалении записей из таблицы можно использовать информацию из других связанных с ней таблиц. В этом случае синтаксис запроса имеет вид:
```sql
DELETE FROM таблица_1
USING 
    таблица_1 
    INNER JOIN таблица_2 ON ...
WHERE ...
```
**Пример:**Удалить всех авторов, которые пишут в жанре "Поэзия". Из таблицы book удалить все книги этих авторов. В запросе для отбора авторов использовать полное название жанра, а не его id.
```sql
DELETE FROM author
USING 
    author 
    INNER JOIN book ON author.author_id = book.author_id
    INNER JOIN genre ON book.genre_id = genre.genre_id

WHERE genre.name_genre = 'Поэзия';

SELECT * FROM author;
SELECT * FROM book;
```






[**_НАЗАД_**](#T-1)</a>