
# SQL Learnings from Leetcode SQL50

## Day 1
* Relearnt CTE syntax in SQL

* **USING keyword**: If we want to join 2 tables on one or more columns that have the exact same name in both the tables, then we can use the USING keyword instead of ON keyword. Remember that the join column should be within parenthesis when we use USING


```
-- https://leetcode.com/problems/replace-employee-id-with-the-unique-identifier
with emp as (
    select * from Employees
),
emp_uni as (
    select * from EmployeeUNI
)

select 
emp.name,
emp_uni.unique_id
from emp left emp_uni using (id)

```
* One major advantage of the USING keyword is we do not need to qualify the join column with the table alias (infact we should not qualify with alias even if the same column is used elsewhere in the SQL statement, refer 3). While using ON keyword, if we do not qualify the join column with the alias, it gives the error Column name 'id' is ambiguous i.e. it does not know which of the 2 tables to pick the column from


* When we want to filter a column in one table based on common column in another table but not having all values, we can either do a join on that column and then filter nulls (LEFT JOIN + IS NULL) or we can directly use NOT IN, as shown below 

```
-- https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions
-- Goal : To find the IDs of the customers who visited the mall but did not make any transactions and the number of times they made these types of visits (Visits table has customer_id and visit_id, Transactions table has visit_id and transaction_id)
-- Approach 1

with 
Visits as (select * from Visits),
Transactions as (select * from Transactions)

select 
customer_id
,count(*) as count_no_trans
from Visits 
left join Transactions using (visit_id)
where Transactions.transaction_id is null
group by 1

-- Approach 2
SELECT customer_id, COUNT(visit_id) as count_no_trans 
FROM Visits
WHERE visit_id NOT IN (
    SELECT visit_id FROM Transactions
    )
GROUP BY customer_id

```

* We cannot combine USING and ON keyword in the same statement (refer 2). 


### References
1. https://stackoverflow.com/questions/11366006/mysql-join-on-vs-using
2. https://stackoverflow.com/questions/52206667/can-the-sql-join-using-clause-be-used-with-one-or-more-and-on-clauses-fo
3. https://www.geeksforgeeks.org/sql-using-clause/

## Day 2

* Cross join generates all possible combinations of values/rows. Cross join is useful when we want to create a grid like structure with all possible combinations. In the example below, we want to create a grid with all possible combinations of Students and Subjects, and then use that to see how many times a student has taken an exam. Very useful to find cases when student has not taken an exam even once, which would not be captured without a cross join

```

-- Without CTE
select 
    StudentsAllSubjects.student_id, 
    StudentsAllSubjects.student_name, 
    StudentsAllSubjects.subject_name, 
    count(Examinations.student_id) as attended_exams
from
(select * from Students cross join Subjects) as StudentsAllSubjects
left join Examinations 
on 
StudentsAllSubjects.student_id = Examinations.student_id 
and StudentsAllSubjects.subject_name = Examinations.subject_name

group by 1,2,3
order by 1,3

-- With CTE

with StudentsAllSubjects as (
    select * from Students cross join Subjects
)

select 
    a.student_id, 
    a.student_name, 
    a.subject_name, 
    coalesce(count(e.student_id), 0) as attended_exams
from
    StudentsAllSubjects as a

left join Examinations e
on 
a.student_id = e.student_id 
and a.subject_name = e.subject_name

group by 1,2,3
order by 1,3

```

* Using CTE can make code more readable as shown above. Also remember that while using CTE, we use WITH keyword only once in the beginning, not before each table

```
-- wrong syntax
with Visits as (select * from Visits),
with Transactions as (select * from Transactions)

select * from Visits left join Transactions using (id)

```

* Coalesce returns first non null argument in list of arguments
```
SELECT COALESCE(NULL, 1, 2, 'W3Schools.com'); -- Output:1

```

* COALESCE vs IFNULL : ifnull takes 2 arguments, whereas coalesce can take 2 or more than 2 arguments. Ifnull returns the first argument if it is not null, and the second argument if the first argument is null. coalesce return the first non null argument in the list of arguments, and if all the arguments are null, it returns null

### References
1. https://stackoverflow.com/questions/219716/what-are-the-uses-for-cross-join
2. https://stackoverflow.com/questions/8974328/mysql-multiple-joins-in-one-query

## Day 3

* Self join : You use a self join when a table references data in itself (for example managerId references to employeeId in the same table). In other words, foreign key refers to a primary key in the same table


```
-- https://leetcode.com/problems/managers-with-at-least-5-direct-reports/?envType=study-plan-v2&envId=top-sql-50
with Managers as (
    select managerId from Employee
    group by managerId
    having count(*) >= 5
)
select e.name 
from Employee e 
inner join Managers m 
on e.id = m.managerId

```

