## SQL

### Body Count

**Task:** One of our employees, Jimmie Castora, kept database backups on his computer. DEADFACE compromised his computer and leaked a portion of the database. Can you figure out how many customers are in the 
database? We want to get ahead of this and inform our customers of the breach.

As I started this challenge, I hadn't MySQL installed on my machine and I wanted to procede without installing new software.

For this challenge, I opened the sql file in my text editor and I extracted the part where we inser `consumers`.
```sql
INSERT INTO `customers` VALUES (1,'Hanstock','Juditha','jhanstock0@ebay.co.uk','06 Everett Crossing','Indianapolis','IN','US','46239','F','12/10/1967'),(...),(...) ...
```

I figured out that each consumer is sorrounded by parenthesis, so I just searched for the number of parenthesis in that part and I found the solution : 10000 consumers.

### Keys

**Task:** One of De Monne's database engineers is having issues rebuilding the production database. He wants to know the name of one of the foreign keys on the loans database table. Submit one foreign key name as the flag: flag{foreign-key-name} (can be ANY foreign key).

In the previous challenge, I got around installing a software to manage DB on my computer and I've just used a text editor.
As I am preceding in the sql challenges, it is time to install software to be able to make sql requests.

First, I installed sqlite but I wasn't able to import the database. So, I've decided to launch a MySQL DB and phpMyAdmin using docker-compose.

```yaml
version: '3'
 
services:
  db:
    image: mysql:latest
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: app_db
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "6033:3306"
    volumes:
      - dbdata:/var/lib/mysql
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: pma
    links:
      - db
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_ARBITRARY: 1
    restart: always
    ports:
      - 8081:80
volumes:
  dbdata:
```

Then, I used phpMyAdmin ([http://localhost:8081/]http://localhost:8081/) in my browser to manage the DB.
By default, we cannot import more than 2 Mb sql files using phpMyAdmin. A simple get around is to zip the sql file.

Then in order to found foreign keys on the loans table, I checked the structure page of that table on phpMyAdmin. (You can also use a text editor to check that.)
`fk_loans_loan_type_id`, `fk_loans_employee_id` and `fk_loans_cust_id` are all valid solutions.


![foreign keys](https://raw.githubusercontent.com/hhassen/writeup_deadface/main/images/loan_fk.png)


### City Lights

**Task:** De Monne wants to know how many branch offices were included in the database leak. This can be found by figuring out how many unique cities the employees live in. Submit the flag as flag{#}.

I solved this challenge directly using `distinct values` button on phpMyAdmin without the need to make a sql request.

Here is the corresponding sql request:
```sql
SELECT COUNT(*) FROM (SELECT DISTINCT city FROM customers) as dt; 
```

Anwser : 460 cities.

### Boom
**Task:** DEADFACE actors will be targeting customers they consider low-hanging fruit. Check out Ghost Town and see who they are targeting. Submit the number of target candidates as the flag: flag{#}
**Ghost Town thread:**  We should try to find someone in their late 50s or in their 70s to target. They usually make better targets than younger folks. Try finding a baby boomer. Once you get the database from JC, hmu and we’ll find out which targets are best.
Let’s hit all boomers, not just Michigan. Go big or go home haha.

[https://www.investopedia.com/terms/b/baby_boomer.asp]https://www.investopedia.com/terms/b/baby_boomer.asp

So basically, we need to look for babyboomer' custumers, meaning they are born between 1946 and 1964.

Now it is time to start making sql requests.

```sql
SELECT email, STR_TO_DATE(dob,'%m/%d/%Y'), datediff(now(), STR_TO_DATE(dob,'%m/%d/%Y'))/365.0 AS age , datediff(STR_TO_DATE(dob,'%m/%d/%Y'), STR_TO_DATE('01/01/1946','%m/%d/%Y')) as diff FROM `customers` WHERE (datediff(STR_TO_DATE(dob,'%m/%d/%Y'), STR_TO_DATE('12/31/1964','%m/%d/%Y')) <= 0); 
```
Answer: 2809 consumers.


### El Paso

**Task:** The regional manager for the El Paso branch of De Monne Financial is afraid his customers might be targeted for further attacks. He would like you to find out the dollar value of all outstanding loan balances issued by employees who live in El Paso. Submit the flag as flag{$#,###.##}.

We need a query to select from one table based on a column value in other table.
**alternative 1 : inner join**
```sql
select
    A.*
from
    loans A
inner join employees B
    on A.employee_id = B.employee_id
where
    B.city = 'El Paso';
```

**alternative 2: correlated sub select**

```sql
select
    A.*
from
    loans A
where
    A.employee_id in (
        select B.employee_id from employees B where B.city = 'El Paso'
)
```

The alternative 1 is faster in caluclation time for me.

**Final sum of loans:**
```sql
select sum(A.balance) from loans A inner join employees B on A.employee_id = B.employee_id where B.city = 'El Paso'; 
```

### All A-loan

**Task:** De Monne has reason to believe that DEADFACE will target loans issued by employees in California. It only makes sense that they'll then target the city with the highest dollar value of loans issued. Which city in California has the most money in outstanding Small Business loans? Submit the city and dollar value as the flag in this format: flag{City_$#,###.##}

```sql
select
    sum(A.balance), B.city
from
    loans A
inner join employees B
    on A.employee_id = B.employee_id
where
    B.state = 'CA' AND A.loan_type_id = 3
GROUP BY B.city
ORDER BY sum(A.balance) DESC;
```

`flag{Oakland_$90,600.00}`