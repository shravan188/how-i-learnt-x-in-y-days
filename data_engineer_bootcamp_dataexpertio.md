# Data Engineer bootcamp by DataExpert-io (Data with Zach)

## Day 1

### Duration : 1 hour

### Learnings

* pgadmin : A tool for managing postgressql. It provides a graphical interface for creating, maintaining and using database objects

* Used docker compose to start docker container with this (docker_compose.yml)[https://github.com/DataExpert-io/data-engineer-handbook/blob/main/bootcamp/materials/1-dimensional-data-modeling/docker-compose.yml], then opened http://localhost:5050 to open pgadmin. Logged into using .env variables (postgres@postgres.com and postgres)

```
cd bootcamp\materials\1-dimensional-data-modeling

docker compose up -d

# get running containers and their id
docker ps

# 
docker exec -it <container_id> bash 

# 
pg_restore -U $POSTGRES_USER -d $POSTGRES_DB /docker-entrypoint-initdb.d/data.dump


psql -U postgres

\l

# connect to postgres database
\c postgres

\dt

select count(*) from events;

# exit psql
\q

# exit docker container
exit

```
* pg_restore : Allows you to restore a database from a .dump file (which is a logical backup file created using the command pg_dump)


### Doubts
1. What are volumes in docker and why do we use the -v flag when we do docker compose down?

### References

1. https://stackoverflow.com/questions/37694987/connecting-to-postgresql-in-a-docker-container-from-outside
2. https://stackoverflow.com/questions/129445/postgresql-psql-i-how-to-execute-script-in-a-given-path
3. https://www.timescale.com/learn/a-guide-to-pg_restore-and-pg_restore-example
4. https://simplebackups.com/blog/postgresql-pgdump-and-pgrestore-guide-examples/

## Day 2
### Learnings
Watched course lectures - Dimensional modeling theory and Lab

### Doubts
1. Why do we need separate data modeling for transactional and analytical processing?
2. What is an OLAP cube?
3. What is the difference bw struct, map and array?
4. What is cumulative data design? What is the compactness vs usability tradeoff?
5. What are the different scd types and how is it differernt from daily snapshots?
6.


## Day 3
### Duration : 0.5 hours
### Learnings
* Setup postgres and pgadmin 4 with docker using steps in day 1 

* Created new server connection with Host Name : host.docker.internal

* Viewed existing tables by opening Servers > pgtest > Databases > postgres > schemas > public > tables in the Object Explorer pane

* Opened Query Tool and then ran the following SQL queries

```
-- to see postgres version
select version();

select * from actor_films;

```

### Doubts
1. What does host name host.docker.internal mean?

### References
1. https://www.youtube.com/watch?v=UjQiwonRMas

## Day 4 and 5
### Duration : 3.5 + 1 hours

### Learnings
* Postgres has multiple built in types. Some of the main ones are
   * INTEGER, REAL (numeric types)
   * BOOLEAN
   * CHAR, VARCHAR, TEXT (character types)
   * DATE, TIME, TIMESTAMP (temporal types)
   * Array
   * User defined types (including enum)

* We can create PostgreSQL user-defined data type using CREATE DOMAIN or CREATE TYPE statements. CREATE TYPE is used to create a composite type

* Enum type is a custom data type that allows you to define a list of possible values for a column.If you attempt to insert or update a row with a value not in the list, PostgreSQL will issue an error. When we do ORDER BY with an enum field, it orders the rows based on the order in which we list them when we define the enum.

* We can define an array and insert into it as follows. Also by using the UNNEST function, we can expands the array into multiple individual rows

```
--- Run code in https://onecompiler.com/postgresql
-- create
CREATE TABLE EMPLOYEE (
  empId INTEGER PRIMARY KEY,
  name TEXT NOT NULL,
  dept TEXT NOT NULL,
  phone_numbers TEXT []
);

-- insert
INSERT INTO EMPLOYEE VALUES (0001, 'Clark', 'Sales', ARRAY ['123-456-789', '123-456-789']);
INSERT INTO EMPLOYEE VALUES (0002, 'Dave', 'Accounting',  ARRAY ['123-456-789', '123-456-789']);
INSERT INTO EMPLOYEE VALUES (0003, 'Ava', 'Sales',  ARRAY ['123-456-789', '123-456-789']);

-- fetch 
SELECT * FROM EMPLOYEE WHERE dept = 'Sales';

-- fetch with unnest
SELECT empId, name, dept, unnest(phone_numbers) FROM EMPLOYEE WHERE dept = 'Sales';



```

* The goal of this exercise is to create a new cumulative table called players which has only 1 row per player - and all the fields that change or keep getting added with time such as season, pts, assists, rebounds, etc are stored as an array in a single column. This new table design helps compress information and makes joins easier

```
-- create a custom type that contains all the temporal fields i.e. fields that change with time
CREATE TYPE season_stats AS (
      season INTEGER,
      pts REAL,
      ast REAL,
      reb REAL,
      weight INTEGER
);

-- 
CREATE TYPE scoring_class as ENUM('bad', 'average', 'good', 'star');

-- seasons field stores an array of season_stats, so similar to an array of tuples in python
-- scoring_class, years_since_last_active, is_active and current_season are derived columns
CREATE TABLE players (
     player_name TEXT,
     height TEXT,
     college TEXT,
     country TEXT,
     draft_year TEXT,
     draft_round TEXT,
     draft_number TEXT,
     seasons season_stats[],
     scoring_class scoring_class,
     years_since_last_active INTEGER,
     is_active BOOLEAN,
     current_season INTEGER,
     PRIMARY KEY (player_name, current_season)
);

-- today cte is the seed query, as it populates the database with initial data
-- we have to run this repeatedly with successive years to get the entire data (next current_season = 1996 and season = 1997 and so on)
WITH yesterday AS (
   select * from players
   where current_season = 1995
),
today AS (
   select * from player_seasons
   where season = 1996
)
INSERT INTO players
SELECT
   COALESCE(y.player_name, t.player_name) as player_name,
   COALESCE(y.height, t.height) as height,
   COALESCE(y.college, t.college) as college,
   COALESCE(y.country, t.country) as country,
   COALESCE(y.draft_year, t.draft_year) as draft_year,
   COALESCE(y.draft_round, t.draft_round) as draft_round,
   COALESCE(y.draft_number, t.draft_number) as draft_number,
   COALESCE(y.seasons, ARRAY[]::season_stats[]) ||
            ARRAY[ROW(
               t.season,
               t.pts,
               t.ast,
               t.reb, 
               t.weight)::season_stats
            ]
            as seasons,
   CASE
             WHEN t.season IS NOT NULL THEN
                 (CASE WHEN t.pts > 20 THEN 'star'
                    WHEN t.pts > 15 THEN 'good'
                    WHEN t.pts > 10 THEN 'average'
                    ELSE 'bad' END)::scoring_class
             ELSE y.scoring_class
      END,

   CASE WHEN t.season IS NOT NULL THEN 0
   ELSE y.years_since_last_active + 1
   END as years_since_last_active,
   t.season IS NOT NULL as is_active,
   COALESCE(t.season, y.current_season + 1) as current_season

FROM yesterday y FULL OUTER JOIN today t
ON y.player_name = t.player_name;

-- to see history of Michael Jordan as of 1997
select player_name, height, college, country, (unnest(seasons)::season_stats).* from players
where player_name like '%Michael Jordan%' and current_season = 1997;

```


* Note that single quotes and double quotes have different meaning in SQL, unlike Python. When doing string comparison, we have to use single quotes not double quotes eg. `where player_name like '%Michael%'`

### Doubts
1. What are functions in Postgres and how do we define and use them?
2. Why is the order in coalesce previous followed by current?
3. What is difference b/w execute script and execute query in pgadmin?
4. What is the difference b/w procedural code and SQL in Postgres? Why is this difference not there in SQL server?
5. https://stackoverflow.com/questions/54351802/how-i-can-run-parts-of-sql-query-separate-in-pgadmin4
6. Suppose we have to fill the cumulative table till 2021, should we have to manually do it from 1996 to 2021, or is there a shorter way?
7. The code above keeps adding rows with null values for season_stats column, even after player has retired. Can we improve that?

### References
1. https://neon.tech/postgresql/postgresql-tutorial/postgresql-data-types
2. https://onecompiler.com/postgresql (playground to run postgresql)
3. https://popsql.com/learn-sql/postgresql/how-to-insert-data-into-an-array-in-postgresql
4. https://www.w3resource.com/PostgreSQL/postgresql_unnest-function.php

## Day 6
### Duration : 1 hour

### Learnings

* Facts and Dimensions : Facts represent measurable events (e.g., sales), while dimensions provide descriptive context (e.g., customer, product) for those facts

* Slowly Changing Dimensions : A Slowly Changing Dimension (SCD) is a dimension (basically a column) while generally stable, may change over time, often in an unpredictable manner. Common examples include geographical location(such as address), customer/employee details (level 4 manager name), product attributes

* Rapidly changing dimensions :  Rapidly changing dimension are dimensions which undergo frequent updates, for example transactional parameters like customer ID, product ID and price

* 2 approaches to create SCD table : 
      * By creating the entire SCD table from scratch every refresh (use entire history/full refresh)
      * By incrementally adding data to already existing SCD table (incremental refresh)

```
WITH streak_started AS (
   SELECT player_name,
         current_season,
         scoring_class,
         LAG(scoring_class, 1) OVER 
            (PARTITION BY player_name ORDER BY current_season) <> scoring_class
            OR LAG(scoaring_class, 1) OVER
            (PARTITION BY player_name ORDER BY current_season) IS NULL
            AS did_change,
       -- LAG(is_active, 1) OVER(PARTITION BY player_name ORDER BY current_season) <> is_active AS is_active_change_indicator,
      FROM players
),
   streak_identified AS (
      SELECT player_name,
            scoring_class,
            current_season,
            SUM(CASE WHEN did_change THEN 1 ELSE 0 END)
               OVER (PARTITION BY player_name) as streak_identifier,
            is_active,

         FROM streak_started
   )

SELECT
   player_name,
   scoring_class,
   is_active,
   streak_identifier,
   MIN(current_season) AS start_date,
   MAX(current_season) AS end_date,
   2009 AS current_season -- last season in the players data is 2009
FROM streak_identified
GROUP BY 1,2,3


```
* We are essentially using GROUP BY to track fields which change. In this case, we are tracking change in scoring_class for each player, so when we group by player_name and scoring_class, we can get from which season (i.e. start_date) to which season (i.e. end_date) a player was in a particular scoring_class. We can track more than one fields using the same logic, for example we want to track scoring_class and is_active, we can add is_active to GROUP BY clause, and we will get min and max seasons based on the combination of those 2 fields (streak_identifier is dependent on scoring_class hence adding it to group by makes no additional impact)

* If before converting to SCD2 a value is same across 10 rows (in this case 10 seasons), after creating SCD2 it will be just one row with the value, the start_date and the end_date. To achieve this we use GROUP BY

* When using GROUP BY, column must appear in the GROUP BY clause or be used in an aggregate function


### Doubts
1. Why are we computing the streak_identifier in the first place? WHat value does it add?
2. If we try to create an SCD table using all of history, we may get wrong values if the person goes from value 1 to value 2 and then back to value 1 over a period of time. For example, Aaron McKie is bad from 1996 to 1999, avarage from 2000 to 2001 and again bad from 2002 to 2005. So for him there will be only 1 row with scoring_class as "bad" with start_date as 1996 and end_date as 2005, instead of 2 rows of "bad" 1996 to 1999 and 2002 to 2005. How do we avoid this?

### References
1. https://www.youtube.com/watch?v=nyu-8Si21ec
2. https://www.sqlshack.com/implementing-slowly-changing-dimensions-scds-in-data-warehouses/
3. https://stackoverflow.com/questions/77483950/implement-scd-type-2-on-periodic-snapshot-table

## Day 7 and 8
### Duration : 1.5 + 2

### Learnings
* As mentioned in Doubts of Day 6, using GROUP BY may give incorrect information, hence used a different approach involving just the WHERE clause and LEAD window function as show below

```
--- better code to create scd table
WITH streak_started AS (
	SELECT player_name,
         current_season,
         scoring_class,
		 is_active,
         LAG(scoring_class, 1) OVER (PARTITION BY player_name ORDER BY current_season) <> scoring_class
            OR LAG(scoring_class, 1) OVER (PARTITION BY player_name ORDER BY current_season) IS NULL
			OR LAG(is_active, 1) OVER(PARTITION BY player_name ORDER BY current_season) <> is_active
         AS did_change
      FROM players
)

SELECT
   player_name,
   scoring_class,
   is_active,
   current_season AS start_date,
   lead(current_season, 1, 9999) OVER (PARTITION BY player_name ORDER BY current_season) AS end_date
FROM streak_started
WHERE did_change 
ORDER BY player_name, start_date;

-- SELECT * FROM players
-- WHERE player_name = 'Aaron McKie'
```

* Instead of creating SCD table fully (i.e. full refresh) using above approach, we can only add data to it incrementally (i.e. incremental refresh). When we add data to a SCD table incrementally, we have to handle 3 types of scenarios:
      * Existing rows unchanged i.e. all dimension values remains same for an existing player
      * Existing rows change i.e. any one or more of the dimensions for an exisiting player change
      * New rows created i.e. a new player is added
Each of these 3 scenarios must be handled separately, as shown in the diagram and code below


```
CREATE TYPE scd_type AS (
                    scoring_class scoring_class,
                    start_season INTEGER,
                    end_season INTEGER
   )


WITH historical_scd as (
   SELECT 
      player_name,
      scoring_class,
      is_active,
      start_season,
      end_season
   FROM player_scd
   WHERE current_season = 2021
   AND end_season < 2021
),
last_season_scd as (
   -- we will not be using this cte in the final union all, hence we just select all columns
   SELECT *         
   FROM player_scd
   WHERE current_season = 2021
   AND end_season = 2021
),
this_season_data as (
   -- we will not be using this cte in the final union all, hence we just select all columns
   SELECT * FROM players
   WHERE current_season = 2022
)
unchanged_records as (
   SELECT 
      ls.player_name,
      ls.scoring_class,
      ls.is_active,
      ls.start_season,
      ts.current_season as end_season
   FROM this_season_data ts
   INNER JOIN last_season_scd ls
   ON ts.player_name = ls.player_name
   WHERE ts.scoring_class = ls.scoring_class AND ts.is_active = ls.is_active
),
changed_records as (
   SELECT 
      ts.player_name
      UNNEST(ARRAY[
         ROW(
            ls.scoring_class,
            ls.is_active,
            ls.start_season,
            ls.end_season   
         ),
         ROW(
            ts.scoring_class,
            ts.is_active,
            ts.start_season,
            ts.end_season
         )

      ])
   FROM last_season_scd ls
   LEFT JOIN this_season_data ts
   ON ls.player_name = ts.player_name
   WHERE ts.scoring_class <> ls.scoring_class OR ts.is_active <> ls.is_active
),
new_records as (
   SELECT 
      ts.player_name,
      ts.scoring_class,
      ts.is_active,
      ts.current_season as start_season,
      ts.current_season as end_season
   FROM this_season_data ts
   LEFT JOIN last_season_scd ls
   WHERE ls.player_name IS NULL

)

SELECT *, 2022 AS current_season FROM (
   SELECT * FROM historical_scd
   UNION ALL
   SELECT * FROM unchanged_records
   UNION ALL
   SELECT * FROM changed_records
   UNION ALL
   SELECT * FROM new_records
)


```

* A subtle point to remember - if the current season is 2022 and previous season is 2021, we are taking historical scd with end_season < 2021 and not end_season <= 2021, because if dimension values are unchanged we do not want the row with end_season as 2021, we will be creating a new row with end_season as 2022 for the player. As a side effect of this, we cannot club changed_records with new_records because, for changed_records, we will have to create two rows - the one with end_season 2021 and the second row with start_season and end_season as 2022. On the other hand for new_records, we create only a single row, with start_season and end_season as 2022

* Although current_season is not required from an SCD standpoint, it is helpful in handling the different scenarios when incrementally adding data to the SCD table


### Doubts
1. In changed_records cte, why last_season LEFT JOIN this_season and not vice versa?
2. How to select rows present in table 1 but not in table 2 while joining 2 tables? (check new_records cte)
3. How to determine the WHERE clauses for historical_scd and last_season_scd?
4. Why do we need current_season in SCD table? Cant we just have start_season and end_season?
5. Why do we have separate CTE for changed_records and new_records? Can't we club both of these scenarios into a single one as in both cases we just have to add a new row with the new values and start_season and end_season as current_season?

### References
1. https://community.spiceworks.com/t/difference-between-scd-load-and-incremental-load-in-informatica/864169/2
2. https://medium.com/analytics-vidhya/slowly-changing-dimensions-at-scale-bf9ce9157951

## Day 9, 10, 11, 12

### Duration : 4 + 2 + 1 + 1 hours

### Learnings
* Worked on W1 Homework on dimensional modeling


* Encountered `ERROR:  operator does not exist: integer = text LINE 40: ON py.actorid = ty.actorid`, because actorid was text in one table, integer in another. To solve this dropped table actors using below code, and recreated with actorid as text instead of integer

```
DROP TABLE actors
```

* Encountered `ERROR:  Cannot cast type integer[] to integer in column 2. cannot cast type record to film_stats ` 

* ERROR:  column films of table actors depends on type film_stats[] cannot drop type film_stats because other objects depend on it 

* ERROR:  cannot drop type film_stats because other objects depend on it
SQL state: 2BP01
Detail: column films of table actors depends on type film_stats[]

* On trying ALTER TYPE film_stats ALTER ATTRIBUTE votes SET DATA TYPE INTEGER[];
ERROR:  cannot alter type "film_stats" because column "actors.films" uses it 

* ERROR:  CASE types text and quality_class cannot be matched
LINE 31:  (CASE WHEN ty.avg_rating > 8 THEN 'star'
 
* ERROR:  invalid input syntax for type integer: "tt13236566" 

```
CREATE TYPE film_stats AS (
film TEXT[],
votes INTEGER[],
rating REAL[],
filmid TEXT[]
);

CREATE TYPE quality_class AS ENUM('bad','average','good','star');

CREATE TABLE actors (
actor TEXT,
actorid TEXT,
current_year INTEGER,
films film_stats[],
quality_class TEXT,
is_active BOOLEAN
);

WITH previous_year AS(
	SELECT * FROM actors	
	WHERE current_year = 2019
),
actor_films_ratings AS (
SELECT *,
ROUND(AVG(rating) OVER (PARTITION BY actorid)::numeric, 2) as avg_rating
FROM actor_films
WHERE year = 2020
),
this_year AS (
SELECT 
	actor,
	actorid,
	year,
	min(avg_rating) as avg_rating,
	array_agg(film) as film,
	array_agg(votes) as votes,
	array_agg(rating) as rating,
	array_agg(filmid) as filmid

FROM actor_films_ratings
GROUP BY actor, actorid, year
)
SELECT 
	COALESCE(py.actor, ty.actor) as actor,
	COALESCE(py.actorid, ty.actorid) as actorid,
	COALESCE(ty.year, py.current_year + 1) as current_year,
	COALESCE(py.films, ARRAY[]::film_stats[]) || ARRAY[ROW(ty.film, ty.votes, ty.rating, ty.filmid)::film_stats] as films,
	CASE WHEN ty.actorid IS NOT NULL THEN
	(CASE WHEN ty.avg_rating > 8 THEN 'star'
	     WHEN ty.avg_rating > 7 and ty.avg_rating <=8 THEN 'good'
	     WHEN ty.avg_rating >6 and ty.avg_rating <=7 THEN 'average'
	     ELSE 'bad' END)
	ELSE py.quality_class
	END as quality_class,
	ty.actorid IS NOT NULL as is_active

FROM previous_year py FULL OUTER JOIN this_year ty
ON py.actorid = ty.actorid

-- Repeat this query for multiple years

CREATE TABLE actors_history_scd (
actorid TEXT,
actor TEXT,
quality_class TEXT,
is_active BOOLEAN,
start_date INTEGER,
end_date INTEGER,
current_year INTEGER
)

WITH cte AS (

SELECT 
actor,
actorid,
current_year,
quality_class,
is_active,
LAG(quality_class, 1) OVER (PARTITION BY actorid ORDER BY current_year) <> quality_class
OR LAG(is_active, 1) OVER (PARTITION BY actorid ORDER BY current_year) <> quality_class
OR LAG(quality_class, 1) OVER (PARTITION BY actorid ORDER BY current_year) IS NULL
AS did_change,
FROM actors

)

SELECT 
actor,
actorid,
quality_class,
is_active,
current_year as start_date,
LEAD(current_year, 1, 9999) OVER (PARTITION BY actorid ORDER BY current_year) as end_date,
2021 AS current_year
FROM cte
WHERE did_change
oRDER BY actorid, start_date;


WITH previous_year AS (

SELECT * FROM actors_history_scd
WHERE current_year = 2020
AND end_date = 2020
),
current_year AS (

SELECT * FROM actors
WHERE current_year = 2021

)

historical_scd AS
(
SELECT actor, actorid, quality_class, is_active, start_date, end_date


FROM actors_history_scd
WHERE current_year = 2020
AND end_date < 2020

),
unchanged_records AS (

SELECT py.actor, py.actorid, py.quality_class, py.is_active, py.start_date, ty.current_year as end_date
FROM previous_year py INNER JOIN this_year ty
ON py.actorid = ty.actorid 
WHERE py.is_active = ty.is_active and py.quality_class = ty.quality_class
),
changed_records AS (
SELECT 
ty.actor,
ty.actorid,
UNNEST(ARRAY(ROW(py.quality_class, py.is_active, py.start_date, ty.current_year as end_date), ROW(ty.quality_class, ty.is_active, ty.current_year as start_date, ty.current_year as end_date)))

FROM previous_year py INNER JOIN this_year ty
ON py.actorid = ty.actorid
WHERE py.is_active <> ty.is_active OR py.quality_class <> ty.quality_class

),
new_records AS (

SELECT
ty.actor, ty.actorid, ty.quality_class, ty.is_active, ty.current_year as start_date, ty.current_year as end_date
FROM this_year ty left join previous_year py
ON py.actorid = ty.actorid 
WHERE py.actorid IS NULL

)

SELECT * FROM historical_scd
UNION ALL
SELECT * FROM unchanged_records
UNION ALL
SELECT * FROM changed_records
UNION ALL
SELECT * FROM new_records

```


### Doubts
1. What is the syntax for creating table and deleting table in postgres?
2. When to use TEXT vs VARCHAR?
3. What are we doing the join bw current and previous year on and why? How to determine order in the coalesce - previous and current?
4. Why cant you put index on a text column but put in on a varchar column in mysql?
5. Can we directly apply coalesce on an array? 
6. How to concatenate a row to an existing array in postgres?
7. What if an actor has multiple films? How does ROW(cy.film, cy.votes) work in that scenario? (to solve this we have to use array_agg)
8. How to determine if an actor is active or not? Can we use any column from this_year cte and check if it is null?
9. What is struct type in postgres?
10. When trying to do ROUND(AVG(rating),2) in postgres, it gives error function `round(double precision, integer) does not exist`. Why is this the case?
11. How to define array of integers in postgres table?
12. What is a form in Postgres?
13. PostgreSQL is strictly typed environment - it is big difference from MSSQL procedures, where you can returns anything. What does this mean?
14. How to define and use a procedure in postgres? When to use it?
15. How to expand elements of a ROW into separate columns in Postgres? 

### References
1. https://stackoverflow.com/questions/25300821/difference-between-varchar-and-text-in-mysql
2. https://www.postgresql.org/docs/current/rowtypes.html
3. https://stackoverflow.com/questions/13113096/how-to-round-an-average-to-2-decimal-places-in-postgresql
4. https://stackoverflow.com/questions/35338711/cannot-drop-table-users-because-other-objects-depend-on-it
5. https://stackoverflow.com/questions/7162903/how-to-alter-a-columns-data-type-in-a-postgresql-table
6. https://stackoverflow.com/questions/44852403/cannot-alter-composite-type-because-a-column-is-using-it
7. https://dba.stackexchange.com/questions/22362/list-all-columns-for-a-specified-table
8. https://stackoverflow.com/questions/32469937/how-to-add-new-column-with-data-on-existing-table
9. https://neon.tech/docs/functions/array_agg
10. https://stackoverflow.com/questions/24444008/how-can-you-expand-a-condensed-postgresql-row-into-separate-columns
11. https://stackoverflow.com/questions/62709365/how-to-insert-a-row-in-the-middle-of-a-slowly-changing-dimension-type-2-based-on



## Day N
### Duration : 1.5 hours

### Learnings
* Partition : Atomic chunk of data (which is stored on a node in a cluster)
* Partitioning enables parallelism i.e. a data transformation can be performed on multiple data chunks in parallel, thus job finishes faster
* RDD (Resilient Distributed dataset) : A large dataset split across multiple machines/nodes. Each unit which the data is split into is called a partition
* Repartition : Change number of partitions data is divided into. Done to improve performance (more details later). Can be done in 2 ways : 
    * repartion (recreates new equal sized partitions from scratch, can increase or decrease total no. of partitions, involves shuffling across nodes) 
    * coalesce (combines existing partitions to create larger partitions, decreases total no. of partitions, no shuffling)
    (check interview questions in 11)


* Memory/inmemory vs disk : Memory is where computer stores data temporarily, while storage is where you save files permanently. Memory is much faster than other forms of storage, such as a disk (or even ssd) due to various reasons (refer 3)

* Spark does all computation in memory

* Forked repo https://github.com/DataExpert-io/data-engineer-handbook and cloned it locally. Uploaded devices.csv, events.csv and event_data_pyspark.ipynb on Databricks using Databricks community edition. Since DataBricks uses DBFS(databricks file system), had to replace file paths with /FileStore/tables/events.csv and /FileStore/tables/devices.csv

* Since pyspark 3.4.0, you can use the withColumnsRenamed() method to rename multiple columns at once. For version before that, we can either use withColumnRenamed() multiple times or use select as shown below (along with get function of dictionary)

```
from pyspark.sql import SparkSession
from pyspark.sql.functions import expr, col

spark = SparkSession.builder.appName("Jupyter").getOrCreate()

events = spark.read.option("header", "true").csv("/FileStore/tables/events.csv").withColumn("event_date", expr("DATE_TRUNC('day', event_time)"))
devices = spark.read.option("header","true").csv("/FileStore/tables/devices.csv")

df = events.join(devices, on="device_id", how="left")

mapping = dict(zip(['browser_type', 'os_type'], ['browser_family', 'os_family']))
mapping = {'browser_type': 'browser_family', 'os_type': 'os_family'}

# if the column name is not in mapping, just return column name, else return the mapped one
# this is where .get function of dictionary is so useful
df = df.select([col(c).alias(mapping.get(c, c)) for c in df.columns])

print(df.count()) # 404814
```

* Spark 3 can create tables in any Iceberg catalog with the clause USING iceberg. To use Iceberg in Spark, first configure Spark catalogs. (refer 7)

### Doubts
1. What exactly is special about HDFS and DBFS?
2. How exactly does repartition work when we specify a column name, especially when number of categories in the column is less than number of partitions? (refer 6)
3. When we create database using spark where and how is it stored? How does it compare to a postgres database? What is difference b/w operational and analytoc workload?

### References
1. https://medium.com/@zaiderikat/apache-spark-repartitioning-101-f2b37e7d8301
2. https://www.geeksforgeeks.org/difference-between-memory-and-hard-disk/
3. https://superuser.com/questions/1696557/what-in-the-hardware-makes-ram-faster-than-drive
4. https://stackoverflow.com/questions/40732962/spark-rdd-is-partitions-always-in-ram?rq=1
5. https://stackoverflow.com/questions/38798567/rename-more-than-one-column-using-withcolumnrenamed
6. https://stackoverflow.com/questions/58286502/spark-repartitioning-by-column-with-dynamic-number-of-partitions-per-column
7. https://iceberg.apache.org/docs/1.7.0/spark-ddl/
8. https://spark.apache.org/docs/3.5.3/sql-ref-syntax-ddl-create-database.html
9. https://www.reddit.com/r/dataengineering/comments/1g03cyw/how_does_spark_compare_to_postgres_in_large_scale/
10. https://stackoverflow.com/questions/76242658/if-spark-isnt-a-storage-system-how-do-tables-work
11. https://www.java-success.com/5-apache-spark-coalesce-vs-repartition-scenarios-interview-qas/




## Day N + 1

* Repartitioning : Redistributing the data across different partitions in a spark rdd

* Repartitioning using column : When we reparition using column, we may get skewed partitions since we are using the column values to do the partitioning

```
from pyspark.sql.functions import spark_partition_id


os_df = spark.createDataFrame(["Windows","Windows","Windows","Windows","Windows","Windows","Windows","Linux","Mac","Mac"], "string").toDF("os")

os_df = os_df.withColumn("partitionId", spark_partition_id())
print(os_df.rdd.getNumPartitions()) # 8
os_df.show() # partition ids from 0 to 7

os_df = os_df.repartition(2)
print(os_df.rdd.getNumPartitions()) # 2
os_df = os_df.withColumn("partitionId", spark_partition_id())
os_df.show() # partition ids 0 (six rows, combination of all 3 categories) and 1 (four rows)

## repartition by column with 2 paritions
os_df = os_df.repartition(2, col("os"))
print(os_df.rdd.getNumPartitions()) #2
os_df = os_df.withColumn("partitionId", spark_partition_id())
os_df.show() # all windows and linux rows have partition id 0, mac has partition id 1

## repartition by column with 3 paritions

os_df = os_df.repartition(3, col("os"))
print(os_df.rdd.getNumPartitions()) # 3 (but 1 partition is empty)
os_df = os_df.withColumn("partitionId", spark_partition_id()) 
os_df.show() # linux row has index 0, all windows and 

```

* To distribute data across partitions spark needs somehow to convert value of the column to index of the partition. There are two default partitioners in Spark - HashPartitioner and RangePartitioner. We can also implement our own partitioner if we want better distribution (refer 3)

* Sorting data : There are 2 ways of sorting:
    * sort() : does a global sort i.e. sorts data across all partitions. Slower as it involves shuffling
    * sortWithinPartitions() : sorts data only within each partition. We can imagine a fourth column called partitionId as the primary sorting column

```
sorted = df.repartition(10, col("event_date"))\
    .sortWithinPartitions(col("event_date"), col("host"))\
    .withColumn("event_time", col("event_time").cast("timestamp")) 


sortedTwo = df.repartition(10, col("event_date"))\
    .sort(col("event_date"), col("host"))\
    .withColumn("event_time", col("event_time").cast("timestamp")) 


```
* If we do df.repartition(1).sortWithinPartitions() it is equivalent to global sort i.e. df.sort()

* We can use .explain() to see the query plan 

### Doubts
1. How and when to implement custom partitioning in Pyspark?
2. How to implement SCD Type 1 upsert in Pyspark? How about SCD Type 2?
3. How to write code to detect empty partitions in Pyspark?
4. How does hash partitioning work for date columns or any non categorical column?
5. Does repartitioning create near equal sized partitions?

### References
1. https://stackoverflow.com/questions/47674311/how-to-create-a-sample-single-column-spark-dataframe-in-python
2. https://stackoverflow.com/questions/46032320/apache-spark-get-number-of-records-per-partition
3. https://stackoverflow.com/questions/50694848/why-do-i-get-so-many-empty-partitions-when-repartionning-a-spark-dataframe
4. https://stackoverflow.com/questions/66534193/how-does-sortwithinpartitions-sort
5. https://stackoverflow.com/questions/31610971/spark-repartition-vs-coalesce


## Day N + 2

### Duration : 1 hour

### Learnings

* Join strategies in Spark when we join two datasets : shuffle join, broadcast join

* Broadcast join : When one of the 2 tables to be joined is small, that small table is sent to all the worker nodes in the cluster. This allows join to be performed locally within each node (thus avoiding redistributing data amongst nodes i.e. shuffling)

* spark.sql.autoBroadcastJoinThreshold : This threshold is the maximum size in bytes for a table to be automatically broadcast by spark to all worker nodes. Default value is 10485760 i.e. 10MB. By setting this value to -1, broadcasting can be disabled.

* We can explicitly perform broadcast join using the broadcast function

```
from pyspark.sql import SparkSession
# create spark session
spark = SparkSession.builder.appName("homework").getOrCreate()
# disable automatic broadcast join
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)
# check if configuration is done as expected
print(spark.conf.get("spark.sql.autoBroadcastJoinThreshold"))

maps = spark.read.option("header","true").csv("maps.csv")
medals = spark.read.option("header","true").csv("medals.csv")
matches = spark.read.option("header","true").csv("matches.csv")

print(maps.count(), medals.count(), matches.count())

matches = matches.join(maps, matches.mapid == maps.mapid, "left")
matches.explain()

from pyspark.sql.functions import broadcast
maps = spark.read.option("header","true").csv("maps.csv")
matches = spark.read.option("header","true").csv("matches.csv")
matches = matches.join(broadcast(maps), matches.mapid == maps.mapid, "left")
matches.explain()

```

* The different query plans we get are as follows

```
### Without broadcast join
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [mapid#762], [mapid#697], LeftOuter
   :- Sort [mapid#762 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(mapid#762, 200), ENSURE_REQUIREMENTS, [plan_id=1112]
   :     +- FileScan csv [match_id#761,mapid#762,is_team_game#763,playlist_id#764,game_variant_id#765,is_match_over#766,completion_date#767,match_duration#768,game_mode#769,map_variant_id#770] Batched: false, DataFilters: [], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/matches.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string...
   +- Sort [mapid#697 ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(mapid#697, 200), ENSURE_REQUIREMENTS, [plan_id=1113]
         +- Filter isnotnull(mapid#697)
            +- FileScan csv [mapid#697,name#698,description#699] Batched: false, DataFilters: [isnotnull(mapid#697)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/maps.csv], PartitionFilters: [], PushedFilters: [IsNotNull(mapid)], ReadSchema: struct<mapid:string,name:string,description:string>


### With broadcast join
== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- BroadcastHashJoin [mapid#890], [mapid#866], LeftOuter, BuildRight, false
   :- FileScan csv [match_id#889,mapid#890,is_team_game#891,playlist_id#892,game_variant_id#893,is_match_over#894,completion_date#895,match_duration#896,game_mode#897,map_variant_id#898] Batched: false, DataFilters: [], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/matches.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string...
   +- BroadcastExchange HashedRelationBroadcastMode(List(input[0, string, false]),false), [plan_id=1172]
      +- Filter isnotnull(mapid#866)
         +- FileScan csv [mapid#866,name#867,description#868] Batched: false, DataFilters: [isnotnull(mapid#866)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/maps.csv], PartitionFilters: [], PushedFilters: [IsNotNull(mapid)], ReadSchema: struct<mapid:string,name:string,description:string>




```

### Doubts

1. What is a worker node?


### References
1. https://spark.apache.org/docs/3.5.3/sql-performance-tuning.html
2. https://aspinfo.medium.com/what-is-broadcast-join-how-to-perform-broadcast-in-pyspark-699aef2eff5a
3. https://github.com/DataExpert-io/data-engineer-handbook/tree/main/bootcamp/materials/3-spark-fundamentals

## Day N + 3
### Duration : 1.5 hours

### Learnings
Watched course lectures
    * Spark + Iceberg in 1 Hour - Memory Tuning, Joins, Partition 
    * High Performance Spark in 1 hour - DataFrame, Dataset, UDFs, Caching

### Doubts
1. How is bucketing different from partitioning?
2. What is executor and how is it related to the partitions created?
3. What is salting and how exactly does it solve the skew issue?
4. Is the main selling point of Databricks using notebooks in production instead of spark submit?
5. Why do api call happen on driver?
6. While bucketing why does overwrite not work but append does?

### References
1. https://stackoverflow.com/questions/19128940/what-is-the-difference-between-partitioning-and-bucketing-a-table-in-hive

## Day N + 4
### Duration : 1.75 hour

### Learnings
* Bucketing is used primarily to improve the performance of join operations. Before you perform a join, we have to bucket the records **on the join column**

* Spark applies a hash function to the specified column and divides the data into buckets corresponding to the hash values.

* bucketBy became available in version 2.3 of Spark

* Number of bucketing files = number of buckets * number of partitions

* Number of partitions depends on the cluster manager, the default parallelism is the number of CPU cores. To find number of cpu cores, we can use the multiprocessing library as follows

```
import multiprocessing

cores = multiprocessing.cpu_count() # Count the number of cores in a computer
print(cores)


```

* On doing saveAsTable, spark-warehouse folder is created

```
from pyspark.sql import SparkSession
# create spark session
spark = SparkSession.builder.appName("homework").getOrCreate()

matches = spark.read.option("header","true").csv("matches.csv")
match_details = spark.read.options("header", "true").csv("match_details.csv")

# creates 16 buckets based on match_id which is the join column
matches.write.bucketBy(16, "match_id").saveAsTable("matches_bucketed", format="parquet")
match_details.write.bucketBy(16, "match_id").saveAsTable("match_details_bucketed", format="parquet")

print(matches.rdd.getNumPartitions()) # 2 
# Google Colab has 2 cores by default, hence default parallelism is 2
# Total number of bucketing files created = num partitions * num buckets = 2 * 16 = 32
# Hence 32 files created in match_details_bucketed folder (excluding _SUCCESS file) inside spark-warehouse folder (refer 5)

# spark.table same as spark.read.table
# load the bucketed tables
matches_bucketed = spark.table("matches_bucketed")
match_details_bucketed = spark.table("match_details_bucketed")

matches_bucketed.join(match_details_bucketed, matches_bucketed.match_id == match_details_bucketed.match_id, "left").count()

```



```

== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [match_id#160], [match_id#197], LeftOuter
   :- Sort [match_id#160 ASC NULLS FIRST], false, 0
   :  +- Exchange hashpartitioning(match_id#160, 200), ENSURE_REQUIREMENTS, [plan_id=678]
   :     +- FileScan csv [match_id#160,mapid#161,is_team_game#162,playlist_id#163,game_variant_id#164,is_match_over#165,completion_date#166,match_duration#167,game_mode#168,map_variant_id#169] Batched: false, DataFilters: [], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/matches.csv], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string...
   +- Sort [match_id#197 ASC NULLS FIRST], false, 0
      +- Exchange hashpartitioning(match_id#197, 200), ENSURE_REQUIREMENTS, [plan_id=679]
         +- Filter isnotnull(match_id#197)
            +- FileScan csv [match_id#197,player_gamertag#198,previous_spartan_rank#199,spartan_rank#200,previous_total_xp#201,total_xp#202,previous_csr_tier#203,previous_csr_designation#204,previous_csr#205,previous_csr_percent_to_next_tier#206,previous_csr_rank#207,current_csr_tier#208,current_csr_designation#209,current_csr#210,current_csr_percent_to_next_tier#211,current_csr_rank#212,player_rank_on_team#213,player_finished#214,player_average_life#215,player_total_kills#216,player_total_headshots#217,player_total_weapon_damage#218,player_total_shots_landed#219,player_total_melee_kills#220,... 12 more fields] Batched: false, DataFilters: [isnotnull(match_id#197)], Format: CSV, Location: InMemoryFileIndex(1 paths)[file:/content/match_details.csv], PartitionFilters: [], PushedFilters: [IsNotNull(match_id)], ReadSchema: struct<match_id:string,player_gamertag:string,previous_spartan_rank:string,spartan_rank:string,pr...


== Physical Plan ==
AdaptiveSparkPlan isFinalPlan=false
+- SortMergeJoin [match_id#371], [match_id#391], LeftOuter
   :- Sort [match_id#371 ASC NULLS FIRST], false, 0
   :  +- FileScan parquet spark_catalog.default.matches_bucketed[match_id#371,mapid#372,is_team_game#373,playlist_id#374,game_variant_id#375,is_match_over#376,completion_date#377,match_duration#378,game_mode#379,map_variant_id#380] Batched: true, Bucketed: true, DataFilters: [], Format: Parquet, Location: InMemoryFileIndex(1 paths)[file:/content/spark-warehouse/matches_bucketed], PartitionFilters: [], PushedFilters: [], ReadSchema: struct<match_id:string,mapid:string,is_team_game:string,playlist_id:string,game_variant_id:string..., SelectedBucketsCount: 16 out of 16
   +- Sort [match_id#391 ASC NULLS FIRST], false, 0
      +- Filter isnotnull(match_id#391)
         +- FileScan parquet spark_catalog.default.match_details_bucketed[match_id#391,player_gamertag#392,previous_spartan_rank#393,spartan_rank#394,previous_total_xp#395,total_xp#396,previous_csr_tier#397,previous_csr_designation#398,previous_csr#399,previous_csr_percent_to_next_tier#400,previous_csr_rank#401,current_csr_tier#402,current_csr_designation#403,current_csr#404,current_csr_percent_to_next_tier#405,current_csr_rank#406,player_rank_on_team#407,player_finished#408,player_average_life#409,player_total_kills#410,player_total_headshots#411,player_total_weapon_damage#412,player_total_shots_landed#413,player_total_melee_kills#414,... 12 more fields] Batched: true, Bucketed: true, DataFilters: [isnotnull(match_id#391)], Format: Parquet, Location: InMemoryFileIndex(1 paths)[file:/content/spark-warehouse/match_details_bucketed], PartitionFilters: [], PushedFilters: [IsNotNull(match_id)], ReadSchema: struct<match_id:string,player_gamertag:string,previous_spartan_rank:string,spartan_rank:string,pr..., SelectedBucketsCount: 16 out of 16




```


### Doubts

1. How does bucketing happen and is there any way to check the bucketing in the parquet files?
2. Should we sort the table after bucketing and if so why?
3. Should we save the bucketed table using saveAsTable or can we use the bucketed data directly without saving as table?
4. How does repartitioning before bucketing help optimize bucketing?
5. What is bucket pruning?

### References
1. https://www.reddit.com/r/GoogleColab/comments/1edmrxl/how_to_turn_off_code_complete_in_notebooks/?rdt=49194
2. https://luminousmen.com/post/the-5-minute-guide-to-using-bucketing-in-pyspark/
3. https://stackoverflow.com/questions/73346771/why-does-spark-re-sort-the-data-when-the-join-of-the-two-tables-are-bucketed-and
4. https://medium.com/@diehardankush/what-all-about-bucketing-and-partitioning-in-spark-bc669441db63
5. https://books.japila.pl/spark-sql-internals/bucketing/#creating-bucketed-tables
6. https://stackoverflow.com/questions/67135876/how-many-cpu-cores-does-google-colab-assigns-when-i-keep-n-jobs-8-is-there-an
7. https://www.reddit.com/r/apachespark/comments/1d57daf/how_to_decide_optimal_number_of_partitions/
8. https://aspinfo.medium.com/how-to-improve-performance-with-bucketing-in-pyspark-c66a899e70b5


## Day N + 5

### Duration : 1.5 hours
* Spark mainly optimizes using 2 ways - by being distribted processing and by lazy evaluation. Important concepts linked to each of these are below
   * Distributed processing : partitioning, rdd, architecture (worker node, driver, executor, core), repartition, coalesce, shuffle, broadcasting, bucket, Out of Memory (OOM) errors,  
   * Lazy evaluation : DAG, Transformation, Action (Job, Stage, Task), Cache, Persist, Checkpoint, Salting , catalyst optimizer, predicate , partition pruning, 

* Process : Process is an instance of a computer program that is being executed

* Spark uses a master/slave architecture. It has one central coordinator (called Driver) that communicates with many distributed workers (called executors)

* Node : A physical or virtual machine

* Worker: Worker is actually a node or machine. Either you say it worker or worker node both are same

* Executor : Executor is a process inside the worker node and a single worker node can have multiple executors. Spark executor is a single JVM instance on a node that serves a single spark application

* Core(aka.Thread or Slot) : Core property controls the number of concurrent tasks an executor can run. For example if you request 2 executor each with 2 cores then you can run 4 concurrent tasks at the same time during your job execution.

* Driver : 

* Spark perform all its computation in memory. It would just spill the data to the disk only when it does not fit in memory

* Partitions could be anywhere and might not be equally distributed in most of the cases.It could happen some of the executors does not have a single partition and other executors have more than 2 partitions.

* Task : the lowest unit of work that performs the actual ask. 

### Doubts
1. What is difference bw worker and executor in Spark?
2. Is the idea of a JVM associated with an executor process or at a higher "worker" level?
3. A partition is a portion of the actual data. This split could happen using hashing, round robin or range. What determines the location of these data partitions?
4. If I have less executors (say, 10) than the partitions (say, 20), does it mean that only 10 tasks will be executed in parallel at any point? is the degree of parallelism constrained by the number of executors?
5. How to do group by in Pyspark
6. How does bucketing look like in Spark dag?

### References
1. https://stackoverflow.com/questions/34514545/sort-in-descending-order-in-pyspark
2.https://stackoverflow.com/questions/24696777/what-is-the-relationship-between-workers-worker-instances-and-executors
3.https://stackoverflow.com/questions/68560515/what-is-the-relationship-between-a-node-worker-executor-task-and-partition


## Day N + 6, N+7, N+8
### Duration : 4 + 3 + 2 hours

* Worked on and completed Pyspark homework

* Errors faced :
   * AMBIGUOUS_REFERENCE - Reference `match_id` is ambiguous. Cause : In the previous line join was done on match_id, hence 2 columns with the same name match_id. Solution : Changed the join statement, specify the join column as an array. This is similar to the USING keyword in SQL
```
# old expression
matches_combined_df = matches_bucketed.join(match_details_bucketed, matches_bucketed.match_id == match_details_bucketed.match_id, "left")

# new expression
matches_combined_df = matches_bucketed.join(match_details_bucketed, ["match_id"], "left")

```
   * py4j.Py4JException: Method executePlan([class org.apache.spark.sql.catalyst.plans.logical.Filter]) does not exist, when running code below Cause : CHange in PySpark internals in newer version
```
catalyst_plan = matches_combined_df._jdf.queryExecution().logical()
size_bytes = spark._jsparkSession.sessionState().executePlan(catalyst_plan).optimizedPlan().stats().sizeInBytes()
```



* If 2 tables have same column names, and if we are joining on those columns, then we can specify join columns in an array or use withColumnRenamed before join. The second approach can be used even if we are not joining on the column with same names (just like in the assignment)

* To get size of a dataframe in MB we can use `df.explain('cost')`. 

* To see version of Pyspark use `pyspark --version` in terminal (prefix with ! if jupyter notebook). Similarly `spark-submit --version` and `spark-shell --version` 


```
### Homework3.py

from pyspark.sql import SparkSession
from pyspark.sql.functions import broadcast
import pyspark.sql.functions as F

# create spark session
spark = SparkSession.builder.appName("homework").getOrCreate()

# disable automatic broadcast join
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

# load data
maps = spark.read.option("header","true").csv("maps.csv")
medals = spark.read.option("header","true").csv("medals.csv")
matches = spark.read.option("header","true").csv("matches.csv")
match_details = spark.read.option("header","true").csv("match_details.csv")
medals_matches_players = spark.read.option("header","true").csv("medals_matches_players.csv")

# store data in buckets for optimised join
matches.write.mode("overwrite").bucketBy(16, "match_id").saveAsTable("matches_bucketed", format="parquet")
match_details.write.mode("overwrite").bucketBy(16, "match_id").saveAsTable("match_details_bucketed", format="parquet")
medals_matches_players.write.mode("overwrite").bucketBy(16, "match_id").saveAsTable("medals_matches_players_bucketed", format="parquet")

# join bucketed tables
# matches_bucketed = spark.read.parquet("matches_bucketed")
matches_bucketed = spark.table("matches_bucketed")
match_details_bucketed = spark.table("match_details_bucketed")
medals_matches_players_bucketed = spark.table("medals_matches_players_bucketed")

matches_combined_df = matches_bucketed.join(match_details_bucketed, ["match_id"], "left")
matches_combined_df = matches_combined_df.join(medals_matches_players_bucketed, ["match_id", "player_gamertag"], "left")


# broadcast join maps
matches_combined_df = matches_combined_df.join(broadcast(maps), ["mapid"], "left")
# broadcast join medals
medals = medals.withColumnRenamed("name","medal_name")
matches_combined_df = matches_combined_df.join(broadcast(medals), ["medal_id"], "left")

# find player who averages the most kills per game
avg_kills_df = matches_combined_df.groupBy("player_gamertag").agg(F.avg("player_total_kills").alias("avg_kills")).orderBy("avg_kills", ascending=False)

# find playlist that gets played the most
playlist_count_df = matches_combined_df.groupBy("playlist_id").agg(F.count("match_id").alias("playlist_count")).orderBy("playlist_count", ascending=False)

# find map that gets played the most
top_maps = matches_combined_df.groupby('mapid').count().orderBy("count", ascending=False).limit(3)

# find map players get the most Killing Spree medals on
killing_spree_df = matches_combined_df.filter(F.col("medal_name") == "Killing Spree")
top_maps_ks = killing_spree_df.groupBy("mapid").agg(F.count("match_id").alias("killing_spree_medal_count")).orderBy("killing_spree_medal_count", ascending=False)

# Try different .sortWithinPartitions to see which has the smallest data size
matches_combined_df = matches_combined_df.repartition(4, F.col("completion_date")).sortWithinPartitions(F.col("completion_date"))
print(matches_combined_df.explain('cost'))
matches_combined_df = matches_combined_df.repartition(4, F.col("mapid")).sortWithinPartitions(F.col("mapid"))
print(matches_combined_df.explain('cost'))
matches_combined_df = matches_combined_df.repartition(4, F.col("match_id")).sortWithinPartitions(F.col("mapid"))
print(matches_combined_df.explain('cost'))



```

* Py4j : Bridge between Apache Spark JVM codebase and Python client applications. PySpark's DataFrame has a private _jdf property which is an instance of Py4j's JavaObject referencing DataFrame instance on the JVM side

* 

### Doubts
1. While joining 2 tables in Pyspark, how to handle same column name in both tables, when join is done on the common column and when the common column name is not the join key?
2. SessionState is the state separation layer between Spark SQL sessions, including SQL configuration, tables, functions, UDFs, SQL parser, and everything else that depends on a SQLConf. What does this mean?
3. When to use alias vs withColumnRenamed?
4. What is the difference b/w SparkContext and SparkSession in Pyspark?

### References
1. https://stackoverflow.com/questions/33778664/spark-dataframe-distinguish-columns-with-duplicated-name
2. https://kb.databricks.com/data/join-two-dataframes-duplicated-columns.html
3. https://stackoverflow.com/questions/50287558/how-to-rename-duplicated-columns-after-join
4. https://stackoverflow.com/questions/62411830/how-to-find-size-in-mb-of-dataframe-in-pyspark
5. https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/spark-sql-SessionState.html
6. https://stackoverflow.com/questions/62793652/select-number-of-partitions-on-basis-of-size-of-file-read-by-spark
7. https://stackoverflow.com/questions/74192463/difference-between-alias-and-withcolumnrenamed
8. https://www.waitingforcode.com/pyspark/pyspark-jvm-introduction-2/read
9. https://stackoverflow.com/questions/54124386/capturing-the-result-of-explain-in-pyspark

## Milestone - Successfully submitted Spark Homework on 15-Jan-2025

## Day P

### Duration : 1.5 hours

* Event store : An event store is a type of database optimized for storage of events

* Stream processing : A data management technique that involves ingesting a continuous data stream to quickly analyze, filter or transform the data in real time. Once processed, the data is passed off to an application, data store or another stream processing engine.

* Event sourcing : Event Sourcing is an architectural design pattern where changes that occur in a domain are immutably stored as events in an append-only log. In simple words, whenever there is a change (say for example Order Status changed from Pending to Paid), a new record is created in the database instead of overwriting the existing record

* Apache Kafka : Apache Kafka is a distributed event store and stream-processing platform. 

* Apache Flink : Apache Flink is a stream processing framework with powerful stream- and batch-processing capabilities

* Apache Flink can connect with (i.e. write the data to) various storage backends such as
   * Apache HDFS for batch access (with different storage format such as Parquet, ORC, custom binary) 
   * Apache Kafka if you want to access the data as a stream 
   * Key-value store such as Apache HBase and Apache Cassandra for point access to data 
   * Databases such as MongoDB, MySQL, Postgresql

* Journey to setup Flink for W4 lab
   * Set environment variables in flink-env.env
   * While trying to run docker compose faced following error `Docker | failed to solve with frontend dockerfile.v0: failed to create LLB definition: rpc error: code = Unknown desc`
   * To solve issue, deleted docker config file - Went to `C:\Users\<Username>\.docker` and deleted config.json 
   * Ran the following docker command (from folder containing dockerfile) : `docker compose --env-file flink-env.env up --build --remove-orphans  -d`
   * Started the Postgres container from W1 and then ran the following SQL
   ```
   -- Create processed_events table
   CREATE TABLE IF NOT EXISTS processed_events (
      ip VARCHAR,
      event_timestamp TIMESTAMP(3),
      referrer VARCHAR,
      host VARCHAR,
      url VARCHAR,
      geodata VARCHAR
   );
   ```

###
1. Is Flink just a processing library or can it also store data? In that sense is it similar to Spark, which is a processing library i.e. Spark only stores data temporarily in memory during processing and does not have a persistent storage?



### References
1. https://stackoverflow.com/questions/74583214/docker-failed-to-solve-with-frontend-dockerfile-v0-failed-to-create-llb-defin
2. https://www.kurrent.io/event-sourcing
3. https://stackoverflow.com/questions/34767316/can-we-use-apache-spark-to-store-data-or-is-it-only-a-data-processing-tool
4. https://stackoverflow.com/questions/31951978/storage-in-apache-flink
5. https://quix.io/blog/pyflink-deep-dive



## Day P + 2
## Duration : 2.5 hours

### Learnings
* Learnt the fundamental terms used in Kafka

* Messages : The unit of data within Kafka is called a message. 

* Event : What is called "event" in the streaming context (if we speak about Kafka Streams API) is a "message" in the normal Kafka usage. 

* Kafka, at its core, only transfers data in byte format. There is no data verification that’s being done at the Kafka cluster level. In fact, Kafka doesn’t even know what kind of data it is sending or receiving; whether it is a string or integer

* Topic : Named stream of records/collection of messages. Kafka topics are categories used to organize messages, and are analogous to tables in a database or folders in a filesystem

* Partitioning : A single topic log is broken into multiple logs, each of which can live on a separate node in the Kafka cluster, and this is known as partitioning. This allows Kafka to be scalable, because if there was no partitioning, then a topic would be constrained to live entirely on one node, and would be limited to the size of that node. Partitions allow a topic's log to scale beyond a size that will fit on a single server (a broker) and act as the unit of parallelism. Hence Kafka is a distributed system, as it can handle multiple partitions stored across multiple nodes.

* Messages are written to a partition in an append-only fashion and are read in order from beginning to end.

* Producer : Producers create new messages. A message will be produced to a specific topic

* Consumer : Applications that read data from Kafka topics are called consumers. The consumer subscribes to one or more topics and reads the messages in the order in which they were produced to each partition.

* Offset : Kafka offsets are identifiers of messages within a Kafka partition. They represent the order of a message from the beginning of a partition. Hence each message in a Kafka topic has a partition ID and an offset ID attached to it

* Offset commit :  Kafka allows consumers to use Kafka to track their position (offset) in each partition. The action of updating the current position in the partition is called an offset commit (just like we have commit operation in a database)

* Consumer group : Consumers that are part of the same application and therefore performing the same "logical job" can be grouped together as a Kafka consumer group. In order for indicating to Kafka consumers that they are part of the same specific group , we must specify the consumer-side setting group.id. Consumer groups are used for parallel processing of event messages from the specific topic.


* Group coordinator :  Group coordinator (which is one of the brokers) helps to distribute the data in the subscribed topics to the consumer group instances evenly. The coordinator uses an internal Kafka topic to keep track of group metadata.

* Broker : A single Kafka server i.e. network of machines (physical or vm) is called a broker. Each broker hosts some set of partitions and handles incoming requests to write new events to those partitions or read events from them. In simpler words, broker receives messages from producers, assigns offsets to those messages, and writes the messages to storage on disk. On the consumer side, it responds to fetch requests for partitions and responds with the messages that have been published.

* Cluster : Combination of Kafka brokers

* Replication factor : A replication factor is the number of copies of same data stored over multiple brokers. This is similar to Spark, and is done so that even if one broker goes down, we still have backup copies of the data

* Kafka separates storage from compute. Storage is handled by the brokers and compute is mainly handled by consumers or frameworks built on top of consumers (like apache flink, kafka streams)

* In one consumer group, each partition of a topic will be processed by one consumer only. These are the possible scenarios:
   * `Num consumers > num topic partitions` : Some consumers will be idle 
   * `Num consumers = num topic partitions` : Each consumer can work on one partition
   * `Num consumers < num topic partitions` : Each consumer can work on multiple partition
   In any case, there cannot be more than consumer for a single partition. Note that consumer should be aware of the number of partitions in a given topic

* Python libraries that can be used with Kafka include
   * kafka-python
   * confluent-kafka
   * quixstreams

* Quix Streams is an end-to-end framework for real-time Python data engineering, operational analytics and machine learning on Apache Kafka data streams. 

* Bootstrap servers are Kafka brokers.

* Steps followed to launch Apache Kafka using Docker
   * Create the following docker-compose.yml file in a new folder

```
version: '2'

services:
  zookeeper:
    image: wurstmeister/zookeeper:latest
    ports:
      - "2181:2181"

  kafka:
    image: wurstmeister/kafka:latest
    ports:
      - "9092:9092"
    expose:
      - "9093"
    environment:
      KAFKA_ADVERTISED_LISTENERS: INSIDE://kafka:9093,OUTSIDE://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
      KAFKA_LISTENERS: INSIDE://0.0.0.0:9093,OUTSIDE://0.0.0.0:9092
      KAFKA_INTER_BROKER_LISTENER_NAME: INSIDE
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_CREATE_TOPICS: "my-topic:1:1"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

```
   * Open a terminal in the directory where the docker-compose.yml file is located and run the following command `docker-compose up -d`
   * Run the following commands to create/list topics

```
## create a topic called my-topic
docker exec -it <kafka-container-id> /opt/kafka/bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic my-topic

## list all topics in a cluster
docker exec -it ccf45bb56f71 /opt/kafka/bin/kafka-topics.sh --list --zookeeper zookeeper:2181

## list all topics in a cluster (pass the Kafka cluster address directly using the –bootstrap-server option)
docker exec -it ccf45bb56f71 /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server=localhost:9092

## look at details of a specific topic (such as number of partitions and replicas)
docker exec -it ccf45bb56f71 /opt/kafka/bin/kafka-topics.sh --
bootstrap-server=localhost:9092 --describe --topic weather_data_demo

```

* Produced data to a Kafka topic using quixstreams library

```
# pip install requests
# pip install quixstreams

import requests
import json
from quixstreams import Application

response = requests.get(
    "https://api.open-meteo.com/v1/forecast",
    params = {
        "latitude":51.1,
        "longitude":-0.11,
        "current":"temperature_2m",
    }
)

print(response.json())
weather = response.json()

app = Application(
    broker_address="localhost:9092",
    loglevel="DEBUG"
)

# producer is Kafka parlance for write-only connection
with app.get_producer() as producer:
    producer.produce(
        topic="weather_data_demo",
        key="London", # to identify the data
        value=json.dumps(weather) 
    )


```

### Doubts
1. What is the difference bw a topic and queue (as mentioned in reference 12)?
2. What is the real world use case of consumers gorups vs a single consumer?

### References

1. https://medium.com/@amberkakkar01/getting-started-with-apache-kafka-on-docker-a-step-by-step-guide-48e71e241cf2
2. https://medium.com/slalom-technology/introduction-to-schema-registry-in-kafka-915ccf06b902
3. https://developer.confluent.io/courses/apache-kafka/partitions/
4. https://www.redpanda.com/guides/kafka-architecture-kafka-offset
5. https://learn.conduktor.io/kafka/kafka-consumer-groups-and-consumer-offsets/
6. https://developer.confluent.io/courses/apache-kafka/brokers/
7. https://github.com/ixaxaar/kafka-tutorial
8. https://www.baeldung.com/ops/kafka-list-topics
9. https://www.youtube.com/watch?v=D2NYvGlbK0M
10. https://github.com/Sumanshu-Nankana/kafka-python/blob/main/src/producer.py
11. https://stackoverflow.com/questions/45196003/event-vs-topic-apache-kafka
12. https://abhishek1987.medium.com/kafka-is-it-a-topic-or-a-queue-30c85386afd6
13. https://medium.com/@emer.kurbegovic/queues-vs-topics-a-simple-guide-with-real-world-examples-1d32947cb574
14. https://stackoverflow.com/questions/38024514/understanding-kafka-topics-and-partitions
15. https://developer.confluent.io/courses/architecture/consumer-group-protocol/
16. https://stackoverflow.com/questions/75390937/what-is-the-need-of-consumer-group-in-kafka

## Day P + 3

### Duration : 1 hour

### Learnings

* State : State means the current state of things. For example when you open a water bottle, the state is open, when you close it the state is closed. In Flink state enables to store information of past messages. State can be as simple as a flag or a complex object. For example:
   * When an application searches for certain event patterns (say fraud detection in a bank), the state will store the sequence of events encountered so far (series of trasactions for the bank account)
   * When aggregating events per minute, the state holds the pending aggregates.
   * When training a machine learning model over a stream of data points, the state holds the current version of the model parameters.

* Table API : For batch an stream processing, allows to write queries similar to SQL

* DataStream API : Suitable for building stateful stream processing applications.  Stateful stream processing maintains a memory of past events. This memory, or "state", allows the processing system to track changes over time or across different data points.

* Job : What work is done, basically a set of processes. A process is an instance of a program that is being executed (note that these are all operating system concepts)

* A Flink datastream consists of multiple moving parts, including Sources, Sinks, and Operators. 

* Source : Origin of the data

* Sink : Database or streaming platform FLink writes to

* The real power of Flink comes from its ability to transform data in a distributed streaming pipeline. Contains a variety of operators such as :
   * Column operations (e.g., add, replace, remove, rename columns). 
   * Row-based operations (e.g., map). 
   * Aggregations such as grouping by clause and aggregating group rows.
   * Joins (for example, inner, outer, and interval joins). 
   * Windowing (e.g., sliding, tumbling, group windows).
   * Customer operations using UDF 

* Transformation operations in Spark Streaming include : map(), flatMap(), filter(), repartition(numPartitions), union(otherStream), count(), reduce(), countByValue(), reduceByKey(func, [numTasks]), join(otherStream, [numTasks]), cogroup(otherStream, [numTasks]), transform(), updateStateByKey(), Window()

* Serialization : Process of transforming objects into compact, easily transmittable byte streams

* Checkpoint : A mechanism to store states. This is triggered automatically at periodic intervals

* Savepoint : Triggered manually by the user. Checkpoints are taken automatically and are used for automatic restarting job in case of a failure.whereas savepoints are taken manually, are always stored externally and are used for starting a "new" job

### Doubts
1. Does storing state meaning storing previous messages or storing info of the processing task?

### References
1. https://www.reddit.com/r/learnprogramming/comments/rem1ec/eli5_what_exactly_is_state/
2. https://nightlies.apache.org/flink/flink-docs-release-1.3/dev/stream/state.html
3. https://www.youtube.com/watch?v=VAhgOmyOyRY
4. https://quix.io/blog/navigating-stateful-stream-processing
5. https://quix.io/blog/pyflink-deep-dive
6. https://stackoverflow.com/questions/3073948/job-task-and-process-whats-the-difference
7. https://medium.com/@parinpatel094/wrangling-data-with-speed-a-deep-dive-into-apache-flinks-kryo-dadf5da81ab7
8. https://www.alibabacloud.com/blog/flink-checkpoints-principles-and-practices-flink-advanced-tutorials_596631
9. https://data-flair.training/blogs/apache-spark-streaming-tutorial/

## Day P + 4
### Duration : 1.25 hours

### Learnings

* Docker image : A lightweight, standalone, executable package of software that includes everything needed to run an application

* Process : An instance of a program that is being executed. In simple words, a program in execution

* Docker container : Container is simply an isolated process with all of the files it needs to run.  They provide a way of creating an isolated environment(sandbox) in which applications and their dependencies can live.

* Container networking : Ability for containers to connect to and communicate with each other, or to non-Docker workloads.

* User defined network : A custom network created by user to connect multiple containers.  We can create a docker network using `docker network create <network-name>`

* Network drivers : Software that runs your networking hardware (they activates the actual transmission and receipt of data over the network). Some network drivers available in Docker are
   * bridge : default one
   * host : Remove network isolation between the container and the Docker host.

* The execution of an application in Flink mainly involves three entities: the Client, the JobManager and the TaskManagers. 

* Client : Client is responsible for submitting the application to the cluster, for example:
   * Command line
   * SQL client : For submitting SQL tasks to Flink (`sql-client.sh` in bin directory)
   * Scala Shell : For submitting Table API tasks (`start-scala-shell.sh` in bin directory)

* JobManager : Responsible for the necessary bookkeeping during execution i.e. keeps track of distributed tasks, decides when to schedule the next task/ tasks, and reacts to finished tasks or execution failures.

* TaskManagers : The ones doing the actual computation.

* Application mode : Spin up a Flink cluster for each submitted job, which is available to that job only. When the job finishes, the cluster is shut down and any lingering resources (e.g. files) are cleaned up. In Docker, in this mode, you start a Flink cluster that is dedicated to run only the Flink Jobs which have been bundled with the images

* Session mode : Assumes an already running cluster and uses the resources of that cluster to execute any submitted application. Basically new application can be submiited on the go, and applications executed in the same session use same resources

* To see which mode (application or sesssion) is used, in Docker look at the command key in docker-compose.yml

* Dockerfile vs docker-compose.yml : Dockerfile is used to build images while the docker-compose.yaml file is used to run images. 

* docker-compose can be considered a wrapper around the docker CLI. So instead of doing docker build on the Dockerfile, we have a `build : .` in docker-compose.yml



### Doubts

### References

1. https://www.javatpoint.com/process-vs-program
2. https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/
3. https://docs.docker.com/engine/network/drivers/
4. https://www.alibabacloud.com/blog/apache-flink-fundamentals-five-modes-of-client-operations_595729
5. https://medium.com/@chunilalkukreja/apache-flink-application-vs-session-mode-35c830d9d49c
6. https://flink.apache.org/2020/07/14/application-deployment-in-flink-current-state-and-the-new-application-mode/
7. https://nightlies.apache.org/flink/flink-docs-master/docs/internals/job_scheduling/
8. https://nightlies.apache.org/flink/flink-docs-master/docs/deployment/resource-providers/standalone/docker/
9. https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Dockerfile-vs-docker-compose-Whats-the-difference
10. https://stackoverflow.com/questions/50230399/what-is-the-difference-between-docker-compose-build-and-docker-build
11. https://stackoverflow.com/questions/29480099/whats-the-difference-between-docker-compose-vs-dockerfile