* Conditional aggregation in SQL is very useful when we want percentage of each category in column (say colA - action) while grouping by another column (say colB - user_id) and not the category column itself

```

-- https://leetcode.com/problems/confirmation-rate/description/?envType=study-plan-v2&envId=top-sql-50

select user_id, 
    round(coalesce(sum(case when action = "confirmed" then 1 else 0 end), 0) * 1.0 / count(*), 2)as confirmation_rate
from
Signups left join Confirmations 
using (user_id)
group by user_id

```
### References
1. https://stackoverflow.com/questions/770579/how-to-calculate-percentage-with-a-sql-statement
2. https://stackoverflow.com/questions/51489103/how-do-i-get-percentage-amount-of-categorical-variables-per-day-using-sql
3. https://stackoverflow.com/questions/3362038/what-is-self-join-and-when-would-you-use-it
4. https://stackoverflow.com/questions/18680680/can-a-foreign-key-refer-to-a-primary-key-in-the-same-table

## Day 4

* Use the mod function or % to find modulus of number by another number
* Use round function to round of result to a given number of decimal places

```
-- https://leetcode.com/problems/not-boring-movies/?envType=study-plan-v2&envId=top-sql-50

select * 
from Cinema
where id % 2 = 1 and lower(description) <> "boring"
order by rating desc


-- https://leetcode.com/problems/project-employees-i/?envType=study-plan-v2&envId=top-sql-50
select 
    project_id, 
    round(avg(experience_years), 2) as average_years
from 
    Project 
left join 
    Employee 
using (employee_id)
group by 1

```

## Day 5

* Note that select count(user_id) from Users although an expression returns a constant value and does not depend on the group by fields

```
-- Approach 1
select 
    contest_id, 
    round((count(*) * 100) / (select count(*) from Users), 2) as percentage
from
Register 
left join Users
using (user_id)
group by contest_id
order by percentage desc, contest_id

-- Approach 2 (better approach)
-- left join is not required in this scenario
-- also we need to do distinct count
select 
    contest_id, 
    round((count(distinct user_id) * 100) / (select count(user_id) from Users), 2) as percentage
from
    Register 
group by 
    contest_id
order by 
    percentage desc, 
    contest_id asc
```

### References
1. https://stackoverflow.com/questions/2051162/sql-multiple-column-ordering


## Day 6

* To format date to a specific format, use date_format function


```
-- https://leetcode.com/problems/monthly-transactions-i/?envType=study-plan-v2&envId=top-sql-50
-- Approach 1
select
  left(trans_date, 7) as month,
  country,
  count(id) as trans_count,
  sum(case when state="approved" then 1 else 0 end) as approved_count,
  sum(amount) as trans_total_amount,
  sum(case when state="approved" then amount else 0 end) as approved_total_amount
from Transactions
group by month, country


-- Approach 2
select
  date_format(trans_date, '%Y-%m') as month,
  country,
  count(id) as trans_count,
  sum(case when state="approved" then 1 else 0 end) as approved_count,
  sum(amount) as trans_total_amount,
  sum(case when state="approved" then amount else 0 end) as approved_total_amount
from Transactions
group by month, country

```

## Day 7

* Cannot use next_login in the where clause of corresponding to the select, hence had to put it as a subquery

* Need to alias the sUbquery to select from it, else an error

```
# 12/15 cases passed
select round(count(*) / (select count(distinct player_id) from Activity),2) as fraction 
from 
    (select * from 
        (select 
            player_id, 
            event_date, 
            lead(event_date) over (order by player_id, event_date) as next_login
         from Activity) a1
    where event_date = (select min(event_date) from Activity a2 where a1.player_id=a2.player_id) and datediff(event_date, next_login) = -1
    ) x

# 14/15 cases passed
select round(count(distinct player_id)/(select count(distinct player_id) from Activity),2) as fraction 
from 
    (select * from 
        (select player_id, 
        event_date, 
        lead(event_date) over (order by player_id, event_date) as next_login
        from Activity) a1
    where event_date = (select min(event_date) from Activity a2 where a1.player_id=a2.player_id) and datediff(event_date, next_login) = -1
    )x

```

## Day 8

*  To compare two dates we can use arithmetic operators like >, >=, <, <=, == and <>. For these operators to work, the two dates should be in the ANSI-standard YYYY-MM-DD format. Other keywords for comparison are BETWEEN, ISNULL, DATEDIFF


* Be careful about the edge cases i.e. >= and >, in this case since 27th July 2019 is included, it should be > with -30 days and >= with -29 days

