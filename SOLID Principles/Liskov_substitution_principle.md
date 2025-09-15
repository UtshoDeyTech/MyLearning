# Liskov Substitution Principle (LSP)

## Table of Contents
1. [Introduction](#introduction)
2. [Definition and Theory](#definition-and-theory)
3. [Why LSP Matters](#why-lsp-matters)
4. [When to Apply LSP](#when-to-apply-lsp)
5. [Consequences of Violating LSP](#consequences-of-violating-lsp)
6. [Python Examples](#python-examples)
7. [Behavioral Contracts](#behavioral-contracts)
8. [Benefits of Following LSP](#benefits-of-following-lsp)
9. [Common Pitfalls](#common-pitfalls)
10. [Best Practices](#best-practices)
11. [Conclusion](#conclusion)

## Introduction

The Liskov Substitution Principle (LSP) is the third principle in the SOLID principles of object-oriented design, formulated by Barbara Liskov in 1987. It's a fundamental principle that ensures proper inheritance relationships and polymorphism work correctly in object-oriented systems.

LSP is often considered one of the most subtle and important principles in object-oriented design, as it defines what it means for inheritance to be done correctly.

## Definition and Theory

### Formal Definition
**"Objects of a superclass should be replaceable with objects of a subclass without altering the correctness of the program."**

### Barbara Liskov's Original Formulation (1987)
**"If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2, then S is a subtype of T."**

### Simplified Definition
**"Derived classes must be substitutable for their base classes."**

### Core Concepts

#### Behavioral Subtyping
LSP is fundamentally about behavioral subtyping, not just structural inheritance. A subclass should not just inherit the interface of its parent class, but also adhere to the behavioral contract.

#### Contract Compliance
Subclasses must honor the contracts established by their parent classes:
- **Preconditions** cannot be strengthened in subtypes
- **Postconditions** cannot be weakened in subtypes
- **Invariants** must be preserved in subtypes

#### IS-A vs BEHAVES-LIKE-A
LSP distinguishes between the "IS-A" relationship (structural inheritance) and the "BEHAVES-LIKE-A" relationship (behavioral inheritance). True substitutability requires both.

## Why LSP Matters

### 1. **Polymorphism Reliability**
Ensures that polymorphism works correctly and predictably.

### 2. **Code Reusability**
Allows existing code to work with new subclasses without modification.

### 3. **System Robustness**
Prevents unexpected behavior when substituting objects.

### 4. **Design Integrity**
Maintains the logical consistency of inheritance hierarchies.

### 5. **Client Code Stability**
Client code doesn't need to know about specific subclasses.

### 6. **Testing Simplification**
Tests written for base classes should work for all subclasses.

## When to Apply LSP

### You Should Apply LSP When:

1. **Designing inheritance hierarchies**
2. **Creating polymorphic interfaces**
3. **Building frameworks or libraries**
4. **Implementing design patterns that rely on substitutability**
5. **Refactoring existing class hierarchies**
6. **Creating abstract base classes**

### Warning Signs That LSP is Violated:

- Subclasses throw exceptions that the parent class doesn't throw
- Subclasses have empty or no-op implementations of parent methods
- Client code checks the type of objects before using them
- Subclasses strengthen preconditions or weaken postconditions
- Subclasses change the expected behavior in unexpected ways
- Comments like "this method doesn't apply to this subclass"
- Runtime type checking to handle different subclass behaviors

## Consequences of Violating LSP

### 1. **Broken Polymorphism**
Polymorphic code cannot reliably work with all subclasses.

### 2. **Fragile Client Code**
Client code becomes dependent on specific subclass implementations.

### 3. **Unexpected Runtime Errors**
Substituting objects can cause crashes or incorrect behavior.

### 4. **Increased Testing Complexity**
Each subclass requires specific handling and testing.

### 5. **Reduced Code Reusability**
Generic algorithms cannot work with all subclasses.

### 6. **Design Confusion**
Inheritance hierarchies become confusing and counterintuitive.

## Python Examples

### Example 1: The Classic Rectangle/Square Problem

#### Violating LSP

```python
# BAD: Violates LSP
class Rectangle:
    def __init__(self, width, height):
        self._width = width
        self._height = height
    
    @property
    def width(self):
        return self._width
    
    @width.setter
    def width(self, value):
        self._width = value
    
    @property
    def height(self):
        return self._height
    
    @height.setter
    def height(self, value):
        self._height = value
    
    def area(self):
        return self._width * self._height
    
    def __str__(self):
        return f"Rectangle(width={self.width}, height={self.height})"

class Square(Rectangle):
    """Violates LSP: Changes the behavior of width/height setters"""
    def __init__(self, side):
        super().__init__(side, side)
    
    @Rectangle.width.setter
    def width(self, value):
        self._width = value
        self._height = value  # Problem: Changes height when setting width
    
    @Rectangle.height.setter
    def height(self, value):
        self._width = value   # Problem: Changes width when setting height
        self._height = value
    
    def __str__(self):
        return f"Square(side={self.width})"

# Client code that works with Rectangle but breaks with Square
def resize_rectangle(rectangle: Rectangle, new_width: int, new_height: int):
    """This function expects Rectangle behavior but breaks with Square"""
    print(f"Original: {rectangle}")
    
    rectangle.width = new_width
    print(f"After setting width to {new_width}: {rectangle}")
    
    rectangle.height = new_height
    print(f"After setting height to {new_height}: {rectangle}")
    
    expected_area = new_width * new_height
    actual_area = rectangle.area()
    
    print(f"Expected area: {expected_area}")
    print(f"Actual area: {actual_area}")
    print(f"Areas match: {expected_area == actual_area}")
    print("-" * 50)

# Usage - demonstrates LSP violation
rectangle = Rectangle(3, 4)
square = Square(5)

print("Testing with Rectangle:")
resize_rectangle(rectangle, 6, 8)

print("Testing with Square (LSP violation):")
resize_rectangle(square, 6, 8)  # This breaks our expectations!
```

#### Following LSP

```python
# GOOD: Following LSP
from abc import ABC, abstractmethod

class Shape(ABC):
    """Abstract base class for all shapes"""
    
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height
    
    @property
    def width(self) -> float:
        return self._width
    
    @width.setter
    def width(self, value: float):
        if value <= 0:
            raise ValueError("Width must be positive")
        self._width = value
    
    @property
    def height(self) -> float:
        return self._height
    
    @height.setter
    def height(self, value: float):
        if value <= 0:
            raise ValueError("Height must be positive")
        self._height = value
    
    def area(self) -> float:
        return self._width * self._height
    
    def perimeter(self) -> float:
        return 2 * (self._width + self._height)
    
    def resize(self, width: float, height: float):
        """Method to resize the rectangle"""
        self.width = width
        self.height = height
    
    def __str__(self):
        return f"Rectangle(width={self.width}, height={self.height})"

class Square(Shape):
    """Square as a separate class, not inheriting from Rectangle"""
    def __init__(self, side: float):
        self._side = side
    
    @property
    def side(self) -> float:
        return self._side
    
    @side.setter
    def side(self, value: float):
        if value <= 0:
            raise ValueError("Side must be positive")
        self._side = value
    
    def area(self) -> float:
        return self._side * self._side
    
    def perimeter(self) -> float:
        return 4 * self._side
    
    def resize(self, side: float):
        """Method to resize the square"""
        self.side = side
    
    def __str__(self):
        return f"Square(side={self.side})"

# Alternative approach: If you must have Square inherit from Rectangle
class MutableRectangle(Shape):
    """Rectangle that can be freely resized"""
    def __init__(self, width: float, height: float):
        self._width = width
        self._height = height
    
    @property
    def width(self) -> float:
        return self._width
    
    @property
    def height(self) -> float:
        return self._height
    
    def set_dimensions(self, width: float, height: float):
        """Safe way to change dimensions"""
        self._width = width
        self._height = height
    
    def area(self) -> float:
        return self._width * self._height
    
    def perimeter(self) -> float:
        return 2 * (self._width + self._height)
    
    def __str__(self):
        return f"MutableRectangle(width={self.width}, height={self.height})"

class ImmutableSquare(Shape):
    """Square that maintains its constraint while being substitutable"""
    def __init__(self, side: float):
        self._side = side
    
    @property
    def width(self) -> float:
        return self._side
    
    @property
    def height(self) -> float:
        return self._side
    
    @property
    def side(self) -> float:
        return self._side
    
    def area(self) -> float:
        return self._side * self._side
    
    def perimeter(self) -> float:
        return 4 * self._side
    
    def __str__(self):
        return f"ImmutableSquare(side={self.side})"

# Client code that works properly with LSP
def calculate_shape_properties(shapes: list[Shape]):
    """Works with any Shape implementation"""
    for shape in shapes:
        print(f"{shape}")
        print(f"  Area: {shape.area():.2f}")
        print(f"  Perimeter: {shape.perimeter():.2f}")
        print()

def resize_rectangle_safely(rectangle: Rectangle, width: float, height: float):
    """Works specifically with Rectangle objects"""
    print(f"Resizing {rectangle}")
    rectangle.resize(width, height)
    print(f"New dimensions: {rectangle}")
    print(f"Area: {rectangle.area()}")

# Usage
shapes = [
    Rectangle(3, 4),
    Square(5),
    ImmutableSquare(6)
]

print("Calculating properties for all shapes:")
calculate_shape_properties(shapes)

print("Resizing rectangles:")
rect = Rectangle(3, 4)
resize_rectangle_safely(rect, 6, 8)
```

### Example 2: Bird Flying Problem

#### Violating LSP

```python
# BAD: Violates LSP
class Bird:
    def __init__(self, name):
        self.name = name
    
    def fly(self):
        return f"{self.name} is flying"
    
    def make_sound(self):
        return f"{self.name} makes a sound"

class Sparrow(Bird):
    def fly(self):
        return f"{self.name} flies quickly with rapid wing beats"
    
    def make_sound(self):
        return f"{self.name} chirps melodiously"

class Eagle(Bird):
    def fly(self):
        return f"{self.name} soars majestically through the sky"
    
    def make_sound(self):
        return f"{self.name} screeches powerfully"

class Penguin(Bird):
    """Violates LSP: Cannot fly but inherits fly method"""
    def fly(self):
        # Problem: Penguins can't fly, but they inherit the fly method
        raise NotImplementedError("Penguins cannot fly")
        # Alternative bad approaches:
        # return ""  # Returns empty string
        # return "Penguin cannot fly"  # Changes expected behavior
        # pass  # Returns None instead of string
    
    def make_sound(self):
        return f"{self.name} trumpets loudly"
    
    def swim(self):
        return f"{self.name} swims gracefully underwater"

class Ostrich(Bird):
    """Also violates LSP: Cannot fly"""
    def fly(self):
        raise NotImplementedError("Ostriches cannot fly")
    
    def make_sound(self):
        return f"{self.name} makes deep booming sounds"
    
    def run(self):
        return f"{self.name} runs at high speed"

# Client code that expects all birds to fly
def make_birds_fly(birds: list[Bird]):
    """This function breaks when used with non-flying birds"""
    for bird in birds:
        try:
            print(bird.fly())
        except NotImplementedError as e:
            print(f"Error with {bird.name}: {e}")

def bird_airshow(birds: list[Bird]):
    """Airshow function assumes all birds can fly"""
    print("Welcome to the Bird Airshow!")
    for bird in birds:
        print(f"Next up: {bird.fly()}")

# Usage - demonstrates LSP violation
birds = [
    Sparrow("Chirpy"),
    Eagle("Majesty"),
    Penguin("Waddles"),  # This will cause problems
    Ostrich("Runner")    # This will also cause problems
]

print("Trying to make all birds fly:")
make_birds_fly(birds)
print()

print("Airshow (will crash):")
try:
    bird_airshow(birds)
except NotImplementedError as e:
    print(f"Airshow crashed: {e}")
```

#### Following LSP

```python
# GOOD: Following LSP
from abc import ABC, abstractmethod

class Bird(ABC):
    """Abstract base class for all birds"""
    def __init__(self, name: str):
        self.name = name
    
    @abstractmethod
    def make_sound(self) -> str:
        pass
    
    @abstractmethod
    def move(self) -> str:
        pass

class FlyingBird(Bird):
    """Abstract class for birds that can fly"""
    
    @abstractmethod
    def fly(self) -> str:
        pass
    
    def move(self) -> str:
        return self.fly()

class SwimmingBird(Bird):
    """Abstract class for birds that can swim"""
    
    @abstractmethod
    def swim(self) -> str:
        pass

class RunningBird(Bird):
    """Abstract class for birds that primarily run"""
    
    @abstractmethod
    def run(self) -> str:
        pass
    
    def move(self) -> str:
        return self.run()

# Flying birds
class Sparrow(FlyingBird):
    def fly(self) -> str:
        return f"{self.name} flies quickly with rapid wing beats"
    
    def make_sound(self) -> str:
        return f"{self.name} chirps melodiously"

class Eagle(FlyingBird):
    def fly(self) -> str:
        return f"{self.name} soars majestically through the sky"
    
    def make_sound(self) -> str:
        return f"{self.name} screeches powerfully"

# Swimming birds (that can also move on land)
class Penguin(SwimmingBird):
    def swim(self) -> str:
        return f"{self.name} swims gracefully underwater"
    
    def make_sound(self) -> str:
        return f"{self.name} trumpets loudly"
    
    def move(self) -> str:
        return self.swim()
    
    def waddle(self) -> str:
        return f"{self.name} waddles on land"

# Running birds
class Ostrich(RunningBird):
    def run(self) -> str:
        return f"{self.name} runs at high speed up to 70 km/h"
    
    def make_sound(self) -> str:
        return f"{self.name} makes deep booming sounds"

# Multi-capability birds
class Duck(FlyingBird, SwimmingBird):
    def fly(self) -> str:
        return f"{self.name} flies with steady wing beats"
    
    def swim(self) -> str:
        return f"{self.name} paddles smoothly on water"
    
    def make_sound(self) -> str:
        return f"{self.name} quacks loudly"
    
    def move(self) -> str:
        return f"{self.name} can fly or swim"

# Client code that works properly with LSP
def bird_sounds_concert(birds: list[Bird]):
    """Works with any bird - they all can make sounds"""
    print("Bird Sounds Concert:")
    for bird in birds:
        print(f"  {bird.make_sound()}")

def bird_movement_demo(birds: list[Bird]):
    """Works with any bird - they all can move"""
    print("\nBird Movement Demo:")
    for bird in birds:
        print(f"  {bird.move()}")

def flying_competition(flying_birds: list[FlyingBird]):
    """Works only with flying birds"""
    print("\nFlying Competition:")
    for bird in flying_birds:
        print(f"  {bird.fly()}")

def swimming_race(swimming_birds: list[SwimmingBird]):
    """Works only with swimming birds"""
    print("\nSwimming Race:")
    for bird in swimming_birds:
        print(f"  {bird.swim()}")

def running_race(running_birds: list[RunningBird]):
    """Works only with running birds"""
    print("\nRunning Race:")
    for bird in running_birds:
        print(f"  {bird.run()}")

# Usage
all_birds = [
    Sparrow("Chirpy"),
    Eagle("Majesty"),
    Penguin("Waddles"),
    Ostrich("Runner"),
    Duck("Quackers")
]

flying_birds = [bird for bird in all_birds if isinstance(bird, FlyingBird)]
swimming_birds = [bird for bird in all_birds if isinstance(bird, SwimmingBird)]
running_birds = [bird for bird in all_birds if isinstance(bird, RunningBird)]

# All these work without violations
bird_sounds_concert(all_birds)
bird_movement_demo(all_birds)
flying_competition(flying_birds)
swimming_race(swimming_birds)
running_race(running_birds)
```

### Example 3: Database Storage System

#### Violating LSP

```python
# BAD: Violates LSP
class DatabaseStorage:
    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connected = False
    
    def connect(self):
        print(f"Connecting to database: {self.connection_string}")
        self.connected = True
        return True
    
    def disconnect(self):
        print("Disconnecting from database")
        self.connected = False
    
    def save(self, data):
        if not self.connected:
            raise RuntimeError("Must connect before saving")
        print(f"Saving to database: {data}")
        return True
    
    def load(self, key):
        if not self.connected:
            raise RuntimeError("Must connect before loading")
        print(f"Loading from database: {key}")
        return f"data_for_{key}"
    
    def begin_transaction(self):
        if not self.connected:
            raise RuntimeError("Must connect before starting transaction")
        print("Beginning database transaction")
    
    def commit_transaction(self):
        print("Committing database transaction")
    
    def rollback_transaction(self):
        print("Rolling back database transaction")

class FileStorage(DatabaseStorage):
    """Violates LSP: File storage doesn't support transactions"""
    def __init__(self, file_path):
        super().__init__(file_path)  # Misuses connection_string for file_path
        self.file_path = file_path
    
    def connect(self):
        print(f"Opening file: {self.file_path}")
        self.connected = True
        return True
    
    def disconnect(self):
        print("Closing file")
        self.connected = False
    
    def save(self, data):
        if not self.connected:
            raise RuntimeError("Must open file before saving")
        print(f"Writing to file: {data}")
        return True
    
    def load(self, key):
        if not self.connected:
            raise RuntimeError("Must open file before loading")
        print(f"Reading from file: {key}")
        return f"file_data_for_{key}"
    
    def begin_transaction(self):
        # Problem: File storage doesn't support transactions
        raise NotImplementedError("File storage doesn't support transactions")
    
    def commit_transaction(self):
        raise NotImplementedError("File storage doesn't support transactions")
    
    def rollback_transaction(self):
        raise NotImplementedError("File storage doesn't support transactions")

class MemoryStorage(DatabaseStorage):
    """Violates LSP: Memory storage doesn't need connection"""
    def __init__(self):
        super().__init__("memory")
        self.data = {}
        self.connected = True  # Always "connected"
    
    def connect(self):
        # Problem: Memory storage doesn't need connection, but we inherit it
        print("Memory storage is always connected")
        return True
    
    def disconnect(self):
        # Problem: Can't really disconnect from memory
        print("Cannot disconnect from memory storage")
    
    def save(self, data):
        # Ignores connected state
        key = f"key_{len(self.data)}"
        self.data[key] = data
        print(f"Stored in memory: {key} -> {data}")
        return True
    
    def load(self, key):
        # Ignores connected state
        return self.data.get(key, f"default_data_for_{key}")
    
    def begin_transaction(self):
        # Problem: Memory storage doesn't need transactions
        print("Memory storage doesn't need explicit transactions")
    
    def commit_transaction(self):
        print("Memory storage auto-commits")
    
    def rollback_transaction(self):
        raise NotImplementedError("Memory storage cannot rollback")

# Client code that expects database behavior
def perform_database_operations(storage: DatabaseStorage):
    """This function expects full database behavior"""
    try:
        storage.connect()
        
        # Try to use transaction
        storage.begin_transaction()
        
        storage.save("important_data")
        storage.save("more_data")
        
        # Simulate some condition
        if True:  # Some business logic
            storage.commit_transaction()
        else:
            storage.rollback_transaction()
        
        result = storage.load("some_key")
        print(f"Loaded: {result}")
        
        storage.disconnect()
        
    except NotImplementedError as e:
        print(f"Operation failed: {e}")

# Usage - demonstrates LSP violation
storages = [
    DatabaseStorage("postgresql://localhost/mydb"),
    FileStorage("/tmp/data.txt"),  # Will fail on transactions
    MemoryStorage()  # Will fail on rollback
]

for storage in storages:
    print(f"\nTesting {storage.__class__.__name__}:")
    perform_database_operations(storage)
```

#### Following LSP

```python
# GOOD: Following LSP
from abc import ABC, abstractmethod
from typing import Any, Optional

class Storage(ABC):
    """Abstract base class for all storage systems"""
    
    @abstractmethod
    def save(self, key: str, data: Any) -> bool:
        pass
    
    @abstractmethod
    def load(self, key: str) -> Optional[Any]:
        pass
    
    @abstractmethod
    def delete(self, key: str) -> bool:
        pass
    
    @abstractmethod
    def exists(self, key: str) -> bool:
        pass

class ConnectionMixin:
    """Mixin for storage systems that require connection management"""
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
        self.connected = False
    
    def connect(self) -> bool:
        if self.connected:
            return True
        print(f"Connecting to: {self.connection_string}")
        self.connected = True
        return True
    
    def disconnect(self) -> None:
        if self.connected:
            print("Disconnecting...")
            self.connected = False
    
    def _ensure_connected(self) -> None:
        if not self.connected:
            raise RuntimeError("Must connect before performing operations")

class TransactionalMixin:
    """Mixin for storage systems that support transactions"""
    def __init__(self):
        self.in_transaction = False
    
    def begin_transaction(self) -> None:
        if self.in_transaction:
            raise RuntimeError("Transaction already in progress")
        print("Beginning transaction")
        self.in_transaction = True
    
    def commit_transaction(self) -> None:
        if not self.in_transaction:
            raise RuntimeError("No transaction in progress")
        print("Committing transaction")
        self.in_transaction = False
    
    def rollback_transaction(self) -> None:
        if not self.in_transaction:
            raise RuntimeError("No transaction in progress")
        print("Rolling back transaction")
        self.in_transaction = False

class DatabaseStorage(Storage, ConnectionMixin, TransactionalMixin):
    """Database storage with connection and transaction support"""
    
    def __init__(self, connection_string: str):
        ConnectionMixin.__init__(self, connection_string)
        TransactionalMixin.__init__(self)
        self.data = {}
    
    def save(self, key: str, data: Any) -> bool:
        self._ensure_connected()
        print(f"Saving to database: {key} -> {data}")
        self.data[key] = data
        return True
    
    def load(self, key: str) -> Optional[Any]:
        self._ensure_connected()
        print(f"Loading from database: {key}")
        return self.data.get(key)
    
    def delete(self, key: str) -> bool:
        self._ensure_connected()
        if key in self.data:
            del self.data[key]
            print(f"Deleted from database: {key}")
            return True
        return False
    
    def exists(self, key: str) -> bool:
        self._ensure_connected()
        return key in self.data

class FileStorage(Storage, ConnectionMixin):
    """File storage with connection management but no transactions"""
    
    def __init__(self, file_path: str):
        ConnectionMixin.__init__(self, file_path)
        self.file_path = file_path
        self.data = {}
    
    def connect(self) -> bool:
        if self.connected:
            return True
        print(f"Opening file: {self.file_path}")
        # Simulate loading data from file
        self.data = {}  # In reality, would read from file
        self.connected = True
        return True
    
    def disconnect(self) -> None:
        if self.connected:
            print(f"Closing file: {self.file_path}")
            # In reality, would save data to file
            self.connected = False
    
    def save(self, key: str, data: Any) -> bool:
        self._ensure_connected()
        print(f"Writing to file: {key} -> {data}")
        self.data[key] = data
        return True
    
    def load(self, key: str) -> Optional[Any]:
        self._ensure_connected()
        print(f"Reading from file: {key}")
        return self.data.get(key)
    
    def delete(self, key: str) -> bool:
        self._ensure_connected()
        if key in self.data:
            del self.data[key]
            print(f"Deleted from file: {key}")
            return True
        return False
    
    def exists(self, key: str) -> bool:
        self._ensure_connected()
        return key in self.data

class MemoryStorage(Storage):
    """Simple in-memory storage - always available, no connection needed"""
    
    def __init__(self):
        self.data = {}
    
    def save(self, key: str, data: Any) -> bool:
        print(f"Storing in memory: {key} -> {data}")
        self.data[key] = data
        return True
    
    def load(self, key: str) -> Optional[Any]:
        print(f"Loading from memory: {key}")
        return self.data.get(key)
    
    def delete(self, key: str) -> bool:
        if key in self.data:
            del self.data[key]
            print(f"Deleted from memory: {key}")
            return True
        return False
    
    def exists(self, key: str) -> bool:
        return key in self.data

# Specialized interfaces for specific capabilities
class TransactionalStorage(Storage, TransactionalMixin):
    """Interface for storage systems that support transactions"""
    pass

class ConnectedStorage(Storage, ConnectionMixin):
    """Interface for storage systems that require connections"""
    pass

# Client code that works with different storage types appropriately
def basic_storage_operations(storage: Storage):
    """Works with any storage system"""
    print(f"\nTesting basic operations with {storage.__class__.__name__}:")
    
    # Basic operations that all storage systems support
    storage.save("user1", {"name": "Alice", "age": 30})
    storage.save("user2", {"name": "Bob", "age": 25})
    
    print(f"User1 exists: {storage.exists('user1')}")
    user_data = storage.load("user1")
    print(f"Loaded user1: {user_data}")
    
    storage.delete("user2")
    print(f"User2 exists after deletion: {storage.exists('user2')}")

def connected_storage_operations(storage: ConnectedStorage):
    """Works with storage systems that require connection management"""
    print(f"\nTesting connected operations with {storage.__class__.__name__}:")
    
    storage.connect()
    basic_storage_operations(storage)
    storage.disconnect()

def transactional_operations(storage: TransactionalStorage):
    """Works with storage systems that support transactions"""
    print(f"\nTesting transactional operations with {storage.__class__.__name__}:")
    
    if isinstance(storage, ConnectionMixin):
        storage.connect()
    
    storage.begin_transaction()
    storage.save("tx_data1", "value1")
    storage.save("tx_data2", "value2")
    storage.commit_transaction()
    
    storage.begin_transaction()
    storage.save("tx_data3", "value3")
    storage.rollback_transaction()
    
    print(f"tx_data1 exists: {storage.exists('tx_data1')}")
    print(f"tx_data3 exists (should be False): {storage.exists('tx_data3')}")
    
    if isinstance(storage, ConnectionMixin):
        storage.disconnect()

# Usage
memory_storage = MemoryStorage()
file_storage = FileStorage("/tmp/data.txt")
db_storage = DatabaseStorage("postgresql://localhost/mydb")

# All storage types work with basic operations
basic_storage_operations(memory_storage)
connected_storage_operations(file_storage)
connected_storage_operations(db_storage)

# Only transactional storage works with transactions
transactional_operations(db_storage)

# This would be a compile-time error if using type hints properly:
# transactional_operations(file_storage)  # FileStorage doesn't implement TransactionalMixin
```

### Example 4: Vehicle Speed Control

#### Violating LSP

```python
# BAD: Violates LSP
class Vehicle:
    def __init__(self, max_speed):
        self.max_speed = max_speed
        self.current_speed = 0
    
    def accelerate(self, amount):
        self.current_speed = min(self.current_speed + amount, self.max_speed)
        return self.current_speed
    
    def brake(self, amount):
        self.current_speed = max(self.current_speed - amount, 0)
        return self.current_speed
    
    def get_speed(self):
        return self.current_speed

class Car(Vehicle):
    def __init__(self):
        super().__init__(180)  # 180 km/h max speed

class Bicycle(Vehicle):
    def __init__(self):
        super().__init__(50)   # 50 km/h max speed

class Train(Vehicle):
    """Violates LSP: Trains cannot brake immediately or accelerate arbitrarily"""
    def __init__(self):
        super().__init__(300)  # 300 km/h max speed
        self.is_at_station = True
    
    def accelerate(self, amount):
        # Problem: Trains can only accelerate when not at station
        if self.is_at_station:
            raise RuntimeError("Cannot accelerate while at station")
        
        # Problem: Trains accelerate slowly and in fixed increments
        if amount > 10:
            amount = 10  # Trains can't accelerate more than 10 km/h at once
        
        return super().accelerate(amount)
    
    def brake(self, amount):
        # Problem: Trains need long distance to brake
        if self.current_speed > 100 and amount > 20:
            raise RuntimeError("Cannot brake that hard at high speed")
        
        return super().brake(amount)
    
    def leave_station(self):
        self.is_at_station = False
    
    def arrive_at_station(self):
        if self.current_speed > 0:
            raise RuntimeError("Must stop before arriving at station")
        self.is_at_station = True

class Airplane(Vehicle):
    """Violates LSP: Airplanes have complex speed requirements"""
    def __init__(self):
        super().__init__(900)  # 900 km/h max speed
        self.altitude = 0
        self.is_airborne = False
    
    def accelerate(self, amount):
        # Problem: Airplanes need minimum speed to stay airborne
        if self.is_airborne and self.current_speed - amount < 200:
            raise RuntimeError("Cannot slow down below stall speed while airborne")
        
        return super().accelerate(amount)
    
    def brake(self, amount):
        # Problem: Airplanes can only brake on ground
        if self.is_airborne:
            raise RuntimeError("Cannot brake while airborne")
        
        return super().brake(amount)
    
    def takeoff(self):
        if self.current_speed < 250:
            raise RuntimeError("Insufficient speed for takeoff")
        self.is_airborne = True
    
    def land(self):
        if self.current_speed > 300:
            raise RuntimeError("Too fast to land safely")
        self.is_airborne = False

# Client code that expects simple vehicle behavior
def emergency_stop(vehicle: Vehicle):
    """Tries to stop vehicle immediately - breaks with some subclasses"""
    print(f"Emergency stop for {vehicle.__class__.__name__}")
    current_speed = vehicle.get_speed()
    print(f"Current speed: {current_speed}")
    
    # Try to brake hard
    try:
        while vehicle.get_speed() > 0:
            vehicle.brake(50)  # Hard brake
        print("Vehicle stopped successfully")
    except RuntimeError as e:
        print(f"Emergency stop failed: {e}")

def speed_test(vehicle: Vehicle):
    """Tests vehicle acceleration - breaks with some subclasses"""
    print(f"\nSpeed test for {vehicle.__class__.__name__}")
    
    try:
        # Accelerate rapidly
        for _ in range(5):
            speed = vehicle.accelerate(30)
            print(f"Speed after acceleration: {speed}")
        
        # Emergency brake
        emergency_stop(vehicle)
        
    except RuntimeError as e:
        print(f"Speed test failed: {e}")

# Usage - demonstrates LSP violations
vehicles = [
    Car(),
    Bicycle(),
    Train(),
    Airplane()
]

for vehicle in vehicles:
    speed_test(vehicle)
```

#### Following LSP

```python
# GOOD: Following LSP
from abc import ABC, abstractmethod
from enum import Enum

class VehicleState(Enum):
    STOPPED = "stopped"
    MOVING = "moving"
    MAINTENANCE = "maintenance"

class Vehicle(ABC):
    """Abstract base class with common vehicle behavior"""
    
    def __init__(self, max_speed: float):
        self.max_speed = max_speed
        self.current_speed = 0.0
        self.state = VehicleState.STOPPED
    
    @abstractmethod
    def can_accelerate(self, amount: float) -> bool:
        """Check if vehicle can accelerate by given amount"""
        pass
    
    @abstractmethod
    def can_brake(self, amount: float) -> bool:
        """Check if vehicle can brake by given amount"""
        pass
    
    def accelerate(self, amount: float) -> float:
        """Accelerate if possible, otherwise do nothing"""
        if self.can_accelerate(amount):
            old_speed = self.current_speed
            self.current_speed = min(self.current_speed + amount, self.max_speed)
            if old_speed == 0 and self.current_speed > 0:
                self.state = VehicleState.MOVING
            return self.current_speed
        return self.current_speed
    
    def brake(self, amount: float) -> float:
        """Brake if possible, otherwise do nothing"""
        if self.can_brake(amount):
            self.current_speed = max(self.current_speed - amount, 0)
            if self.current_speed == 0:
                self.state = VehicleState.STOPPED
            return self.current_speed
        return self.current_speed
    
    def get_speed(self) -> float:
        return self.current_speed
    
    def get_state(self) -> VehicleState:
        return self.state
    
    def stop_gradually(self) -> bool:
        """Stop the vehicle using its normal braking capability"""
        while self.current_speed > 0:
            old_speed = self.current_speed
            self.brake(self.get_max_brake_amount())
            if self.current_speed == old_speed:
                return False  # Cannot brake further
        return True
    
    @abstractmethod
    def get_max_brake_amount(self) -> float:
        """Get maximum safe braking amount for this vehicle"""
        pass
    
    @abstractmethod
    def get_max_acceleration_amount(self) -> float:
        """Get maximum acceleration amount for this vehicle"""
        pass

class RoadVehicle(Vehicle):
    """Base class for vehicles that operate on roads"""
    
    def can_accelerate(self, amount: float) -> bool:
        return (self.state != VehicleState.MAINTENANCE and 
                amount <= self.get_max_acceleration_amount() and
                self.current_speed + amount <= self.max_speed)
    
    def can_brake(self, amount: float) -> bool:
        return (self.state != VehicleState.MAINTENANCE and
                amount <= self.get_max_brake_amount())

class Car(RoadVehicle):
    def __init__(self):
        super().__init__(180)  # 180 km/h max speed
    
    def get_max_brake_amount(self) -> float:
        return 50  # Cars can brake hard
    
    def get_max_acceleration_amount(self) -> float:
        return 30  # Cars can accelerate quickly

class Bicycle(RoadVehicle):
    def __init__(self):
        super().__init__(50)   # 50 km/h max speed
    
    def get_max_brake_amount(self) -> float:
        return 20  # Limited braking power
    
    def get_max_acceleration_amount(self) -> float:
        return 10  # Limited by human power

class RailVehicle(Vehicle):
    """Base class for rail-based vehicles"""
    
    def __init__(self, max_speed: float):
        super().__init__(max_speed)
        self.is_at_station = True
    
    def can_accelerate(self, amount: float) -> bool:
        return (not self.is_at_station and 
                self.state != VehicleState.MAINTENANCE and
                amount <= self.get_max_acceleration_amount() and
                self.current_speed + amount <= self.max_speed)
    
    def can_brake(self, amount: float) -> bool:
        return (self.state != VehicleState.MAINTENANCE and
                amount <= self.get_max_brake_amount())
    
    def leave_station(self) -> bool:
        if self.is_at_station and self.current_speed == 0:
            self.is_at_station = False
            return True
        return False
    
    def arrive_at_station(self) -> bool:
        if not self.is_at_station and self.current_speed == 0:
            self.is_at_station = True
            return True
        return False

class Train(RailVehicle):
    def __init__(self):
        super().__init__(300)  # 300 km/h max speed
    
    def get_max_brake_amount(self) -> float:
        # Trains brake more gently at high speeds
        if self.current_speed > 100:
            return 15
        return 25
    
    def get_max_acceleration_amount(self) -> float:
        return 10  # Trains accelerate slowly

class AerialVehicle(Vehicle):
    """Base class for flying vehicles"""
    
    def __init__(self, max_speed: float, min_airborne_speed: float):
        super().__init__(max_speed)
        self.min_airborne_speed = min_airborne_speed
        self.altitude = 0
        self.is_airborne = False
    
    def can_accelerate(self, amount: float) -> bool:
        return (self.state != VehicleState.MAINTENANCE and
                amount <= self.get_max_acceleration_amount() and
                self.current_speed + amount <= self.max_speed)
    
    def can_brake(self, amount: float) -> bool:
        if self.state == VehicleState.MAINTENANCE:
            return False
        
        if self.is_airborne:
            # Cannot brake below stall speed while airborne
            return self.current_speed - amount >= self.min_airborne_speed
        
        return amount <= self.get_max_brake_amount()
    
    def takeoff(self) -> bool:
        if not self.is_airborne and self.current_speed >= self.min_airborne_speed:
            self.is_airborne = True
            return True
        return False
    
    def land(self) -> bool:
        if self.is_airborne and self.current_speed <= self.min_airborne_speed * 1.2:
            self.is_airborne = False
            return True
        return False

class Airplane(AerialVehicle):
    def __init__(self):
        super().__init__(900, 200)  # 900 km/h max, 200 km/h min airborne
    
    def get_max_brake_amount(self) -> float:
        if self.is_airborne:
            return 0  # Cannot brake while airborne
        return 40
    
    def get_max_acceleration_amount(self) -> float:
        return 25

# Client code that works properly with LSP
def gentle_stop(vehicle: Vehicle) -> bool:
    """Stops vehicle using its natural braking capabilities"""
    print(f"Gently stopping {vehicle.__class__.__name__}")
    print(f"Initial speed: {vehicle.get_speed()}")
    
    success = vehicle.stop_gradually()
    print(f"Final speed: {vehicle.get_speed()}")
    print(f"Stop successful: {success}")
    return success

def accelerate_safely(vehicle: Vehicle, target_speed: float) -> float:
    """Accelerates vehicle safely within its limits"""
    print(f"\nSafely accelerating {vehicle.__class__.__name__} to {target_speed}")
    
    while vehicle.get_speed() < target_speed:
        max_accel = vehicle.get_max_acceleration_amount()
        needed = target_speed - vehicle.get_speed()
        accel_amount = min(max_accel, needed)
        
        old_speed = vehicle.get_speed()
        vehicle.accelerate(accel_amount)
        
        if vehicle.get_speed() == old_speed:
            break  # Cannot accelerate further
        
        print(f"Speed: {vehicle.get_speed()}")
    
    return vehicle.get_speed()

def vehicle_performance_test(vehicle: Vehicle):
    """Test that works with any vehicle type"""
    print(f"\n=== Testing {vehicle.__class__.__name__} ===")
    
    # Test acceleration
    final_speed = accelerate_safely(vehicle, 60)
    print(f"Reached speed: {final_speed}")
    
    # Test braking
    gentle_stop(vehicle)
    
    print(f"Final state: {vehicle.get_state()}")

# Specialized functions for specific vehicle types
def rail_vehicle_operations(train: RailVehicle):
    """Operations specific to rail vehicles"""
    print(f"\n=== Rail Operations for {train.__class__.__name__} ===")
    
    print("Leaving station...")
    if train.leave_station():
        accelerate_safely(train, 100)
        gentle_stop(train)
        print("Arriving at station...")
        train.arrive_at_station()
    else:
        print("Cannot leave station")

def aerial_vehicle_operations(aircraft: AerialVehicle):
    """Operations specific to aerial vehicles"""
    print(f"\n=== Aerial Operations for {aircraft.__class__.__name__} ===")
    
    # Taxi and takeoff
    accelerate_safely(aircraft, aircraft.min_airborne_speed)
    if aircraft.takeoff():
        print("Takeoff successful!")
        accelerate_safely(aircraft, 400)
        
        # Prepare for landing
        while aircraft.get_speed() > aircraft.min_airborne_speed * 1.2:
            aircraft.brake(aircraft.get_max_brake_amount())
        
        if aircraft.land():
            print("Landing successful!")
            gentle_stop(aircraft)

# Usage
vehicles = [
    Car(),
    Bicycle(),
    Train(),
    Airplane()
]

# Test basic vehicle operations (works with all vehicles)
for vehicle in vehicles:
    vehicle_performance_test(vehicle)

# Test specialized operations
train = Train()
rail_vehicle_operations(train)

airplane = Airplane()
aerial_vehicle_operations(airplane)
```

## Behavioral Contracts

### Understanding Contracts in LSP

LSP is fundamentally about maintaining behavioral contracts. When a subclass inherits from a parent class, it enters into a contract to behave in a way that's compatible with the parent's contract.

#### Preconditions
**Rule: Preconditions cannot be strengthened in subtypes**

```python
# BAD: Strengthening preconditions
class Calculator:
    def divide(self, a: float, b: float) -> float:
        # Precondition: b != 0
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b

class StrictCalculator(Calculator):
    def divide(self, a: float, b: float) -> float:
        # BAD: Strengthening precondition by adding b > 0
        if b <= 0:  # Stronger condition than parent
            raise ValueError("Divisor must be positive")
        return a / b

# GOOD: Weakening or maintaining preconditions
class LenientCalculator(Calculator):
    def divide(self, a: float, b: float) -> float:
        # GOOD: Weaker precondition (handles more cases)
        if b == 0:
            return float('inf') if a > 0 else float('-inf') if a < 0 else float('nan')
        return a / b
```

#### Postconditions
**Rule: Postconditions cannot be weakened in subtypes**

```python
# BAD: Weakening postconditions
class FileProcessor:
    def save_file(self, filename: str, data: str) -> bool:
        # Postcondition: Returns True if file was saved successfully
        with open(filename, 'w') as f:
            f.write(data)
        return True

class UnreliableFileProcessor(FileProcessor):
    def save_file(self, filename: str, data: str) -> bool:
        # BAD: Weakening postcondition (might not actually save)
        try:
            with open(filename, 'w') as f:
                f.write(data)
            return True
        except:
            return False  # Returns False even when it should guarantee success

# GOOD: Strengthening or maintaining postconditions
class ReliableFileProcessor(FileProcessor):
    def save_file(self, filename: str, data: str) -> bool:
        # GOOD: Stronger postcondition (also creates backup)
        with open(filename, 'w') as f:
            f.write(data)
        # Additional guarantee: create backup
        with open(filename + '.backup', 'w') as f:
            f.write(data)
        return True
```

#### Invariants
**Rule: Invariants must be preserved in subtypes**

```python
# Example of maintaining invariants
class BankAccount:
    def __init__(self, initial_balance: float):
        self._balance = initial_balance
    
    @property
    def balance(self) -> float:
        return self._balance
    
    def withdraw(self, amount: float) -> bool:
        # Invariant: balance >= 0
        if amount <= self._balance:
            self._balance -= amount
            return True
        return False

class SavingsAccount(BankAccount):
    def __init__(self, initial_balance: float, min_balance: float = 100):
        super().__init__(initial_balance)
        self.min_balance = min_balance
    
    def withdraw(self, amount: float) -> bool:
        # GOOD: Maintains parent invariant while adding stricter constraint
        if amount <= self._balance - self.min_balance:
            self._balance -= amount
            return True
        return False
```

## Benefits of Following LSP

### 1. **Reliable Polymorphism**
Code that works with base classes automatically works with all subclasses.

### 2. **Simplified Testing**
Tests written for base classes validate the behavior of all subclasses.

### 3. **Reduced Client Code Complexity**
Client code doesn't need type checking or special handling for different subclasses.

### 4. **Enhanced Code Reusability**
Generic algorithms can work with any conforming subclass.

### 5. **Improved Design Quality**
Forces careful consideration of inheritance relationships.

### 6. **Better Maintainability**
Changes to subclasses are less likely to break existing client code.

## Common Pitfalls

### 1. **Refusing Bequest**
Creating subclasses that throw exceptions for inherited methods they cannot support.

### 2. **Strengthening Preconditions**
Making subclasses more restrictive than their parents about what inputs they accept.

### 3. **Weakening Postconditions**
Making subclasses provide weaker guarantees than their parents.

### 4. **Violating Invariants**
Breaking class invariants that were established by the parent class.

### 5. **Changing Expected Behavior**
Implementing methods in ways that surprise users familiar with the parent class.

### 6. **Using Implementation Inheritance Instead of Interface Inheritance**
Inheriting for code reuse rather than substitutability.

## Best Practices

### 1. **Design by Contract**
Clearly define preconditions, postconditions, and invariants for your classes.

### 2. **Prefer Composition Over Inheritance**
Use inheritance only when true substitutability is needed.

### 3. **Use Abstract Base Classes**
Define clear interfaces that subclasses must implement.

### 4. **Test Substitutability**
Write tests that verify subclasses can replace parent classes.

### 5. **Follow the IS-A Principle Carefully**
Just because something "is a" something else doesn't mean it should inherit from it.

### 6. **Document Behavioral Contracts**
Clearly document the expected behavior of methods and classes.

### 7. **Use Type Hints and Protocols**
Leverage Python's type system to enforce correct usage.

### 8. **Regular Code Reviews**
Review inheritance hierarchies for LSP compliance.

## Measuring LSP Compliance

### Code Metrics to Watch:
- **Exception Types**: Subclasses throwing different exceptions than parents
- **Return Type Consistency**: Subclasses returning different types than expected
- **Method Signature Changes**: Subclasses changing method signatures
- **Behavioral Divergence**: Subclasses behaving differently than documented

### Questions to Ask:
1. Can I substitute any subclass for its parent without breaking client code?
2. Do all subclasses honor the contracts established by their parent classes?
3. Are there any methods in subclasses that simply throw "not implemented" exceptions?
4. Do I need to check the type of an object before calling its methods?
5. Do my unit tests pass when run against all subclasses?

### LSP Validation Checklist:
- [ ] All subclasses can be used wherever the parent class is expected
- [ ] No subclass strengthens preconditions of inherited methods
- [ ] No subclass weakens postconditions of inherited methods
- [ ] All class invariants are maintained in subclasses
- [ ] No subclass throws exceptions that the parent doesn't throw
- [ ] Client code works with all subclasses without modification
- [ ] Unit tests for the base class pass for all subclasses

## Conclusion

The Liskov Substitution Principle is crucial for creating robust object-oriented designs that support reliable polymorphism. By ensuring that subclasses can truly substitute for their parent classes, you create systems that are more predictable, maintainable, and extensible.

LSP forces you to think carefully about inheritance relationships and to distinguish between "is-a" relationships (structural similarity) and "behaves-like-a" relationships (behavioral compatibility). True substitutability requires both structural and behavioral compatibility.

Remember that LSP is not just about avoiding runtime errors - it's about maintaining the logical consistency and predictability of your object hierarchies. When you violate LSP, you're essentially breaking the promises made by your class interfaces, which leads to fragile and unpredictable code.

The key to successful LSP implementation is to design your inheritance hierarchies around behavioral contracts rather than just code reuse. Focus on what objects do, not just what they are, and ensure that all subclasses can fulfill the behavioral promises made by their parent classes.

By following LSP, you create inheritance hierarchies that are logically sound, easy to understand, and safe to extend, leading to more robust and maintainable object-oriented systems.