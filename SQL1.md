# Домашнее задание к занятию "`SQL. Часть 1`" - `Исаков Михаил`

Задание можно выполнить как в любом IDE, так и в командной строке.

### Задание 1

Получите уникальные названия районов из таблицы с адресами, которые начинаются на “K” и заканчиваются на “a” и не содержат пробелов.

#### *Ответ:*
```sql
SELECT DISTINCT district
FROM address
WHERE district LIKE 'K%a' AND district NOT LIKE '% %';
```
![z](https://github.com/632456/hw-02/blob/main/SQL1/1.png)

### Задание 2

Получите из таблицы платежей за прокат фильмов информацию по платежам, которые выполнялись в промежуток с 15 июня 2005 года по 18 июня 2005 года **включительно** и стоимость которых превышает 10.00.

#### *Ответ:*
```sql
SELECT payment_date, amount
FROM payment
WHERE amount > 10 AND payment_date BETWEEN '2005-06-15 00:00:00' AND '2005-06-18 23:59:59';
```
![z](https://github.com/632456/hw-02/blob/main/SQL1/2.png)

### Задание 3

Получите последние пять аренд фильмов.

#### *Ответ:*
```sql
SELECT rental_date
FROM rental
ORDER BY rental_date DESC
LIMIT 5;
```
![z](https://github.com/632456/hw-02/blob/main/SQL1/3.png)
### Задание 4

Одним запросом получите активных покупателей, имена которых Kelly или Willie. 

Сформируйте вывод в результат таким образом:
- все буквы в фамилии и имени из верхнего регистра переведите в нижний регистр,
- замените буквы 'll' в именах на 'pp'.

#### *Ответ:*
```sql
SELECT CONCAT((REPLACE(LOWER(first_name), 'll', 'pp')), " ", LOWER(last_name)) AS 'Имя и фамилия' , active
FROM customer
WHERE active = 1 AND first_name IN ('Kelly', 'Willie');
```
![z](https://github.com/632456/hw-02/blob/main/SQL1/4.png)