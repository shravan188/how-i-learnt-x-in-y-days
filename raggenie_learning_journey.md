# RagGenie Learning Journey

### High level goal : To contribute to Raggenie codebase

## Day 0

### Duration : 1 hour

### Learnings

* Backend in app folder, frontend in ui folder
* List of important libraries used in the backend
    * Poetry (for setup)
    * FastAPI + Starlette (for backend server)
    * SQLAlchemy (for database connections)
    * psycopg2 (for postgres)
    * Loguru (for logging)
    * Langchain
    * Chromadb (vector database)
    * Onnxruntime 
    * Pydantic (data validation package)


### Doubts
1. What are abstract base classes? What is abstract method?
2. What is a mixin in Python?
3. What is Open Neural Network Exchange (ONNX)?
4. What s difference bw Pydantic, typing, data classes, annotations and typeddicts?
5. Can we use Trafilatura in website plugin or url reader?

### Resources
Nil

## Day 1

### Duration : 1.5 hour

### Learnings
* **Abstract class** : blueprint for other classes i.e. any child class of the abstract class must declare the methods defined in the abstract class. Abstract class can have constructors, variables, abstract methods, non-abstract methods. Abstract classes are used when we want to provide a common interface for different implementation of a component. We use abc class in Python to create abstract class.

* **Abstract method** : a method that has a declaration but no implementation. If this method is not implemented in subclass, it will throw an exception

```
from abc import ABC, abstractmethod

# inherit from abstract base class
class Animal(ABC):
  @abstractmethod
  def feed(self):
    pass
 
  #sleep is not defined as an abstract method, hence even if the subclasses do not implement it, it will not throw an error
  def sleep(self):
    pass


# wrong definition - will throw an error when creating an object, as feed abstract method is not defined
class Lion(Animal):
  def roar(self):
    print("Lion roars")

# right definition - will work properly during instantiation
class Lion(Animal):
  def feed(self):
    print("Lion eats")
    
  def roar(self):
    print("Lion roars")

lion1 = Lion()
isinstance(lion1, Lion) # True
lion1.roar() # Lion roars

```
* To see is a class is subclass of another class, we can use the function issubclass. Similarly, isinstance to see if an object is a instance of a class. 

* Multiple inheritance in python : A single class can inherit from multiple parent classes

* Method vs function : A method is a function which is associated with an object. A function is just a block of reusable code that accepts arguments 

* **Classmethod** : method that is bound to class rather than object created from that class (Python has 3 kind of methods : Instance method, class method and static method)

```
class A(object):
    def x(self):
        print(self)

    @classmethod
    def y(cls):
        print(cls)

a = A()
b = A()

## self object in x is tied to the object, whereas cls is tied to class from which the object is created

print(a.x()) # <__main__.A object at 0x7eecdaeb65f0>
print(b.x()) # <__main__.A object at 0x7eecdaeb6c20>
print(a.y()) # <class '__main__.A'>
print(b.y()) # <class '__main__.A'>

```
* The use of self as an argument for instance method and cls for classmethod is just a naming convention, but it is better to stick to it

### Doubts
1. How to simulate abstract method withhout abc?
2. Can abstract class have non abstract methods? Can abstract method have an implementation in the abstract class itself?
3. What is the difference between abstract class and metaclass?
4. What is diamond problem in multiple inheritance?
5. When do we use classmethod vs static method?

### Resources
1. https://www.reddit.com/r/learnprogramming/comments/1bynrj2/in_python_what_is_the_difference_between_a_method/?rdt=47989
2. https://stackoverflow.com/questions/12179271/meaning-of-classmethod-and-staticmethod-for-beginner
3. https://stackoverflow.com/questions/8689964/why-do-some-functions-have-underscores-before-and-after-the-function-name
4. https://realpython.com/python-double-underscore/
5. https://stackoverflow.com/questions/27186296/can-i-pass-self-as-the-first-argument-for-class-methods-in-python


## Day 2
### Duration : 3 hours

### Learnings


* Docker image : It is a template (a file) from which the docker container is created