```
-- https://leetcode.com/problems/user-activity-for-the-past-30-days-i/?envType=study-plan-v2&envId=top-sql-50

-- Approach 1 (using date_add)

select 
    activity_date as day,
    count(distinct user_id) as active_users
from Activity
where activity_date <= '2019-07-27' and activity_date > date_add('2019-07-27', INTERVAL -30 DAY)
group by activity_date

-- Approach 2 (using datediff)
-- TO DO : Add sql code using datediff approach

```

* Dateadd syntax is different in mysql compared to other databases


### References
1. https://www.dbvis.com/thetable/how-to-compare-sql-dates/

## Day 9

*  We can use WHERE...IN with multiple columns simulataneously i.e. 

```

SELECT * FROM tableA
WHERE
   ([C1], [C2],..., [Cn]) IN (SELECT [D1], [D2],..., [Dn] FROM tableB)

```
In some databases like SQL Server this is not supported. In that case, you can use EXISTS, as mentioned in reference 1

* Correlated subqueries (which is used in approach 1) are very slow. 

```

-- https://leetcode.com/problems/product-sales-analysis-iii/?envType=study-plan-v2&envId=top-sql-50
-- Approach 1 (was timing out for some test cases)
select product_id, year as first_year, quantity, price
from Sales s1
where year = (
    select min(year) from Sales s2 where s1.product_id = s2.product_id
)

-- Apporach 2
select product_id, year as first_year, quantity, price
from Sales 
where (product_id, year) in (
    select product_id, min(year) from Sales group by product_id
)


```

### Doubts
1. How do you define a correlated subquery? What are some situations in which only correlated subquery works?


### References
1. https://dba.stackexchange.com/questions/247382/where-in-based-on-multiple-columns


## Day 10

* WHERE vs GROUP BY : Use Having clause when you are to filter out on the table which is already grouped by. Use Where clause when you are to filter out on the table before group by.

```

-- https://leetcode.com/problems/number-of-unique-subjects-taught-by-each-teacher/?envType=study-plan-v2&envId=top-sql-50
select 
    teacher_id, 
    count(distinct subject_id) as cnt
from Teacher
group by teacher_id

-- https://leetcode.com/problems/classes-more-than-5-students/?envType=study-plan-v2&envId=top-sql-50

select class 
from Courses
group by class
having count(student) >= 5


-- https://leetcode.com/problems/find-followers-count/?envType=study-plan-v2&envId=top-sql-50


select user_id, count(follower_id) as followers_count
from Followers
group by user_id
order by user_id asc


-- https://leetcode.com/problems/biggest-single-number/submissions/1482926065/?envType=study-plan-v2&envId=top-sql-50

--- Approach 1 (using CTE)
with SingleNumbers as (
    select num
    from MyNumbers
    group by num
    having count(*) = 1
)
select max(num) as num from SingleNumbers


--- Approach 2 (using subqueries)
TO DO : Add subqueries approach

-- https://leetcode.com/problems/customers-who-bought-all-products/submissions/1482940486/?envType=study-plan-v2&envId=top-sql-50
-- can we solve this problem without using count
select customer_id 
from Customer
group by customer_id
having count(distinct product_key) = (select count(product_key) from Product)

```

## Day 11

```
-- https://leetcode.com/problems/the-number-of-employees-which-report-to-each-employee/?envType=study-plan-v2&envId=top-sql-50
--- Approach 1 (using cte)
with Managers as 
(
select 
    reports_to as employee_id, 
    count(distinct employee_id) as reports_count, 
    round(avg(age), 0) as average_age
from Employees
where reports_to is not null
group by reports_to
)
select m.employee_id, e.name, m.reports_count, m.average_age
from Managers m
left join Employees e
on m.employee_id = e.employee_id

--- Approach 2 (using self join)
--- TO DO : Add self join code

-- https://leetcode.com/problems/primary-department-for-each-employee/?envType=study-plan-v2&envId=top-sql-50

--- Approach 1 (using union)

select employee_id, min(department_id) as department_id
from Employee 
group by employee_id
having count(*) = 1

union

select employee_id, department_id
from Employee
where primary_flag='Y'

--- Apporach 2 (without using union, just where clause)
--- TO DO : Add union code


```

## Day 12

```
-- https://leetcode.com/problems/triangle-judgement/?envType=study-plan-v2&envId=top-sql-50
select 
    x,
    y,
    z,
    case when x + y > z and y + z > x and x+z > y then "Yes" else "No" end as triangle
from Triangle

-- https://leetcode.com/problems/consecutive-numbers/description/?envType=study-plan-v2&envId=top-sql-50
select distinct num as ConsecutiveNums 
from 
(
    select 
        id, 
        num, 
        lag(num) over (order by id) as num1, lag(num, 2) over (order by id) as num2
    from 
        Logs
) LogsAdjacent
where num = num1 and num1 = num2

-- Write the same code using CTE, look for other logics


```

