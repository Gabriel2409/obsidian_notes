#sd 
TODO: redo this note. DI is not a design pattern. It uses patterns such as Strategy pattern or factory, or builder.

- provide dependencies instead of creating them internally
- reduces coupling, make tests easier (mocking)


```python

class Database:
	def connect(self):
		return "Connection"

class ServiceNoDi:
	def __init__(self):
		self.db = Database()
	def connect(self):
		return self.db.connect()

class ServiceWithDi:
	def __init__(self,db:Database):
		self.db = db
	def connect(self):
		return self.db.connect()
```

In the second case, as long as db correctly implements the Database interface, we can use any kind of db

