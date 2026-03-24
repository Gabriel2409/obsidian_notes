#sd

The **Composite design pattern** is a structural pattern that lets you compose objects into **tree structures** to represent part-whole hierarchies. It allows clients to treat individual objects and compositions of objects uniformly.

```python
from abc import ABC, abstractmethod

# 1. Component Interface
class FileSystemComponent(ABC):
    """Declares the interface for objects in the composition."""
    
    @abstractmethod
    def display(self, indent=""):
        """Displays the component's name and structure."""
        pass
    
    # Optional: Methods for managing children (only implemented by Composite)
    def add(self, component):
        raise NotImplementedError("Cannot add components to a Leaf.")
    
    def remove(self, component):
        raise NotImplementedError("Cannot remove components from a Leaf.")

# 2. Leaf
class File(FileSystemComponent):
    """Represents the individual objects (files) in the composition."""
    
    def __init__(self, name, size):
        self.name = name
        self.size = size

    def display(self, indent=""):
        print(f"{indent}📄 File: {self.name} (Size: {self.size}KB)")

# 3. Composite
class Folder(FileSystemComponent):
    """Represents the groups (folders) that can hold other components."""
    
    def __init__(self, name):
        self.name = name
        self.children = []

    def add(self, component):
        self.children.append(component)

    def remove(self, component):
        self.children.remove(component)

    def display(self, indent=""):
        print(f"{indent}📁 Folder: {self.name}/")
        # The key Composite logic: it forwards the request to its children
        for child in self.children:
            child.display(indent + "  |--")

# --- Usage Example ---

# Create Files (Leaves)
file1 = File("readme.txt", 10)
file2 = File("index.html", 50)
file3 = File("profile.jpg", 300)

# Create Folders (Composites)
root_folder = Folder("C:")
src_folder = Folder("src")
docs_folder = Folder("docs")

# Build the tree structure
# C:
# |--docs/
# |  |--readme.txt
# |--src/
# |  |--index.html
# |  |--assets/
# |     |--profile.jpg

docs_folder.add(file1)

src_folder.add(file2)

assets_folder = Folder("assets")
assets_folder.add(file3)
src_folder.add(assets_folder)

root_folder.add(docs_folder)
root_folder.add(src_folder)


print("--- Displaying the Entire Root Folder (Composite) ---")
root_folder.display()

print("\n--- Displaying a Single File (Leaf) ---")
file3.display()
```