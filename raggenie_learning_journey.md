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


## Day 6
### Duration : 1.5 hours


### Learnings


* An alternative approach to get dictionary output from sqlite query is to set row factory to dqlite3.Row as shown below

```
import sqlite3

# Connect to database raggenie_test if it exists, else create a new one
conn = sqlite3.connect(database="raggenie_test.db")

conn.row_factory = sqlite3.Row

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

# Result of setting row factory as sqlite3.Row is not directly dictionaries but objects like <sqlite3.Row object at 0x...>
# To get dictionary as output, we need to cast to type dict
print(dict(res.fetchone())) # {'title': 'Monty Python and the Holy Grail', 'year': 1975, 'score': 8.2}

# Get all the metadata from sqlite schema table ie sqlite_master
metadata = cur.execute("SELECT * from sqlite_master")
print(dict(metadata.fetchone())) # {'type': 'table', 'name': 'movie', 'tbl_name': 'movie', 'rootpage': 2, 'sql': 'CREATE TABLE movie(title, year, score)'}

# Get name of table
name = cur.execute("SELECT name from sqlite_master") 
print([dict(n) for n in name.fetchall()]) # [{'name': 'movie'}]

res = cur.execute("SELECT 1;")
print([dict(r) for r in res.fetchall()]) # [{'1': 1}]


conn.close()

```

* When we do a rollback, it goes to start of transaction, all series of changes will be lost

* We can do error handling in SQLite3 and get error codes as shown below (we try to create a table which already exists)
```
import sqlite3

conn = sqlite3.connect('raggenie_test.db')
cur = conn.cursor()
try:
  cur.execute("CREATE TABLE movie(title, year, score)")
except SQLite3.Error as er:
   # Python 3.11 onwards
  print(er.sqlite_errorcode) # 1
  print(er.sqlite_errorname) # SQLITE_ERROR (As table already exists)

```
Note that in versions prior to Python 3.11 you couldn't get error codes through Python's sqlite3 module

* Types of error supported by sqlite3 in general include:
  * sqlite3.Error (base error class)
  * sqlite3.DatabaseError
  * sqlite3.IntegrityError
  * sqlite3.ProgrammingError

* Using the sqlite3 command line program, we can execute SQL commands against a sqlite database as shown below
```
# Start the sqlite3 program
sqlite3

# Reopen persistent database raggeine_test.db
.open raggenie_test.db

# list all records in table movie
select * from movie;

# 
select * from sqlite_master;

# gives information about columns in the table
# It gives the following details about each column : name, data type, null or not, default value, primary key
PRAGMA table_info(movie);

# query information from the table-valued function corresonding to table_info pragma for the table movie
select * from pragma_table_info('movie');

# get all column names for the table movie
select name from pragma_table_info('movie');

# Exit sqlite3 in terminal
.quit

```
* The PRAGMA statement is an SQL extension (extension to SQL language) specific to SQLite and used to modify the operation of the SQLite library or to query the SQLite library for internal (non-table) data. The PRAGMA statement is issued using the same interface as other SQLite commands (e.g. SELECT, INSERT) but is different in the following important respects - the pragma command is specific to SQLite and is not compatible with any other SQL database engine.

* PRAGMAs that return results and that have no side-effects (eg table_info, index_info) can be accessed from ordinary SELECT statements as table-valued functions. To do so, the corresponding table-valued function has the same name as the PRAGMA with "pragma_" prefix.


* __repr__ is a special method used to define how an object should be represented in a string format

```
class Point:

    def __init__(self, x, y):

        self.x = x

        self.y = y

    def __repr__(self):

        return f"Point(x={self.x}, y={self.y})"

p1 = Point(3,4)
print(repr(p1)) # Point(x=3, y=4)
print(str(p1)) # Point(x=3, y=4)


```
* The standard practice is to define __repr__ in such a way that the return value of __repr__ should be an expression using which we can recreate the object (as shown in the above example). This might not always be practical though

* If __str__ is not defined, the using str() will call __repr__ (as shown above)

### Doubts
1. What is a table-valued function?
2. How does python differentiate normal functions, functions beginning with one underscore and dunder methods?
3. What if a Python class does not have __init__ constructor method?