* Docker image layer : A layer, or image layer is a change on an image, or an intermediate image. Because they can become quite large, docker images are designed to be composed of layers of other images, allowing a minimal amount of data to be sent when transferring images over the network. The concept of layers comes in handy at the time of building images. Because layers are intermediate images, if you make a change to your Dockerfile, docker will rebuild only the layer that was changed and the ones after that. This is called layer caching.

* Docker container : An instance of the image. A running container has a currently executing process

* Dockerfile → (Build) → Image → (Run) → Container.

* Below are the steps followed to run postgres within docker

```
# to pull an image from docker hub
docker pull postgres

# to see all images
docker images

# to see all the layers in the postgres docker iamge
docker image history postgres


# create a container with the name postgres-rg from the image postgres
# e is env variable, set POSTGRES_PASSWORD environment variable as password
# d means detach i.e. run container in background
# p binds containers port to host port
docker run --name postgres-rg -e POSTGRES_PASSWORD=password -d -p 5432:5432 postgres

# start postgres-rg container
docker start postgres-rg

# to see all running containers
docker ps

# exec allows you to run/execute commands within a running container (in this case bash command)
# -i keeps stdin connected; if you don't specify -i, the shell will simply exit.
# -t allocates a tty device; if you don't specify -t, you won't have a very pleasant interactive experience (there will be no shell prompt or job control, for example)
# bash means we can run bash within the postgres-rg container
# type exit to come out of bash in container
docker exec -it postgres-rg bash

# login as user postgres, postgres refers to operating system user/default user
# If we just run psql it gives gives error 
# psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" 
# failed: FATAL:  role "root" does not exist
psql -U postgres


# list all users in the current database server.
\du

# create database
create database test; (dont miss semicolon)

# List all databases
\l 

# quit the file
Q

# Connect to database with name test
\c test 

# create table test with columns id, num and data
CREATE TABLE test (id serial PRIMARY KEY, num integer, data varchar);

# List tables from current schema
\dt

# insert values into test table, returns number of rows inserted
INSERT INTO test (num, data) VALUES (100, 'abc');

# see all rows from test table of test database
SELECT * FROM test;

# Quit i.e. exit psql
\q

# sample output of psql
postgres=# Select 1;
 ?column?
----------
        1
(1 row)
```

* tty : short for teletype and perhaps more commonly called a terminal.  It helps perform input and output on a character-by-character basis

* For a freshly initialized postgres system, there is one user with default username as postgres. Its role is always a “superuser”. 

* Once we create a postgres database in Docker, we connect to it using psycopg2

```
#### postgres.py (remember to install psycopg2 within virtual environment)

import psycopg2

# Connect to an existing database with name test as user postgres
conn = psycopg2.connect(dbname="test", user="postgres", password="password")

# Open a cursor to perform database operations
cur = conn.cursor()

# Execute insert command against database, test table already created using psql
cur.execute("INSERT INTO test (num, data) VALUES (%s, %s)", (00, "def"))

# Execute select command against database
cur.execute("SELECT * FROM test;")

# Make changes to database persistent
conn.commit()

# Close communication with database
cur.close()
conn.close()

```

### Doubts

1. What is the difference between docker image and container? 
2. How to restart docker image using its id?
3. What exactly is differene between running container in foreground versus in background?
4. What happens if we do not commit back to the database?

### References
1. https://tomcam.github.io/postgres/#opening-a-connection-locally
2. https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/
3. https://docs.docker.com/reference/cli/docker/container/run/
4. https://stackoverflow.com/questions/39666950/how-restart-a-stopped-docker-container
5. https://docs.docker.com/reference/cli/docker/container/exec/
6. https://itsfoss.com/what-is-tty-in-linux/
7. https://stackoverflow.com/questions/30172605/how-do-i-get-into-a-docker-containers-shell
8. https://stackoverflow.com/questions/11919391/postgresql-error-fatal-role-username-does-not-exist
9. https://stackoverflow.com/questions/17963348/how-to-disconnect-from-a-database-and-go-back-to-the-default-database-in-postgre
10. https://stackoverflow.com/questions/31222377/what-are-docker-image-layers 
11. Postgres sharding demo - https://github.com/smcclure17/sharding-demo/tree/main/sharding_demo
12. https://stackoverflow.com/questions/23735149/what-is-the-difference-between-a-docker-image-and-a-container


