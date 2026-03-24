#sd 

The Observer Pattern is a behavioral design pattern that creates a one-to-many dependency between objects.

| Component | Role | Analogy |
|---|---|---|
| Subject (Publisher) | The object holding the state; it maintains a list of dependents. | A newspaper agency. |
| Observer (Subscriber) | The object that receives notifications when the Subject's state changes. | A subscriber who gets the daily paper. |
| Goal | Loose Coupling: The Subject doesn't need to know the concrete type of its Observers. |  |
 ## RxJS Observables in Angular
 
RxJS (Reactive Extensions for JavaScript) uses the Observer pattern as its foundation but extends it for reactive and asynchronous programming.

| RxJS Term | Pattern Role | Key Difference |
|---|---|---|
| Observable | Producer (Subject) | Lazy: Only executes when subscribed to. Emits multiple values over time (a stream). |
| Observer | Consumer | Defines three methods: next() (for data), error(), and complete(). |
| Subscription | The active link | Must be explicitly unsubscribed from to prevent memory leaks. |
| Subject | A special Observable that is both a Producer and a Consumer. | Used for multicasting (broadcasting a value to many subscribers at once). |
High-Level Principle: Observables are used everywhere in Angular (HTTP, forms, events) because they allow you to treat all data streams—synchronous or asynchronous—in a unified, declarative way using powerful operators (like map, filter).

## Observer example

```python
# The Observer Interface
class Observer:
    """Defines the update interface for objects that should be notified of changes in a subject."""
    def update(self, subject):
        raise NotImplementedError

# The Subject (Publisher)
class NewsAgency:
    """Maintains a list of observers and notifies them of state changes."""
    def __init__(self, news="No news yet"):
        self._observers = []
        self._news = news

    def attach(self, observer: Observer):
        """Attaches an observer to the subject."""
        print(f"Attached: {type(observer).__name__}")
        self._observers.append(observer)

    def detach(self, observer: Observer):
        """Detaches an observer from the subject."""
        self._observers.remove(observer)
        print(f"Detached: {type(observer).__name__}")

    def notify(self):
        """Notifies all observers about an event."""
        print("\n--- NOTIFYING OBSERVERS ---")
        for observer in self._observers:
            observer.update(self)

    @property
    def news(self):
        """State getter for the news."""
        return self._news

    @news.setter
    def news(self, new_news):
        """State setter which triggers notification."""
        self._news = new_news
        self.notify()

# Concrete Observers (Subscribers)
class Newspaper(Observer):
    def update(self, subject: NewsAgency):
        print(f"Newspaper received: '{subject.news}' (Preparing to Print)")

class TVChannel(Observer):
    def update(self, subject: NewsAgency):
        print(f"TVChannel received: '{subject.news}' (Broadcasting Live)")

# Client Code
agency = NewsAgency()

# Create observers
paper = Newspaper()
channel = TVChannel()

# Observers subscribe to the agency
agency.attach(paper)
agency.attach(channel)

# The state changes and all attached observers are notified automatically
agency.news = "Major breakthrough in clean energy technology!"

# An observer decides to unsubscribe
agency.detach(paper)

# Another state change, only the remaining observer is notified
agency.news = "Local sports team wins the championship."
```