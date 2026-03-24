#sd

The Adapter design pattern is a structural pattern that allows objects with incompatible interfaces to collaborate. It works like a physical adapter (e.g., a power plug adapter) by translating the interface of one object into an interface the client expects.


The adapter pattern involves:

- A Client who wants to use a service.
- A Target Interface that the client understands.
- An Adaptee that provides the service but has an incompatible interface.
- The Adapter itself, which implements the target interface and holds an instance of the adaptee, translating calls from the client to the adaptee's methods.

The adapter acts as a middleman, converting the requests. This allows you to reuse existing code without modifying it

```python
# The Adaptee: The new coffee maker with an incompatible interface
class NewCoffeeMaker:
	def brew(self):
		return "Brewing a new style coffee."

# The Target: The interface the client expects
class CoffeeMachine:
    def make_coffee(self):
        raise NotImplementedError

# The Adapter: It implements the target interface and wraps the adaptee
class CoffeeMakerAdapter(CoffeeMachine):
    def __init__(self, new_maker):
        self.new_maker = new_maker

    def make_coffee(self):
        # The adapter translates the method call
        return self.new_maker.brew()

# The Client: It only knows how to use the old interface
class Customer:
    def order_coffee(self, machine):
        return machine.make_coffee()

# Usage
new_maker = NewCoffeeMaker()
adapter = CoffeeMakerAdapter(new_maker)

customer = Customer()

# The customer seamlessly uses the new machine via the adapter
print(customer.order_coffee(adapter))
# Output: Brewing a new style coffee.
```