## Day 3

### Duration : 2 hours

### Learnings
* In a function call **d means "treat the key-value pairs in the dictionary as additional named arguments to this function call."

```
dict = {"a":1, "b":2}

def foo(a, b):
    print(a, b)

foo(**dict) # translates to foo(a=1, b=2)

```

* Cursor factory : helps us set the type of output we want from the executed query. The default output is a list of tuples but by using cursor factor we can change it to other data types like list of dictionaries (so that we can use column names as keys to access data)

```
## Normal cursor
import psycopg2

# Connect to an existing database with name test as user postgres
conn = psycopg2.connect(dbname="test", user="postgres", password="password")

# Open a cursor to perform database operations
cur = conn.cursor()

# Execute select command against database
cur.execute("SELECT * FROM test;")

# Show result of query execution
print(cur.fetchall())

# Close communication with database
cur.close()
conn.close()

## Output is in form of list of tuples 
## Output : [(1, 100, 'abc'), (2, 0, 'def')]
## It looks like a list,  but it's a DictRow. This means that you can still use the column names as keys to access the data :
## rows = cur.fetchall()
## print([row['datname'] for row in rows])



## Replace curr = conn.cursor() with cur = conn.cursor(cursor_factory=extras.RealDictCursor)
## Output : [RealDictRow([('id', 1), ('num', 100), ('data', 'abc')]), RealDictRow([('id', 2), ('num', 0), ('data', 'def')])]

## Replace curr = conn.cursor() with cur = conn.cursor(cursor_factory=extras.DictCursor)
## Output : [[1, 100, 'abc'], [2, 0, 'def']]


```

* While configuring datasource (example doing an insert or delete) we need to do connection.commit() after cursor.execute(), but while fetching data, we do not need to commit, instead after cursor.execute() we do cursor.fetchall(). In short, remember to commit the transaction after executing INSERT or DELETE

* select 1 from table will return the constant 1 for every row of the table that matches the WHERE clause, otherwise it returns nothing. It is used by some databases as a query to test a connection to see if it's alive.

* EXISTS operator is used to test for the existence of any record in a subquery. It returns True if subquery returns 1 or more records

```
-- Get all supplier names who sell product with a price of 22
SELECT SupplierName
FROM Suppliers
WHERE EXISTS (SELECT 1 FROM Products WHERE Products.SupplierID = Suppliers.supplierID AND Price = 22);

```

* To get table schema from postgres, we use INFORMATION_SCHEMA

### Doubts
1. Where are the plugins used? (all plugins are called in DSLoader class in loader.py (search for DSLoader class) -> in app/providers/container.py datasources attribute of Cntainer class uses DSLoader class (so search for Container class) -> found Container class in app/main.py where it is stored in config dictionary variable and then svc.update_datasource_documentations is called on that (search for update_datasource_documentations function) -> In app/services/connector.py, there is update_datasource_documentations function which class functions within the plugin like healthcheck, get data etc)
2. Notice that ? placeholders are used to bind data to the query. Always use placeholders instead of string formatting to bind Python values to SQL statements, to avoid SQL injection attacks. But why does string formatting cause SQL injection?
3. What is difference b/w exists and in in SQL?
4. What is the difference b/w RealDict and NamedTupleCursor?

### References
1. https://stackoverflow.com/questions/21809112/what-does-tuple-and-dict-mean-in-python
2. https://stackoverflow.com/questions/7171041/what-does-it-mean-select-1-from-table
3. https://www.psycopg.org/docs/extras.html
4. https://stackoverflow.com/questions/6739355/dictcursor-doesnt-seem-to-work-under-psycopg2
5. https://stackoverflow.com/questions/50666600/psycopg2-extras-dictcursor-not-returning-dict-in-postgres


## Day 4
### Duration : 2 hours

