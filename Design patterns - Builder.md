#sd

The **Builder** pattern is a creational pattern designed to separate the **construction of a complex object** from its representation. It allows you to create different variations of an object using the same construction process, especially when the object has many optional or required parameters.

**Principle**: Constructs a complex object step-by-step using a dedicated builder object.

**Use Case**: When a class has a large number of parameters, and the object's creation can be a multi-step process with varying configurations.

The idea is to return the instance of the builder at each step to chain the methods and finish with build, which will validate the config before returning the underlying object
```python
# The Product
class Vehicule:
    def __init__(self, engine, transmission, tires, spoiler=False):
        self.engine = engine
        self.transmission = transmission
        self.tires = tires
        self.spoiler = spoiler

    def show_specs(self):
        print(f"Vehicle Specs:")
        print(f"  - Engine: {self.engine}")
        print(f"  - Transmission: {self.transmission}")
        print(f"  - Tires: {self.tires}")
        print(f"  - Spoiler: {'Yes' if self.spoiler else 'No'}")

# The Builder
class VehiculeBuilder:
    def __init__(self):
        self.engine = None
        self.transmission = None
        self.tires = None
        self.spoiler = False

    def set_engine(self, engine_type):
        self.engine = engine_type
        return self

    def set_transmission(self, transmission_type):
        self.transmission = transmission_type
        return self

    def set_tires(self, tire_size):
        self.tires = tire_size
        return self

    def add_spoiler(self):
        self.spoiler = True
        return self

    def build(self):
        return Vehicule(self.engine, self.transmission, self.tires, self.spoiler)

```