* When we create a subquery we need to alias it, else we get an error

* There are different types of window functions listed below
    * Aggregate window functions: Perform operations on sets of rows within a window, such as using SUM(), MAX(), and COUNT().
    * Ranking window functions : Rank the rows within a given window, such as using RANK(), DENSE_RANK(), or ROW_NUMBER()
    * Value window functions : Combine multiple operations into one, such as using LAG(), LEAD(), or FIRST_VALUE().

### Doubts
1. What are the different types of window functions? 


## Day 13
* Have filtered multiple columns simulataneously using WHERE....IN. Had to consider multiple scenarios
    * No change_date before 2019-08-16
    * Having one or more change_date only before 2019-08-16
    * Having change_date both before and after 2019-08-16
    * Having a change_date on 2019-08-16
    Since we are not concerned with change_date post 2019-08-16, 2,3,4 can be handled simulataneously

```
-- https://leetcode.com/problems/product-price-at-a-given-date/?envType=study-plan-v2&envId=top-sql-50
with PricePrior as (
    -- for products which have atleast once change_date before or on 2019-08-16
    select product_id, new_price, change_date
    from Products
    where change_date <= "2019-08-16"
    
    union all
    -- for products where there is no change_date before 2019-08-16
    select product_id, 10 as new_price, "1900-01-01" as change_date
    from Products
)

select distinct
    product_id, new_price as price
from PricePrior
where (product_id, change_date) in 
    (select product_id, max(change_date) from PricePrior group by product_id)

```

* UNION ALL will perform better than UNION when you're not concerned about eliminating duplicate records because you're avoiding an expensive distinct sort operation

### Doubts
1. Suppose we have to find the value at the end of each month of the year, and we have new records in the table only when there is a change, how do we write a sql for that?

### References
1. https://stackoverflow.com/questions/3627946/performance-of-union-versus-union-all-in-sql-server


## Day 14

