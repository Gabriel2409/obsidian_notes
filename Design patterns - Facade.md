#sd 

The Facade design pattern is a structural pattern that provides a simplified, unified interface to a complex system of classes, a library, or a framework. It wraps a set of complex subsystems with a single, easy-to-use class, hiding the underlying complexity from the client.

It is a single object that provides a simple entry point to a complex subsystem.

- Subsystems: These are the multiple classes that perform the complex work. The client would normally have to interact with all of them to get a task done.
- Facade: This is the single, simplified class. It knows which subsystem classes are needed to fulfill a request and delegates the work to them.

```python
# The Subsystems (Complex components)
class Amplifier:
    def on(self):
        print("Amplifier is on.")
    def set_volume(self, volume):
        print(f"Volume set to {volume}.")
    def off(self):
        print("Amplifier is off.")

class Projector:
    def on(self):
        print("Projector is on.")
    def off(self):
        print("Projector is off.")

class DvdPlayer:
    def on(self):
        print("DVD Player is on.")
    def play(self, movie):
        print(f"Playing '{movie}'.")
    def off(self):
        print("DVD Player is off.")

# The Facade: A simple, unified interface
class HomeTheaterFacade:
    def __init__(self):
        self.amp = Amplifier()
        self.proj = Projector()
        self.dvd = DvdPlayer()

    def watch_movie(self, movie):
        print("Get ready to watch a movie...")
        self.amp.on()
        self.amp.set_volume(10)
        self.proj.on()
        self.dvd.on()
        self.dvd.play(movie)

    def end_movie(self):
        print("Shutting down the theater...")
        self.dvd.off()
        self.proj.off()
        self.amp.off()

# The Client: Interacts only with the facade
```