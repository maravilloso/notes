Python3
=======

For debugging, use ``python -i`` which runs your program and then enters the Python interactive shell afterwards. Quite useful for debugging, testing, etc.
```bash
bash % python3 -i helloworld.py
hello world
>>>
```
## PIP
```bash
# Forzar la reinstalación desde cero de un paquete
pip install --force-reinstall <package-name>

# Instalar modulo desde detras de un proxy
pip --proxy http://mufarma:mufarmam@10.1.140.173:8080 install tableize

# Descargar solamente el modulo sin instalarlo
pip download <package-name>

# Instalar desde un fichero para el usuario actual solamente
pip install --user file.egg
```
## Formatted Printing
f-strings:
```python
print(f"{' [ Run Status ] ':=^50}")
print(f"[{time:%H:%M:%S}] Training Run {run_id=} status: {progress:.1%}")
print(f"Summary: {total_samples:,} samples processed")
print(f"Accuracy: {accuracy:.4f} | Loss: {loss:#.3g}")
print(f"Memory: {memory / 1e9:+.2f} GB")
print(f'{name:>10s} {shares:>10d} {price:>10.2f}')
```
Example output:
```txt
=================== [ Run Status ] ===================
[11:16:37] Training Run run_id=42 status: 87.4%
Summary: 12,345,678 samples processed
Accuracy: 0.9876 | Loss: 0.0123
Memory: +2.75 GB
    Marcos       1234     567.89
```

With ``.format()`` method:
```python
print('{:10s} {:10d} {:10.2f}'.format(name,shares,price))
```

Use % operator:
```python
print('%10s %10d %10.2f' % (name, shares, price))
```
## Main program check
It is standard practice for modules that can run as a main program to use this convention:
```python
# foo.py
...
if __name__ == '__main__':
	# Running as the main program
	...
	statements
	...
```
## Convenient variations on Class definitions
**Slots** save memory and run faster.
```python
class Stock:
  	__slots__ = ('name', 'shares', 'price')
	def __init__(self, name, shares, price):
		self.name = name
		self.shares = shares
		self.price = price
```

**Dataclasses** reduce coding (less verbosity) Some useful methods get created automatically, but mind that types are NOT enforces tho.
```python
from dataclasses import dataclass

@dataclass
class Stock:
	name : str
	shares : int
	price: float
```
**Named Tuples**: inmutables while allowing other features from tuples like unpacking and indexing:
```python
import typing

class Stock(typing.NamedTuple):
	name: str
	shares: int
	price: float

>>> s = Stock('GOOG', 100, 490.1)
>>> s[0]
'GOOG'
>>> name, shares, price = s
>>> print('%10s %10d %10.2f' % s)
GOOG
100
490.10
>>> isinstance(s, tuple)
True
>>> s.name = 'ACME'
Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
>>>
```
## Composite keys for dicts
Use tuples for multi-part keys:
```python
prices = {
	('ACME','2017-01-01')
	('ACME','2017-01-02')
	('ACME','2017-01-03')
	('SPAM','2017-01-01')
	('SPAM','2017-01-02')
	('SPAM','2017-01-03')
}

# And then you can do things like:
p = prices['ACME', '2017-01-01']
prices['ACME','2017-01-04'] = 515.20
```
## Collections Module
Provides some variations on common data structures (useful for specialized problems)
**defaultdict** allows initialization of missing dict keys, so for example I could implement nested dictionaries for an arbitrary number of levels (Perl like) doing:
```python
from collections import defaultdict
import json

rec_dd = lambda: defaultdict(rec_dd)

>>> x = rec_dd()
>>> x['a']['b']['c']['d'] = 'Hello'
>>> json.dumps(x)
'{"a": {"b": {"c": {"d": "Hello"}}}}'
```
**Counter** is a dictionary specialized for counting items:
```python
from collections import Counter

>>> totals = Counter()
>>> totals['IBM'] += 20
>>> totals['AA'] += 50
>>> totals['ACME'] += 75
>>> totals
Counter({'ACME': 75, 'AA': 50, 'IBM': 20})
>>> totals.most_common(2)
[('ACME', 75), ('AA', 50)]
```
**deque** is a double-ended queue which is more efficient than a list for queuing problems:
```python
from collections import deque

>>> q = deque()
>>> q.append(1)
>>> q.append(2)
>>> q.appendleft(3)
>>> q.appendleft(4)
>>> q
deque([4, 3, 1, 2])
>>> q.pop()
2
>>> q.popleft()
4
```
## Speed up by using ``@cache`` decorator
Whenever you have an assumed expensive calculation:
```python
from functools import cache

@cache
def fib(n):
    return n if n < 2 else fib(n-1) + fib(n-2)
```