### Learnings
* Schema : Within a database/catalogue we have schemas. A schema is a collection of database objects, including tables, views, indexes and procedures, grouped together

* Table : The primary component of a schema that stores data as rows and columns

* Information_schema : This is a built in schema which is common to every PostgresSQL database. It is a collection of views, with each view containing information about the objects in the database. In short it contains the metadata of the database. Some of the important columns in the tables view of information_schema is :
  * table_schema 
  * table_name
  * table_type (eg. base table, view)
  * is_insertable_into 
  Below are the steps to see information_schema schema using psql


```
# start docker container 
docker start postgres-rg

#
docker ps 

#
docker exec -it postgres-rg bash

# 
psql -U postgres

#
select * from information_schema.tables;

#
Q

# See
select * from information_schema.tables where table_schema = 'public';


```

* To see information about columns, we have to use the information_schema.columns view. Some of the important columns in this view are

  * table_catalog
  * table_schema
  * table_name
  * column_name
  * ordinal_position (i.e. position of the column relative to other columns)
  * column_default
  * is_nullable
  * data_type
  * character_maximum_length
  * numeric_precision
  * domain related columns (domain_catalog, domain_schema, domain_name) (business vs technical metadata)


```
#
SELECT * FROM information_schema.columns WHERE table_name = 'test';

#
SELECT column_name, data_type, character_maximum_length FROM information_schema.columns WHERE table_name = 'test'

```

* SQLParse is a library used to parse SQL. It can parse, split and format SQL statements as shown below

```
import sqlparse
raw = "select * from foo where foo.column1 = 'item1'"

statements = sqlparse.split(raw)  #["select * from foo where foo.column1 = 'item1'"]
query = statements[0] # select * from foo where foo.column1 = 'item1'
formated_query = sqlparse.format(query, reindent=True, keyword_case='upper') # SELECT *\nFROM foo\nWHERE foo.column1 = 'item1'
parsed = sqlparse.parse(formated_query)[0] # <Statement 'SELECT...' at 0x7EC103ED2DC0>

print(parsed.get_type()) # SELECT
print(parsed.tokens) # [<DML 'SELECT' at 0x7EC103EBA3E0>, <Whitespace ' ' at 0x7EC103EBA4A0>, <Wildcard '*' at 0x7EC103EBA500>, <Newline ' ' at 0x7EC103EBA560>, <Keyword 'FROM' at 0x7EC103EBA5C0>, <Whitespace ' ' at 0x7EC103EBA620>, <Identifier 'foo' at 0x7EC103ED2E40>, <Newline ' ' at 0x7EC103EBA6E0>, <Where 'WHERE ...' at 0x7EC103ED2CC0>]

```


### Doubts
1. What is the difference between database and catalog?
2. What are indexes and procedures?
3. What does table schema public and pg_catalog mean? If all tables in the current database have the table_schema as public, what about other table_schema?
4. How to get list of attributes and methods of an object? For example in the above code parsed variable is of type <class 'sqlparse.sql.Statement'>. How do we get all attributed and methods of this object?

### References
1. https://www.beekeeperstudio.io/blog/postgresql-information-schema
2. https://medium.com/@diehardankush/catalogue-schema-and-table-understanding-database-structures-ec54347f85c7
3. https://cloud.google.com/spanner/docs/information-schema-pg
4. https://stackoverflow.com/questions/2276644/list-all-tables-in-postgresql-information-schema
5. https://github.com/andialbrecht/sqlparse?tab=readme-ov-file


## Day 5
### Duration : 1.5 hours

### Learnings
* You don't need to install sqlite3 module. It is included in the standard library 

* Sqlite3 unlike other databases like mysql and postgres does not have access control, hence no username and password while connecting. Some other things which SQLite does not have for sake of simplicity are high concurrency, stored procedures and rich set of built in functions

* Transaction : series of logical operations performed to access and modify the contents of the database as per the user's request. It is a single unit of logic/work

* Commit : A COMMIT statement in SQL ends a transaction within a relational database management system (RDBMS) and makes all changes visible to other users. Thus it makes all data modifications since the start of the transaction a permanent part of the database, frees the transaction's resources. Note that a transaction is not committed automatically, if we want to commit data we need to call connection.commit 

