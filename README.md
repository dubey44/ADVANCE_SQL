# ADVANCE_SQL_Assignment

```
create database advance_sql_assignment;
use advance_sql_assignment;

CREATE TABLE actor(
actor_id  integer,
first_name VARCHAR(40),
last_name VARCHAR(40),
last_update  timestamp
);
CREATE TABLE address(
address_id  integer,
address VARCHAR(300),
address2 VARCHAR(300),
district varchar(40),
city_id VARCHAR(50),
postal_code VARCHAR(50),
phone VARCHAR(50),
last_update  timestamp
);


CREATE TABLE category(
category_id  INTEGER,
name_  VARCHAR(300),
last_update  TIMESTAMP);

CREATE TABLE city(
city_id  INTEGER,
city  VARCHAR(300),
country_id  INTEGER,
last_update  TIMESTAMP);

CREATE TABLE country(
country_id  INTEGER,
country  VARCHAR(300),
last_update  TIMESTAMP);

CREATE TABLE customer(
customer_id  INTEGER,
store_id  INTEGER,
first_name  VARCHAR(300),
last_name  VARCHAR(300),
email  VARCHAR(300),
address_id  INTEGER,
activebool  BOOLEAN,
create_date  TIMESTAMP,
last_update  TIMESTAMP,
active_  INTEGER);

CREATE TABLE film_actor(
actor_id  INTEGER,
film_id  INTEGER,
last_update  TIMESTAMP);

CREATE TABLE film_category(
film_id  INTEGER,
category_id INTEGER,
last_update  TIMESTAMP);




CREATE TABLE film(
film_id  INTEGER,
title  VARCHAR(300),
description_  VARCHAR(300),
release_year  INTEGER,
language_id  INTEGER,
original_language_id  INTEGER,
rental_duration  INTEGER,
rental_rate  DECIMAL,
length_  INTEGER,
replacement_cost  DECIMAL,
rating  VARCHAR(300),
last_update  TIMESTAMP,
special_features  VARCHAR(300),
fulltext_  VARCHAR(300));

CREATE TABLE inventory(
inventory_id  INTEGER,
film_id  INTEGER,
store_id  INTEGER,
last_update  TIMESTAMP);

CREATE TABLE language(
language_id  INTEGER,
name_  VARCHAR(300),
last_update  TIMESTAMP);

CREATE TABLE payment(
payment_id  INTEGER,
customer_id  INTEGER,
staff_id  INTEGER,
rental_id  INTEGER,
amount  DECIMAL(20,5),
payment_date  TIMESTAMP);

SELECT * FROM PAYMENT

CREATE TABLE rental(
rental_id  INTEGER,
rental_date  TIMESTAMP,
inventory_id  INTEGER,
customer_id  INTEGER,
return_date  TIMESTAMP,
staff_id  INTEGER,
last_update  TIMESTAMP);

CREATE TABLE staff(
staff_id  INTEGER,
first_name  VARCHAR(300),
last_name  VARCHAR(300),
address_id  INTEGER,
email  VARCHAR(300),
store_id  INTEGER,
active_  VARCHAR(10),
username  VARCHAR(300),
password_  VARCHAR(300),
last_update  TIMESTAMP,
picture  VARCHAR(300));

CREATE TABLE store(
store_id  INTEGER,
manager_staff_id  INTEGER,
address_id  INTEGER,
last_update  TIMESTAMP);
```





Q1. Write a query that gives an overview of how many films have replacements costs in the following cost ranges

low: 9.99 - 19.99
medium: 20.00 - 24.99
high: 25.00 - 29.99

```
create view sal_garding
as
select  case when replacement_cost between 9.99 and 19.99 then 'low'
when replacement_cost between 20 and 24.99 then 'medium'
when replacement_cost between 25 and 29.99 then 'high'
end 
as R_costs
from film
where replacement_cost<30;

select R_costs,count(*) from sal_garding
group by R_costs
```

<img width="939" alt="image" src="https://user-images.githubusercontent.com/122452570/223478094-b4b64d12-8758-4b40-ac6e-d6bb4803ff7e.png">


Q2. Write a query to create a list of the film titles including their film title, 
film length and film category name ordered descendingly by the film length.
 Filter the results to only the movies in the category 'Drama' or 'Sports'.
Eg.	"STAR OPERATION"	"Sports"	181
	"JACKET FRISCO"		"Drama"		181  
 

```
SELECT TITLE,LENGTH_,NAME_ AS CATEGORY
FROM FILM F,FILM_CATEGORY FC,CATEGORY C
WHERE F.FILM_ID=FC.FILM_ID AND FC.CATEGORY_ID=C.CATEGORY_ID AND C.NAME_ IN ('SPORTS','DRAMA')
ORDER BY F.LENGTH_ DESC
```

