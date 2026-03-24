#sd

The **Abstract Factory** pattern provides an interface for creating families of related objects without specifying their concrete classes.

**Principle**: A factory of factories. It centralizes the creation of multiple related products.
    
**Use Case**: When your system needs to be independent of how its products are created, composed, and represented. 


Diff with Factory method:
- Factory Method: Delegation of single-object creation to subclasses, useful for postponing the decision of which concrete class to instantiate.
- Abstract Factory: Grouping multiple create methods into a single factory implementation to ensure that all created objects belong to a compatible family (consistency).


```python
from abc import ABC, abstractmethod

# Abstract Products
class Car(ABC):
    @abstractmethod
    def drive(self):
        pass

class Motorcycle(ABC):
    @abstractmethod
    def ride(self):
        pass

# Concrete Products - Luxury Family
class LuxuryCar(Car):
    def drive(self):
        print("Driving a luxurious car.")

class LuxuryMotorcycle(Motorcycle):
    def ride(self):
        print("Riding a powerful, luxury motorcycle.")

# Concrete Products - Economy Family
class EconomyCar(Car):
    def drive(self):
        print("Driving an economical car.")

class EconomyMotorcycle(Motorcycle):
    def ride(self):
        print("Riding a fuel-efficient motorcycle.")

# Abstract Factory
class VehicleFactory(ABC):
    @abstractmethod
    def create_car(self):
        pass

    @abstractmethod
    def create_motorcycle(self):
        pass

# Concrete Factories
class LuxuryVehicleFactory(VehicleFactory):
    def create_car(self):
        return LuxuryCar()

    def create_motorcycle(self):
        return LuxuryMotorcycle()

class EconomyVehicleFactory(VehicleFactory):
    def create_car(self):
        return EconomyCar()

    def create_motorcycle(self):
        return EconomyMotorcycle()
```

## Difference with factory method

The difference is in **scope and intent**: **Factory Method** uses a class hierarchy to delegate the creation of a **single** product type to subclasses, acting like a virtual constructor. **Abstract Factory** provides an interface to create **families of related products** (multiple types) that share a common theme, ensuring consistency across the whole set.