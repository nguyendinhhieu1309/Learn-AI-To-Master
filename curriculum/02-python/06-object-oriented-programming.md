# Object-Oriented Programming in Python

Object-oriented programming (OOP) groups data with behavior that operates on that data. It can model entities with stable rules, but simple functions and dictionaries are often clearer for simple transformations.

## Learning objectives

- Explain class, instance, attribute, and method.
- Use constructors, instance/class/static methods, properties, and special methods.
- Apply encapsulation, inheritance, and polymorphism.
- Choose composition over inheritance when components vary independently.

## 1. Class and instance

A class describes a kind of object. An instance is one object created from that class. Instance attributes hold per-object state.

~~~python
class Sensor:
    def __init__(self, sensor_id: str, unit: str) -> None:
        self.sensor_id = sensor_id
        self.unit = unit

    def describe(self) -> str:
        return f"{self.sensor_id} measures {self.unit}"

temperature_sensor = Sensor("temp-01", "degrees C")
print(temperature_sensor.describe())
~~~

The initializer receives the new instance as self. Each instance has its own sensor_id and unit. Avoid mutable class attributes when each instance needs a separate list or dictionary.

## 2. Instance, class, and static methods

An instance method receives self and can read or change that object's state. A class method receives cls and can construct or inspect the class. A static method receives neither automatically and is best for a related helper that needs no object state.

~~~python
class Prediction:
    valid_labels = {"cat", "dog"}

    def __init__(self, label: str, score: float) -> None:
        self.label = label
        self.score = score

    @classmethod
    def from_probability(cls, label: str, score: float) -> "Prediction":
        return cls(label, score)

    @staticmethod
    def valid_score(score: float) -> bool:
        return 0.0 <= score <= 1.0
~~~

## 3. Encapsulation and properties

Encapsulation keeps an object's rules near its state. Python uses naming conventions rather than strict private fields: a leading underscore means “internal; use cautiously.”

~~~python
class Account:
    def __init__(self, balance: float = 0.0) -> None:
        self._balance = balance

    @property
    def balance(self) -> float:
        return self._balance

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("deposit must be positive")
        self._balance += amount
~~~

The property provides a read interface; deposit enforces a state rule. Do not add a property for every field when direct attributes are sufficient.

## 4. Inheritance and polymorphism

Inheritance represents a true “is a” relationship and allows a subtype to specialize behavior. Use super to call the parent implementation.

~~~python
class ImageSensor(Sensor):
    def __init__(self, sensor_id: str, width: int, height: int) -> None:
        super().__init__(sensor_id, "pixels")
        self.width = width
        self.height = height

    def describe(self) -> str:
        return f"{self.sensor_id}: {self.width}x{self.height}"
~~~

Polymorphism means different objects can support the same operation. Python often uses duck typing: if an object provides the needed method, a caller can use it without requiring the same parent class.

Do not build deep inheritance chains to share a few lines of code. Prefer composition when an object “has a” component:

~~~python
class Camera:
    def __init__(self, calibration) -> None:
        self.calibration = calibration
~~~

Camera has a calibration object; calibration need not inherit from Camera.

## 5. Special methods and data classes

Special methods let user-defined objects work with Python operations. Common examples include __repr__ for debugging, __str__ for display, __eq__ for equality, and __len__ for length. Implement them only when their behavior is clear and consistent.

For a plain record, a data class may be simpler than writing boilerplate:

~~~python
from dataclasses import dataclass

@dataclass
class BoundingBox:
    x_min: float
    y_min: float
    x_max: float
    y_max: float
~~~

Data classes generate methods such as an initializer and readable representation. They do not automatically validate coordinates.

## 6. OOP design checklist

- Does this object have meaningful state and behavior that belong together?
- Are invariants checked at the boundary?
- Is the class easier to use than a function plus a simple record?
- Can composition avoid a fragile inheritance relationship?
- Are side effects and object mutation visible?

## Practice with answers

1. What is the difference between a class and an instance?
2. When should a method receive self?
3. What does a leading underscore communicate?
4. Why might composition be better than inheritance?

### Answers

1. A class defines a kind of object; an instance is one concrete object.
2. When the operation needs that instance's state or changes it.
3. The attribute is internal by convention.
4. Components that vary independently can be replaced without creating a deep subtype hierarchy.