<img width="945" alt="image" src="https://user-images.githubusercontent.com/122452570/223478876-bee88794-27e3-4856-a9a0-b3d82bc2f9f3.png">


    
    
    
    

Q3. Write a query to create a list of the addresses that are not associated to any customer.

```
SELECT ADDRESS 
FROM ADDRESS A 
WHERE A.ADDRESS_ID NOT IN (SELECT ADDRESS_ID FROM CUSTOMER)
```

<img width="937" alt="image" src="https://user-images.githubusercontent.com/122452570/223479125-46265139-5436-45e4-885c-8b9a3883c137.png">



Q4. Write a query to create a list of the revenue (sum of amount) grouped by a column in the format "country, city" ordered in decreasing amount of revenue.
eg. 	"Poland, Bydgoszcz"		52.88

```
SELECT DISTINCT CONCAT(c.country,', ',ct.city) AS country_city_name,
SUM(p.amount) OVER(PARTITION BY c.country, ct.city) AS revenue 
FROM country c,city ct,address ad,customer cu,payment p        
WHERE c.country_id=ct.country_id
AND ad.city_id=ct.city_id
AND cu.address_id=ad.address_id
AND p.customer_id=cu.customer_id
ORDER BY revenue DESC;                                         
```
 
<img width="942" alt="image" src="https://user-images.githubusercontent.com/122452570/223485374-9ebae90e-c1c1-4b97-87d0-cd194b835a01.png">





Q5. Write a query to create a list with the average of the sales amount each staff_id has per customer.
result: 	2	56.64
	1	55.91

```
SELECT DISTINCT d.staff_id,
ROUND(AVG(d.SUM_by_staff_customer) OVER(PARTITION BY d.staff_id),2) AS Avg_sales_each_staff_per_cust
FROM
(
SELECT DISTINCT p.staff_id,p.customer_id,
SUM(p.amount) OVER(PARTITION BY p.staff_id,p.customer_id) AS SUM_by_staff_customer
FROM payment p
) AS d                                         -- d is the alias for this derived table
ORDER BY Avg_sales_each_staff_per_cust DESC;
```

<img width="941" alt="image" src="https://user-images.githubusercontent.com/122452570/223485849-6760c831-f8b0-4ccd-ac5d-94d8b01b9bf0.png">

    
    

Q6. Write a query that shows average daily revenue of all Sundays.


```
SELECT SUM(AMOUNT) /(SELECT  COUNT(DISTINCT DATE(PAYMENT_DATE))  FROM PAYMENT WHERE WEEKDAY(DATE(PAYMENT_DATE))=6) as Average
FROM PAYMENT 
WHERE WEEKDAY((PAYMENT_DATE))=6
```

<img width="942" alt="image" src="https://user-images.githubusercontent.com/122452570/223480322-947215bd-e943-4601-8e74-bbcc66ffd439.png">


Q7. Write a query to create a list that shows how much the 
average customer spent in total (customer life-time value) grouped by the different districts.

```
SELECT DISTRICT Dist, 
SUM(AMOUNT)/(SELECT COUNT(CUSTOMER_ID) 
FROM CUSTOMER  JOIN ADDRESS ON CUSTOMER.ADDRESS_ID=ADDRESS.ADDRESS_ID WHERE DISTRICT=Dist) AVERAGE
FROM PAYMENT 
JOIN CUSTOMER ON PAYMENT.CUSTOMER_ID=CUSTOMER.CUSTOMER_ID
JOIN ADDRESS ON CUSTOMER.ADDRESS_ID=ADDRESS.ADDRESS_ID
GROUP BY DISTRICT
ORDER BY AVERAGE DESC
```

<img width="934" alt="image" src="https://user-images.githubusercontent.com/122452570/223481197-adc56cf5-68c7-438b-9bcb-7b30b3a23f06.png"> 



Q8. Write a query to list down the highest overall revenue collected 
(sum of amount per title) by a film in each category. 
Result should display the film title, category name and total revenue.
eg. 	"FOOL MOCKINGBIRD"		"Action"	175.77
	"DOGMA FAMILY"			"Animation"	178.7
	"BACKLASH UNDEFEATED"	"Children"	158.81
 
 ```
CREATE VIEW TOTAL_REV AS(
SELECT F.TITLE X, C.NAME_ Y, ROUND(SUM(P.AMOUNT),2) T
FROM FILM F,INVENTORY I, RENTAL R, FILM_CATEGORY FC,PAYMENT P, CATEGORY C
WHERE F.FILM_ID=I.FILM_ID AND I.INVENTORY_ID=R.INVENTORY_ID
AND R.RENTAL_ID=P.RENTAL_ID
AND F.FILM_ID=FC.FILM_ID
AND FC.CATEGORY_ID=C.CATEGORY_ID      
GROUP BY F.TITLE, C.NAME_ );
      
 CREATE VIEW RANKING AS     
(SELECT  TOTAL_REV.X NAME_,TOTAL_REV.Y CATEGORY_, TOTAL_REV.T, RANK() OVER(PARTITION BY TOTAL_REV.Y ORDER BY TOTAL_REV.T DESC) RANKS
FROM TOTAL_REV);

SELECT * FROM RANKING
WHERE RANKS=1;
```
 
 
 <img width="939" alt="image" src="https://user-images.githubusercontent.com/122452570/223481645-57ff0325-ac07-4df8-8ba0-6e06ed3d6494.png">


