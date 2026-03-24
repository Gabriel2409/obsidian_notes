#sd

The Decorator design pattern is a structural pattern that allows you to add new behaviors or responsibilities to an object dynamically, without changing its code. It provides a flexible alternative to subclassing for extending functionality.

The pattern works by wrapping an object with a decorator that has the same interface as the original object.  This allows you to chain multiple decorators to an object, each adding a new layer of functionality.

## High level concept

```python
class BasicCoffee:
    def get_cost(self): 
	    return 5.0
    def get_description(self): 
	    return "Basic Coffee"

class MilkDecorator:
    def __init__(self, coffee):
        self.coffee = coffee
    def get_cost(self): 
	    return self.coffee.get_cost() + 1.5
    def get_description(self): 
	    return self.coffee.get_description() + ", Milk"

coffee = BasicCoffee()
milk_coffee = MilkDecorator(coffe) # and it behaves like a basic coffee because it shares the same interface
```

Note: Most examples use : 
- **Component**: The interface that defines the behavior of both the base object and its decorators.
- **Concrete Component**: The original object that you want to decorate.
- **Decorator**: An abstract class that has a reference to a Component object and implements the same Component interface.
- **Concrete Decorator**: The specific decorators that add new functionalities to the Component.

This makes the pattern more extensible
```python
from abc import ABC, abstractmethod

# 1. Component Interface
class Coffee(ABC):
    @abstractmethod
    def get_cost(self):
        pass

    @abstractmethod
    def get_description(self):
        pass

# 2. Concrete Component
class SimpleCoffee(Coffee):
    def get_cost(self):
        return 5.0
    
    def get_description(self):
        return "Simple Coffee"

# 3. Decorator (Base Decorator)
class CoffeeDecorator(Coffee):
    def __init__(self, decorated_coffee):
        self._decorated_coffee = decorated_coffee
    
    def get_cost(self):
        return self._decorated_coffee.get_cost()
    
    def get_description(self):
        return self._decorated_coffee.get_description()

# 4. Concrete Decorators
class MilkDecorator(CoffeeDecorator):
    def __init__(self, decorated_coffee):
        super().__init__(decorated_coffee)
        
    def get_cost(self):
        return super().get_cost() + 1.5
        
    def get_description(self):
        return super().get_description() + ", Milk"
```

## Python specific implementation

https://github.com/Gabriel2409/pytricks/blob/master/c9_01_02_decorator_metadata.py
https://github.com/Gabriel2409/pytricks/blob/master/c9_04_06_decorator_with_args.py
https://github.com/Gabriel2409/pytricks/blob/master/c9_09_class_decorators.py