* Used the same idea that I had learnt from (Day 7 problem)[https://leetcode.com/problems/game-play-analysis-iv/solutions/3673167/best-optimum-solution-with-explanation/?envType=study-plan-v2&envId=top-sql-50]. In that problem, to join a row with the next day row, we joined with the on condition having datediff = 1. In this problem, for every turn, I wanted to join with all the previous turns, and then get sum of all those rows to get the cumulative sum hence the join condition

* When there is a missing catgory, then we have to handle each category separately with union

```
-- https://leetcode.com/problems/last-person-to-fit-in-the-bus/?envType=study-plan-v2&envId=top-sql-50
with CumulativeQueue as (
    select q1.person_id, q1.person_name, q1.weight, q1.turn, sum(q2.weight) as net_weight,
    case when sum(q2.weight) <=1000 then 0 else 1 end as exceeds 
    from Queue q1
    left join Queue q2
    on q1.turn >= q2.turn
    group by 1,2,3,4
    order by q1.turn asc
)
select person_name
from CumulativeQueue
where turn = (select max(turn) from CumulativeQueue where exceeds = 0)

--- Optimize above code

--- Solve the problem using window function

--- Solve the problem using SQL variables


-- https://leetcode.com/problems/count-salary-categories/submissions/1486960437/?envType=study-plan-v2&envId=top-sql-50
--- Wrong approach : Does not work because if there is a category with no entries it does not appear in final result
with AccountsWithCategory as
(
    select
        account_id,
        case when income < 20000 then "Low Salary"
            when income >= 20000 and income <= 50000 then "Average Salary"
            when income > 50000 then "High Salary" 
        end as category
    from
    Accounts
   
)

select category, count(account_id) as accounts_count from AccountsWithCategory group by category

--- Approach 1
select "Low Salary" as category, count(account_id) as accounts_count from Accounts where income < 20000

union

select "Average Salary" as category, count(account_id) as accounts_count from Accounts where income >= 20000 and income <= 50000

union
select "High Salary" as category, count(account_id) as accounts_count from Accounts where income > 50000

```

### Doubts
1. How to compute cumulative sum in Mysql? How about other databases?
2. What are variables in mysql and how do we use them?

### References
1. https://stackoverflow.com/a/61904024
2. https://stackoverflow.com/questions/2563918/create-a-cumulative-sum-column-in-mysql


## Day 15

* LAG() function is used to get value from the row that precedes the current row. 
* LEAD() function is used to get value from the row next to the current row

* To optimize the code, instead of swapping ids, keep ids and swap students, because if we swap ids, then we again have to create a subquery to sort the new ids, but we swap the students ordering is not required.

```
-- https://leetcode.com/problems/employees-whose-manager-left-the-company/?envType=study-plan-v2&envId=top-sql-50

select employee_id
from Employees
where salary < 30000 and manager_id not in (select employee_id from Employees)
order by employee_id


-- https://leetcode.com/problems/exchange-seats/?envType=study-plan-v2&envId=top-sql-50
with NextRowSeat as (
    select 
        id,
        lead(id) over (order by id asc) as next_row_id,
        student
    from
        Seat
),
NewSeat as (

    select     
    case when id % 2 = 0 then id - 1
         when id % 2 = 1 and next_row_id is not null then id + 1
         else id
    end as new_id,
    student

    from NextRowSeat
)

select new_id as id, student from NewSeat
order by new_id asc

```

## Day 16


```

-- incorrect sql because we cannot use subquery alias in immediate where clause
select min(name) as results from Users
where user_id in 
(
select user_id 
from
(select user_id, count(distinct movie_id) as num_movies
from MovieRating
group by user_id) a
where a.num_movies = (select max(num_movies) from a)
)

-- https://leetcode.com/problems/movie-rating/?envType=study-plan-v2&envId=top-sql-50

with cte1 as (
    select user_id, count(distinct movie_id) as num_movies
    from MovieRating
    group by user_id

),
cte2 as (
    select user_id from cte1 
    where cte1.num_movies = (select max(num_movies) from cte1)
),
resultname as (
    select min(name) as results from Users
    where user_id in (select user_id from cte2)
),
cte4 as (
    select movie_id, avg(rating) as avg_rating from MovieRating
    where date_format(created_at,"%Y-%m") = "2020-02"
    group by movie_id     
),
cte5 as (
    select movie_id from cte4
    where avg_rating = (select max(avg_rating) from cte4)
),
resultmovies as (
    select min(title) as results from Movies
    where movie_id in (select movie_id from cte5)

)

select * from resultname 

union all

select *  from resultmovies

-- Try to write a more optimized code with just 2 ctes with join instead of 6 ctes

```

### Doubts
1. Can't we use a subquery alias in the WHERE statement

### References
1. https://stackoverflow.com/questions/11678269/using-subquerys-alias-in-a-where-statement

## Day 17

* To find difference bw two dates we use datediff 

* Joins with inequality are very powerful, as shown in the problem below

* To filter based on number of rows that are created after GROUP BY, we use HAVING with COUNT(*). In the problem, we only want those dates for which previous 7 days are available, therefore once we join with the previous days, we have to use the HAVING COUNT(*) logic

* In this problem, we need to first aggregate at a daily level, else we will get duplicated if there are more than 1 entry for a given day

```
-- https://leetcode.com/problems/restaurant-growth/?envType=study-plan-v2&envId=top-sql-50
with daily_total as
(
    select visited_on, sum(amount) as amount
    from Customer
    group by visited_on
)

select 
    d1.visited_on, 
    sum(d2.amount) as amount, 
    round(avg(d2.amount), 2) as average_amount 

from 
    daily_total d1
left join 
    daily_total d2
on 
    datediff(d1.visited_on, d2.visited_on) < 7 and 
    datediff(d1.visited_on, d2.visited_on) >= 0

group by d1.visited_on
having count(*) = 7 -- this

order by d1.visited_on

--- Solve the problem using window functions, look at ways to speeden up code

```
### Doubts
1. Cant we do date1 - date2?
2. How to filter based on number of rows available post aggregation using group by?

### References
1. https://stackoverflow.com/questions/5664103/filter-by-count

## Day 18

```
-- https://leetcode.com/problems/friend-requests-ii-who-has-the-most-friends/solutions/3803211/union-all-is-what-you-need-don-t-overcomplicate/?envType=study-plan-v2&envId=top-sql-50

with cte as (
    select requester_id as id from RequestAccepted
    union all
    select accepter_id as id from RequestAccepted
),
NumFriends as (
    select id, count(*) as num
    from cte
    group by id
)

select id, num
from NumFriends
where num = (select max(num) from NumFriends)

--- Do using only 1 cte using order by + limit

```

### Doubts
1. How to do distinct on 2 or more columns?
2. How to remove any row which has a duplicate, based on 1 column? based on 2 columns?

## Day 19
### Learnings

* SUBSTRING(string, start, length) function extracts some characters from a string. It follows 1 indexing
* CONCAT(string1, string2, ...., string_n) adds two or more stings together
* LENGTH() function is used to find length of string column in mysql (it is LEN() in some other dbs)
* UPPER() and LOWER() functions used to change case of string column

```
-- https://leetcode.com/problems/fix-names-in-a-table/?envType=study-plan-v2&envId=top-sql-50
select user_id, 
    concat(upper(substring(name, 1, 1)), lower(substring(name, 2, length(name)-1))) as name
from Users
order by user_id

-- https://leetcode.com/problems/patients-with-a-condition/?envType=study-plan-v2&envId=top-sql-50
select patient_id, patient_name, conditions
from Patients
where conditions like 'DIAB1%' or conditions like '% DIAB1%'

```

### Doubts
1. How can I achieve initcap functionality in MySQL?
2. How to get index of last character/length of a string/substring from middle till end of string?
3. How to use DELETE statement in MySQL?
4. What are SUBSTRING_INDEX(), CHARINDEX() and LEFT() functions in sql?
5. Can we use GROUP BY on a string column in SQL?
6. What does You can't specify target table 'p1' for update in FROM clause mean?

### References

## Day 20
### Duration : 1 hour

### Learnings

* GROUP_CONCAT() function in MySQL is an aggregation function that combines data from multiple rows into a single string. Important cluases we can use within GROUP_CONCAT() are :
    * DISTINCT (to eliminate repeated values)
    * ORDER BY (to get the values in a certain order before concatenating)
    * SEPARATOR (the character using which the values are concatenated)

```
--- 
-- https://leetcode.com/problems/group-sold-products-by-the-date/?envType=study-plan-v2&envId=top-sql-50
select 
    sell_date, 
    count(distinct product) as num_sold,
    group_concat(distinct product separator ',') as products
from Activities
group by sell_date

```

* DELETE syntax in MYSQL is DELETE FROM TABLENAME WHERE CONDITION1

* You need to specify the table where you want to delete the rows when joining tables in DELETE statement, by using the table alias as shown below

```
-- https://leetcode.com/problems/delete-duplicate-emails/?envType=study-plan-v2&envId=top-sql-50
-- we want to delete rows from table P1 that satisfy the where clause

delete p1 
from Person p1, Person p2
where p1.email = p2.email and p1.id > p2.id

```

* The goal here is when a value is repeated multiple times in a column, we want to indentify all the repititions i.e. all occurences of the value except the first one. To do so, we do a self cross join (lets sat p1 and p2). In the resultant table, we filter out rows which have the same value in both tables(p1.email = p2.email). In that filtered set, only the first occurence will have its id (in p1) lesser than the joined value from the corresponding table (i.e. p2), all other occurences will have atleast one row where the id in p1 is greater than id in p2. So once we find those rows, we select only those rows from p1 and eliminate them

```
--- run this code in https://onecompiler.com/mysql
-- create
CREATE TABLE EMPLOYEE (
  id INTEGER PRIMARY KEY,
  email TEXT NOT NULL
);

-- insert
INSERT INTO EMPLOYEE VALUES (1, 'john@example.com');
INSERT INTO EMPLOYEE VALUES (2, 'bob@example.com');
INSERT INTO EMPLOYEE VALUES (3, 'john@example.com');
INSERT INTO EMPLOYEE VALUES (4, 'john@example.com');

-- fetch 
-- SELECT * FROM EMPLOYEE E1, EMPLOYEE E2 WHERE E1.email = E2.email;

-- Get all the repetition rows
SELECT E1.id, E1.email FROM EMPLOYEE E1, EMPLOYEE E2
WHERE E1.email = E2.email and E1.id > E2.id
```

### References
1. https://www.geeksforgeeks.org/mysql-group_concat-function/
2. https://dev.mysql.com/doc/refman/8.4/en/delete.html
3. https://stackoverflow.com/questions/15767312/mysql-cross-table-delete

## Milestone
Completed 40 / 50 questions in Leetcode SQL50 on 31-Dec-2024 


## Day 21 and 23

* In date_format function, we use %Y to match a year and %m to match a month

```
-- https://leetcode.com/problems/list-the-products-ordered-in-a-period/?envType=study-plan-v2&envId=top-sql-50
select 
    p.product_name, 
    sum(o.unit) as unit
from Orders o left join Products p
using (product_id)
where date_format(order_date, '%Y-%m') = '2020-02'
group by product_id
having sum(unit) >= 100

```
* We can have window functions without PARTITION BY. If the optional PARTITION BY clause is present, the rankings are reset for each group of rows. If not, the window function is computed over all the rows

* To use window functions like rank(), row_number() or dense_rank() over entire table, we just have to use the order by + the column based on which we want the ranking, without the partition by in the OVER() clause as shown below
```
CREATE TABLE EMPLOYEE (
  empId INTEGER,
  name TEXT NOT NULL,
  day_of_salary DATE NOT NULL,
  salary INTEGER NOT NULL
);

-- insert
INSERT INTO EMPLOYEE VALUES (0001, 'Clark', '2025-01-01', 2000);
INSERT INTO EMPLOYEE VALUES (0001, 'Clark', '2025-01-02', 3000);
INSERT INTO EMPLOYEE VALUES (0002, 'Dave', '2025-01-01', 1000);
INSERT INTO EMPLOYEE VALUES (0002, 'Dave', '2025-01-02', 4000);
INSERT INTO EMPLOYEE VALUES (0002, 'Dave', '2025-01-03', 5000);
INSERT INTO EMPLOYEE VALUES (0003, 'Ava', '2025-01-01', 2000);
INSERT INTO EMPLOYEE VALUES (0003, 'Ava', '2025-01-02', 2000);
INSERT INTO EMPLOYEE VALUES (0003, 'Ava', '2025-01-03', 6000);



-- fetch 
-- SELECT *, MIN(salary) OVER () as min_salary_across_all_employees 
-- FROM EMPLOYEE

-- compute rank of salary across all employees
-- SELECT *, RANK() OVER (ORDER BY salary) as salary_rank_across_all_employees 
-- FROM EMPLOYEE

-- compute rank of salary in comparison to salary of the same employee on a different day
SELECT *, RANK() OVER (PARTITION BY empId ORDER BY salary) as salary_rank
FROM EMPLOYEE
ORDER BY empid

```

* SQL OFFSET is used to skip a specified number of rows in a query result.
```


-- Approach 1
with cte as (
    select 
        id, 
        salary, 
        dense_rank() over (ORDER BY salary desc) as salary_rank
from Employee
)

select distinct salary 
as SecondHighestSalary 
from cte
where salary_rank=2

union all

select null as SecondHighestSalary
where not exists (
    select 1 from cte where salary_rank=2
)

-- Approach 2 (just use ifnull instead of union with null)

with cte as (
    select 
        id, 
        salary, 
        dense_rank() over (ORDER BY salary desc) as salary_rank
from Employee
)

select ifnull(
(select distinct salary 
from cte
where salary_rank=2),
null
)
as SecondHighestSalary


-- Approach 3 (using limit with offset)

select(
select distinct salary 
from Employee
order by salary desc
limit 1 offset 1
) as SecondHighestSalary

-- Does not work as it does not return null when no records match
select distinct salary as SecondHighestSalary
from Employee
order by salary desc
limit 1 offset 1

```

### Doubts
1. How to remember if year is represented by Y and y in date_format function?
2. How to use window function on entire table i.e. without any partitions?
3. What does window function with only ORDER BY do?
4. How to return a null value when query returns zero records? Can we do coalesce(column, null) or ifnull(column, null)?
5. Is offset calculated before or after limit keyword?

### References
1. https://stackoverflow.com/questions/65481229/check-if-string-consists-of-letters-or-numbers-only-in-transact-sql
2. https://www.geeksforgeeks.org/mysql-regular-expressions-regexp/
3. https://docs.aws.amazon.com/redshift/latest/dg/r_WF_DENSE_RANK.html
4. https://stackoverflow.com/questions/2679865/return-a-value-if-no-rows-are-found-in-microsoft-tsql
5. https://www.datacamp.com/tutorial/sql-offset
6. https://stackoverflow.com/questions/65791960/window-functions-how-to-partition-over-nothing
7. https://docs.getdbt.com/sql-reference/rank

## Day 22
### Learnings
* Regular expression is a tool used for flexible pattern matching within string data
* Standard LIKE doesn't handle regex
* ^ : starts with given string
* $ : end of string
* . : match any single character
* To match a dot (like in an email), we need to escape the dot "\." or use it inside a character class "[.]", as it is a meta-character in regex
* [afg] : matches any character listed within the square brackets
* [A-Z] : match any upper case alphabet, the hyphen means a range in this case
* [0-9] : match any digit from 0 to 9, the hyphen means a range in this case
* [0-9.] : match any digit from 0 to 9 or a dot
* \* : Match zero of more instances of the string/expression preceding it

```
-- https://leetcode.com/problems/find-users-with-valid-e-mails/?envType=study-plan-v2&envId=top-sql-50
select *
from Users
where mail REGEXP '^[A-Za-z][A-Za-z0-9_.-]*@leetcode[.]com$'

-- . should be in square brackets else this is a valid email : winston@leetcode?com
-- $ is required at end else this is valid email : winston@leetcode.comAVD

```


### Doubts
1. Does [abx] match only one occurence of the 3 characters (a,b and x) or any number of occurences?

### References
1. https://stackoverflow.com/questions/13989640/regular-expression-to-match-a-dot
2. https://www.geeksforgeeks.org/mysql-regular-expressions-regexp/


### Day 24

* Condition in ON clause vs WHERE clause : In the example below if we put the date condition in the WHERE clause, we are basically creating multiple combinatations based on the join id and then filtering only the meaningful ones (i.e those with purchase dates b/w start and end dates). Refer 1

* For an INNER join does not matter if you put condition in ON or WHERE clause. But for outer joins it makes a difference

```
-- https://leetcode.com/problems/average-selling-price/?envType=study-plan-v2&envId=top-sql-50
-- Approach 1
select 
    p.product_id, 
    coalesce(round(sum(price * units)/sum(units),2),0) as average_price
from 
    Prices p left join UnitsSold u
on 
    p.product_id = u.product_id and 
    u.purchase_date >= p.start_date and 
    u.purchase_date <= p.end_date
group by 
    p.product_id


-- Approach 2 (try the same thing by joining on only product_id and using where clause for dates)


```

* In where clause instead of `bonus is null or bonus < 1000` we can have `coalesce(bonus, 0) < 1000`

```
-- https://leetcode.com/problems/employee-bonus/

--- Approach 1
select name, bonus
from Employee left join Bonus
using (empId)
where bonus is null or bonus < 1000

-- Approach 2
select name, bonus
from Employee left join Bonus
using (empId)
where coalesce(bonus, 0) < 1000

```


### References
1. https://stackoverflow.com/questions/354070/sql-join-what-is-the-difference-between-where-clause-and-on-clause


## Day 25

```
-- https://leetcode.com/problems/queries-quality-and-percentage/?envType=study-plan-v2&envId=top-sql-50

--- Approach 1
select
    query_name,
    round(avg(rating / position), 2) as quality,
    round(sum(case when rating < 3 then 1 else 0 end) * 100.0/count(rating),2) as poor_query_percentage
from Queries
group by query_name

--- Approach 2 (just use average instead of using sum and count, also use boolean instead of case)

```
### Doubts
1. When we do avg with a condition inside, what is the denominator? Is it the total number of rows in that group? 

### References
1. https://stackoverflow.com/questions/9462873/basic-sql-selecting-avg-values-from-the-same-column-multiple-times-in-one-qu


## Day 26 and 27

### Learnings
* Faced this error while working on problem 2: This version of MySQL doesn't yet support 'LIMIT & IN/ALL/ANY/SOME subquery'

```
-- https://leetcode.com/problems/immediate-food-delivery-ii?envType=study-plan-v2&envId=top-sql-50
with first_orders_cte as (
    select *
    from Delivery
    where (customer_id, order_date) in (select customer_id, min(order_date)
    from Delivery group by customer_id)
)

select 
    round(avg(case when order_date = customer_pref_delivery_date then 1 else 0 end) * 100,2) as immediate_percentage
from first_orders_cte



-- https://leetcode.com/problems/department-top-three-salaries/?envType=study-plan-v2&envId=top-sql-50

-- Wrong approach
with HighEarners as (
select id, name 
from Employee
where (departmentId, salary) in (select distinct departmentId, salary from Employee group by departmentId order by salary desc limit 3)
)
select d.name, he.employee, he.salary
from HighEarners he left join Department d
on he.departmentId = d.id


-- Approach 1 
with WithRank as (
select *, DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary desc) as salaryRank
from Employee
),
HighEarners as (
    select id, name, salary, departmentId
    from WithRank
    where salaryRank in (1,2,3)
)
select 
    d.name as Department, 
    he.name as Employee, 
    he.salary as Salary
from HighEarners he left join Department d 
on he.departmentId = d.id


-- Approach 1 condensed with just 1 CTE
with EmployeeWithRank as (
select 
    *, 
    DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary desc) as salaryRank
from Employee
)

select 
    d.name as Department, 
    e.name as Employee, 
    e.salary as Salary
from EmployeeWithRank e left join Department d 
on e.departmentId = d.id
where e.salaryRank in (1,2,3)


-- Approach 2 (try writing the same code without using window function)

```

* We cannot use window functions where there is a group by

### Doubts
1. How to find the top 2 earners for each department? How about top 3 for each department?
2. What is the different b/w RANK + dostinct and DENSE_RANK?
3. Can't we use DISTINCT in OVER clause eg. `RANK() OVER (DISTINCT salary, departmentId)`? 
4. How to combine PARTITION BY with ORDER BY in OVER clause while using window function? Can we use ORDER BY without PARTITION BY?

### References
1. https://stackoverflow.com/questions/17892762/mysql-this-version-of-mysql-doesnt-yet-support-limit-in-all-any-some-subqu
2. https://stackoverflow.com/questions/46371935/find-the-3rd-maximum-salary-for-each-department-based-on-table-data
3. https://mode.com/sql-tutorial/sql-window-functions

* Group by vs lag vs where clause vs correlated subquery when do we use which? Give some examples.
