


## Day 1
* Started on 8-March-2025

* Class variable vs Instance variable : Class variables is defined outside all methods, shared across all instances of class. Instance variable is particular to a given instance and defined within a method, generally `__init__` 

* It is best practice to use a class name to change the value of a class variable. Because if we try to change the class variable’s value by using an object, a new instance variable is created for that particular object, which shadows the class variables.

```
### wrong approach, because same items_list will be used across all instances of order i.e. all orders
class Order:
  items_list = []
  prices = []
  staus = "open"

  def add_item(self, item, price):
    self.items_list.append(item)
    self.prices.append(price)

  def total_price(self):
    total = 0
    for price in self.prices:
      total = total + price

    return total


### right approach, declare items_list as instance variable not class variable as each order hadd unique items

class Order:
    def __init__(self):
        self.items_list = []
        self.prices = []
        self.status = "open"

    def add_item(self, item, price):
        self.items_list.append(item)
        self.prices.append(price)

    def total_price(self):
        return sum(self.prices)

    def set_status(self, status):
        self.status = status

```

* SOLID principle : Set of design principles in object-oriented software development. SOLID principles make arguments for how code should be split up, which parts should be internal or exposed, and how code should use other code

* Single Responsibility (S) (SRP): Class should have only 1 responsibilty 

```
### Before applying Single Responsibility principle
class Order:
    def __init__(self):
        self.items_list = []
        self.prices = []
        self.status = "open"

    def add_item(self, item, price):
        self.items_list.append(item)
        self.prices.append(price)

    def total_price(self):
        return sum(self.prices)

    def set_status(self, status):
        self.status = status

    def pay(self, payment_type, security_code):
      if payment_type == "debit":
        print("Processing debit payment")
        print(f"Verifying security code {security_code}")
        self.set_status("paid")

      elif payment_type == "credit":
        print("Processing credit payment")
        print(f"Verifying security code {security_code}")
        self.set_status("paid")

      else:
        raise Exception("Unknown payment type")


### After applying Single Responsibility principle
class Order:
    def __init__(self):
        self.items_list = []
        self.prices = []
        self.status = "open"

    def add_item(self, item, price):
        self.items_list.append(item)
        self.prices.append(price)

    def total_price(self):
        return sum(self.prices)

    def set_status(self, status):
        self.status = status


class PaymentProcessor:
  def pay(self, payment_type, security_code, order):
    if payment_type == "debit":
      print("Processing debit payment")
      print(f"Verifying security code {security_code}")
      order.set_status("paid")

    elif payment_type == "credit":
      print("Processing credit payment")
      print(f"Verifying security code {security_code}")
      order.set_status("paid")

    else:
      raise Exception("Unknown payment type")

order1 = Order()
order1.add_item("Shoes", 10)
processor = PaymentProcessor()
processor.pay("debit", 123, order1)

```

* Open/Closed principle (O) (OCP): Classes (or in general software entities) should be open for extension but closed for modification

* In the above case after applying SRP, in payment processor, when we want to add a new payment type, then we need to modify the PaymentProcessor class, which is against OCP, hence we need to create a more basic class, and then for each payment type, create a new class that inherits from that basic class

```
### After applying Open Closed principle to above code
class Order:
    def __init__(self):
        self.items_list = []
        self.prices = []
        self.status = "open"

    def add_item(self, item, price):
        self.items_list.append(item)
        self.prices.append(price)

    def total_price(self):
        return sum(self.prices)

    def set_status(self, status):
        self.status = status


class PaymentProcessor:
  def pay(self, security_code, order):
    raise NotImplementedError()

class DebitPaymentProcessor(PaymentProcessor):
  def pay(self, security_code, order):
    print("Processing debit payment")
    print(f"Verifying security code {security_code}")
    order.set_status("paid")

class CreditPaymentProcessor(PaymentProcessor):
  def pay(self, security_code, order):
    print("Processing credit payments")
    print(f"Verifying security code {security_code}")
    order.set_status("paid")

class PaypalPaymentProcessor(PaymentProcessor):
  def pay(self, security_code, order):
    print("Processing Paypal payments")
    print(f"Verifying security code {security_code}")
    order.set_status("paid")


order1 = Order()
order1.add_item("Shoes", 10)
processor = DebitPaymentProcessor()
processor.pay(123, order1)


```

* In the above code, the issue is Paypal works with email not with security code. To solve this we use Liskov Substitution principle (if something that will not be common amongst all children of a class then we remove it from the parent class and moving it instead to init method of the child class)

### Doubts
1. Why use OOPS? (classes act like tempoary databases during the lifetime of a program ie they help retain state of the variables/attributes)
2. Can we use SOLID principles in functional programming? In React?

### References
1. https://www.youtube.com/watch?v=w6Yul2XHI3s
2. https://www.youtube.com/watch?v=OuNOyFg942M
3. https://dev.to/patferraggi/do-the-solid-principles-apply-to-functional-programming-56lm
4. https://stackoverflow.blog/2021/11/01/why-solid-principles-are-still-the-foundation-for-modern-software-architecture/
5. https://refactoring.guru/design-patterns