# SQL Работа с тренировочной базой данных по продажам книжного интернет магазина
Здесь вы можете ознакомиться с задачами и моим видением их решения c помощью различных функций. 

Со схемой базы данных можно ознакомиться в репозитории в файле "Книжный магазин";

##  Задания:

№1 Вывести номера заказов и названия этапов,  на которых они в данный момент находятся. Если заказ доставлен –  информацию о нем не выводить.

```sql
SELECT buy_id, name_step
FROM step 
     INNER JOIN buy_step USING (step_id)
WHERE buy_step.date_step_beg IS NOT NULL AND buy_step.date_step_end IS NULL
ORDER BY buy_id ASC;
```


№2 Для заказов, прошедших этап транспортировки, вывести количество дней, за которое заказ доставлен в город. А также, если заказ доставлен с опозданием, указать количество дней задержки в отдельном столбце. 

```sql
SELECT buy_id, DATEDIFF(date_step_end, date_step_beg) AS Количество_дней, 
    IF(DATEDIFF(date_step_end, date_step_beg) > city.days_delivery, DATEDIFF(date_step_end, date_step_beg) - city.days_delivery, 0) AS Опоздание
FROM city
     JOIN client USING(city_id)
     JOIN buy USING(client_id)
     JOIN buy_step USING(buy_id)
     JOIN step USING(step_id)
WHERE step_id = 3 AND date_step_end IS NOT NULL;
```


№3
Вывести жанр(ы), в которых было заказано больше всего экземпляров книг, и указать это количество. 

```sql
SELECT name_genre, SUM(buy_book.amount) AS Количество
FROM genre
     INNER JOIN book USING (genre_id)
     INNER JOIN buy_book USING (book_id)
GROUP BY name_genre
HAVING SUM(buy_book.amount) = (
    SELECT MAX(sum_amount) AS max_sum_amount
    FROM
    (SELECT genre_id, SUM(buy_book.amount) AS sum_amount
     FROM buy_book
     INNER JOIN book USING (book_id)
     GROUP BY genre_id) query_in);
```

№4
Вывести автора, название, количество, цену (Розничная_цена). 
Для тех книг количество которых больше или равно 10 - отобразить оптовую скидку 15% и вывести оптовую цену с учетом скидки -15% (Оптовая_цена). 

```sql
SELECT author AS Автор, title AS Название_книги, amount AS Количество, price AS Розничная_цена, IF(amount>=10, 15, 0) AS Скидка, 
ROUND(IF(IF(amount>=10, 15, 0) = 15, price*0.85, price), 2) AS Оптовая_цена
FROM book
ORDER BY author ASC, title ASC;
```

№5
Вывести жанр(ы), в котором было заказано меньше всего экземпляров книг, указать это количество. Учитывать только жанры, в которых была заказана хотя бы одна книга.

```sql
WITH get_genre_buy_amount(name_genre, amount) AS
  (SELECT name_genre, sum(buy_book.amount) as amount
  FROM book
    JOIN buy_book using(book_id)
    JOIN genre using(genre_id)
  GROUP BY name_genre
  HAVING SUM(buy_book.amount) >= 1)
SELECT name_genre, amount as Количество
FROM get_genre_buy_amount 
WHERE amount = (SELECT MIN(amount) 
  FROM get_genre_buy_amount);
```


№6
Снизить цены книг на 20%, цена которых больше 600 рублей. Для тех книг, на которые скидка не действует, в последних двух столбцах вывести символ  "-".  

```sql
SELECT author, title, price, amount, 
       CASE 
       WHEN price >600 THEN ROUND(price*0.2, 2)
       ELSE "-"
       END AS sale_20,
       CASE 
       WHEN price > 600 THEN ROUND(price*0.8, 2)
       ELSE "-"
       END AS price_sale
FROM book
ORDER BY author asc, title ASC;
```

№7 
Завершить этап «Оплата» для заказа под номером 5, вставив в столбец date_step_end дату 13.09.2025, и начать следующий этап («Упаковка»), задав в столбце date_step_beg для этого этапа ту же дату.

```sql
UPDATE buy_step
  JOIN step USING (step_id)
SET buy_step.date_step_end = "2025.09.13"
WHERE  name_step = "Оплата" AND buy_step.buy_id =5;
 
UPDATE buy_step 
  JOIN step USING (step_id)
SET buy_step.date_step_beg = "2025.09.13"
WHERE  name_step = "Упаковка" AND buy_step.buy_id =5;
```

№8 
Удалить из таблиц book и supply книги, цены которых заканчиваются на 99 копеек. 

```sql
DELETE FROM book 
WHERE price LIKE "%.99";

DELETE FROM supply 
WHERE price LIKE "%.99";
```

№9
В последний заказ клиента Баранов Павел добавить по одному экземпляру всех книг Достоевского.

```sql
INSERT INTO buy_book (buy_id, book_id, amount)
SELECT (SELECT MAX(buy_id) FROM buy JOIN client USING (client_id) WHERE name_client = "Баранов Павел") AS buy_id,
book.book_id, 1
FROM book 
    JOIN author USING (author_id)
WHERE name_author LIKE "Достоевский%";
```

### Благодарю за внимание к моей работе. 
Для ознакомления с более сложными и интресными запросами, предлагаю ознакомиться с моим проектом "SQL Работа с тренировочной базой данных по доставке продуктов".