### References
1. https://stackoverflow.com/a/58566730
2. https://stackoverflow.com/questions/25371636/how-to-get-sqlite-result-error-codes-in-python
3. https://github.com/python/cpython/pull/28076
4. https://askubuntu.com/questions/714305/how-to-exit-sqlite3-in-terminal
5. https://stackoverflow.com/questions/9057787/opening-database-file-from-within-sqlite-command-line-shell
6. https://www.sqlite.org/pragma.html
7. https://www.sqlite.org/pragma.html#pragma_table_info
8. https://stackoverflow.com/questions/947215/how-to-get-a-list-of-column-names-on-sqlite3-database
9. https://codedamn.com/news/python/what-is-repr-in-python
10. https://stackoverflow.com/questions/1984162/purpose-of-repr-method
11. https://stackoverflow.com/questions/6578487/init-as-a-constructor


## Day 7 and 8
### Duration: 1 + 1.5 

### Learnings

* Origin remote points to our fork of the main project. We alo setup a remote that points to the main github repo, generally called upstream (to keep our code in sync with what is happening in master repository)

```
### setting up a repo
### first fork the original repo
git clone https://github.com/personalname/raggenie.git
cd raggenie
# list all remote set
git remote -v
# 
git remote add upstream 

virtualenv venv 
venv\Scripts\activate 
poetry install 

```

* While trying to install libraries ran into following error - RuntimeError : uvloop does not support Windows at the moment

* Tried removing uvloop from pyproject.toml and running poetry install. Got the following error : pyproject.toml changed significantly since poetry.lock was last generated. Run `poetry lock [--no-update]` to fix the lock file. To solve this, deleted the lock file and then ran poetry install again. This time installation was successful with following warning : Warning: The current project could not be installed: No file/folder found for package raggenie
If you do not want to install the current project use --no-root.
If you want to use Poetry only for dependency management but not for packaging, you can disable package mode by setting package-mode = false in your pyproject.toml file.

* Although installation was successful, app was not running - could not figure out the reason. Hence decided to use the docker approach, as shown below

```
## cd to the folder containing docker-compose.yaml / compose.yaml 

#
docker compose build
#
# d flag : detached i.e. container will run in the background and will not block the terminal
docker compose up -d
# see all the images
docker images
# see active containers
docker ps

```

* The docker approach was successful, was able to setup frontend and backend successfully. 

## Doubts 
1. What does `super().__init__(__name__)` do?
2. Does fetchmany work in sqlite3?
3. Do select name from sqlite_master and select table_name from information_schema.tables give a similar output, and what are differences in any?
4. Do get a dictionary output from sqlite, we have to define a dict_factory function. Should this function be defined as a classmethod/staticmethod or a normal one?
5. What is fetch_feedback function doing?
6. What is an ordered dict?
7. What happens when you add code in `__init__.py` file?
8. Should we use name or tbl_name in sqlite_master ?

### References 
1. https://www.youtube.com/watch?v=OODDLyvePr8
2. https://docs.docker.com/compose/gettingstarted/



## Day 9
### Duration : 3 hours

### Learnings

* Started draft pr for creating sqlite plugin. Had to check output of each line to make sure it is consisted with the codes of the other databases
```
import psycopg2
# Connect to an existing database with name test as user postgres
conn = psycopg2.connect(dbname="test", user="postgres", password="password")

# Open a cursor to perform database operations
# Although output looks like list of lists, internally each "list" behaves like an ordered dict
cur = conn.cursor(cursor_factory=extras.DictCursor)
cur.execute("SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'")
table_names = cur.fetchall() # [['test']]
for table in table_names:
    print(table['table_name']) # 'test'
    print(table[0]) # 'test' 

```

