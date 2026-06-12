# Python Hands-On Guide

## Table of Contents
1. [Getters and Setters](#1-getters-and-setters)
2. [List and Tuple Differences](#2-list-and-tuple-differences)
3. [Dictionary and Set](#3-dictionary-and-set)
4. [List and Dict Comprehension](#4-list-and-dict-comprehension)
5. [Decorators, Generators and Iterators](#5-decorators-generators-and-iterators)
6. [Lambda Functions](#6-lambda-functions)
7. [Args and Kwargs](#7-args-and-kwargs)
8. [Shallow and Deep Copy](#8-shallow-and-deep-copy)
9. [Pass by Assignment](#9-pass-by-assignment)
10. [Abstract Class](#10-abstract-class)
11. [Encapsulation](#11-encapsulation)
12. [Polymorphism](#12-polymorphism)
13. [Inheritance](#13-inheritance)
14. [Instance, Static, Class Methods](#14-instance-static-class-methods-and-attributes)
15. [Multithreading](#15-multithreading)
16. [Memory Management](#16-memory-management)
17. [Custom Exceptions](#17-custom-exceptions)
18. [Zero-Based Indexing](#18-zero-based-indexing)
19. [Database Configuration](#19-database-configuration)
20. [Docker](#20-docker)
21. [Context Manager](#21-Context-Manager)
---

## 1. Getters and Setters

### For Protected and Private Attributes

```python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age
        
    def get_name(self):
        return self._name
    
    def set_name(self, name):
        self._name = name
    
    def get_age(self):
        return self._age
        
    def set_age(self, age):
        self._age = age
        
p = Person('sanjay', 25)
print(p.get_name())
p.set_age(22)
print(p.get_age())
```

**Output:**
```
sanjay
22
```

### For Public Attributes with Magic Methods

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return f"Person(name='{self.name}', age={self.age})"

    def __repr__(self):
        return f"Person('{self.name}', {self.age})"

    def __eq__(self, other):
        return isinstance(other, Person) and self.name == other.name and self.age == other.age

    def __hash__(self):
        return hash((self.name, self.age))

p1 = Person("Sanjay", 25)
p2 = Person("Sanjay", 25)
p3 = Person("Ram", 22)

print(p1)         # Uses __str__: Person(name='Sanjay', age=25)
print(repr(p1))   # Uses __repr__: Person('Sanjay', 25)

print(p1 == p2)   # True (compares values)
print(p1 == p3)   # False

# Now can use in sets!
people = {p1, p2, p3}
print(people)     # Only 2 unique people, since p1 == p2
```

> **Note:** Without `__eq__`, Python compares memory addresses, so you'd get False.

---

## 2. List and Tuple Differences

| Feature     | List              | Tuple              |
|-------------|-------------------|--------------------|
| Mutable     | ✅ Yes            | ❌ No              |
| Syntax      | `[item1, item2]`  | `(item1, item2)`   |
| Performance | Slightly slower   | Faster (read-only) |
| Use case    | When data changes | Fixed data         |
| Hashing     | NO                | Yes                |

### List Example

```python
nums = [1, 2, 3]
nums.append(4)
print(nums)  # [1, 2, 3, 4]
```

### List Operations Comparison

| Operation  | Removes by  | Returns            | Can delete slices? | Error if not found? | Notes                                                                                              |
|------------|-------------|--------------------|--------------------|---------------------|----------------------------------------------------------------------------------------------------|
| `pop()`    | Index       | ✔ Removed element  | ❌ No              | ✔ IndexError        | Useful when you need the removed value (e.g., stack operations). Default removes last element.     |
| `remove()` | Value       | ❌ None            | ❌ No              | ✔ ValueError        | Removes the first matching value only. Good for removing by content, not position.                 |
| `del`      | Index/Slice | ❌ Nothing         | ✔ Yes              | ✔ IndexError        | More powerful: can delete slices or entire variables. Doesn't return anything.                     |

### Tuple Example

```python
coordinates = (10, 20)
print(coordinates[0])  # 10
```

**❌ Unsupported (Because Tuples Are Immutable)**
- `t[0] = 100` → ❌ Error
- `.append()`, `.remove()`, `.pop()` → ❌ Not available

### Difference Between `append()` and `extend()`

#### ✅ 1. `append()`
- **Purpose:** Adds one element to the end of the list
- **Behavior:** Adds the item as a single object, even if it's another list

```python
l = [1, 2, 3]
l.append([4, 5])
print(l)  # [1, 2, 3, [4, 5]]
```

#### ✅ 2. `extend()`
- **Purpose:** Adds multiple elements to the list
- **Behavior:** Iterates through the argument and adds elements one by one

```python
l = [1, 2, 3]
l.extend([4, 5])
print(l)  # [1, 2, 3, 4, 5]
```

| Operation  | Adds           | Input Type Accepted                                      | How it behaves                          | Example Input    | Output          | Notes                                        |
|------------|----------------|----------------------------------------------------------|-----------------------------------------|------------------|-----------------|----------------------------------------------|
| `append()` | One item       | Any object (int, string, list, tuple, dict, set, etc.)  | Adds the entire object as a single element | `append([4,5])` | `[1,2,3,[4,5]]` | Use when adding one element only.            |
| `extend()` | Multiple items | Any iterable (list, tuple, string, set, dict keys)      | Iterates and adds elements one by one   | `extend([4,5])`  | `[1,2,3,4,5]`   | Use when merging lists or adding multiple values. |

### 🔍 Quick Notes
- **`append()`**
  - Input type: ANY object
  - Adds the object as it is
- **`extend()`**
  - Input type: ONLY iterables
  - Adds the contents of the iterable

### ⚠️ Common Pitfall with List Multiplication

```python
a = [[]] * 5 
print(a)
a[1].append(20)
a[2].append(30)
print(a)
```

**Output:**
```python
[[], [], [], [], []]
[[20, 30], [20, 30], [20, 30], [20, 30], [20, 30]]
```

**🧠 Interview Takeaway:**
- `*` with mutable objects → shared references
- Safe for immutable types (like ints, strings)
- Dangerous for mutable types (list, dict, set)

**✅ Correct Way (Independent Lists)**

```python
a = [[] for _ in range(5)]
```

Now each element is a different list.

---

## 3. Dictionary and Set

### Dictionary
A dictionary is an unordered collection of key-value pairs. Keys must be unique and immutable.

person = dict(name="Alice", age=25, city="New York")
print(person)
# Output: {'name': 'Alice', 'age': 25, 'city': 'New York'}

for k in person.keys():
    if k == 'age':
        print(person[k])

l = [1, 1, 2, 2, 3, 3, 4, 5]
d = dict()
for i in l:
    if i in d.keys():
        d[i] += 1
    else:
        d[i] = 1
        
print(d)
```

**Output:**
```
{'name': 'Alice', 'age': 25, 'city': 'New York'}
25
{1: 2, 2: 2, 3: 2, 4: 1, 5: 1}
```

#### Using a List of Tuples

```python
print(person)
# Output: {'name': 'Alice', 'age': 25, 'city': 'New York'}
```

#### Using `zip()` with Two Lists

```python
values = ["Bob", 30, "London"]
person = dict(zip(keys, values))
print(person)
# Output: {'name': 'Bob', 'age': 30, 'city': 'London'}
```

### Set

A set is an unordered collection of unique elements.

```python
print(numbers)  # Output: {1, 2, 3, 4}

empty_set = set()
numbers.add(5)        # Add element
numbers.remove(2)     # Remove element (KeyError if not present)
numbers.discard(10)   # Safe remove (no error if not present)
```

### Dictionary vs Set Comparison

| Feature    | Dictionary (`dict`)         | Set (`set`)               |
|------------|-----------------------------|---------------------------|
| Stores     | Key-value pairs             | Unique values only        |
| Indexing   | By key                      | Not supported (unordered) |
| Mutable    | ✅ Yes                      | ✅ Yes                    |
| Duplicates | Keys must be unique         | Elements must be unique   |
| Common use | Structured data (name: age) | Membership checks, math   |

---

## 4. List and Dict Comprehension
List comprehension in Python is a compact, readable, and faster way to create lists using a single line of code.

✅ Definition
List comprehension allows you to create a new list by applying an expression to each item in an iterable, optionally filtering items with a condition.

### 🧠 Basic Syntax

```python
new_list = [expression for item in iterable]
```

**With condition:**
```python
new_list = [expression for item in iterable if condition]
```

### Benefits

| Feature          | Benefit                                |
|------------------|----------------------------------------|
| Shorter Code     | Reduces multiple lines into one        |
| Faster           | Usually faster than loops              |
| Readability      | Cleaner and more Pythonic              |
| Functional Style | Combines loop + condition in one place |

### Comprehension Types

| Type               | Syntax Example                         | Result Type     |
|--------------------|----------------------------------------|-----------------|
| List comprehension | `[x*2 for x in range(5)]`              | List            |
| Dict comprehension | `{x: x**2 for x in range(5)}`          | Dictionary      |
| With condition     | `[x for x in range(10) if x % 2 == 0]` | List (filtered) |

### Examples

```python
lc = [i*i for i in l if i % 2 == 0]
print(lc)

d = {x: x*x for x in l if x % 2 == 0}
print(d)
```

**Output:**
```
[4, 16]
{2: 4, 4: 16}
```

```python
values = [1, 2, 3]
combined = {k: v for k, v in zip(keys, values)}
print(combined)
# Output: {'a': 1, 'b': 2, 'c': 3}
enumerate() Function
enumerate() is a built-in Python function that lets you loop over a sequence while automatically getting the index of each item.

✅ Definition
enumerate(iterable, start=0)
Returns pairs: (index, item) for each element in the iterable.

🧠 Why use enumerate()?
It avoids manually writing:

for i in range(len(list)):
    print(i, list[i])
Instead, you use:

for i, val in enumerate(list):
    print(i, val)
Much cleaner and more Pythonic.

#### 🔧 Syntax
```python
enumerate(iterable, start=0)
```
- **iterable** → list, tuple, string, etc.
- **start** → optional, default is 0

#### Examples

**Example 1: Basic usage**
```python

for index, fruit in enumerate(fruits):
    print(index, fruit)
```
**Output:**
```
0 apple
1 banana
2 mango
```

**Example 2: Custom start index**
```python
    print(index, fruit)
Output:

1 apple
2 banana
3 mango
**Example 3: With list comprehension**
```python
pairs = [(i, x) for i, x in enumerate(['a', 'b', 'c'])]
print(pairs)
```
**Output:**
```
[(0, 'a'), (1, 'b'), (2, 'c')]
```

**Example 4: Using enumerate on a string**
```python
    print(i, ch)
Output:

0 p
1 y
2 t
3 h
4 o
5 n
```
---

## 5. Decorators, Generators and Iterators
Decorators
Decorators in Python are essentially functions that add functionality to an existing function without changing the structure of the function itself. They are represented by @decorator_name and are called in a bottom-up fashion.

```python
def mydecorator(func):
    def wrapper():
        print("before")
        func()
        print("After")
    return wrapper

@mydecorator
def hello():
    print("hello")

hello()
```

**Output:**
```
before
hello
After
```

#### Decorator with Arguments

```python
def mydec(func):
    def wrapper(*args):
        print("before")
        func(*args)
        print("after")
    return wrapper

@mydec
def hello(s, s2):
    print(f"hello {s} and {s2}")
    
hello("sanjay", "kumar")

Output:
before
hello sanjay and kumar
after

```

#### Practical Example: Logging Decorator

```python
def log(func):
    def wrapper(n):
        print(f"Calling {func.__name__} with n = {n}")
        result = func(n)
        print(f"{func.__name__}({n}) = {result}")
        return result
    return wrapper

@log
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Test
print(fibonacci(5))
```

**Output:**
```
Calling fibonacci with n = 5
Calling fibonacci with n = 4
Calling fibonacci with n = 3
...
fibonacci(1) = 1
fibonacci(2) = 1
...
fibonacci(5) = 5
```

### Iterators
An iterator is an object that allows you to iterate over a sequence, one element at a time.

🔹 How it works:
Implements two methods:

__iter__() → returns the iterator object itself
__next__() → returns the next value or raises StopIteration
#### Iterable vs Iterator

**✔ Iterable**
- Can be looped over
- Examples: list, tuple, string, dict, set
- Contains elements but does NOT know how to give the next element by itself

```python
nums = [1, 2, 3, 4]  # iterable
```

**✔ Iterator**
- Knows how to give the next element
- Uses `__next__()` internally
- Can be used with `next(iterator)`
- One-time use (gets exhausted)

```python
print(next(i))  # 1
print(next(i))  # 2
print(next(i))  # 3
print(next(i))  # 4
```

**Note:** `next(nums)` will NOT work → ❌ TypeError: 'list' object is not an iterator

#### Custom Iterator Example

```python
class Numbers:
    def __init__(self):
        self.num = 1
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.num <= 10:
            val = self.num
            self.num += 1
            return val
        else:
            raise StopIteration

n = Numbers()
for i in n: 
    print(i)
```

### Generators

A generator is a simpler way to create an iterator using a function with `yield`.

```python
def fib(n):
    a, b = 0, 1
    for i in range(n):
        yield a
        a, b = b, a+b
        
f = fib(7)
for i in f:
    print(i)
```

```python
def count_up_to(n):
    num = 1
    while num <= n:
        yield num
        num += 1

for i in count_up_to(5):
    print(i)
# Output: 1 2 3 4 5
```

#### Regular Function Equivalent

```python
def count_up_to_reg(n):
    result = []
    num = 1
    while num <= n:
        result.append(num)
        num += 1
    return result
```

**Key Difference:**
- **Generator** = produce values lazily (one by one), low memory, pause/resume
- **Regular function** = produce everything at once, high memory usage

### Iterator vs Generator Comparison

| Feature     | Iterator                          | Generator                    |
|-------------|-----------------------------------|------------------------------|
| Definition  | Class with `__iter__`, `__next__` | Function with `yield`        |
| State saved | Manually handled                  | Automatically by Python      |
| Syntax      | Verbose                           | Compact and Pythonic         |
| Use Case    | Custom iteration logic            | Efficient looping, streaming |

---

## 6. Lambda Functions
A lambda is a short, anonymous function (i.e., a function without a name), defined using the lambda keyword.

### Syntax
```python
lambda arguments: expression
```

- You can have multiple arguments
- The function body must be a single expression
- Automatically returns the result of the expression (no return needed)

### Examples

```python
print(mul(2, 5))  # output => 10
def make_multiplier(n):
    return lambda x: x * n

double = make_multiplier(2)
print(double(5))  # Output: 10

triple = make_multiplier(3)
print(triple(5))  # Output: 15
```

### Common Functions with Lambda

**Syntax:**
```python
map(function, iterable)
filter(function, iterable)
from functools import reduce
reduce(function, iterable)
```

#### `map()` Example

```python
squared = list(map(lambda x: x**2, nums))
print(squared)  # [1, 4, 9, 16]
```

#### `filter()` Example

```python
evens = list(filter(lambda x: x % 2 == 0, nums))
print(evens)  # [2, 4, 6]
```

#### `reduce()` Example

```python

nums = [1, 2, 3, 4]
res = reduce(lambda a, b: a + b, nums)
print(res)  # 10
```

**How it works:**
```
1 + 2 → 3
3 + 3 → 6
6 + 4 → 10
```

**🧠 Use when:**
- ✔ You need one final result
- ✔ Aggregation logic

#### `sorted()` with Lambda

```python
sorted_students = sorted(students, key=lambda x: x[1])
print(sorted_students)
# Output: [('Bob', 75), ('Alice', 85), ('Charlie', 90)]
```

---

## 7. Args and Kwargs

### `*args`

`*args` is a special syntax used in the function definition to pass variable-length arguments.
- `*` means variable length
- `args` is the name used by convention

### `**kwargs`

`**kwargs` is a special syntax used in the function definition to pass variable-length keyworded arguments.
- `kwargs` is used just by convention
- Keyworded argument means a variable that has a name when passed to a function
- It is actually a dictionary of the variable names and its value

### Example

```python
def profile(title, *args, **kwargs):
    print("Title:", title)
    print("Positional:", args)
    print("Keyword:", kwargs)

profile("Engineer", "Python", "AI", name="Sanjay", experience=3, stack="py")
```

**Output:**
```
Title: Engineer
Positional: ('Python', 'AI')
Keyword: {'name': 'Sanjay', 'experience': 3, 'stack': 'py'}
```

---

## 8. Shallow and Deep Copy

### ### Shallow Copy
A bit-wise copy of an object. The copied object created has an exact copy of the values in the original object. If either of the values is a reference to other objects, just the reference addresses for the same are copied.

### Deep Copy
Copies all values recursively from source to target object, i.e., it even duplicates the objects referenced by the source object.

### Example

```python

list_1 = [1, 2, [3, 5], 4]

## shallow copy
list_2 = copy(list_1) 
list_2[3] = 7
list_2[2].append(6)
list_2    # output => [1, 2, [3, 5, 6], 7]
list_1    # output => [1, 2, [3, 5, 6], 4]

## deep copy
list_3 = deepcopy(list_1)
list_3[3] = 8
list_3[2].append(7)
list_3    # output => [1, 2, [3, 5, 6, 7], 8]
list_1    # output => [1, 2, [3, 5, 6], 4]
```

**Key Difference:**
- **Shallow copy** → copies only the outer list; inner lists are shared
- **Deep copy** → copies everything, including nested lists

---

## 9. Pass by Assignment

### Pass by Value vs Pass by Reference

- **Pass by value:** Copy of the actual object is passed. Changing the value of the copy of the object will not change the value of the original object.
- **Pass by reference:** Reference to the actual object is passed. Changing the value of the new object will change the value of the original object.

**In Python, arguments are not passed by value or reference, but through pass-by-assignment or pass by object.**

---

## 10. Abstract Class
Abstraction means hiding complex internal details and showing only the essential features of an object.

Python uses the `abc` module to define abstract classes and methods.
👉 `abc` stands for: **Abstract Base Class**

To create an abstract class we have to import `from abc import ABC, abstractmethod`

### Example

```python
from abc import ABC, abstractmethod
class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass

class Dog(Animal):
    def make_sound(self):
        return "Bark!"

class Cat(Animal):
    def make_sound(self):
        return "Meow!"

# Create objects
dog = Dog()
cat = Cat()

print(dog.make_sound())  # Output: Bark!
print(cat.make_sound())  # Output: Meow!
```

---

## 11. Encapsulation
Encapsulation is the practice of wrapping data (attributes) and code (methods) together inside a class and restricting direct access to some of the object's components.

### ✅ Goals of Encapsulation:
- Hide internal data
- Control access using getters and setters
- Prevent accidental modification of data

### Example

```python
class Person:
    def __init__(self, name, age):
        self.__name = name  # private attribute
        self.__age = age    # private attribute

    # Getter for name
    def get_name(self):
        return self.__name

    # Setter for name
    def set_name(self, name):
        self.__name = name

    # Getter for age
    def get_age(self):
        return self.__age

    # Setter for age
    def set_age(self, age):
        if age > 0:
            self.__age = age

p = Person("Sanjay", 25)

# Access through methods (safe)
print(p.get_name())  # Output: Sanjay

# Modify value
p.set_age(30)
print(p.get_age())   # Output: 30

# Direct access is blocked:
# print(p.__name)    # ❌ AttributeError
```

---

## 12. Polymorphism
Polymorphism means "many forms." In Python, it allows the same method or operator to behave differently depending on the object it's acting on.

### ✅ Method Overriding (Runtime Polymorphism)

**📌 What is it?**
Method overriding occurs when a subclass provides a specific implementation of a method that is already defined in its superclass.

```python
class Animal:
    def speak(self):
        print("Animal speaks")

class Dog(Animal):
    def speak(self):  # Overriding the parent method
        print("Dog barks")

d = Dog()
d.speak()  # Output: Dog barks
```

### ✅ Method Overloading (Compile Time - Python Style)

**📌 What is it?**
Method overloading means having multiple methods in the same class with the same name but different parameters.

🛑 **Python does not support traditional method overloading like Java or C++.**
Instead, Python handles it using default arguments or `*args`.

```python
class Calculator:
    def add(self, *args):
        return sum(args)

c = Calculator()
print(c.add(2, 3))        # Output: 5
print(c.add(1, 2, 3, 4))  # Output: 10
```

---

## 13. Inheritance
Inheritance allows a class (child class) to reuse the properties and methods of another class (parent class). It promotes code reusability and makes it easier to maintain and extend code.

### Example

```python
class Animal:
    def speak(self):
        print("Animal speaks")

# Dog inherits from Animal
class Dog(Animal):
    def bark(self):
        print("Dog barks")

# Create an instance of Dog
d = Dog()
d.speak()  # Inherited method
d.bark()   # Dog's own method
```

**Output:**
```
Animal speaks
Dog barks
```

### Types of Inheritance

| Type             | Description                     | Example                          |
|------------------|----------------------------------|----------------------------------|
| **Single**       | One child, one parent           | `class Dog(Animal)`              |
| **Multiple**     | One child, multiple parents     | `class Child(Father, Mother)`    |
| **Multilevel**   | Inheritance chain               | `class C(B); class B(A)`         |
| **Hierarchical** | One parent, many children       | `class Cat(Animal), Dog(Animal)` |
| **Hybrid**       | Combination of any of the above | Complex structures               |

### What is MRO?

MRO (Method Resolution Order) is the order in which Python searches for methods and attributes in an inheritance hierarchy. It is especially important in multiple inheritance to resolve ambiguity. Python uses the C3 Linearization Algorithm to compute the MRO, which can be viewed using:
ClassName.mro()
```python
class A:
    def show(self):
        print("A")

class B(A):
    def show(self):
        print("B")

class C(A):
    def show(self):
        print("C")

class D(B, C):
    pass

obj = D()
obj.show()
```
Output:
```
B

Why?

Python follows the MRO:

print(D.mro())

Output:

[D, B, C, A, object]

Search order:

D → B → C → A → object

Since B has show(), Python stops there.
```

---

## 14. Instance, Static, Class Methods and Attributes

### 1. Instance Methods & Instance Attributes

Belong to each object (instance) of the class.
Always take `self` as their first parameter.
Can read and write instance attributes, which are stored on `self`.

```python
class Circle:
    def __init__(self, radius):          # constructor
        self.radius = radius             # instance attribute

    def area(self):                      # instance method
        return 3.1416 * self.radius**2   # uses self.radius

c1 = Circle(5)
c2 = Circle(10)

print(c1.area())  # 78.54
print(c2.area())  # 314.16
```

### 2. Class Methods & Class Attributes

Belong to the class itself, not to any particular object.
Defined with the `@classmethod` decorator and take `cls` as their first parameter.
Can read/write class attributes—shared by all instances.

```python
class Circle:
    pi = 3.1416                    # class attribute

    def __init__(self, radius):
        self.radius = radius

    @classmethod
    def set_pi(cls, new_pi):       # class method
        cls.pi = new_pi            # modifies class attribute

    def area(self):                # instance method still uses pi
        return Circle.pi * self.radius**2

# before change
print(Circle.pi)        # 3.1416
# change pi for all circles
Circle.set_pi(3.14)
print(Circle.pi)        # 3.14

c = Circle(5)
print(c.area())         # 78.5  (uses updated pi)
```

### 3. Static Methods

Behave like plain functions but live in the class's namespace.
Defined with `@staticmethod`.
Don't take `self` or `cls`—they cannot access instance or class data unless explicitly passed.

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b

    @staticmethod
    def is_even(n):
        return n % 2 == 0

print(MathUtils.add(3, 4))   # 7
print(MathUtils.is_even(10)) # True
```

### Comparison Table

| Feature            | Defined As                      | First Argument | Can Access…               |
|--------------------|---------------------------------|----------------|---------------------------|
| Instance method    | `def foo(self, …):`             | `self`         | Instance + class data     |
| Class method       | `@classmethod\ndef foo(cls…)`   | `cls`          | Class + (other passed)    |
| Static method      | `@staticmethod\ndef foo(…)`     | —              | Only what you pass in     |
| Instance attribute | `self.x = …`                    | N/A            | Unique to each object     |
| Class attribute    | `Class.x = …`                   | N/A            | Shared across all objects |

---

## 15. Multithreading

### 🧠 What is Multithreading?

Multithreading means running multiple threads (smaller units of a process) concurrently within the same process.

Each thread:
- Shares the same memory space
- Runs different parts of the program seemingly in parallel

### 🧩 Useful for:
- I/O-bound tasks (e.g., file I/O, API calls, database queries)
- Not great for CPU-bound tasks (because of Python's GIL — explained below)

### ⚙️ The GIL (Global Interpreter Lock)

Python has a **Global Interpreter Lock (GIL)** — which allows only one thread to execute Python bytecode at a time.

So even if you have multiple threads, only one executes Python code at a time.

But when a thread waits for I/O (e.g., network or disk), other threads can run — giving concurrency benefits.

### 🧩 Hence:
- Use **multithreading** for I/O-bound tasks
- Use **multiprocessing** for CPU-bound tasks

### 🧩 Example 1: Simple Multithreading

```python
import threading
import time

def task(name):
    print(f"Starting task {name}")
    time.sleep(2)
    print(f"Task {name} completed")

# Create threads
t1 = threading.Thread(target=task, args=("A",))
t2 = threading.Thread(target=task, args=("B",))

# Start threads
t1.start()
t2.start()

# Wait for both to finish
t1.join()
t2.join()

print("All tasks completed!")
```

**Output:**
```
Starting task A
Starting task B
Task A completed
Task B completed
All tasks completed!
```

⏱️ **Total time ≈ 2 seconds (not 4)** → because both threads run concurrently.

### Common Threading Methods

| Method / Attribute               | Description                |
|----------------------------------|----------------------------|
| `Thread(target=func, args=())`   | Create a new thread        |
| `start()`                        | Start the thread           |
| `join()`                         | Wait for thread to finish  |
| `current_thread()`               | Get current thread object  |
| `name`                           | Thread name                |
| `is_alive()`                     | Check if thread is running |

### When to Use Threading

| Task Type                               | Recommended                            |
|-----------------------------------------|----------------------------------------|
| I/O-bound (API calls, DB queries, etc.) | ✅ `threading` / `ThreadPoolExecutor`  |
| CPU-bound (math, data processing, ML)   | 🚫 `threading` — use `multiprocessing` |

### GIL Effect Summary

| Scenario               | GIL Effect               | Result                     |
|------------------------|--------------------------|----------------------------|
| I/O-bound tasks        | GIL released during wait | Threads run concurrently ✅ |
| CPU-bound tasks        | GIL always held          | No parallelism ❌          |
| `sleep()`, network, DB | OS handles waiting       | Overlap execution ✅       |

**Note:** Threads appear concurrent because Python releases the GIL during I/O operations, allowing other threads to run while one thread is waiting.

---

## 16. Memory Management
### 🧠 What Is Memory Management in Python?

Python automatically handles memory allocation and deallocation — you don't manually free memory like in C or C++.

Python uses:
- **Reference Counting** (primary mechanism)
- **Garbage Collection** (secondary mechanism — for circular references)

### ### 🧩 2. Reference Counting (Core Mechanism)

Every Python object keeps track of how many references point to it.

👉 Each time you:
- Assign an object to a variable → count increases
- Pass it as a function argument → count increases
- Delete a reference or variable → count decreases

When count becomes zero, Python frees the memory.

```python

a = [1, 2, 3]
print(sys.getrefcount(a))  # e.g., 2 (one from a, one from function argument)

b = a
print(sys.getrefcount(a))  # count increases (e.g., 3)

del b
print(sys.getrefcount(a))  # count decreases (e.g., 2)
```

### 🧮 Explanation:
- `sys.getrefcount()` adds one temporary reference when called
- When count reaches 0 → object is destroyed → memory reclaimed

### ### 🧩 3. Problem: Circular References

Reference counting fails when two objects reference each other.

**Example:**

```python
class Node:
    def __init__(self):
        self.ref = None

a = Node()
b = Node()

a.ref = b
b.ref = a

del a
del b
```

Even after deleting `a` and `b`, the memory isn't freed because:
- `a.ref` → points to `b`
- `b.ref` → points to `a`

Reference count never becomes zero.

🧨 **This creates a circular reference** — a memory leak risk.

### 🧹 4. Python's Garbage Collector (GC)
To handle such circular references, Python has a garbage collector in the gc module.

### ✨ How it works:
- GC periodically checks for objects that are unreachable (not accessible from any variable)
- Even if they still reference each other, they're removed if no external references exist

### Summary Table

| Concept                | Description                                                                         |
|------------------------|-------------------------------------------------------------------------------------|
| **Reference Counting** | Counts how many references exist for each object. Frees immediately when count = 0. |
| **Garbage Collector**  | Handles circular references that reference counting can't free.                     |
| **PyMalloc**           | Python's custom memory allocator for small objects (faster than OS allocation).     |
| **`gc.collect()`**     | Manually triggers garbage collection.                                               |
| **PyMalloc**           | Python's custom memory allocator for small objects (faster than OS allocation).     |
| **`gc.collect()`**     | Manually triggers garbage collection.                                               |

---

## 17. Custom Exceptions
### 💡 What Are Exceptions?

An exception is an error that interrupts the normal flow of a program.

**Examples of built-in exceptions:**
- `ZeroDivisionError`
- `ValueError`
- `KeyError`
- `TypeError`
- `IndexError`

When an exception occurs, Python raises it — and if it's not handled, your program crashes.

### 🧩 Why Create Custom Exceptions?
You create custom exceptions when:
- Built-in exceptions don't clearly describe your error
- You want to make your codebase more readable and maintainable
- You want to handle specific domain-related errors (e.g., `InvalidUserError`, `PaymentFailedError`)

### Example

```python
class InvalidAgeError(Exception):
    """Raised when age is below the allowed limit."""
    pass

def validate_age(age):
    if age < 18:
        raise InvalidAgeError("Age must be at least 18!")
    else:
        print("Access granted!")

try:
    validate_age(10)
    # validate_age("abc")
    
except InvalidAgeError as e:
    print(f"Error: {e}")
except Exception as e:
    print(f"Error: {e}")
```

**Output:**
```
Error: Age must be at least 18!
```

### Summary Table

| Concept                          | Description                    |
|----------------------------------|--------------------------------|
| `class MyError(Exception): pass` | Define custom exception        |
| `raise MyError("message")`       | Raise it                       |
| `try: ... except MyError:`       | Handle it                      |
| `__init__`, `__str__`            | Customize behavior             |
| Inherit hierarchy                | For organized error management |

### else block in Exception Handling

The else block executes only if no exception occurs.
```python
try:
    num = int("123")
except ValueError:
    print("Invalid number")
else:
    print("No exception occurred")
finally:
    print("Always executes")

Output:

No exception occurred
Always executes

If an exception occurs, else is skipped.
```

---

## 18. Zero-Based Indexing
### 🧠 Interview One-Line Answer (BEST)

**Why zero-based indexing?**

> "Zero-based indexing simplifies memory address calculation and pointer arithmetic, making array access faster and less error-prone."

---

## 19. Database Configuration
### Django Database Configuration Example

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',  # or 'mysql', 'sqlite3', etc.
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': config('DB_HOST', default='localhost'),
        'PORT': config('DB_PORT', default='5432'),
    }
}
```

### Database Drivers

| Database   | Driver Installation      | Engine                            |
|------------|--------------------------|-----------------------------------|
| Oracle SQL | `pip install cx-Oracle`  | `'django.db.backends.oracle'`     |
| PostgreSQL | `pip install psycopg2`   | `'django.db.backends.postgresql'` |

---

## 20. Docker
### 🧠 What is Docker?

Docker is a platform that allows developers to package applications and their dependencies into a standardized unit called a **container**. These containers can then be run consistently on any system, regardless of the underlying environment.

### Key Benefits:
- Docker ensures that the application will run the same way in development, testing, and production
- Developers can avoid the common **"it works on my machine"** problem

### Docker Philosophy

> **Docker = "Build once, run anywhere"**

It improves efficiency, reliability, and scalability in software development and deployment.

---

## 21. Context Manager

A context manager is a Python construct that manages resource acquisition and cleanup automatically using the with statement. It ensures resources such as files, database connections, and locks are properly released even if exceptions occur. We can create a custom context manager either by implementing __enter__() and __exit__() methods in a class or by using the @contextmanager decorator from the contextlib module.

The most common example is file handling:

```python
with open("test.txt", "r") as f:
    data = f.read()
```
Here:

Before entering the with block → file is opened.
After leaving the with block (even if an exception occurs) → file is closed automatically.

Without a context manager:
```python
f = open("test.txt", "r")
try:
    data = f.read()
finally:
    f.close()
```
The with statement makes this cleaner and safer.

### Why do we need Context Managers?

They help manage resources such as:

Database connections
Files
Network sockets
Locks/Mutexes
Transactions

and guarantee cleanup even when exceptions occur.

A context manager implements two special methods:
```python
__enter__()
__exit__()

Flow:

with MyContext() as obj:
    # code block

Execution order:

1. __enter__()
2. code inside with block
3. __exit__()
```

### Custom Context Manager Using a Class

```python
class MyContextManager:

    def __enter__(self):
        print("Resource acquired")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Resource released")


with MyContextManager():
    print("Inside block")

Output:

Resource acquired
Inside block
Resource released
```

### Custom Context Manager Using contextlib

Python provides the contextlib module.
```python
from contextlib import contextmanager

@contextmanager
def my_context():
    print("Acquire resource")

    try:
        yield
    finally:
        print("Release resource")


with my_context():
    print("Inside block")

Output:

Acquire resource
Inside block
Release resource
```

The code before yield behaves like __enter__(), and the code after yield behaves like __exit__().

### Real-world FastAPI Example

The get_db() dependency is essentially a context manager:
```python
def get_db():
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()
```

When a request arrives:

1. Create DB session
2. Yield session to endpoint
3. Endpoint executes
4. Session closes automatically

This prevents database connection leaks.

---

## 🎯 End of Python Hands-On Guide

This guide covers essential Python concepts for interviews and practical development. Practice these examples and understand the underlying concepts to master Python programming!