Q9. Modify the table "rental" to be partitioned using PARTITION command based on ‘rental_date’ in below intervals:
	<2005
between 2005–2010
	between 2011–2015
	between 2016–2020
	>2020 - Partitions are created yearly
 
 ```
ALTER TABLE rental 
PARTITION BY RANGE(YEAR(rental_date))
(
PARTITION rental_less_than_2005 VALUES LESS THAN (2005),
PARTITION rental_between_2005_2010 VALUES LESS THAN (2011),
PARTITION rental_between_2011_2015 VALUES LESS THAN (2016),
PARTITION rental_between_2016_2020 VALUES LESS THAN (2021),
PARTITION rental_greater_than_2020 VALUES LESS THAN MAXVALUE
);
```
    
    
    

Q10. Modify the table "film" to be partitioned using PARTITION command based on ‘rating’ from below list. Further apply hash sub-partitioning based on ‘film_id’ into 4 sub-partitions.

partition_1 - "R"
partition_2 - "PG-13", "PG"
partition_3 - "G", "NC-17"

```
ALTER TABLE film
PARTITION BY LIST(rating)
SUBPARTITION BY HASH(film_id) SUBPARTITIONS 4
(
PARTITION PR values('R'),
PARTITION Pgs values('PG-13', 'PG'),
PARTITION GNC values('G', 'NC-17')
);
```


Q11. Write a query to count the total number of addresses from the “address” table where the ‘postal_code’ is of the below formats.
Use regular expression.

9*1**, 9*2**, 9*3**, 9*4**, 9*5**

eg. postal codes - 91522, 80100, 92712, 60423, 91111, 9211
result - 2

```
SELECT count(postal_code) 
FROM address
WHERE postal_code REGEXP '^9[0-9][1-5][0-9]{2}'; 
```

<img width="937" alt="image" src="https://user-images.githubusercontent.com/122452570/223484089-0c2aad62-6b06-4a22-bc3b-8572262f1132.png">


Q12. Write a query to create a materialized view from the “payment” table 
where ‘amount’ is between(inclusive) $5 to $8. 
The view should manually refresh on demand. Also write a query to manually refresh the created materialized view.

```
DELIMITER $$
CREATE EVENT refresh_payment_between_5_8    
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  CREATE OR REPLACE VIEW payment_between_5_8 AS
  SELECT *
  FROM payment
  WHERE amount BETWEEN 5 AND 8;
END$$
DELIMITER ;

SELECT * FROM payment_between_5_8;
```


<img width="938" alt="image" src="https://user-images.githubusercontent.com/122452570/223483853-08575c9c-5b52-43bf-af37-e9752a863533.png">



Q13. Write a query to list down the total sales of each staff with each customer from the ‘payment’ table.
 In the same result, list down the total sales of each staff i.e. sum of sales from all customers 
 for a particular staff. Use the ROLLUP command. Also use GROUPING command to indicate null values.
 
 ```
 SELECT p.staff_id,p.customer_id,
GROUPING(p.staff_id) as staff,
GROUPING(p.customer_id) as customer,sum(p.amount) as sum_of_sales
FROM payment p
GROUP BY p.staff_id,p.customer_id
WITH ROLLUP;
```

<img width="933" alt="image" src="https://user-images.githubusercontent.com/122452570/223483351-5a1f1f1e-6136-4290-ba35-a69c403a59f1.png">

 
 

**Q.14 Write a single query to display the customer_id, staff_id, payment_id, amount, amount on immediately previous payment_id, 
amount on immediately next payment_id ny_sales for the payments from customer_id ‘269’ to  staff_id ‘1’.**

```
select customer_id,payment_id,staff_id,
lead(amount) over(order by payment_id) next_payment,
lag(amount) over(order by payment_id) previous_amount,
lead(amount) over(Partition by customer_id,staff_id ORDER BY payment_id) as ny_sales,
lag(amount) over(Partition by customer_id,staff_id ORDER BY payment_id) as py_sales
from payment
where customer_id=269 and staff_id=1;
```

<img width="934" alt="image" src="https://user-images.githubusercontent.com/122452570/223482756-b728ab82-773a-4fd9-9fda-f4de2ef7b9f6.png">