* Every SQLite database contains a single "schema table" that stores the schema for that database. The schema for a database is a description of all of the other tables, indexes, triggers, and views that are contained within the database. Schema table is called sqlite_master or sqlite_schema and has the following fields
  * type (table, index, view)
  * name
  * tbl_name (name of a table or view that the object is associated with)
  * root_page
  * sql


```
import sqlite3

# Connect to database raggenie_test if it exists, else create a new one
conn = sqlite3.connect("raggenie_test.db")

# Open a database cursor to execute sql statments
cur = conn.cursor()

# Execute create statement against database to create movie table with 3 columns - title, year, score
cur.execute("CREATE TABLE movie(title, year, score)")

# Execute insert statement against table in databse
# The INSERT statement implicitly opens a transaction, which needs to be committed before changes are saved in the database
cur.execute("""
    INSERT INTO movie VALUES
        ('Monty Python and the Holy Grail', 1975, 8.2),
        ('And Now for Something Completely Different', 1971, 7.5)
""")

# Commit the transaction to the database
conn.commit()

# Execute select statement to fetch all data from movie table
res = cur.execute("select * from movie")

# Return all the resulting rows
print(res.fetchall()) # [('Monty Python and the Holy Grail', 1975, 8.2), ('And Now for Something Completely Different', 1971, 7.5)]

metadata = cur.execute("SELECT * from sqlite_master")
print(metadata.fetchall()) # [('table', 'movie', 'movie', 2, 'CREATE TABLE movie(title, year, score)')]

res = cur.execute("SELECT 1;")
print(res.fetchall()) # [(1,)]

conn.close()
```

* To get the results as list of dictionary instead of list of tuples, we should change the row_factory attribute as shown below

```

import sqlite3

# Connect to database raggenie_test if it exists, else create a new one
conn = sqlite3.connect("raggenie_test.db")

def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

conn.row_factory = dict_factory

# Open a database cursor to execute sql statments
cur = conn.cursor()

# Execute create statement against database to create movie table with 3 columns - title, year, score
cur.execute("CREATE TABLE movie(title, year, score)")

# Execute insert statement against table in databse
# The INSERT statement implicitly opens a transaction, which needs to be committed before changes are saved in the database
cur.execute("""
    INSERT INTO movie VALUES
        ('Monty Python and the Holy Grail', 1975, 8.2),
        ('And Now for Something Completely Different', 1971, 7.5)
""")

# Commit the transaction to the database
conn.commit()

# Execute select statement to fetch all data from movie table
res = cur.execute("select * from movie")

# Return one row rows
print(res.fetchone()) # {'title': 'Monty Python and the Holy Grail', 'year': 1975, 'score': 8.2}

metadata = cur.execute("SELECT * from sqlite_master")
print(metadata.fetchone()) # {'type': 'table', 'name': 'movie', 'tbl_name': 'movie', 'rootpage': 2, 'sql': 'CREATE TABLE movie(title, year, score)'}

name = cur.execute("SELECT name from sqlite_master") 
print(name.fetchall()) # [{'name': 'movie'}]

res = cur.execute("SELECT 1;")
print(res.fetchall()) # [{'1': 1}]


conn.close()

```


### Doubts
1. What do the following properties of transactions mean - atomicity, consistency, isolation, durability?
2. What exactly is the row_factory attribute?

### References
1. https://stackoverflow.com/questions/1807320/how-can-i-set-a-username-and-password-in-sqlite3
2. https://stackoverflow.com/questions/1279613/what-is-an-orm-how-does-it-work-and-how-should-i-use-one
3. https://docs.python.org/3/library/sqlite3.html#sqlite3-tutorial
4. https://learn.microsoft.com/en-us/sql/t-sql/language-elements/commit-transaction-transact-sql?view=sql-server-ver16
5. https://www.sqlite.org/schematab.html
6. https://stackoverflow.com/questions/3300464/how-can-i-get-dict-from-sqlite-query
