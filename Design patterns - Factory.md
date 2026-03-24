#sd

Creational pattern that provides a way to create objects without specifying the exact class of object that will be created.

For both examples below, we use

```python
from abc import ABC, abstractmethod

# The Product interface
class Vehicule(ABC):
    @abstractmethod
    def show_specs(self):
        pass

# Concrete Products
class Car(Vehicule):
    def show_specs(self):
        print("I'm a car with four wheels.")

class Moto(Vehicule):
    def show_specs(self):
        print("I'm a motorcycle with two wheels.")
```
## Simple factory

The **Simple Factory** (often referred to as a "class factory" or just "factory") is a simple implementation where a **single, dedicated class** (the factory) handles the creation of all related objects. This factory class has a single method that takes a parameter (like a string or an enum) to determine which specific object to create.

**Principle**: Centralizes object creation in one class.

**Use Case**: When you have a fixed set of object types to create and want to encapsulate the creation logic in a single place. For example, a coffee shop app might have a simple factory to create different types of coffee (e.g., latte, espresso) based on a user's selection.

```python
class VehicleFactory:

	@staticmethod
	def create_vehicule(type_vehicule):
		if type_vehicule == 'car':
			return Car()
		elif type_vehicule == 'moto':
			return Moto()
```

Important: the `create_vehicule` is not in the Vehicule class ! The whole point of the Factory Method is to **decouple the creation logic from the usage logic**
## Factory method

In the **Factory Method** pattern, the object creation is handled by a method within a **class hierarchy**. A superclass defines a method for creating an object (the "factory method"), but subclasses are responsible for implementing that method and deciding which concrete class to instantiate.

**Principle**: Delegates the instantiation of objects to subclasses.

**Use Case**: When a class can't anticipate the class of objects it must create. For example, a document editor might need to create different types of documents (e.g., PDF, Word), and a subclass for each document type would implement the factory method to create the appropriate object.

```python
from abc import ABC, abstractmethod


# The Creator interface (with the Factory Method)
class VehiculeFactory(ABC):
    @abstractmethod
    def create_vehicule(self):
        # This is the Factory Method. Subclasses must implement it.
        pass

# Concrete Creators
class CarFactory(VehiculeFactory):
    def create_vehicule(self):
        return Car()

class MotoFactory(VehiculeFactory):
    def create_vehicule(self):
        return Moto()
```

Note: While theoretically, you could pass arguments to a Factory Method, it would need to be able to handle all possible arguments for all possible products. This can lead to a long list of parameters, many of which might be `None` for certain subclasses. 
For this usecase, use the [[Design patterns - Builder]]. 

A **Factory** is like a vending machine. You put in a request (`'car'`), and you get a single, pre-configured product. It can take some basic choices (`'car'`, `blue`, `2024`), but it's not designed for highly customized orders.

A **Builder** is like a custom sandwich shop. You start with a base (`bread`), then add ingredients one by one (`cheese`, `lettuce`, `sauce`). You can arrange them in any order and leave out what you don't want, creating a highly customized final product.


