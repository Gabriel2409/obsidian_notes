#sd 

**Use Case:** Ensure only one instance exists globally (database connections, configuration settings, logging)

**Private constructor:** Prevents direct instantiation. Without this, clients could create multiple instances, defeating the pattern's purpose.

**Static access method:** Provides controlled access to the single instance. This is where the "only one instance" logic lives.

**Note for multithreading**: [[Double check locking]]

**Note on python impl**: In python, we have the same logic. By defining `__new__` and not `__init__`, we can control what happens when we call `Singleton()`. Indeed `__new__` is called before `__init__` and, because we set the instance at the class level and check for its existence, we never instanciate a new one. Calling the class also serves as the static access method
```python
class Singleton:
	_instance = None

	def __new__(cls):
		if cls._instance is None:
			cls._instance = super().__new__(cls)
		return cls._instance 
		
s1 = Singleton()
s2 = Singleton()
print(s1 is s2) # True
```

```c#
// sealed: prevents inheritance
public sealed class Singleton
{
	// private static: belongs to class, not instance
	// readonly: can only be set once at declaration
	// Lazy<T> : delays object creation until first access
    private static readonly Lazy<Singleton> _instance = 
	    // lambda method explaining how to create the instance
        new Lazy<Singleton>(() => new Singleton());
    
    // private constructor: prevents direct instanciation. 
    // We can't use new Singleton()
    private Singleton() { }
    
    // public getter returning the singleton (Value is a property of Lazy<T>)
    public static Singleton Instance => _instance.Value;
}

// Usage
var s1 = Singleton.Instance;
var s2 = Singleton.Instance;
Console.WriteLine(ReferenceEquals(s1, s2)); // True
```