* Tried running the code using docker approach (docker compose build -> docker compose up -d), frontend was running but backend was shutting down in a few seconds, because there was some bug in the code I had written. Removed sys.exit(1) in llm.py so that code does not exit on error, but still backend was shutting down after few seconds. Saw that llm.py called create_app function hence went to file app/main.py. Using the log files (docker container_name logs), found out that code is exiting somewhere after "initializing plugin providers". So changed logger.add to sys.stderr and then tracing the function call added a bunch of log statements in app/main.py, services/provider.py, repository/provider.py (this was where the error came up). Found out error was due to **UNIQUE constraint failed** in provider name. Went to `__init__.py` file in sqlite and changed the generic_name in Connection argument from 'Database name' to 'SQLite Database name'. This error occured because postgres already was using 'Database name', hence Sqlite could not use the same name because of the unique constraint 

```
2024-12-11 10:18:21 2024-12-11 04:48:21.387 | DEBUG    | commands.cli:cli:21 - Debug mode enabled
2024-12-11 10:18:21 2024-12-11 04:48:21.388 | INFO     | commands.cli:cli:24 - loading configurations
2024-12-11 10:18:21 2024-12-11 04:48:21.394 | INFO     | commands.llm:llm:21 - Intializing fastapi application server
2024-12-11 10:18:21 2024-12-11 04:48:21.394 | INFO     | app.main:create_app:36 - creating application
2024-12-11 10:18:21 2024-12-11 04:48:21.395 | INFO     | app.main:create_app:37 - creating container object
2024-12-11 10:18:21 2024-12-11 04:48:21.395 | INFO     | app.main:create_app:39 - loading necessary configurations
2024-12-11 10:18:21 2024-12-11 04:48:21.405 | INFO     | app.main:create_app:52 - Shravan 9.58 am test
2024-12-11 10:18:21 2024-12-11 04:48:21.405 | INFO     | app.main:create_app:54 - creating database tables
2024-12-11 10:18:21 2024-12-11 04:48:21.452 | INFO     | app.main:create_app:58 - initializing vector store
2024-12-11 10:18:21 2024-12-11 04:48:21.453 | INFO     | app.vectordb.loader:load_class:22 - vectordb provider: chroma
2024-12-11 10:18:21 2024-12-11 04:48:21.453 | INFO     | app.vectordb.chromadb:__init__:15 - initializing with configs
2024-12-11 10:18:22 2024-12-11 04:48:22.887 | INFO     | app.embeddings.loader:load_embclass:31 - embedding class: chroma_default
2024-12-11 10:18:22 2024-12-11 04:48:22.888 | INFO     | app.embeddings.chroma_default:__init__:7 - Initialising embedding providers
2024-12-11 10:18:23 2024-12-11 04:48:23.252 | INFO     | app.vectordb.chromadb:connect:48 - Connected to ChromaDB
2024-12-11 10:18:23 2024-12-11 04:48:23.253 | INFO     | app.main:create_app:62 - initializing plugin providers
2024-12-11 10:18:23 2024-12-11 04:48:23.335 | INFO     | app.repository.provider:insert_or_update_data:13 - Shravan 10.12am
2024-12-11 10:18:23 2024-12-11 04:48:23.382 | INFO     | app.repository.provider:insert_or_update_data:13 - Shravan 10.12am
2024-12-11 10:18:23 2024-12-11 04:48:23.403 | INFO     | app.repository.provider:insert_or_update_data:13 - Shravan 10.12am
2024-12-11 10:18:23 2024-12-11 04:48:23.414 | INFO     | app.repository.provider:insert_or_update_data:13 - Shravan 10.12am
2024-12-11 10:18:23 2024-12-11 04:48:23.528 | INFO     | app.repository.provider:insert_or_update_data:13 - Shravan 10.12am
2024-12-11 10:18:23 2024-12-11 04:48:23.720 | INFO     | app.repository.provider:insert_or_update_data:13 - Shravan 10.12am
2024-12-11 10:18:23 2024-12-11 04:48:23.733 | INFO     | app.repository.provider:insert_or_update_data:31 - (sqlite3.IntegrityError) UNIQUE constraint failed: providerconfig.name
2024-12-11 10:18:23 [SQL: INSERT INTO providerconfig (name, description, field, slug, value, enable, config_type, "order", required, provider_id, updated_at, deleted_at) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) RETURNING id, created_at]
2024-12-11 10:18:23 [parameters: ('Database name', 'Database name', 'database', 'database', 'null', 1, 1, 5, 1, 7, None, None)]
2024-12-11 10:18:23 (Background on this error at: https://sqlalche.me/e/20/gkpj)
2024-12-11 10:18:23 2024-12-11 04:48:23.734 | CRITICAL | commands.llm:llm:33 - Failed to start the LLM server: 'ConnectionArgument' object has no attribute 'name'
2024-12-11 10:18:23 2024-12-11 04:48:23.734 | CRITICAL | commands.llm:llm:33 - Failed to start the LLM server: 'ConnectionArgument' object has no attribute 'name'

```
* Once error was fixed, was able to see Sqlite plugin loading in the frontend. Push the code to github and created a draft pr using the git commands below

