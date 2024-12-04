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