## Iterating on Tuples
Unpacking can be used when iterating collections of tuples, even using the "throwaway" value:
```python
portfolio = [
	('GOOG', 100, 490.1),
	('IBM', 50, 91.1),
	('CAT', 150, 83.44),
	('IBM', 100, 45.23),
	('GOOG', 75, 572.45),
	('AA', 50, 23.15)
]

for name, _, price in portfolio:
	print(name,price)
```
## Iterating on Varying Records
Wildcard unpacking can be used for such cases:
```python
prices = [
	['GOOG', 490.1, 485.25, 487.5 ],
	['IBM', 91.5],
	['HPE', 13.75, 12.1, 13.25, 14.2, 13.5 ],
	['CAT', 52.5, 51.2]
]

for name, *values in prices:
	print(name, values)
```
## The zip() function
Iterate on multiple sequences in parallel

```python
columns = ['name','shares','price']
values = ['GOOG',100, 490.1 ]

for colname, val in zip(columns, values):
	# Loops with	colname='name' val='GOOG'
	#				colname='shares' val=100
	#				colname='price' val=490.1
	...
    
# Common use: making dictionaries
record = dict(zip(columns,values))

```
## Unpacking Iterables
More convenient than the ``+`` operator:
```python
a = (1, 2, 3)
b = [4, 5]
c = [ *a, *b ]	# c = [1, 2, 3, 4, 5]	(list)
d = ( *a, *b )	# d = (1, 2, 3, 4, 5)	(tuple)

>>> c = a + b
Traceback (most recent call last):
	File "<stdin>", line 1, in <module>
TypeError: can only concatenate tuple (not "list") to tuple
```
## Unpacking & combining dictionaries

```python
a = { 'name': 'GOOG', 'shares': 100, 'price':490.1 }
b = { 'date': '6/11/2007', 'time': '9:45am' }
c = { **a, **b }
>>> c
{ 'name': 'GOOG', 'shares':100, 'price': 490.1,
	'date': '6/11/2007,'time': '9:45am' }
```
## Generators
Generators can only be consumed once:
```python
>>> nums = [1,2,3,4]
>>> squares = (x*x for x in nums)
>>> for n in squares:
		print(n, end=' ')
1 4 9 16
>>> for n in squares:
		print(n, end=' ')
>>>
# notice no output (spent)
```
## Walrus operator
Really useful for using a value right after checking if it is not None!
```python
if response := get_user_input():
    print('You pressed:', response)
else:
    print('You pressed nothing')
```
## Structural Pattern Matching
Allows for more expressive and readable ways to match patterns and deconstruct data
```python
def handle_data(data):
    match data:
        # Match a literal value
        case 42:
            print("Matched the number 42!")

        # Match a variable with a guard
        case x if x > 100:
            print(f"Matched a number greater than 100: {x}")

        # Match a tuple with specific values
        case (1, 2, 3):
            print("Matched the tuple (1, 2, 3)")

        # Match a tuple with a wildcard
        case (1, _, _):
            print("Matched a tuple starting with 1")

        # Match a list with specific length
        case [1, 2, 3, *_]:
            print("Matched a list starting with [1, 2, 3]")

        # Match a dictionary with specific keys
        case {"name": name, "age": age}:
            print(f"Matched a dictionary with name={name} and age={age}")

        # Match an instance of a class
        case MyClass(attr1=x, attr2=y):
            print(f"Matched an instance of MyClass with attr1={x} and attr2={y}")

        # Match a nested structure
        case {"user": {"name": name, "age": age}, "status": status}:
            print(f"Matched nested structure: name={name}, age={age}, status={status}")

        # Match enums
        case Status.ACTIVE:
            print("Matched an active status!")

        # Match with OR patterns
        case 0 | 1 | 2:
            print("Matched one of the values: 0, 1, or 2")

        # Match with AS patterns
        case [x, y, z] as full_list:
            print(f"Matched a list: {full_list} with elements {x}, {y}, {z}")

        # Match a class with type guards
        case int() if data % 2 == 0:
            print(f"Matched an even integer: {data}")

        # Default case (wildcard)
        case _:
            print("Matched something else!")

# Define a custom class for matching
class MyClass:
    def __init__(self, attr1, attr2):
        self.attr1 = attr1
        self.attr2 = attr2

# Define an enum for matching
from enum import Enum

class Status(Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"

# Test the function with various inputs
handle_data(42)
handle_data(150)
handle_data((1, 2, 3))
handle_data([1, 2, 3, 4, 5])
handle_data({"name": "Alice", "age": 30})
handle_data(MyClass(attr1="value1", attr2="value2"))
handle_data({"user": {"name": "Bob", "age": 25}, "status": "online"})
handle_data(Status.ACTIVE)
handle_data([10, 20, 30])
handle_data(4)
handle_data("unexpected")
```