```
git checkout -b add-sqlite-plugin
git add app/plugins/sqlite
git commit -m "add sqlite plugin"
git push origin add-sqlite-plugin
# Go to main raggenie repo and create a draft pr
```

* A lot is hidden in inheritance
* Check case statement in app/service/connector.py (case 2 is for database)

### Doubts
1. How to see detailed error trackeback? Which is better logging or using debugger? How do we use a debugger within docker? 
2. What is sys.stderr?

### References
1. https://stackoverflow.com/questions/32416585/whats-the-difference-between-name-and-tbl-name-in-sqlite-master
2. https://www.vectorlogo.zone/logos/sqlite/
3. https://stackoverflow.com/questions/47829345/how-to-see-the-logs-of-a-docker-container
4. https://stackoverflow.com/questions/31420317/how-to-understand-sys-stdout-and-sys-stderr-in-python


## Day 10
### Duration : 2 hours

Consolidated previous day learnings and wrote it down. Did not work on any code today. Also had discussion on how to add sqlite plugin as unlike other databases, it is a file based database.


## Day 11
### Duration : 2 hours

### Learnings

* `__name__` is a variable that exists in every Python module, and is set to the name of the module

* `__main__` is the name of the environment where top-level code is run. “Top-level code” is the first user-specified Python module that starts running. It’s “top-level” because it imports all other modules that the program needs. Sometimes “top-level code” is called an entry point to the application 

* Due to previous point, we use the code `if __name__ == '__main__'` in a python file (say foo.py). This implies if we run foo directly from the terminal using `python foo.py` then the code within the if block is executed as when we run the file, `__name__` of the module foo is set to `__main__`. However if we import the foo module in another python file, then the code within the if block does not run (as `__name__` is set to foo)

* Prior to Python 3.3, to make a directory into a package `__init__.py` was required. Since Python 3.3, it is still required if we want to create a regular package. A regular pakage is generally implemented as a directory containing an `__init__.py` file. **When a regular package is imported, this `__init__.py` file is implicitly executed, and the objects it defines are bound to names in the package’s namespace**

* Order in which Python interpreter searches for a module when it is imported is as follows (entire list of directories interpreter searches to get the file can be obtained by printing sys.path)
  * Built in modules
  * Directory of input script/current directory
  * PYTHONPATH
  * site-packages directory

* Built-in function dir() is used to find out which names a module defines

* `__all__` affects the from <module> import * behavior only. Members that are not mentioned in `__all__` are still accessible from outside the module and can be imported with from <module> import <member>.

### Doubts

1. When is `__init__.py` required to import a file as a module, and when is it not required?
2. What is difference bw file, script and module in python?
3. What is the difference bw namespace package and regular package, and when to use namespace package?
4. What are compiled python files?
5. What is difference bw a module and a package?
6. How is the main.py file able to access the class although `__all__` in `__init__.py` only has the get_hello function?

### References
1. https://stackoverflow.com/questions/44834/what-does-all-mean-in-python
2. https://stackoverflow.com/questions/22942650/relative-import-from-init-py-file-throws-error
3. https://stackoverflow.com/questions/11536764/how-to-fix-attempted-relative-import-in-non-package-even-with-init-py
4. https://stackoverflow.com/questions/2996110/what-is-the-difference-between-a-module-and-a-script-in-python
5. https://stackoverflow.com/questions/448271/what-is-init-py-for
6. https://docs.python.org/3/tutorial/modules.html
