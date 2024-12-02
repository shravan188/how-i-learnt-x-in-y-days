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
